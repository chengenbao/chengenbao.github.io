---
layout: note
title: "NVSHMEM 统一地址空间与 Sparse Embedding 技术解析"
permalink: /notes/nvshmem-sparse-embedding/
---

# NVSHMEM 统一地址空间与 Sparse Embedding 技术解析

> 技术：NVIDIA NVSHMEM 3.6 / CUDA UVA / IBGDA  
> 应用场景：大规模推荐系统 Sparse Embedding 分布式 Lookup  
> 笔记整理：chengenbao · 2026-04

---

## 一、为什么 Sparse Embedding 需要统一地址空间

推荐系统和大模型的 Sparse Embedding Table 面临三重困境：

**1. 容量困境**：动辄数十亿行的 embedding table（如 1B × 128dim × 4bytes = **512 GB**），远超单卡 HBM 容量（H100 = 80GB）。

**2. 随机访问困境**：lookup index 高度随机，传统 GPU cache miss 严重，UVM 页迁移开销巨大。

**3. 分布式通信困境**：分表存储后每次 forward 需要 all-to-all 汇聚，传统 NCCL 方案依赖 CPU 发起、集体同步，难以与计算 overlap。

**统一地址空间（PGAS）的价值**：把多卡 HBM 在逻辑上拼成一块大内存，GPU kernel 直接用指针寻址远端数据，绕开 CPU，实现 compute-communication 天然重叠。

---

## 二、NVSHMEM 技术架构

### 2.1 核心模型：PGAS（Partitioned Global Address Space）

```
物理布局（4张GPU）：
┌──────────────────────────────────────────────────────┐
│  GPU0 HBM          GPU1 HBM          GPU2 HBM          GPU3 HBM │
│  embedding[0..N/4) embedding[N/4..N/2) ...             ...      │
└──────────────────────────────────────────────────────┘
         ↓ NVSHMEM 对称分配后的逻辑视图
┌──────────────────────────────────────────────────────┐
│         全局地址空间 [0, N)，每个 PE 持有 1/4        │
│  PE0 owning 0..N/4  PE1 owning N/4..N/2  ...         │
└──────────────────────────────────────────────────────┘
```

每个 PE（Processing Element = GPU进程）调用 `nvshmem_malloc` 分配**对称内存**——所有 PE 分配相同大小，NVSHMEM 自动建立跨卡地址映射。

### 2.2 通信原语

| 原语 | 说明 | 适用场景 |
|---|---|---|
| `nvshmem_get(dest, src, size, pe)` | 从指定 PE 拉取数据（阻塞） | 同步 lookup |
| `nvshmemx_getmem_on_stream(...)` | 异步 stream-based get | 与计算 overlap |
| `nvshmem_put(dest, src, size, pe)` | 推送数据到指定 PE | backward scatter |
| `nvshmem_atomicadd(dest, val, pe)` | 远端原子加（用于梯度累加） | embedding 梯度聚合 |
| `nvshmem_barrier_all()` | 全局同步屏障 | epoch 边界 |

所有这些操作均可在 **CUDA kernel 内部调用**，无需 CPU 介入。

### 2.3 传输通道

```
节点内（intra-node）：
  NVLink / NVSwitch  →  带宽 ~600 GB/s（H100 NVLink 4.0）
                         延迟 ~1 μs

节点间（inter-node）：
  InfiniBand RDMA    →  带宽 ~25 GB/s（IB HDR 200Gb）
  IBGDA（IB GPUDirect Async）→ 网卡直写 GPU，省去 pinned memory 中转
```

### 2.4 与 CUDA UVM 的关键区别

| | CUDA UVM（cudaMallocManaged） | NVSHMEM |
|---|---|---|
| 覆盖范围 | 单机 CPU + GPU | 多机多 GPU |
| 访问触发 | 页错误（page fault）驱动迁移 | 预先映射，无页错误 |
| 发起方 | CPU 或 GPU | **GPU kernel 直接发起** |
| 通信模式 | 被动（reactive） | 主动（proactive） |
| 适合场景 | 内存超出单卡、访问局部性好 | 随机访问、延迟敏感、需 overlap |

---

## 三、Sparse Embedding Lookup 的技术方案

### 3.1 系统架构

```
Forward Pass:

每张GPU持有 embedding table 的 1/N 分片
         ↓
kernel 收到一批 [batch_size, seq_len] 的 sparse index
         ↓
计算每个 index 归属哪个 PE：  pe_id = row_id / (N / num_gpus)
         ↓
本地 row → 直接读 local HBM（zero-copy）
远端 row → nvshmemx_getmem_on_stream() 异步拉取
         ↓
在等待远端数据时，继续执行 dense computation（overlap！）
         ↓
结果汇聚，继续 forward
```

### 3.2 代码示例（CUDA kernel 片段）

```cuda
#include <nvshmem.h>
#include <nvshmemx.h>

__global__ void sparse_embedding_lookup(
    float* output,           // [batch_size, dim]
    const int* indices,      // [batch_size]
    float* sym_emb_table,    // nvshmem_malloc 分配的对称表
    int dim, int rows_per_pe
) {
    int tid = blockIdx.x * blockDim.x + threadIdx.x;
    if (tid >= batch_size) return;

    int row_id = indices[tid];
    int target_pe = row_id / rows_per_pe;
    int local_row  = row_id % rows_per_pe;

    float* src = sym_emb_table + local_row * dim;
    float* dst = output + tid * dim;

    if (target_pe == nvshmem_my_pe()) {
        // 本地访问，直接 memcpy
        memcpy(dst, src, dim * sizeof(float));
    } else {
        // 远端访问，GPU-initiated RDMA get
        nvshmemx_float_get_on_stream(
            dst, src, dim, target_pe, stream
        );
    }
}
```

### 3.3 初始化流程

```python
import nvshmem4py as nvshmem
import torch

# 初始化 NVSHMEM（通常配合 MPI 或 NCCL 做 bootstrap）
nvshmem.init()
my_pe = nvshmem.my_pe()
n_pes = nvshmem.n_pes()

total_rows = 1_000_000_000  # 10亿行
dim = 128
rows_per_pe = total_rows // n_pes

# 对称分配：所有 PE 分配相同大小
emb_table = nvshmem.malloc(rows_per_pe * dim * 4)  # float32

# 初始化本 PE 的分片
...

nvshmem.barrier_all()
```

---

## 四、关键技术挑战与解决方案

### 4.1 碎片 RDMA 问题（最核心挑战）

**问题**：每次 lookup 只取 1 行（如 512 bytes），大量小消息 RDMA 会打爆网卡 QP（Queue Pair）。

**解决方案：Request Coalescing（请求合并）**

```
原始请求：
  [row 12] → PE1
  [row 15] → PE1
  [row 89] → PE1
  [row 3]  → PE2
  ...

合并后：
  PE1 的请求：{12, 15, 89} → 一次 scatter-gather RDMA
  PE2 的请求：{3, ...}    → 一次 scatter-gather RDMA
```

实现方式：
1. **sort by PE**：先对 batch 内的 indices 按 `pe_id` 排序分组
2. **gather 成连续 buffer**：把同一 PE 的请求 index 聚合后一次 `nvshmem_get`
3. **scatter 回输出位置**：按原始顺序写回 output buffer

### 4.2 热点 PE 过载（Hot PE 问题）

**问题**：Zipf 分布下，头部 1% 的 embedding row 承载 ~50% 的请求，对应 PE 成为瓶颈。

**解决方案：热行 Replica + 一致性哈希**

```
Top-K 热行策略：
  统计每个 PE 的请求频次（滑动窗口）
  频次 > threshold 的行 → 在所有 PE 上复制副本
  lookup 时优先读本地副本（zero-copy）

训练时：
  副本梯度 → all-reduce 同步（仅 top-K 行，开销可控）
```

Meta 的 ZionEX、字节的 Persia 均采用类似策略。

### 4.3 对称内存限制

**问题**：`nvshmem_malloc` 要求所有 PE 分配**等量**内存，不支持各卡不同大小。

**应对方案**：
- 按最大 PE 的数据量对齐分配（浪费部分显存）
- 或：用 `nvshmem_malloc` 分配指针数组 + 实际数据用常规 `cudaMalloc`，通过 `nvshmem_ptr()` 访问（需要 NVLink 直连）
- 动态扩容：重新分配 + `nvshmem_barrier_all()` 同步，代价较高

### 4.4 训练时 Backward 梯度聚合

```
Forward:  nvshmem_get  (pull embedding)         ✅ 高效
Backward: nvshmem_atomicadd (scatter gradient)  ⚠️ 吞吐较低

替代方案：
  1. 收集所有远端梯度 → 批量 all-reduce（类似 DDP）
  2. Gradient accumulation window：攒若干 step 再同步
  3. 仅推理场景：完全跳过 backward，难度大降
```

---

## 五、与现有方案的横向对比

| 方案 | 发起方 | 通信模式 | Overlap | 随机访问 | 工程复杂度 |
|---|---|---|---|---|---|
| **NCCL All-to-All** | CPU | 集体同步 | ❌ | 中 | 低 |
| **CUDA UVM MANAGED** | GPU | 页迁移（被动） | ❌ | 差（页粒度） | 低 |
| **软件 Cache + Prefetch** | CPU+GPU | 异步拉取 | 部分 | 好（hot行） | 中 |
| **NVSHMEM GPU-initiated** | GPU | One-sided 异步 | ✅ | 好 | 高 |
| **IBGDA（网卡直写GPU）** | NIC | RDMA 直通 | ✅ | 好 | 很高 |

**工业参考**：
- **DeepEP**（DeepSeek）：用 NVSHMEM 做 MoE all-to-all，带宽利用率逼近硬件上限，验证了路径可行性
- **FBGEMM TBE**（Meta）：`MANAGED_CACHING` 模式用 UVM + HBM Cache，是当前最成熟的工业方案
- **UGACHE**（ACM 2025）：统一多 GPU 缓存，通过 UVA peer access + 频率感知 cache policy 实现近最优命中率
- **字节 FLUX / Persia**：IBGDA 路径，网卡直写 GPU，省去 CPU pinned memory 中转

---

## 六、推荐落地路径

### 阶段一：单机多卡（NVLink，验证 overhead）

```
目标：验证 NVSHMEM one-sided get 在 sparse lookup 场景的基线性能
硬件：4/8× H100，NVLink 互联
步骤：
  1. nvshmem_malloc 把 embedding 均匀分片到各卡
  2. 实现 kernel：sort-by-PE → coalesced get → scatter back
  3. 对比 baseline（NCCL all-to-all）的延迟 / 吞吐
  4. 加入 prefetch pipeline：下一 batch lookup 与当前 dense 计算 overlap
```

### 阶段二：多机扩展（IB，引入 Cache）

```
目标：支持 10B+ 行 embedding，跨节点低延迟
步骤：
  1. 启用 IBGDA transport（NVSHMEM 配置 NVSHMEM_REMOTE_TRANSPORT=ibgda）
  2. 引入 hot embedding cache（per-GPU，LFU/LRU eviction）
  3. 仅 miss 时走 RDMA，hit 直接读本地 cache
  4. 定期同步 top-K hot 行的副本
```

### 关键 Benchmark 指标

| 指标 | 目标值 |
|---|---|
| 单次 lookup 延迟（节点内，无 cache） | < 5 μs |
| 单次 lookup 延迟（跨节点，IBGDA） | < 20 μs |
| 吞吐（QPS，batch=1024） | > 100K QPS/GPU |
| Cache 命中率（热行 1%，Zipf α=1.2） | > 90% |

---

## 七、可行性结论

**推理场景（无 backward）：技术可行，工程难度中等**
- NVSHMEM PGAS + GPU-initiated get 是比 NCCL 更适合 sparse random access 的底层原语
- 核心工程投入：coalescing + cache + 热点处理，约 2-3 个月研发

**训练场景（含 backward）：技术可行，工程难度较高**
- backward 梯度 scatter 吞吐是主要瓶颈
- 建议先做推理验证，再扩展到训练

**最大风险**：碎片 RDMA + 热点倾斜，两者必须同时解决才能达到生产级性能。

---

## 参考资料

- [NVIDIA NVSHMEM 官方文档](https://docs.nvidia.com/nvshmem/api/)
- [NVIDIA NVSHMEM Developer Page](https://developer.nvidia.com/nvshmem)
- [DeepEP: NVSHMEM Integration](https://github.com/deepseek-ai/DeepEP)
- [FBGEMM TBE EmbeddingLocation 文档](https://docs.pytorch.org/FBGEMM/fbgemm_gpu/python-api/tbe_ops_training.html)
- [UGACHE: Unified and Near-optimal Multi-GPU Cache for Embedding (ACM 2025)](https://dl.acm.org/doi/10.1145/3767725)
- NVSHMEM 深度解析：初始化流程与核心机制（知乎，AI闲谈，2025-07）
- DeepEP，nvshmem 和 IBGDA 二三事（知乎，2025-07）
