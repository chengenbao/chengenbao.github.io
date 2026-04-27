---
layout: note
title: "GPU 性能分析基础：Register Spill · Occupancy · Latency-Bound 诊断"
date: 2026-04-27
tags: [GPU, CUDA, 性能分析, Nsight, Occupancy, Warp, H20, Hopper, 内核优化]
---

# GPU 性能分析基础：Register Spill · Occupancy · Latency-Bound 诊断

> **适用场景**：看懂 Nsight Compute / `clock64` / `cuobjdump` 的输出，诊断 CUDA kernel 为什么"不够快"。本文以 **NVIDIA H20（Hopper 架构）** 为例。

---

## 0. 心智模型：GPU 硬件层次

在深入指标之前，先建立一个清晰的硬件视图——所有性能分析的起点。

```
GPU
 ├─ 多个 SM（H20 有 78 个）
 │   ├─ 4 个 warp scheduler
 │   ├─ CUDA Core        → 做 FMA/ADD/MUL（FP32/FP16）
 │   ├─ SFU              → 做 sqrt/rsqrt/div/sin/cos
 │   ├─ Tensor Core      → 做矩阵乘（GEMM）
 │   ├─ 寄存器文件       → 65536 × 32-bit / SM
 │   ├─ Shared Memory / L1 Cache → ~228KB / SM
 │   └─ LD/ST 单元       → 访存
 ├─ L2 Cache（全 GPU 共享，~60MB）
 └─ HBM3 显存（H20 ~96GB，带宽 4 TB/s）
```

**核心规律**：CUDA Core、SFU、LD/ST 单元**并行运转**，kernel 的吞吐由**最慢的那个单元**决定——这就是 **bottleneck**。

---

## 1. Thread / Warp / Block / Grid

| 概念 | 说明 |
|------|------|
| **Thread** | 最小执行单位，有自己的寄存器 |
| **Warp** | 32 个 thread 打包，**硬件上 lockstep 执行**（同一时刻执行同一条指令） |
| **Block** | 若干 warp，共享 Shared Memory，只能在一个 SM 上运行 |
| **Grid** | 若干 block，即一次 kernel launch |

例：`kBlockSize=512` → 一个 block 含 **16 个 warp**（512 / 32）。

---

## 2. Register Spill（寄存器溢出）

### 2.1 什么是 Spill

每个 SM 的寄存器文件有 65536 个 32-bit 寄存器，每个 thread 最多使用 255 个。当编译器判断活跃变量超过上限，多余的变量会被"溢出"到 **local memory**（物理上是 HBM，per-thread 视角访问）。

```
正常情况：          Spill 情况：
变量 a, b, c        变量 a, b → 寄存器（~1 cycle）
  → 寄存器           变量 c   → HBM（~400 cycle）
```

访存延迟放大约 **400×**，是最直接的性能杀手之一。

### 2.2 如何检查

```bash
# 方法 1：编译时加 -Xptxas=-v
nvcc -Xptxas=-v your_kernel.cu
# 输出示例：
# ptxas info: Used 61 registers, 0 bytes spill stores, 0 bytes spill loads
#                                    ↑ 非 0 = 有 spill，要警觉

# 方法 2：反汇编看 SASS 指令
cuobjdump --dump-sass your_binary.so | grep -E "LDL|STL"
# LDL = Load Local，STL = Store Local，即访问 local memory
```

`spill stores = 0 && spill loads = 0` → 寄存器溢出这个嫌疑可排除。

---

## 3. Occupancy（占用率）

$$\text{Occupancy} = \frac{\text{实际驻留 warp 数}}{\text{SM 最大 warp 数（H20 = 64）}}$$

### 3.1 为什么重要

当某个 warp 遇到 stall（等访存、等 SFU 结果、数据依赖），warp scheduler 可以**切换到其他 ready 的 warp**继续执行，从而隐藏延迟。驻留 warp 越多，可切换的候补越多，延迟隐藏效果越好。

### 3.2 影响 Occupancy 的因素

| 因素 | 影响方向 |
|------|---------|
| 每 thread 寄存器数增多 | ↓ 驻留 warp 减少 |
| 每 block shared memory 增多 | ↓ 可并发 block 减少 |
| block size 过大或过小 | ↓ SM 利用不均衡 |
| `__launch_bounds__` 注解 | 可能强制降低寄存器数以提高 occupancy |

> **经验值**：occupancy > 50% 通常已够用；但 occupancy 高不等于性能好，关键看是否存在 latency-bound 问题（见第 5 节）。

---

## 4. 延迟隐藏 vs Warp Lockstep

这是理解"SM 很忙但吞吐很低"的关键。

### 4.1 理想情况：延迟被隐藏

```
Warp 0: [算] [等 SFU ·····] [算]
Warp 1:       [算] [等 SFU ·····] [算]   ← scheduler 切换
Warp 2:             [算] [等 SFU ····]   ← 再切
→ SFU 始终有活干，整体高效
```

### 4.2 坏情况：Warp Lockstep

```
Warp 0: [算] [等 SFU] [算] [等 SFU] ···
Warp 1: [算] [等 SFU] [算] [等 SFU] ···  ← 所有 warp 同步！
Warp 2: [算] [等 SFU] [算] [等 SFU] ···
→ 同一时刻要么全在算，要么全在等 SFU
→ scheduler 无 warp 可切换
```

**结果**：SM 显示 97% busy，但 FMA / SFU / LD 利用率都很低 → **Latency-bound**，最难调的一类。

---

## 5. 三种典型瓶颈

| 瓶颈类型 | 判断方法 | 优化方向 |
|---------|---------|---------|
| **Memory-bound** | HBM 带宽接近峰值（H20 ~4 TB/s） | 减少 HBM 读写、kernel 融合、提高访存合并、使用 Shared Memory |
| **Compute-bound** | FMA / Tensor Core 吞吐打满 | 算法优化、充分利用 Tensor Core、降低算术强度 |
| **Latency-bound** | SM busy 高，但 FMA/MEM 吞吐均低 | 提高 ILP、提高 occupancy、打破 lockstep（错相位执行） |

---

## 6. 常用性能指标

| 指标 | 含义 | 参考健康值 |
|------|------|-----------|
| SM busy % | SM 时钟中有指令在 issue 的比例 | 越高越好，但需结合吞吐看 |
| FMA utilization | FP32 CUDA Core 实际吞吐 / 峰值 | compute-bound kernel 应 > 60% |
| MFU | Model FLOPs Utilization = 有效 FLOP / 峰值 FLOP | 大模型训练 30%–50% 算优秀 |
| Register/thread | 每线程寄存器数 | < 64 通常不影响 occupancy |
| Occupancy | 驻留 warp / SM 最大 warp | > 50% 通常够用 |
| DRAM throughput | HBM 带宽利用率 | memory-bound kernel 应接近峰值 |

**口诀**：**只有 SM busy 高、其他指标都低 = latency-bound**。

---

## 7. 诊断工具与方法

### 7.1 编译期（零运行成本）

```bash
# 查看寄存器 / spill / shared memory 使用量
nvcc -Xptxas=-v your_kernel.cu

# 反汇编看 SASS，检查 LDL/STL（local memory 访问）
cuobjdump --dump-sass your_binary.so
```

### 7.2 运行期 Profiler

```bash
# Nsight Systems：整体 timeline，看 kernel 调度与 overlap
nsys profile -o out python train.py

# Nsight Compute：单 kernel 精细指标（需要 profiling 权限）
ncu --set full -o out python train.py
```

**常见环境踩坑**：
- `RmProfilingAdminOnly=1` → 容器内无 `ncu` 权限
- `perf_event_paranoid` 过高 → 部分硬件计数器不可用
- 没权限时可用 `clock64()` 在 kernel 内手动计时做轻量诊断（见下方）

### 7.3 Kernel 内手动计时（clock64）

无需 profiler 权限的轻量方案：

```cpp
__global__ void my_kernel(...) {
    long long t0 = clock64();
    // ... compute ...
    long long t1 = clock64();
    if (threadIdx.x == 0) {
        atomicAdd(&sum_block_cycles, t1 - t0);
    }
}
```

用 `sum_block_cycles / (num_SM × SM_clock_hz)` 估算 SM busy 时间，与 kernel 总耗时对比，快速判断是否 latency-bound。

---

## 8. 真实案例：FTML v2 Optimizer 调优

```
现象：optimizer 耗时 86ms，FLOP 利用率仅 ~10%

假设 1：kernel launch 次数多
  → 改为 scratch-backed 单次 launch
  → 99ms → 86ms，有改进，但远未达到理论值 ✓（部分有效）

假设 2：register spill 导致访存多
  → cuobjdump 看到 0 bytes spill → 排除 ✗

假设 3：数据依赖导致流水线停顿
  → shared memory 缓存系数、fast-math 均无效，甚至变慢 ✗

假设 4：warp lockstep + SFU 瓶颈 ← 当前方向
  → clock64 实测：SM 97% busy，FMA 12%
  → 确认 latency-bound
  → 优化方向：
     · 缩小 kChunkSize，让 block 更快退出
     · 不同 block 的 warp 错相位执行
     · 打破 lockstep，让 SFU pipeline 不空转
```

**教训**：不要凭直觉猜瓶颈。每个假设都要用数据验证，数据反驳时立刻放弃，换下一个假设。

---

## 9. 诊断 Checklist

```
□ 1. nvcc -Xptxas=-v → 检查 spill stores/loads 是否为 0
□ 2. cuobjdump → 有无 LDL/STL 指令
□ 3. Nsight Compute → SM busy % vs FMA/SFU/MEM utilization
□ 4. 若 SM busy 高、吞吐低 → 确认 latency-bound
     → clock64 实测 block cycle，估算 SM busy 时间占比
□ 5. 根据瓶颈类型选择优化策略（见第 5 节表格）
```

---

## 10. 推荐学习资料

| 资料 | 重点章节 |
|------|---------|
| [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/) | 第 5 章 Performance Guidelines，§5.2.3 Occupancy 计算 |
| [Nsight Compute Kernel Profiling Guide](https://docs.nvidia.com/nsight-compute/ProfilingGuide/) | Roofline Analysis、Warp Stall Reasons |
| GTC Talks（每年） | "CUDA: New Features and Beyond"、"How to Optimize CUDA Applications" |
| *Professional CUDA C Programming*（Cheng / Grossman） | 第 3、4 章：执行模型与内存模型 |

**实战习惯**：每次编译 CUDA 代码都加 `-Xptxas=-v`，养成看 register / spill / shared memory 用量的直觉。

---

## 附：术语速查表

| 术语 | 含义 |
|------|------|
| SM | Streaming Multiprocessor，GPU 的基本计算单元 |
| SFU | Special Function Unit，处理 sqrt/rsqrt/div/sin/cos |
| FMA | Fused Multiply-Add，`a×b+c` 单指令完成 |
| Warp | 32 个 thread 的组，硬件 lockstep 执行 |
| Occupancy | SM 驻留 warp 数 / SM 最大支持 warp 数 |
| Spill | 寄存器装不下，变量溢出到 local memory（HBM） |
| ILP | Instruction-Level Parallelism，同一 thread 内的指令并行 |
| Roofline | 性能上限：min(峰值算力, 带宽 × 算术强度) |
| Lockstep | 所有 warp 同步执行同一指令，scheduler 无 warp 可切 |
| MFU | Model FLOPs Utilization = 有效 FLOP / 峰值 FLOP |
| LDL/STL | Load/Store Local，SASS 中的 spill 指令 |
