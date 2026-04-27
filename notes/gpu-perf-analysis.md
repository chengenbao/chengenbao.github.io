---
layout: note
title: "GPU CUDA Kernel 性能分析深度笔记"
date: 2026-04-27
tags: [GPU, CUDA, 性能分析, Nsight, Occupancy, Warp, Register Spill, Tensor Core, Roofline, 内核优化]
---

# GPU CUDA Kernel 性能分析深度笔记

> 面向工程师实际调优的系统性参考手册  
> 参考来源：NVIDIA Nsight Compute Profiling Guide 13.2、CUDA C++ Best Practices Guide 13.2

---

## 目录

1. [Register Spill 深入](#1-register-spill-深入)
2. [Occupancy 深入](#2-occupancy-深入)
3. [Memory 访问深入](#3-memory-访问深入)
4. [Warp 执行深入](#4-warp-执行深入)
5. [Compute 深入](#5-compute-深入)
6. [高级优化技术](#6-高级优化技术)
7. [Nsight Compute 指标解读](#7-nsight-compute-指标解读)
8. [实战调优流程](#8-实战调优流程)

---

## 1. Register Spill 深入

### 1.1 什么是 Register Spill

每个 SM sub-partition 拥有一个共享的寄存器文件（Register File），大小固定（Ampere/Hopper 架构每个 SM 共 65536 个 32-bit 寄存器）。当 kernel 所需寄存器超过可分配数量时，编译器会将部分变量溢出（spill）到 **Local Memory**，而 Local Memory 实际位于 **Device DRAM**，延迟与 Global Memory 相当（400~800 cycles）。

**Spill 的直接代价：**
- 每次 spill load/store 约 400~800 cycles 延迟（L2 miss 情况下）
- L1/L2 命中时约 20~80 cycles
- Spill 指令增加指令数，降低 IPC（Instructions Per Cycle）
- Spill 访问会与 global memory 访问竞争带宽

**识别方式（Nsight Compute）：**
```
# 查看 local memory 使用
ncu --metrics l1tex__data_pipe_lsu_wavefronts_mem_lg_cmd_load.sum,
             l1tex__data_pipe_lsu_wavefronts_mem_lg_cmd_store.sum \
    ./my_kernel

# 查看 PTX 中是否有 local store/load
cuobjdump --dump-ptx my_kernel.cubin | grep "\.local"

# 查看 SASS 中 spill 指令
cuobjdump --dump-sass my_kernel.cubin | grep -E "LDL|STL"
```

**关键 Nsight Compute 指标：**
- `l1tex__t_sectors_pipe_lsu_mem_local_op_ld.sum` — local load 扇区数
- `l1tex__t_sectors_pipe_lsu_mem_local_op_st.sum` — local store 扇区数
- `LaunchStats` section 中 `Registers Per Thread` 字段

---

### 1.2 `__launch_bounds__` 的作用机制与副作用

`__launch_bounds__(maxThreadsPerBlock, minBlocksPerSM)` 是给编译器的**硬约束**提示：

```cuda
// 声明：每个 block 最多 256 线程，每个 SM 至少同时跑 2 个 block
__global__ __launch_bounds__(256, 2)
void myKernel(float* data) { ... }
```

**作用机制：**

1. `maxThreadsPerBlock`（必填）：告知编译器最大 block 大小
   - 编译器据此计算每线程可用的寄存器上限：
     ```
     max_regs_per_thread = floor(65536 / maxThreadsPerBlock)
     ```
     例如：`maxThreadsPerBlock=256` → 每线程最多 `65536/256 = 256` 个寄存器（上限）
   - 实际上，硬件还会按 block 对齐分配（通常以 4 或 8 为粒度）

2. `minBlocksPerSM`（可选）：告知编译器期望每 SM 至少 N 个 block
   - 编译器会进一步限制寄存器使用，使得资源足够 N 个 block 驻留
   - 计算公式：`max_regs_per_thread = floor(65536 / (maxThreadsPerBlock * minBlocksPerSM))`
   
   例：`__launch_bounds__(128, 4)` → `floor(65536 / (128*4)) = 128` 个寄存器/线程

**副作用（必须注意）：**

| 副作用 | 说明 |
|--------|------|
| 强制寄存器限制 | 若 kernel 自然需要 200 个寄存器，但 `__launch_bounds__` 限制为 128，会产生大量 spill |
| 可能增加 spill | 过于激进的 `minBlocksPerSM` 导致寄存器极度压缩 |
| 影响编译时间 | 编译器要重新布局寄存器分配 |
| 不影响 runtime | launch 时仍可用更小的 block，但不会自动受益于更多寄存器 |

**最佳实践：**
```cuda
// Step 1: 不加 __launch_bounds__ 先编译，用 cuobjdump 查实际寄存器数
// cuobjdump --dump-sass my.cubin | head -5  →  // .used_regs = 64

// Step 2: 如需控制 occupancy，保守设置
__global__ __launch_bounds__(256)  // 只限制 max，不指定 min，让编译器自由优化
void myKernel(...) { }

// Step 3: 仅在确定 occupancy 受寄存器限制时，才加 minBlocksPerSM
__global__ __launch_bounds__(128, 4)  // 确保 4 blocks/SM，接受可能的 spill
void myKernelHighOccupancy(...) { }
```

---

### 1.3 `maxrregcount` 编译选项

`-maxrregcount=N` 是 **nvcc 全局选项**，对整个编译单元中所有 kernel 生效：

```bash
nvcc -maxrregcount=64 -arch=sm_86 kernel.cu -o kernel
```

与 `__launch_bounds__` 的对比：

| 特性 | `__launch_bounds__` | `-maxrregcount` |
|------|---------------------|-----------------|
| 作用粒度 | 单个 kernel | 整个 .cu 文件 |
| 优先级 | 高（覆盖 maxrregcount） | 低 |
| 与编译器协商 | 是（基于 block 大小推算） | 否（硬性上限） |
| 调试友好 | 较好 | 较差 |

**使用场景：**
```bash
# 场景1：快速测试不同寄存器数对性能的影响
for REG in 32 48 64 96 128; do
    nvcc -maxrregcount=$REG -arch=sm_86 kernel.cu -o kernel_$REG
    ./benchmark kernel_$REG
done

# 场景2：保守限制，避免单个复杂 kernel 占满寄存器文件
# 但要在 PTX/SASS 中确认是否产生 spill
nvcc -maxrregcount=128 -arch=sm_90 -Xptxas="-v" kernel.cu
# -v 会输出: Used 128 registers, 48 bytes smem, 0 bytes lmem
# 如果 lmem > 0 则有 spill
```

**诊断命令：**
```bash
# 查看每个 kernel 的寄存器使用
nvcc -Xptxas="-v" -arch=sm_86 kernel.cu 2>&1 | grep "registers"
# 输出: ptxas info : Used 96 registers, 12288 bytes smem, 400 bytes lmem
# lmem > 0 意味着有 spill！
```

---

### 1.4 PTX vs SASS 层面的 Spill 对比

**PTX 层面的 Spill 标志：**
```ptx
// PTX 中 local memory 变量声明
.local .align 4 .b8 __local_depot[256];  // 256 字节 local memory

// local store（spill out）
st.local.f32   [%rd0+0], %f1;

// local load（spill reload）
ld.local.f32   %f2, [%rd0+4];
```

**SASS 层面的 Spill 指令：**
```sass
// Ampere SASS
STL.64    [R1+0x10], R4          // store to local (spill out)
LDL.64    R6, [R1+0x10]          // load from local (spill reload)

// 诊断命令
cuobjdump --dump-sass my.cubin | awk '/LDL|STL/{count++} END{print count, "spill instructions"}'
```

**PTX vs SASS 的关键区别：**

| 层面 | 特点 |
|------|------|
| PTX | 虚拟寄存器，spill 明显可见，但 PTX 优化可消除部分 |
| SASS | 物理寄存器，是最终执行代码，SASS 中无 LDL/STL 表示无 spill |
| 编译器 | 可能在 PTX→SASS 过程中消除某些 PTX spill（寄存器着色优化） |

**重要结论**：只有 SASS 层面的 `LDL/STL` 才是真正的 spill，PTX 中的 `ld.local/st.local` 不一定最终变成 SASS 中的 LDL/STL。

---

### 1.5 通过 ILP 减少 Spill 需求

**ILP（Instruction-Level Parallelism）减少 Spill 的原理：**

当代码具有足够的 ILP 时，编译器可以**复用寄存器**——一个计算完成后，其寄存器立即被另一个独立计算复用，不需要同时持有大量"活跃"变量。

**反面教材（高 spill 风险）：**
```cuda
// 大型展开循环，大量变量同时活跃
float a0 = A[i+0], a1 = A[i+1], a2 = A[i+2], a3 = A[i+3];
float b0 = B[i+0], b1 = B[i+1], b2 = B[i+2], b3 = B[i+3];
// ... 8 个变量同时活跃，需要 8 个寄存器
float c0 = a0 * b0;
float c1 = a1 * b1;
// c0..c3 也需要寄存器 → 12 个寄存器同时活跃
```

**优化方式（交错 ILP）：**
```cuda
// 交错加载-计算，减少同时活跃变量数
float a0 = A[i+0], b0 = B[i+0];
float c0 = a0 * b0;  // a0, b0 可以被复用
float a1 = A[i+1], b1 = B[i+1];
float c1 = a1 * b1;  // 同理，最多只有 3 个变量同时活跃
```

**具体技巧：**

```cuda
// 技巧1：缩短变量生命周期
__global__ void kernel(float* out, float* A, float* B, int N) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx >= N) return;
    
    // 不好：所有 tmp 同时活跃
    // float t0, t1, t2, t3;
    // t0 = A[idx] + 1.0f;
    // t1 = B[idx] * 2.0f;
    // t2 = t0 + t1;
    // t3 = sqrtf(t2);
    // out[idx] = t3;
    
    // 好：流水线式，减少同时活跃的变量
    float t = A[idx] + 1.0f;
    t += B[idx] * 2.0f;   // t 被复用
    t = sqrtf(t);
    out[idx] = t;
}

// 技巧2：手动 loop unroll 控制 unroll factor
// 过度展开（如 #pragma unroll 16）可能导致 16 个循环变量同时活跃
#pragma unroll 4  // 适度展开，平衡 ILP 和寄存器压力
for (int k = 0; k < K; k += 4) {
    acc += A[k] * B[k];
    acc += A[k+1] * B[k+1];
    acc += A[k+2] * B[k+2];
    acc += A[k+3] * B[k+3];
}

// 技巧3：使用 __restrict__ 帮助编译器别名分析
__global__ void kernel(float* __restrict__ out, 
                       const float* __restrict__ A,
                       const float* __restrict__ B) {
    // 编译器知道 out/A/B 不重叠，可以更激进地重用寄存器
}
```

---

## 2. Occupancy 深入

### 2.1 Occupancy 计算基础

**Occupancy 定义：**
```
Occupancy = Active Warps per SM / Maximum Warps per SM
```

- **Theoretical Occupancy**：由 launch config + 资源限制静态计算得出
- **Achieved Occupancy**：kernel 实际运行时，warp scheduler 观测到的平均值

最大 warp 数（以 Ampere sm_86 为例）：
- 每 SM 最大 48 个 warps（= 1536 threads / 32）
- 每 SM 最大 32 个 blocks
- 每 SM 寄存器文件：65536 × 32-bit
- 每 SM 最大共享内存：100 KB（可配置 L1/smem 分区）

**Theoretical Occupancy 计算（手动推导）：**

```python
# 示例：sm_86, block_size=256, regs_per_thread=64, smem_per_block=8192 bytes

max_warps_per_sm = 48         # hardware limit
max_blocks_per_sm = 32        # hardware limit
reg_file_size = 65536         # 32-bit registers per SM
max_smem_per_sm = 102400      # 100 KB

# 限制1：寄存器限制
warps_per_block = 256 // 32   # = 8
regs_per_warp = 64 * 32       # 64 regs/thread * 32 threads = 2048
blocks_by_regs = reg_file_size // (regs_per_warp * warps_per_block)
# = 65536 // (2048 * 8) = 65536 // 16384 = 4 blocks

# 限制2：共享内存限制
blocks_by_smem = max_smem_per_sm // 8192  # = 12 blocks

# 限制3：block 数量限制
blocks_by_count = max_blocks_per_sm  # = 32 blocks

# 取最小值
active_blocks = min(blocks_by_regs, blocks_by_smem, blocks_by_count)  # = 4
active_warps = active_blocks * warps_per_block  # = 4 * 8 = 32
occupancy = active_warps / max_warps_per_sm  # = 32/48 ≈ 66.7%
```

**CUDA Occupancy Calculator API：**
```cuda
#include <cuda_runtime.h>

int minGridSize, blockSize;
// 让 CUDA runtime 推荐最佳 block size
cudaOccupancyMaxPotentialBlockSize(
    &minGridSize,   // 推荐的最小 grid 大小（波次）
    &blockSize,     // 推荐的 block 大小
    myKernel,       // kernel 函数指针
    0,              // 动态 smem 大小（字节）
    0               // 最大 block 大小限制（0=无限制）
);

// 查询特定 block 大小的 occupancy
int numBlocks;
cudaOccupancyMaxActiveBlocksPerMultiprocessor(
    &numBlocks, myKernel, blockSize, sharedMemPerBlock);
float occupancy = (float)(numBlocks * blockSize) / maxThreadsPerSM;
```

---

### 2.2 Theoretical vs Achieved Occupancy 的差异

即使 Theoretical Occupancy = 100%，Achieved Occupancy 可能远低于此。

**差异来源：**

| 来源 | 说明 |
|------|------|
| Grid 大小不足 | 总 block 数 < SMs × blocks_per_SM |
| 尾部效应（Tail Effect） | 最后一波 blocks 不足以填满所有 SM |
| Warp divergence 导致提前退出 | 部分 warp 因条件判断提前结束，占用 slot 但不执行 |
| 资源竞争 | 如 hardware barrier 耗尽 |
| 依赖关系 | global memory 同步导致 warp 提前退出 |

**实际案例：**
```
Theoretical Occupancy: 75%  (48 warps 中 36 个)
Achieved Occupancy:    52%  (平均只有 25 个 warp 活跃)

可能原因：
- Grid 较小，某些 SM 分到的 blocks 较少
- 末尾 blocks 数量不足填满所有 SM
- Wave 数量少，最后一个 wave 只覆盖部分 SM
```

**Nsight Compute 指标：**
```bash
ncu --metrics sm__warps_active.avg.pct_of_peak_sustained_active \
              sm__warps_eligible.avg.pct_of_peak_sustained_active \
    ./my_kernel

# sm__warps_active  = achieved occupancy (%)
# sm__warps_eligible = 真正 ready to issue 的 warps (%)
```

---

### 2.3 Wave Quantization 问题

**Wave（波次）**：一次能同时覆盖所有 SM 的 CTA 组合称为一个 Wave。

```
Wave Size = num_SMs × active_blocks_per_SM

例：A100 有 108 个 SM，每 SM 4 个 blocks → Wave Size = 432 blocks
```

**Wave Quantization 代价：**

如果 grid 大小不是 wave size 的整数倍，最后一个 partial wave 会浪费资源。

```
例：
- Wave size = 432 blocks
- Total blocks = 500
- Wave 1: 432 blocks（全满）
- Wave 2: 68 blocks（只有 68/432 ≈ 15.7% 利用率）
- 整体 "tail efficiency" = (500) / (2 * 432) = 57.9%！
```

**量化损失计算：**
```python
def wave_quantization_efficiency(total_blocks, wave_size):
    full_waves = total_blocks // wave_size
    tail_blocks = total_blocks % wave_size
    if tail_blocks == 0:
        return 1.0
    total_wave_capacity = (full_waves + 1) * wave_size
    return total_blocks / total_wave_capacity

# 调优建议：让 total_blocks 尽量是 wave_size 的整数倍
# 或调整 block_size 改变 wave_size 来更好地匹配 problem_size
```

**实际调优策略：**
```cuda
// 方案1：调整 grid_size 以匹配 wave boundary
int wave_size = num_sms * blocks_per_sm;  // 运行时查询
int grid_size = ((N + block_size - 1) / block_size);
// 向上对齐到 wave_size 的整数倍（若 problem 允许多余计算）
grid_size = ((grid_size + wave_size - 1) / wave_size) * wave_size;

// 方案2：使用 persistent kernel 彻底避免 wave quantization
// （见第6章）
```

---

### 2.4 寄存器数 vs Occupancy Tradeoff

这是一个经典的**三角关系**：

```
高寄存器数 → 低 Occupancy → 低延迟隐藏能力
低寄存器数 → 高 Occupancy → 可能 Spill → 实际更慢
```

**量化分析框架：**

```
Latency = Memory_Latency × (1 - Occupancy_Hide_Factor)

当 Occupancy 足够高（>50%），增加 occupancy 对隐藏延迟的边际效益递减
当 Occupancy 很低（<25%），提高 occupancy 有显著收益
```

**实际寄存器-Occupancy 映射（sm_86）：**

| 寄存器/线程 | 最大 Occupancy（256 threads/block） |
|------------|--------------------------------------|
| 32 | 100% (48 warps) |
| 40 | 75% (36 warps, 3 blocks × 8 warps) |
| 48 | 75% (36 warps) |
| 64 | 50% (24 warps, 2 blocks × 8 warps) |
| 128 | 25% (12 warps, 1 block × 8 warps) |
| 256 | 12.5% (6 warps, 1 block × 6 warps，极限) |

**决策原则：**
1. Memory-bound kernel：优先高 occupancy（多 warps 掩盖 memory latency）
2. Compute-bound kernel：occupancy 影响较小，优先高寄存器（减少 spill，更好 ILP）
3. 经验值：多数 kernel 在 50~75% occupancy 时性能较好

---

## 3. Memory 访问深入

### 3.1 Global Memory Coalescing 原理

**Coalescing 基础：**

Global memory 以 **128-byte cache line**（对应 L2 sector 大小）为单位访问。当一个 warp（32 threads）发出 global memory 请求时：

- 理想情况：32 个 4-byte float 访问连续地址 → 合并为 **1 个 128-byte 事务**
- 最差情况：32 个完全随机地址 → 最多 **32 个独立事务**

**量化公式：**
```
Memory Efficiency = Requested Bytes / Actual Bytes Transferred
= (32 × element_size) / (transactions × 128)
```

**访问模式分析：**

```cuda
// Pattern 1: 完美 coalesced（row-major, 连续访问）
// Thread i 访问 A[i] → 效率 100%
float val = A[threadIdx.x + blockIdx.x * blockDim.x];

// Pattern 2: Stride-2 访问
// Thread i 访问 A[2*i] → 2 个 128B 事务，效率 50%
float val = A[2 * (threadIdx.x + blockIdx.x * blockDim.x)];

// Pattern 3: Stride-32 访问（最差情况之一）
// 每个 thread 隔 32 个 float = 128 bytes 访问 → 32 个事务，效率 3%
float val = A[32 * (threadIdx.x + blockIdx.x * blockDim.x)];

// Pattern 4: 结构体-of-数组 vs 数组-of-结构体
struct Bad { float x, y, z; } soa_bad[N];  // AoS: stride-3 for x-only access
// 改为:
float x[N], y[N], z[N];  // SoA: coalesced for each component
```

**Misaligned 访问代价：**

```cuda
// 未对齐访问（非 128-byte 对齐的起始地址）
// 可能产生额外的 partial cache line 请求
float* misaligned = (float*)((char*)data + 4);  // 偏移 4 bytes
// 即使访问连续，也会产生 2 个 cache line 请求而非 1 个

// 解决方案：使用 cudaMalloc 保证对齐（默认 256-byte 对齐）
// 或使用 __align__ 属性
struct __align__(16) Vec4 { float x, y, z, w; };
```

**实际测量（Nsight Compute 指标）：**
```
l1tex__average_t_sectors_per_request_pipe_lsu_mem_global_op_ld  
# 理想值=1（完全 coalesced），越大越差

# 同等指标
l1tex__t_sectors_pipe_lsu_mem_global_op_ld.sum  /  
l1tex__t_requests_pipe_lsu_mem_global_op_ld.sum
= 平均每请求 sectors 数（1 sector = 32 bytes）
```

---

### 3.2 Shared Memory Bank Conflict

**Bank 结构（Ampere/Volta/Turing）：**
- 32 个 banks，连续的 32-bit words 依次映射到 bank 0, 1, ..., 31, 0, 1, ...
- bank 0 包含地址 0, 128, 256, ... bytes（每 128 bytes 循环一次）
- **4-byte stride（相邻 int/float）→ 相邻 thread → 不同 bank → 无冲突**

```
Bank 映射公式：bank_id = (address / 4) % 32
```

**Bank Conflict 场景：**

```cuda
__shared__ float smem[32][32];

// 无冲突：32 threads 各访问 smem[threadIdx.x][col]（行访问）
// thread 0 → bank 0, thread 1 → bank 1, ..., thread 31 → bank 31
float val = smem[threadIdx.x][col];  // Perfect

// 2-way conflict：stride=2
// thread 0 → bank 0, thread 1 → bank 2, ..., thread 16 → bank 0 (conflict!)
float val = smem[2 * threadIdx.x][0];  // 2-way bank conflict

// 32-way conflict（最坏情况）：所有 thread 访问同一 bank
// thread 0..31 都访问 smem[0][0], smem[32][0], ...(即 column 0)
// 在转置场景中极其常见！
float val = smem[threadIdx.x * 32][0];  // 全部 → bank 0！

// 经典矩阵转置 bank conflict 问题
__shared__ float tile[32][32];
// 写入：无冲突（行写入）
tile[threadIdx.y][threadIdx.x] = A[...];  // OK
__syncthreads();
// 读取：32-way conflict！（列读取，32 threads 访问 column i → 全是 bank i%32）
float v = tile[threadIdx.x][threadIdx.y];  // CONFLICT!
```

**修复方案 - Padding：**

```cuda
// 经典 padding 技巧：在列上加 1（或更多）padding
__shared__ float tile[32][33];  // +1 padding
// 现在 bank_id = (row * 33 + col) / 1 % 32
// 每行起始 bank 错开，消除列访问冲突

// 验证：列访问
// thread 0: tile[0][0] → bank 0
// thread 1: tile[1][0] → bank (1*33/1)%32 = 33%32 = 1 ✓
// thread 2: tile[2][0] → bank 66%32 = 2 ✓
// ... 完美！

// 更通用的 padding 计算
// 目标：相邻行的相同列位于不同 bank
// padding_size = (bank_count / gcd(bank_count, row_width)) 
// 通常 padding 1 个元素已足够（当 row_width 不是 bank_count 的因子时）
```

**检测 Bank Conflict（Nsight Compute）：**
```bash
ncu --metrics l1tex__data_bank_conflicts_pipe_lsu_mem_shared_op_ld.sum \
              l1tex__data_bank_conflicts_pipe_lsu_mem_shared_op_st.sum \
    ./my_kernel

# conflicts 应为 0 或接近 0
# 高冲突数 → 需要 padding 或访问模式重新设计
```

**16-bit 元素的特殊情况（Ampere+）：**
```cuda
// 16-bit 元素：两个连续 half 组成一个 32-bit word，共享同一 bank
// 要避免同一 warp 中相邻 thread 访问同一个 32-bit word 中的不同 half
__shared__ __half smem[32][64];  // 64 halfs per row
// 每 2 个相邻 half → 同一 bank word → 若 2 threads 访问同一 word 可能出问题
// 通常需要 padding 2 个 half（即 1 个 int）
__shared__ __half smem[32][66];  // padding 2 halfs per row
```

---

### 3.3 L1/L2 Cache 命中率分析

**Cache 层次（Ampere sm_86）：**

| 层次 | 大小（per SM） | 延迟（cycles） | 带宽 |
|------|---------------|----------------|------|
| Register | 64K × 32bit | 0 | ~16 TB/s |
| L1 / Shared | 128 KB（可配 32+96 或 64+64 KB） | 28~34 | ~19 TB/s per SM |
| L2 | 4~40 MB（GPU 全局） | 200~300 | 1.6 TB/s (A100) |
| HBM2e (DRAM) | 40~80 GB | 400~800 | 2 TB/s (A100) |

**L1 Cache 配置：**
```cuda
// 配置 L1 / Shared Memory 分区（per kernel）
// 在 A100 上 L1 可配置为 32/64/96/100 KB
cudaFuncSetAttribute(myKernel, 
    cudaFuncAttributePreferredSharedMemoryCarveout, 
    cudaSharedmemCarveoutMaxShared);  // 最大化 shared memory
// 或
cudaFuncSetAttribute(myKernel,
    cudaFuncAttributePreferredSharedMemoryCarveout,
    cudaSharedmemCarveoutMaxL1);      // 最大化 L1

// 特定 shared memory 大小
cudaFuncSetAttribute(myKernel,
    cudaFuncAttributeMaxDynamicSharedMemorySize, 
    98304);  // 96 KB dynamic smem（需要 opt-in）
```

**命中率分析（Nsight Compute）：**
```bash
# L1 命中率
ncu --metrics l1tex__t_sector_hit_rate.pct ./my_kernel

# L2 命中率  
ncu --metrics lts__t_sector_hit_rate.pct ./my_kernel

# DRAM 带宽利用率
ncu --metrics gpu__dram_throughput.avg.pct_of_peak_sustained_elapsed ./my_kernel
```

**提高 L1 命中率的策略：**
```cuda
// 1. 时间局部性：相同数据被多次访问（如矩阵乘法中的 A/B 块）
// 2. 空间局部性：coalesced access
// 3. 数据重用：将热数据缓存到 shared memory
// 4. 非缓存加载（当数据流式访问，不会重用时，避免污染 cache）
float4 val = __ldcs((float4*)&A[idx]);  // cache streaming hint（L2 not L1）
float4 val2 = __ldlu((float4*)&A[idx]); // cache last use hint
```

**L2 持久化（Hopper/Ampere 特性）：**
```cuda
// Ampere 支持 L2 持久化区域，将频繁访问的数据 pin 在 L2
cudaDeviceProp prop;
cudaGetDeviceProperties(&prop, 0);
size_t l2_persist_size = min((size_t)prop.accessPolicyMaxWindowSize, hot_data_size);

cudaStreamAttrValue stream_attr;
stream_attr.accessPolicyWindow.base_ptr = (void*)hot_data;
stream_attr.accessPolicyWindow.num_bytes = l2_persist_size;
stream_attr.accessPolicyWindow.hitRatio = 1.0f;   // 100% 命中时使用持久策略
stream_attr.accessPolicyWindow.hitProp = cudaAccessPropertyPersisting;
stream_attr.accessPolicyWindow.missProp = cudaAccessPropertyStreaming;
cudaStreamSetAttribute(stream, cudaStreamAttributeAccessPolicyWindow, &stream_attr);
```

---

### 3.4 连续 vs 跨步 vs 随机访问模式性能差异

**基准测试结论（A100，float32）：**

| 访问模式 | 有效带宽 | L2 命中率 | 说明 |
|----------|----------|-----------|------|
| 连续（stride=1） | ~2 TB/s | 30~50% | 接近峰值 |
| stride=2 | ~1 TB/s | 30~50% | 50% 效率 |
| stride=4 | ~500 GB/s | 20~40% | 25% 效率 |
| stride=32 | ~80 GB/s | <10% | 3% 效率（每线程独立 cache line）|
| 完全随机 | ~50 GB/s | <5% | L2 几乎全 miss |

**带宽损失的底层原因：**

```
Stride-N 访问时：
- 每个 warp（32 threads）触发 min(N, 32) 个独立的 128B cache line 请求
- Stride=1: 1 cache line → 128B utilized / 128B fetched = 100%
- Stride=2: 2 cache lines → 128B utilized / 256B fetched = 50%
- Stride=32+: 32 cache lines → 128B utilized / 4096B fetched = 3.1%

有效带宽 ≈ Peak_BW × (1 / stride)（stride ≤ 32 时近似成立）
```

**实际代码优化：**
```cuda
// 问题：AoS 导致跨步访问（对 x 分量 stride=3）
struct Particle { float x, y, z; };
Particle* particles = ...;
// Thread i 访问 x: particles[i].x → 地址间隔 12 bytes → stride=3
float x = particles[threadIdx.x].x;  // BAD

// 解决：SoA 布局
float* px, *py, *pz;
float x = px[threadIdx.x];  // GOOD: stride=1, coalesced

// 半精度 SoA（带宽最优）
__half* hx, *hy, *hz;
// 每 2 个 half = 4 bytes = 1 bank → half2 进一步优化
half2 xy_pair = ((half2*)hxy)[threadIdx.x];  // 2 components per load
```

---

## 4. Warp 执行深入

### 4.1 Warp Divergence 代价

**SIMT 模型与 Divergence：**

一个 warp 的 32 threads 在同一时刻执行同一指令（SIMT：Single Instruction Multiple Threads）。当遇到 `if/else` 时：
- 两个分支都会执行，但使用 **predicate mask** 屏蔽不需要执行的 thread
- 代价 = 两条路径的总执行时间（串行化）

**Predicated Execution vs 真正的 Branch：**

编译器对短 if/else 使用 **predicate**（条件执行）而非真正跳转：

```ptx
// CUDA 源码
if (x > 0) y = x * 2;
else       y = -x;

// 编译为 predicated PTX（无跳转指令）：
setp.gt.f32  p, %f1, 0.0;
@p  fma.f32   %f2, %f1, 2.0, 0.0;  // 只有 p=true 的 thread 执行
@!p neg.f32   %f2, %f1;             // 只有 p=false 的 thread 执行
// 两条指令都发射，但各自屏蔽不执行的 thread

// 对于较长的 if 块，编译器改用真正的 branch（BRA 指令）
// 有 divergence 时 BRA 会串行化两个路径
```

**实际代价量化：**

```
完全 divergent（50/50 split）的 warp：
- 执行代价 ≈ branch_A_cost + branch_B_cost（两路串行）
- 相当于每路有效 throughput 降低 50%

最坏情况（1 thread 走 A，31 threads 走 B）：
- 代价几乎等于 1 个 warp 执行两路的 latency
- 但在实际 warp 池中影响相对较小（只有该 warp 效率低）
```

**减少 Divergence 的技巧：**

```cuda
// 技巧1：数据重排，让相同分支的 thread 在同一 warp
// 坏：奇偶分布，导致每个 warp 一半走 A 一半走 B
if (data[tid] % 2 == 0) { ... }  // 随机数据 → 50% divergence

// 好：先按条件 sort/partition 数据，再 launch kernel
// (使用 thrust::partition 预处理)

// 技巧2：用算术代替 branch
// 坏：
float result;
if (x > 0) result = x;
else        result = 0;

// 好（无 branch）：
float result = fmaxf(x, 0.0f);   // 使用内置函数
// 或：float result = x * (float)(x > 0);

// 技巧3：warp-level 条件（所有 thread 条件相同）
// 如 tid-based condition 通常不会导致 divergence
if (blockIdx.x % 2 == 0) { ... }  // 整个 block 条件相同，无 divergence

// 技巧4：__ballot_sync 处理异构任务
uint32_t mask = __ballot_sync(0xffffffff, condition);
if (__popc(mask) > 16) {
    // 多数路径：更高效地处理
}
```

**Branch Efficiency 指标：**
```bash
ncu --metrics smsp__sass_average_branch_targets_threads_uniform.pct \
    ./my_kernel
# 100% = 无 divergence
# 低值 = 高 divergence，需要关注
```

---

### 4.2 Warp Stall 各种原因详解

Warp stall 是 latency hiding 失败的直接体现。以下是 Nsight Compute 中的 stall 类型：

**1. Long Scoreboard (lgscb)**
- **含义**：等待 long-latency 操作（global/local memory）
- **Nsight 指标**：`smsp__warp_issue_stalled_long_scoreboard_per_warp_active`
- **原因**：global load 未完成，相关寄存器被 scoreboard 标记为 "in-flight"
- **延迟**：L2 miss → ~200-800 cycles；L1 hit → ~28-34 cycles
- **解决**：
  ```cuda
  // 1. 提前加载（prefetch）
  // 先发出下一个 iteration 的 load，再处理当前 iteration
  float prefetch = A[i + blockDim.x];  // 提前一个 iteration 加载
  __syncthreads();
  // process current data
  // use prefetch in next iteration
  
  // 2. 增加 occupancy（更多 warps 掩盖 latency）
  // 3. 使用 shared memory + cp.async（见第6章）
  ```

**2. Short Scoreboard (stscb)**
- **含义**：等待 short-latency 操作（shared memory, 部分特殊函数）
- **Nsight 指标**：`smsp__warp_issue_stalled_short_scoreboard_per_warp_active`
- **原因**：shared memory load 结果还未就绪（~20-40 cycles）
- **解决**：在 shared memory load 和使用之间插入独立指令

**3. Barrier (sync)**
- **含义**：等待 `__syncthreads()` 或 cooperative groups barrier
- **Nsight 指标**：`smsp__warp_issue_stalled_barrier_per_warp_active`
- **原因**：barrier 前，部分 warp 先到达，等待其他 warp
- **解决**：
  ```cuda
  // 减少 barrier 次数，批量操作
  // 坏：
  for (int i = 0; i < N; i++) {
      smem[tid] = A[i];
      __syncthreads();
      result += process(smem);
      __syncthreads();
  }
  
  // 好：使用 double buffering 重叠（见第6章）
  ```

**4. Synchronization (sync_wait)**
- **含义**：等待 `__syncwarp()` 或 warp-level 同步原语
- **解决**：尽量用 `__syncthreads()` 替代（或使用 warp-shfl 无需同步）

**5. Wait (wait)**
- **含义**：等待前一个指令管线中的依赖（execution dependency）
- **例**：R1 由 FP instruction 生成，下一条指令立即使用 R1
- **延迟**：FP 约 6 cycles，INT 约 6 cycles，依赖延迟≈管线深度
- **解决**：增加指令间的独立计算（提高 ILP）
  ```cuda
  // 坏（RAW hazard）：
  float x = a * b;
  float y = x + c;  // x 的 latency 约 6 cycles，等待
  
  // 好（ILP）：
  float x0 = a0 * b0;
  float x1 = a1 * b1;  // 与 x0 独立，管线可以重叠
  float y0 = x0 + c0;
  float y1 = x1 + c1;
  ```

**6. Throttle (throt)**
- **含义**：MIO（Memory I/O）管线被 throttle，或调度器没有 eligible warp
- **原因**：
  - 内存请求队列满（过多 in-flight 的 memory 操作）
  - No eligible warp（所有 active warp 都在 stall）
- **Nsight 指标**：`smsp__warp_issue_stalled_mio_throttle_per_warp_active`
- **解决**：
  - 减少 outstanding memory requests 密度
  - 增加 occupancy 增加 warp 数
  - 改善 memory access pattern（coalescing）

**Warp Stall 综合查看命令：**
```bash
ncu --section WarpStateStats \
    --metrics smsp__warp_issue_stalled_long_scoreboard_per_warp_active,\
smsp__warp_issue_stalled_short_scoreboard_per_warp_active,\
smsp__warp_issue_stalled_barrier_per_warp_active,\
smsp__warp_issue_stalled_wait_per_warp_active,\
smsp__warp_issue_stalled_mio_throttle_per_warp_active \
    ./my_kernel
```

---

### 4.3 Active Cycles vs Elapsed Cycles vs SM Active Cycles

**三种 Cycle 定义：**

| 指标 | 定义 | 对应指标 |
|------|------|----------|
| Elapsed Cycles | 从 kernel 开始到结束的总 GPU wall-clock cycles | `gpc__cycles_elapsed` |
| SM Active Cycles | SM 上至少有一个 warp active 的 cycles | `sm__cycles_active` |
| SM Elapsed Cycles | SM 的实际 elapsed cycles（包括 idle）| `sm__cycles_elapsed` |

**关系与用途：**
```
Active Cycles ≤ Elapsed Cycles

SM Utilization = SM Active Cycles / SM Elapsed Cycles

如果 SM Active Cycles << SM Elapsed Cycles：
- Grid 太小，SM 闲置
- 或 kernel 刚启动/即将结束（tail latency）

典型比率：
- 良好：Active/Elapsed > 90%
- 差：Active/Elapsed < 50%（表明 SM 大量空转）
```

**Nsight Compute 指标示例：**
```bash
ncu --metrics sm__cycles_active.sum \
              sm__cycles_elapsed.sum \
              smsp__cycles_active.avg \
              smsp__cycles_elapsed.avg \
    ./my_kernel

# 计算 SM 利用率
SM_Util = sm__cycles_active.sum / sm__cycles_elapsed.sum

# 注意：sm 是 per-SM 级别，smsp 是 per-sub-partition 级别
# 每个 SM 有 4 个 sub-partitions（SMSP）
```

---

## 5. Compute 深入

### 5.1 Tensor Core 使用条件

Tensor Core 是专门执行矩阵乘加（MMA）操作的硬件单元，从 Volta 架构引入。

**Tensor Core 世代与能力（per SM）：**

| 架构 | 代数 | FP16 TOPS | BF16 TOPS | TF32 TOPS | INT8 TOPS | FP8 TOPS |
|------|------|-----------|-----------|-----------|-----------|----------|
| Volta (V100) | 1st | 125 | N/A | N/A | N/A | N/A |
| Turing (T4) | 2nd | 65 | N/A | N/A | 130 | N/A |
| Ampere (A100) | 3rd | 312 | 312 | 312 (TF32) | 624 | N/A |
| Hopper (H100) | 4th | 1000 | 1000 | 500 | 2000 | 2000 |

**矩阵尺寸对齐要求：**

Tensor Core 操作格式：`D = A × B + C`，矩阵尺寸 MxNxK：

```
WMMA API（Volta+）的支持格式：
- FP16: m=16, n=16, k=16（基础格式）
- FP16: m=32, n=8, k=16; m=8, n=32, k=16（Turing+）
- INT8: m=16, n=16, k=32（Turing+）
- BF16: m=16, n=16, k=16（Ampere+）
- TF32: m=16, n=16, k=8（Ampere+）
- FP8: m=16, n=16, k=32（Hopper+）

关键约束：
1. 矩阵必须以特定尺寸的倍数对齐（M、N 是 16 的倍数，K 是 16 或 32 的倍数）
2. 数据必须连续存储（row-major 或 column-major，无自定义 stride 或需对齐）
3. 访问必须 warp-level 协作（整个 warp 共同执行一个 MMA 操作）
```

**通过 WMMA API 使用 Tensor Core：**
```cuda
#include <mma.h>
using namespace nvcuda::wmma;

__global__ void tensorcore_gemm(half* A, half* B, float* C, float* D,
                                 int M, int N, int K) {
    // 声明 fragment（warp-level 矩阵片段）
    fragment<matrix_a, 16, 16, 16, half, row_major> a_frag;
    fragment<matrix_b, 16, 16, 16, half, col_major> b_frag;
    fragment<accumulator, 16, 16, 16, float> c_frag;
    
    // warp 坐标
    int warpM = (blockIdx.x * blockDim.x + threadIdx.x) / warpSize;
    int warpN = (blockIdx.y * blockDim.y + threadIdx.y);
    
    // 初始化 accumulator
    fill_fragment(c_frag, 0.0f);
    
    // K-loop
    for (int k = 0; k < K; k += 16) {
        // 加载矩阵片段
        load_matrix_sync(a_frag, A + warpM * 16 * K + k, K);
        load_matrix_sync(b_frag, B + k * N + warpN * 16, N);
        
        // Tensor Core MMA 操作
        mma_sync(c_frag, a_frag, b_frag, c_frag);
    }
    
    // 存储结果
    store_matrix_sync(D + warpM * 16 * N + warpN * 16, c_frag, N, mem_row_major);
}
```

**cuBLAS 自动使用 Tensor Core：**
```cuda
// cuBLAS 会自动选择 Tensor Core（当精度/尺寸满足时）
cublasHandle_t handle;
cublasCreate(&handle);

// 启用 Tensor Core math（默认在 cuBLAS 中已启用）
cublasSetMathMode(handle, CUBLAS_TENSOR_OP_MATH);

// 对于 FP16 输入，FP32 累加：
cublasGemmEx(handle, CUBLAS_OP_N, CUBLAS_OP_T,
             N, M, K,
             &alpha, B_fp16, CUDA_R_16F, N,
                     A_fp16, CUDA_R_16F, K,
             &beta,  C_fp32, CUDA_R_32F, N,
             CUDA_R_32F,
             CUBLAS_GEMM_DEFAULT_TENSOR_OP);
```

---

### 5.2 Mixed Precision 实际吞吐 vs FP32

**理论 FLOPS 比较（A100 SXM4）：**

| 精度 | 峰值 FLOPS | vs FP32 倍数 |
|------|-----------|-------------|
| FP64 | 19.5 TFLOPS | 0.5× |
| TF32 (Tensor Core) | 312 TFLOPS | 8× |
| FP32 (CUDA Core) | 19.5 TFLOPS | 1× |
| FP16 (Tensor Core) | 312 TFLOPS | 16× |
| BF16 (Tensor Core) | 312 TFLOPS | 16× |
| INT8 (Tensor Core) | 624 TOPS | 32× |
| FP8 (H100 Tensor Core) | 2000 TOPS（H100） | N/A |

**实际注意事项：**

```cuda
// 精度选择策略
// TF32：FP32 的替代，精度损失小，吞吐 8× 提升，无需代码修改
// → 设置 CUBLAS_TENSOR_OP_MATH 即可自动使用

// FP16/BF16：权重和激活使用半精度，梯度用 FP32 累加
// BF16 vs FP16：
// - BF16: 8-bit exponent（与 FP32 相同），7-bit mantissa
// → 数值范围与 FP32 相同，不易溢出，适合训练
// - FP16: 5-bit exponent，10-bit mantissa
// → 精度更高但容易溢出，需要 loss scaling

// 实际代码（FP16 GEMM with FP32 accumulation）
__global__ void fp16_gemm_kernel(
    const __half* A, const __half* B, float* C,
    int M, int N, int K) {
    // 使用 WMMA 或 PTX MMA 指令
    // 输入 FP16，累加器 FP32
}

// AMP（Automatic Mixed Precision）- PyTorch
with torch.cuda.amp.autocast():
    output = model(input)  # 自动选择 FP16/BF16
scaler.scale(loss).backward()
```

---

### 5.3 Arithmetic Intensity 与 Roofline 模型实战

**Arithmetic Intensity（AI）定义：**
```
AI = FLOPs / Bytes_Memory_Transferred
```

**Roofline 模型：**
```
Attainable_Performance = min(
    Peak_FP_Performance,        // compute-bound ceiling
    Peak_Bandwidth × AI         // memory-bound ceiling
)

交叉点（Ridge Point）= Peak_FP_FLOPS / Peak_Bandwidth
```

**A100 SXM4 的 Roofline 参数：**
```
Peak FP32 CUDA:    19.5 TFLOPS
Peak FP16 TCore:   312  TFLOPS
Peak HBM2e BW:     2    TB/s
Ridge Point FP32:  19.5e12 / 2e12 = 9.75 FLOPs/Byte
Ridge Point FP16:  312e12  / 2e12 = 156  FLOPs/Byte
```

**常见算子的 AI 估算：**
```python
# GEMM（M=N=K=4096，FP32）
FLOPs = 2 * 4096 * 4096 * 4096 = 137.4 GFLOPs
Bytes = (4096*4096 + 4096*4096 + 4096*4096) * 4 = 201.3 MB (理想情况，无重复读)
AI = 137.4e9 / 201.3e6 ≈ 682 FLOPs/Byte
# → 远高于 Ridge Point，是 compute-bound（适合 Tensor Core）

# Vector ADD（N=1e8，FP32）
FLOPs = 1e8 (1 add per element)
Bytes = 3 * 1e8 * 4 = 1.2 GB (read A, read B, write C)
AI = 1e8 / 1.2e9 ≈ 0.083 FLOPs/Byte
# → 极度 memory-bound（AI 远低于 Ridge Point）

# BatchNorm（N×C×H×W，FP32）
FLOPs ≈ 5 * N * C * H * W
Bytes ≈ 2 * N * C * H * W * 4  (read input, write output)
AI = 5/8 ≈ 0.625 FLOPs/Byte
# → memory-bound
```

**在 Nsight Compute 中查看 Roofline：**
```bash
# 收集 Roofline 所需指标
ncu --set full \
    --section SpeedOfLight_RooflineChart \
    ./my_kernel

# 关键指标
# sm__throughput.avg.pct_of_peak_sustained_elapsed   → compute 利用率
# gpu__dram_throughput.avg.pct_of_peak_sustained_elapsed → memory 利用率
# sm__sass_thread_inst_executed_op_ffma_pred_on.sum → FP32 FFMA 指令数
```

**Roofline 分析决策树：**
```
如果 kernel 在 Roofline 图中：
├─ 远离两条 roofline → 执行效率低（warp stall、divergence）
├─ 接近 memory bandwidth roofline → memory-bound，优化 coalescing/cache
├─ 接近 compute roofline → compute-bound，优化 IPC/使用 Tensor Core
└─ 在 Ridge Point 右侧但低于 FP16 roofline → 考虑使用混合精度
```

---

### 5.4 FFMA vs HMMA vs IMMA 指令

**CUDA SASS 指令类别：**

| 指令 | 含义 | 数据类型 | 硬件 |
|------|------|----------|------|
| `FFMA` | Fused Floating-point Multiply-Add | FP32 | CUDA Core |
| `DFMA` | Double Fused MA | FP64 | CUDA Core (fewer units) |
| `HMMA` | Half-precision Matrix Multiply-Add | FP16/BF16 | Tensor Core |
| `IMMA` | Integer Matrix Multiply-Add | INT8/INT4 | Tensor Core |
| `QMMA` | FP8 Matrix Multiply-Add | FP8 | Tensor Core (Hopper) |
| `MUFU` | Multi-function unit | sin/cos/sqrt/rcp | SFU |
| `IADD3` | 3-operand integer add | INT32 | INT Core |

**在 SASS 中识别 Tensor Core 使用：**
```bash
cuobjdump --dump-sass my.cubin | grep "HMMA\|IMMA\|QMMA" | wc -l
# 如果有大量 HMMA 指令 → Tensor Core 在工作

# 完整分析
cuobjdump --dump-sass my.cubin | awk '
/FFMA/ {ffma++}
/HMMA/ {hmma++}
/IMMA/ {imma++}
/LDL|STL/ {spill++}
END {
    print "FFMA:", ffma
    print "HMMA:", hmma
    print "IMMA:", imma
    print "Spills:", spill
}'
```

**FP32 FFMA 吞吐计算（A100 SM）：**
```
每个 SM 有 4 个 SMSP，每个 SMSP 有 16 个 FP32 CUDA Core
每 cycle 每 SM 最大 64 个 FP32 FFMA
峰值 = 64 FFMA/cycle × 108 SMs × 1.41 GHz ≈ 9.7 TFLOPS（FP32，双精度操作）
实际峰值 = 19.5 TFLOPS（因为 FFMA = 2 FLOPs）
```

---

## 6. 高级优化技术

### 6.1 Persistent Kernel

**传统 Launch 模式的问题：**
- 每个 kernel launch 有 overhead（~几微秒）
- Grid 不能填满 GPU 时，存在 wave quantization 浪费
- 无法做 producer-consumer 模型

**Persistent Kernel 模式：**
```cuda
// 模式：一个 kernel 持续运行，内部处理任务队列
__global__ void persistent_kernel(
    TaskQueue* queue,
    float* output,
    int total_tasks
) {
    // 每个 thread block 持续抢占任务
    int task_id;
    while ((task_id = atomicAdd(&queue->head, 1)) < total_tasks) {
        // 处理 task_id 对应的任务
        process_task(task_id, queue->tasks[task_id], output);
    }
}

// Launch：只用足够的 blocks 填满 GPU（而不是按数据划分）
int num_sms, blocks_per_sm;
cudaDeviceGetAttribute(&num_sms, cudaDevAttrMultiProcessorCount, 0);
// 每 SM 例如 2 个 blocks，填满但不超过 GPU 容量
int grid_size = num_sms * blocks_per_sm;  // e.g., 108 * 2 = 216 blocks
persistent_kernel<<<grid_size, block_size>>>(queue, output, total_tasks);
```

**优势：**
- 消除 wave quantization（blocks 数恰好填满 GPU）
- 减少 kernel launch overhead
- 支持动态任务分配
- 适合 workloads 大小不确定的场景

**注意事项：**
- 需要原子操作的全局任务计数器（瓶颈）
- 缓存友好性下降（任务 locality 差）
- 需要特别处理 block 间的同步

---

### 6.2 Kernel Fusion

**Kernel Fusion 原理：**

将多个连续的 kernel 合并成一个，避免中间结果写回 DRAM。

```cuda
// 未融合：Elementwise → LayerNorm → Activation
// 每个 kernel 都要完整地读写 DRAM
elementwise_kernel<<<...>>>(A, B, tmp1);   // 写 tmp1 到 DRAM
layernorm_kernel<<<...>>>(tmp1, tmp2);      // 读 tmp1，写 tmp2
activation_kernel<<<...>>>(tmp2, output);   // 读 tmp2，写 output
// 总 DRAM 流量: 2N (elementwise) + 4N (layernorm) + 2N (activation) = 8N

// 融合后：单一 kernel，中间结果在寄存器/shared memory
__global__ void fused_kernel(float* A, float* B, float* output, int N) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx >= N) return;
    
    // Stage 1: elementwise
    float tmp = A[idx] + B[idx];
    
    // Stage 2: layernorm（需要 reduce，用 shared memory）
    __shared__ float smem[256];
    smem[threadIdx.x] = tmp;
    __syncthreads();
    // ... reduction 计算 mean/var
    float normalized = (tmp - mean) / sqrtf(var + eps);
    
    // Stage 3: activation
    output[idx] = tanhf(normalized);  // 或 GELU, ReLU 等
    // 总 DRAM 流量: 2N (读 A,B) + N (写 output) = 3N
}
```

**Fusion 适用条件：**
- 中间数据不需要全局可见性（无 grid-wide reduce 依赖）
- 融合后 shared memory 总需求不超过上限
- 融合后 register 压力可接受（避免 spill）

**工具辅助 Fusion：**
- **torch.compile** (PyTorch 2.0)：自动 kernel fusion
- **Triton**：编写 fused kernel 的高级语言
- **CUDA Graphs** + **cudnn**：也可实现一定程度的 fusion

---

### 6.3 Double Buffering / Software Pipelining

**问题：计算与内存访问的串行化**
```
传统方式：
[Load chunk 0] → [Compute chunk 0] → [Load chunk 1] → [Compute chunk 1]
时间线：  |||load|||....compute....|||load|||....compute....
```

**Double Buffering 思路：**
```
目标：
[Load chunk 1] 与 [Compute chunk 0] 重叠
时间线：  |||load 0|||
                    |||compute 0||| + |||load 1|||
                                     |||compute 1||| + |||load 2|||
```

**使用 Shared Memory 的 Double Buffering：**
```cuda
#define BLOCK_K 32

__global__ void dgemm_double_buffer(
    const float* A, const float* B, float* C,
    int M, int N, int K
) {
    __shared__ float smA[2][BLOCK_K][64];  // 2 buffers
    __shared__ float smB[2][BLOCK_K][64];
    
    int buf = 0;  // 当前 buffer 索引
    
    // 预加载第 0 块
    load_smem(smA[0], smB[0], A, B, 0, ...);
    __syncthreads();
    
    float acc[4][4] = {0};
    
    for (int k = 0; k < K; k += BLOCK_K) {
        int next_buf = 1 - buf;
        
        // 异步加载下一块（与当前计算重叠）
        if (k + BLOCK_K < K) {
            load_smem_async(smA[next_buf], smB[next_buf], A, B, k + BLOCK_K, ...);
        }
        
        // 计算当前块
        compute_tile(acc, smA[buf], smB[buf]);
        
        __syncthreads();  // 等待异步加载完成（或等下次 sync）
        buf = next_buf;
    }
    
    store_result(C, acc, ...);
}
```

---

### 6.4 `cp.async` 指令与 `__pipeline_memcpy_async`

`cp.async` 是 Ampere 引入的**异步内存拷贝指令**，允许 global → shared memory 的拷贝在后台进行，不占用 load/store 管线。

**PTX 层面：**
```ptx
// 异步拷贝 16 bytes from global to shared
cp.async.cg.shared.global [smem_addr], [global_addr], 16;

// 提交当前异步操作到一个 "commit group"
cp.async.commit_group;

// 等待最近的 N 个 commit group 中有 <= M 个尚未完成
cp.async.wait_group 0;  // 等待所有 cp.async 完成
cp.async.wait_group 1;  // 只等到还有 1 个 group 未完成（继续流水）
```

**CUDA C++ API（`cuda/pipeline`）：**
```cuda
#include <cuda/pipeline>

__global__ void async_pipeline_kernel(
    const float* global_in,
    float* output,
    int N
) {
    __shared__ float smem[2][BLOCK_SIZE];  // double buffer in shared
    
    __shared__ cuda::pipeline_shared_state<
        cuda::thread_scope_block, 2> pipeline_state;
    
    auto pipeline = cuda::make_pipeline(
        cuda::this_thread_block(), &pipeline_state);
    
    // 预填充 pipeline：先提交 stage 0
    cuda::memcpy_async(cuda::this_thread_block(),
                       smem[0], &global_in[0 * BLOCK_SIZE],
                       sizeof(float) * BLOCK_SIZE, pipeline);
    pipeline.producer_commit();
    
    for (int stage = 0; stage < N / BLOCK_SIZE; stage++) {
        // 提交下一 stage 的加载（异步）
        if (stage + 1 < N / BLOCK_SIZE) {
            cuda::memcpy_async(
                cuda::this_thread_block(),
                smem[(stage + 1) & 1],
                &global_in[(stage + 1) * BLOCK_SIZE],
                sizeof(float) * BLOCK_SIZE, pipeline);
            pipeline.producer_commit();
        }
        
        // 等待当前 stage 的数据就绪
        pipeline.consumer_wait();
        __syncthreads();
        
        // 使用 smem[stage & 1] 中的数据计算
        process(smem[stage & 1], output + stage * BLOCK_SIZE);
        
        pipeline.consumer_release();
    }
}
```

**`__pipeline_memcpy_async`（旧 API）：**
```cuda
#include <cooperative_groups/memcpy_async.h>

// 通过 __pipeline_memcpy_async 实现
// 现在推荐使用 cuda::memcpy_async（更现代）
```

**性能收益：**
```
传统 smem load：
global load → L1/L2 cache → 填充 smem（占用 LSU 管线）
→ warp 必须等待数据到达才能继续

cp.async：
数据直接从 L2/DRAM 填入 smem（绕过寄存器）
→ 不占用 LSU 管线
→ warp 可以继续执行其他指令
→ 计算与内存传输真正并行

实际收益：
- memory-bound kernel：20~40% 加速
- GEMM 类 kernel：配合 Hopper 的 Tensor Memory Accelerator (TMA) 可达 2× 加速
```

---

### 6.5 Stream 并行与 CUDA Graph

**CUDA Stream 基础：**
```cuda
// 创建多个流
cudaStream_t stream[4];
for (int i = 0; i < 4; i++) cudaStreamCreate(&stream[i]);

// 在不同流中并行执行
for (int i = 0; i < 4; i++) {
    cudaMemcpyAsync(d_in[i], h_in[i], size, cudaMemcpyHostToDevice, stream[i]);
    kernel<<<grid, block, 0, stream[i]>>>(d_in[i], d_out[i]);
    cudaMemcpyAsync(h_out[i], d_out[i], size, cudaMemcpyDeviceToHost, stream[i]);
}

// 同步所有流
for (int i = 0; i < 4; i++) cudaStreamSynchronize(stream[i]);
```

**CUDA Graph（减少 launch overhead）：**
```cuda
// 方法1：流捕获方式
cudaStream_t captureStream;
cudaStreamCreate(&captureStream);

cudaGraph_t graph;
cudaStreamBeginCapture(captureStream, cudaStreamCaptureModeGlobal);

// 在 captureStream 上 "执行" 的所有操作都被记录为节点
cudaMemcpyAsync(d_A, h_A, size, cudaMemcpyHostToDevice, captureStream);
kernel_A<<<grid, block, 0, captureStream>>>(d_A, d_B);
kernel_B<<<grid, block, 0, captureStream>>>(d_B, d_C);
cudaMemcpyAsync(h_C, d_C, size, cudaMemcpyDeviceToHost, captureStream);

cudaStreamEndCapture(captureStream, &graph);

// 实例化 graph（编译为可执行的 graph）
cudaGraphExec_t graphExec;
cudaGraphInstantiate(&graphExec, graph, NULL, NULL, 0);

// 重复执行（极低 overhead，只有 ~几微秒）
for (int iter = 0; iter < 1000; iter++) {
    cudaGraphLaunch(graphExec, captureStream);
    cudaStreamSynchronize(captureStream);
}

// 清理
cudaGraphExecDestroy(graphExec);
cudaGraphDestroy(graph);
```

**CUDA Graph 的性能收益：**
```
传统方式：每次 kernel launch 约 5~20 μs CPU overhead
CUDA Graph：整个 graph 一次 launch，<1 μs overhead

适用场景：
- 推理场景（同一网络重复运行）
- 周期性任务（每帧模拟）
- 小 kernel 密集的工作流（overhead 占比高）

局限性：
- Graph 拓扑固定（条件分支不支持）
- Dynamic parallelism 受限
- 某些 API 不支持流捕获
```

---

## 7. Nsight Compute 指标解读

### 7.1 Source Counter（每行代码 stall 分布）

Source Counter 是 Nsight Compute 最强大的功能之一，将 stall 信息映射回源代码/汇编行。

**收集方式：**
```bash
# 收集 Source Counter（需要 -lineinfo 编译）
nvcc -lineinfo -arch=sm_86 kernel.cu -o kernel
ncu --section SourceCounters \
    --source-folder ./  \
    ./kernel

# 或使用 GUI：Nsight Compute → 打开 profile → Source 页面
```

**Source Counter 指标含义：**

| 指标 | 说明 |
|------|------|
| `Branch` | 分支指令采样次数 |
| `Divergent Branch` | 发生 divergence 的分支 |
| `Active Cycles` | 此行代码活跃的 cycle 数 |
| `Long Scoreboard Stalls` | 因 long-latency 操作停顿的 cycle |
| `No Active Warps Stalls` | 调度器无可用 warp 的 cycle |

**解读示例：**
```
Source Line | Inst  | Stall (Long SB) | Notes
------------------------------------------------------------
float x = A[idx];   | LDG   | 45%             | <-- 主要瓶颈
float y = x * 2.0f; | FFMA  | 0%              |
```

如果 `LDG`（global load）指令的 `Long Scoreboard` stall 占比高（>20%），表明：
1. 内存访问延迟没有被充分隐藏
2. 需要提高 occupancy 或使用 prefetch/cp.async

---

### 7.2 Roofline Chart 解读

```
Roofline Chart 纵轴：Performance（GFLOPS 或 TOPS）
Roofline Chart 横轴：Arithmetic Intensity（FLOPs/Byte）

如何读：
1. 找到 kernel 的数据点（×）
2. 看它落在哪条 roofline 下方
   - 在 memory roofline 附近 → memory-bound（优化内存）
   - 在 compute roofline 附近 → compute-bound（优化计算）
   - 两者都远离 → 效率低，先查 warp stalls

3. 检查 "achieved" 性能离理论 roofline 的距离
   - 距离 roofline < 2× → 良好
   - 距离 roofline > 5× → 需要大幅优化
```

**Nsight Compute Roofline 命令：**
```bash
ncu --section SpeedOfLight_RooflineChart \
    --metrics sm__sass_thread_inst_executed_op_ffma_pred_on.sum,\
              sm__sass_thread_inst_executed_op_fadd_pred_on.sum,\
              sm__sass_thread_inst_executed_op_fmul_pred_on.sum,\
              gpu__dram_throughput.avg.pct_of_peak_sustained_elapsed,\
              l1tex__t_bytes.sum \
    ./my_kernel
```

---

### 7.3 Memory Workload Analysis

Memory Workload Analysis 展示了整个内存层次的流量和效率。

**关键指标与阈值：**

```bash
ncu --section MemoryWorkloadAnalysis ./my_kernel
```

| 指标 | 含义 | 关注阈值 |
|------|------|----------|
| `Memory Throughput` | 内存系统总吞吐利用率 | >80% 接近瓶颈 |
| `Mem Busy` | 内存管线繁忙程度 | 若为限制因素则关注 |
| `Max Bandwidth` | 带宽利用率 | 关键瓶颈指标 |
| `L1/TEX Hit Rate` | L1 缓存命中率 | <30% 考虑访问优化 |
| `L2 Hit Rate` | L2 命中率 | <50% 考虑数据局部性 |
| `DRAM Bandwidth` | HBM/GDDR 带宽利用率 | >80% 表示 memory-bound |

**内存访问效率：**
```bash
# Global Memory Load/Store Efficiency
ncu --metrics l1tex__average_t_sectors_per_request_pipe_lsu_mem_global_op_ld.ratio \
              l1tex__average_t_sectors_per_request_pipe_lsu_mem_global_op_st.ratio \
    ./my_kernel
# 最优值 = 1（理想 coalescing）
# > 4 表示严重 miscoalescing

# 共享内存 bank conflict
ncu --metrics l1tex__data_bank_conflicts_pipe_lsu_mem_shared_op_ld.sum \
              l1tex__data_bank_conflicts_pipe_lsu_mem_shared_op_st.sum \
    ./my_kernel
# = 0 最优
```

---

### 7.4 Compute Workload Analysis

**关键指标解读：**
```bash
ncu --section ComputeWorkloadAnalysis ./my_kernel
```

| 指标 | 含义 | 优化方向 |
|------|------|----------|
| `Executed Ipc Active` | 每 active cycle 执行的指令数 | 目标接近理论 IPC |
| `Executed Ipc Elapsed` | 每 elapsed cycle 执行的指令数 | 受 occupancy 影响 |
| `SM Busy` | SM 活跃程度 | 低则提高 occupancy |
| `Pipe Utilization` | 各管线利用率 | 找到最高利用率的管线 |

**管线利用率（Pipe Utilization）详解：**
```
ALU (FFMA/FADD/FMUL)：FP32 CUDA Core
FP64：双精度浮点
LSU (Load/Store)：内存指令管线
SFU：特殊函数（sin/cos/sqrt/rcp）
Tensor Core：HMMA/IMMA 指令
ADU：地址计算

高利用率的管线就是瓶颈。例：
- LSU 100% → memory-bound（需要 coalescing 或 caching）
- ALU 100% → compute-bound FP32（考虑 Tensor Core 或 FP16）
- Tensor Core 100% → GEMM 运算 perf-bound（几乎最优）
- SFU 100% → 特殊函数太多（用查表或近似函数替代）
```

**IPC 分析：**
```bash
# 实际 IPC vs 理论 IPC
ncu --metrics smsp__inst_executed.sum \
              smsp__cycles_active.sum \
    ./my_kernel

# Achieved IPC = inst_executed / cycles_active
# Theoretical max IPC = 1~4（取决于架构）
# sm_86: 每 SMSP 每 cycle 最多 issue 1 条指令
# 每 SM 有 4 个 SMSP → 每 SM 理论 IPC = 4
```

---

## 8. 实战调优流程

### 8.1 系统化步骤

#### Step 1: Profile - 收集基础 Profile

```bash
# 快速扫描：SpeedOfLight 给出全局视图
ncu --section SpeedOfLight \
    --section LaunchStats \
    --section Occupancy \
    -o baseline_profile \
    ./my_app --kernel-name "myKernel"

# 查看输出
ncu-ui baseline_profile.ncu-rep
# 或命令行
ncu --import baseline_profile.ncu-rep
```

**分析 SpeedOfLight 结论：**
- `Compute Throughput > Memory Throughput` → Compute-bound
- `Memory Throughput > Compute Throughput` → Memory-bound
- 两者都低（<20%）→ Latency-bound（warp stall 为主）

#### Step 2: Identify - 精准定位瓶颈

```bash
# 根据 Step 1 的结论，选择对应 section 深入

# 如果 Memory-bound：
ncu --section MemoryWorkloadAnalysis \
    --section WarpStateStats \
    ./my_app

# 如果 Compute-bound：
ncu --section ComputeWorkloadAnalysis \
    --section InstructionStats \
    ./my_app

# 如果 Latency-bound（warp stall 严重）：
ncu --section SchedulerStats \
    --section WarpStateStats \
    --section SourceCounters \
    ./my_app
```

#### Step 3: Hypothesize - 建立优化假设

根据指标建立假设，按优先级排序：

```
优先级1（最高收益）：
□ Global memory coalescing 差 → 重新排布数据（SoA）
□ Occupancy 极低（<25%） → 减少寄存器/shared memory
□ DRAM bandwidth 接近峰值 → 增加数据重用（shared memory）

优先级2：
□ Shared memory bank conflict 多 → 添加 padding
□ Warp divergence 高 → 重排数据或改用 predicated 代码
□ Long Scoreboard stall 占主导 → 使用 cp.async / 增加 occupancy

优先级3：
□ SFU 管线满载 → 用近似函数或查表
□ 波次量化损失大 → 调整 grid 大小
□ Register spill 存在 → 减少 unroll 或调整 launch_bounds
```

#### Step 4: Verify - 验证优化效果

```bash
# 1. 验证正确性
./my_app --verify  # 对比 CPU/baseline GPU 结果

# 2. 测量性能变化
# 运行 N 次取中位数（避免热身效应和异常值）
for i in {1..5}; do
    ncu --metrics gpu__time_duration.sum ./my_app 2>&1 | grep "gpu__time_duration"
done

# 3. 对比 profile（baseline vs optimized）
ncu -o optimized_profile ./my_app_optimized
# 在 ncu-ui 中 File → Baseline 对比两个 profile
```

#### Step 5: Iterate - 迭代优化

```
Profile → Identify → Hypothesize → Implement → Verify → Profile again
           ↑___________________________________________|

每轮迭代：
1. 只改变一个变量（避免多变量混淆）
2. 记录每次优化的收益和原因
3. 若无收益，回滚并尝试其他假设
4. 当收益 < 5% 时，考虑是否已达架构极限
```

---

### 8.2 常见陷阱

#### 陷阱1：过度优化 occupancy
```
❌ 错误思路："occupancy 越高越好，我要让它达到 100%"
✓ 正确理解：occupancy 是 latency hiding 的手段，不是目标
  → 如果 kernel 是 compute-bound，提高 occupancy 毫无帮助
  → 如果已有足够 warps（比如 50%+），继续提高收益递减
  → 强制高 occupancy（减少寄存器）可能引入 spill，反而更慢
```

#### 陷阱2：忽视 warp stall 的根本原因
```
❌ 错误行为：看到 Long Scoreboard stall 就直接提高 occupancy
✓ 正确流程：
  1. 先看 Source Counter，找到具体是哪行代码产生 stall
  2. 判断是 sequential pattern 还是 random access
  3. 如果是 random access → 改善 access pattern 或用 shared memory cache
  4. 如果是 sequential → cp.async + double buffering
  5. 最后才考虑提高 occupancy
```

#### 陷阱3：在 profiler 下测量性能
```
❌ 误区：在 ncu profiling 模式下测量的 kernel 时间代表真实性能
✓ 现实：
  - ncu 会序列化 kernel 执行（全都在单流中执行）
  - ncu 会 replay kernel 多次（额外 overhead）
  - ncu 可能使 cache 处于不同状态（cache flush 在 replay 间）
  - 正确做法：只用 ncu 分析 metrics，用 nsys 或 cudaEvent 测量真实耗时
```

#### 陷阱4：只看 achieved occupancy 忽视 instruction mix
```
❌ 场景：achieved occupancy = 75%，但 kernel 很慢
✓ 实际问题可能是：
  - 大量 SFU 指令（sin/cos）拖慢了整体 IPC
  - 指令 mix 不均衡（全都是 LDG，没有 FFMA 重叠）
  - 解决：看 ComputeWorkloadAnalysis 的 Pipe Utilization 热图
```

#### 陷阱5：忽视 wave tail 问题（grid 大小设置不当）
```
❌ 常见代码：
grid_size = (N + block_size - 1) / block_size  // 按数据量设定

✓ 更好的方式（考虑 wave quantization）：
int num_sms = ...; // cudaDeviceGetAttribute
int blocks_per_sm = ...; // 根据 occupancy 计算
int wave_size = num_sms * blocks_per_sm;
// 如果 grid_size 是 wave_size 的 1.x 倍，考虑调整 block_size
// 使得 grid_size 更接近 wave_size 的整数倍
```

#### 陷阱6：在 -O0 或 debug 模式下 profile
```
❌ 错误：用 -G（device debug）编译后 profile
  → -G 禁用所有 compiler optimization，register 使用大增，性能完全失真
✓ 正确：始终用 -O2/-O3 + -lineinfo（保留行号，不禁用优化）
  nvcc -O2 -lineinfo -arch=sm_86 kernel.cu
```

#### 陷阱7：混淆 SM busy 与 kernel efficient
```
❌ 误读：SM Active Cycles / SM Elapsed Cycles = 90%，认为"很高效"
✓ 实际含义：SM 上有 warp 活跃（但不代表这些 warp 在有效执行）
  → 还需要看 smsp__cycles_active vs smsp__issue_active
  → "Issue Active" = 调度器真正 issue 指令的 cycle 占比
  → 低 Issue Active + 高 SM Active → warp stall 严重
```

#### 陷阱8：bank conflict 检测漏测 64-bit 访问
```
// 对于 double 或 int64_t，bank conflict 计算方式不同
// double（8 bytes）跨越 2 个 bank（每 bank 4 bytes）
__shared__ double dsmem[32];  // 相邻 thread 访问相差 2 bank，stride-2 conflict！
// 解决：使用 int2 或 float2 + padding
__shared__ double dsmem[33];  // +1 double padding
```

---

### 8.3 常用 ncu 命令速查

```bash
# === 快速诊断 ===
# SpeedOfLight 快速总览
ncu --section SpeedOfLight --section Occupancy ./app

# 完整 profile（慢，但最全面）
ncu --set full ./app

# === 特定问题 ===
# 内存问题
ncu --section MemoryWorkloadAnalysis \
    --metrics l1tex__average_t_sectors_per_request_pipe_lsu_mem_global_op_ld.ratio \
    ./app

# Warp stall 问题
ncu --section WarpStateStats --section SourceCounters ./app

# Occupancy 问题
ncu --section Occupancy --section LaunchStats ./app

# Compute 效率问题
ncu --section ComputeWorkloadAnalysis --section InstructionStats ./app

# Roofline 分析
ncu --section SpeedOfLight_RooflineChart ./app

# === 过滤选项 ===
# 只 profile 特定 kernel
ncu --kernel-name "myKernel" ./app

# 只 profile 第 3 次 kernel 执行
ncu --kernel-id ::myKernel:3 ./app

# 跳过前 100 次 kernel，然后 profile 10 次
ncu --launch-skip 100 --launch-count 10 ./app

# === 输出格式 ===
# 保存到文件
ncu -o my_profile ./app

# CSV 输出（便于脚本处理）
ncu --csv --metrics sm__warps_active.avg.pct_of_peak_sustained_active ./app

# === 与 nsys 配合 ===
# 先用 nsys 找到热点 kernel，再用 ncu 深入分析
nsys profile -o timeline ./app
nsys stats timeline.nsys-rep  # 找到最耗时的 kernel

ncu --kernel-name "hotKernel" -o hotKernel_profile ./app
ncu-ui hotKernel_profile.ncu-rep
```

---

### 8.4 性能优化 Checklist

```
## Launch Configuration
□ Block size 是 32 的整数倍（避免 partial warp）
□ Block size 在 128~256 范围（经验最优区间）
□ Grid size 考虑 wave quantization
□ 使用 cudaOccupancyMaxPotentialBlockSize 验证 block size 选择

## Memory
□ Global memory access coalesced（连续 thread 访问连续地址）
□ 结构体使用 SoA 而非 AoS（除非元素总是一起访问）
□ Shared memory 有无 bank conflict（用 padding 解决）
□ 数据对齐到 16/32/128 bytes
□ 热数据放 shared memory，冷数据直接 global

## Register & Occupancy
□ 检查是否有 register spill（-Xptxas=-v 或 ncu LaunchStats）
□ __launch_bounds__ 与实际 block size 匹配
□ Occupancy 是否达到合理水平（memory-bound 时 ≥50%）
□ Unroll factor 是否合适（过大导致 spill）

## Warp Execution
□ 无显著 warp divergence（或已用 predication 替代）
□ Long Scoreboard stall < 20%（如果高，使用 cp.async / double buffer）
□ Barrier stall 可接受（减少不必要的 __syncthreads）

## Compute
□ Tensor Core 已充分利用（GEMM M/N/K 是 16 的倍数）
□ 已使用混合精度（FP16/BF16）当精度允许时
□ SFU 指令（sin/cos/sqrt）已替换为近似版本（__sinf/__cosf）

## Advanced
□ cp.async 用于 global→shared 数据加载
□ CUDA Graph 用于重复执行的 kernel 序列
□ Stream 并行已启用（H2D copy 与 compute 重叠）
□ L2 Persistence 已配置（频繁访问的小数据集）
```

---

## 附录：关键数字速查表

### Ampere sm_86（RTX 3090 / A5000）
| 参数 | 值 |
|------|-----|
| SMs | 82 |
| CUDA Cores/SM | 128 (FP32) |
| Tensor Cores/SM | 4 (3rd gen) |
| Max Warps/SM | 48 |
| Max Threads/SM | 1536 |
| Max Blocks/SM | 32 |
| Registers/SM | 65536 × 32-bit |
| Max Shared Memory/SM | 100 KB（可配置）|
| L1 Cache/SM | 128 KB（L1+smem 共享）|
| L2 Cache | 6 MB |
| Memory Clock | 9.75 Gbps GDDR6X |
| Peak BW | 936 GB/s |
| FP32 Peak | 35.6 TFLOPS |
| FP16 Peak（TC）| 142 TFLOPS |

### A100 SXM4 80GB
| 参数 | 值 |
|------|-----|
| SMs | 108 |
| Max Warps/SM | 64 |
| Max Threads/SM | 2048 |
| Registers/SM | 65536 × 32-bit |
| Max Shared Memory/SM | 164 KB (dynamic) / 100 KB (static) |
| L2 Cache | 40 MB |
| HBM2e BW | 2 TB/s |
| FP32 Peak | 19.5 TFLOPS |
| TF32 Peak（TC）| 312 TFLOPS |
| FP16 Peak（TC）| 312 TFLOPS |
| INT8 Peak（TC）| 624 TOPS |

### H100 SXM5 80GB
| 参数 | 值 |
|------|-----|
| SMs | 132 |
| Max Warps/SM | 64 |
| Max Threads/SM | 2048 |
| Max Shared Memory/SM | 228 KB |
| L2 Cache | 50 MB |
| HBM3 BW | 3.35 TB/s |
| FP32 Peak | 67 TFLOPS |
| FP16 Peak（TC）| 1000 TFLOPS（稀疏 2000） |
| FP8 Peak（TC）| 2000 TOPS（稀疏 4000）|
| TMA（新特性）| Tensor Memory Accelerator，专用异步 memcpy 引擎 |

### 常见内存延迟（参考值）
| 内存层次 | 延迟（cycles）| 带宽（per SM）|
|----------|--------------|--------------|
| Register | 0 | ~16 TB/s |
| Shared Memory | 20~32 | 4 TB/s |
| L1 Cache (hit) | 28~34 | 2 TB/s |
| L2 Cache (hit) | 200~300 | 200 GB/s |
| HBM (DRAM, L2 miss) | 400~800 | 2 TB/s (A100 全局) |

---

*文档生成时间：2026-04-27*  
*参考版本：NVIDIA Nsight Compute 13.2、CUDA C++ Best Practices Guide 13.2*  
*适用架构：Volta、Turing、Ampere、Hopper（具体数字以目标 GPU 为准）*
