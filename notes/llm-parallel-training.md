---
layout: note
title: "大模型训练并行技术全景：DP / TP / PP / SP / ZeRO"
date: 2026-03-06
tags: [分布式训练, 数据并行, 张量并行, 流水线并行, ZeRO, DeepSpeed, Megatron]
---

# 大模型训练并行技术全景：DP / TP / PP / SP / ZeRO

> **摘要**：训练千亿参数大模型需要跨数百乃至数千张 GPU 协同工作。本文系统梳理五类核心并行策略——数据并行（DP）、张量并行（TP）、流水线并行（PP）、序列并行（SP）和 ZeRO 显存优化，分析各自的切分维度、通信开销与适用场景，并介绍主流框架中的工程实践。

---

## 1. 为什么需要并行训练？

单张 A100 80GB 显存能放下的模型有限：
- **GPT-3 175B**：fp16 权重 ~350GB，单卡远不够
- **梯度 + 优化器状态**：Adam 需额外存 fp32 权重副本 + m/v 动量，约 **16 bytes/param**

以 175B 模型为例：

| 存储项 | 占用 |
|--------|------|
| fp16 权重 | ~350 GB |
| fp32 权重副本（Adam）| ~700 GB |
| 梯度（fp32）| ~700 GB |
| Adam m/v 动量 | ~1400 GB |
| **合计** | **~3150 GB** |

因此需要在数百卡上切分存储与计算。

---

## 2. 数据并行（Data Parallelism, DP）

### 核心思想

每张 GPU 保存**完整模型副本**，不同 GPU 处理不同的数据 batch，训练结束后同步梯度。

```
GPU 0: [完整模型] → batch A → grad A
GPU 1: [完整模型] → batch B → grad B
GPU 2: [完整模型] → batch C → grad C
         ↓ All-Reduce (求和/平均梯度)
         所有 GPU 更新为相同参数
```

### 通信

- **All-Reduce**（Ring-All-Reduce / NCCL）：梯度聚合，通信量 $2(N-1)/N \approx 2$ 倍梯度大小
- PyTorch DDP 使用 **bucket 梯度压缩 + 通信计算 overlap**

### 适用场景

- 模型能放入单卡：直接用 DDP
- 核心瓶颈是**样本吞吐量**，而非显存

### 局限

- 每卡需要存完整模型，**显存不节省**
- 模型参数过大时无法使用

---

## 3. 张量并行（Tensor Parallelism, TP）

### 核心思想

将模型的**权重矩阵**在行或列维度切分到多张 GPU 上，每张 GPU 只计算整个矩阵乘法的一部分。

以 Transformer 的 MLP 层为例（Megatron-LM 方案）：

```
Linear1: [H, 4H] → 按列切成 N 份，每卡存 [H, 4H/N]
Linear2: [4H, H] → 按行切成 N 份，每卡存 [4H/N, H]

前向:  X → [X·W1_i] → GELU → [·W2_i] → All-Reduce → Y
反向:  All-Reduce 发生在 Linear1 输入端
```

每卡只需存 $\frac{1}{N}$ 的参数，但每层需要**1次 All-Reduce**（前向 + 反向共2次）。

### 注意力层切分

Multi-Head Attention 按 head 切分：每卡负责 $H/N$ 个 head，天然并行，无需通信。

### 通信量

每层前向/反向各 1 次 All-Reduce，总通信量与层数成正比。适合 **NVLink 互联的同机多卡**（带宽高），跨节点时通信成为瓶颈。

### 适用场景

- 单层参数量大（如 FFN hidden dim 很大）
- GPU 间 NVLink 带宽充足（同机 8 卡内）

---

## 4. 流水线并行（Pipeline Parallelism, PP）

### 核心思想

将模型**按层**切分到不同 GPU，形成流水线：

```
GPU 0: Layer 1-8   →  micro-batch 1 → micro-batch 2 → ...
GPU 1: Layer 9-16  →            → micro-batch 1 → micro-batch 2 → ...
GPU 2: Layer 17-24 →                        → micro-batch 1 → ...
GPU 3: Layer 25-32 →                                    → micro-batch 1 → ...
```

每张 GPU 只存 $1/N$ 的层，但需要等待上游 GPU 的激活值输出（**点对点通信**）。

### 流水线 bubble 问题

朴素实现中，GPU 在等待时处于空闲状态（bubble），效率为：

$$\text{效率} = 1 - \frac{p-1}{m}$$

其中 $p$ 是流水线阶段数，$m$ 是 micro-batch 数。增大 $m$ 可以减少 bubble，但会增加激活内存。

### 1F1B 调度（Megatron）

交替前向/反向（1 Forward, 1 Backward）来减少内存使用，是 Megatron-LM 的核心优化。

### 适用场景

- 模型层数非常多
- **跨节点**（InfiniBand）：流水线并行通信量小（只传激活值），适合带宽受限场景

---

## 5. 序列并行（Sequence Parallelism, SP）

### 核心思想

将**序列长度维度**切分到不同 GPU，以支持超长上下文训练（如 128K+ tokens）。

Megatron-LM 中的序列并行与张量并行配合使用：TP 区域内各 GPU 持有完整序列但部分参数，SP 区域内各 GPU 持有部分序列但完整激活。

### Ring Attention

DeepSpeed Ulysses 和 Ring Attention 是两种主流实现：

- **Ring Attention**：每个 GPU 持有 $1/N$ 的 Q，循环传递 K/V，实现 $O(N)$ 的序列并行 Attention
- **Ulysses**：对 QKV 做 All-to-All 后执行本地 Attention，再 All-to-All 还原

### 适用场景

- 超长序列（32K+）训练
- 序列长度是显存瓶颈时

---

## 6. ZeRO 优化器（Zero Redundancy Optimizer）

ZeRO 是 DeepSpeed 提出的显存优化方案，核心思想是**消除数据并行中的冗余存储**。

### ZeRO 三个阶段

| 阶段 | 切分内容 | 显存节省 | 通信开销 |
|------|---------|---------|---------|
| ZeRO-1 | Optimizer States | ~4x | 与 DDP 相当 |
| ZeRO-2 | + Gradients | ~8x | 与 DDP 相当 |
| ZeRO-3 | + Parameters | ~Nx（N=GPU数）| 约 1.5x DDP |

### ZeRO-1/2 通信等价性

ZeRO-2 用 **Reduce-Scatter + All-Gather** 替代 DDP 的 All-Reduce，通信量相同但显存更少。

### ZeRO-3 前向通信

前向传播时，每层参数需要从各 GPU **All-Gather** 完整后才能计算，计算完毕立即丢弃，反向时再 All-Gather 一次。

### ZeRO-Infinity（CPU/NVMe Offload）

将优化器状态、梯度甚至参数 offload 到 CPU 内存或 NVMe，理论上可在有限 GPU 上训练任意大小模型，但受 PCIe 带宽限制。

```python
# DeepSpeed ZeRO-3 配置示例
ds_config = {
    "zero_optimization": {
        "stage": 3,
        "offload_optimizer": {"device": "cpu"},
        "offload_param": {"device": "cpu"},
        "overlap_comm": True,
        "contiguous_gradients": True,
    }
}
```

---

## 7. 3D 并行：组合使用

大规模训练（如 GPT-3、Megatron-Turing NLG 530B）通常同时使用多种并行策略：

```
                     ┌─────────────────────────────────┐
                     │         数据并行（DP）              │
                     │   8 个 DP replica（跨节点）         │
                     │                                  │
                     │  ┌──────────────────────────┐   │
                     │  │    流水线并行（PP）          │   │
                     │  │   4 个流水线阶段（跨节点）   │   │
                     │  │                            │   │
                     │  │  ┌────────────────────┐   │   │
                     │  │  │  张量并行（TP）       │   │   │
                     │  │  │  8 卡（同机NVLink）   │   │   │
                     │  │  └────────────────────┘   │   │
                     │  └──────────────────────────┘   │
                     └─────────────────────────────────┘
```

典型配置原则：
- **TP** 在同机 NVLink 内（通信最快）
- **PP** 跨节点（通信量小，适合 IB）
- **DP** 在 PP/TP 都满后，剩余 GPU 做数据并行

---

## 8. 主流框架对比

| 框架 | DP | TP | PP | SP | ZeRO |
|------|----|----|----|----|------|
| PyTorch DDP | ✅ | ❌ | ❌ | ❌ | ❌ |
| DeepSpeed | ✅ | ✅ | ✅ | ❌ | ✅ |
| Megatron-LM | ✅ | ✅ | ✅ | ✅ | ❌ |
| Megatron-DeepSpeed | ✅ | ✅ | ✅ | ✅ | ✅ |
| FSDP（PyTorch 2.0）| ✅ | ❌ | ❌ | ❌ | ZeRO-3 等价 |

---

## 9. 选型决策树

```
模型能放入单卡？
├── 是 → 直接 DDP（数据并行）
└── 否
    ├── 层数多，跨节点 → 流水线并行（PP）
    ├── 单层参数大，同机 NVLink → 张量并行（TP）
    ├── 序列超长（32K+）→ 序列并行（SP）
    └── 想最大化显存效率 → ZeRO-3（+ CPU Offload）
```

实际工程中，**Megatron-DeepSpeed 3D 并行**（TP+PP+DP+ZeRO-1）是训练 100B+ 模型的主流方案。

---

## 参考资料

- [Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism](https://arxiv.org/abs/1909.08053)
- [ZeRO: Memory Optimizations Toward Training Trillion Parameter Models](https://arxiv.org/abs/1910.02054)
- [DeepSpeed ZeRO++: A collection of memory space and communication efficiency optimizations](https://arxiv.org/abs/2306.10209)
- [Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM](https://arxiv.org/abs/2104.04473)
- [Ring Attention with Blockwise Transformers for Near-Infinite Context](https://arxiv.org/abs/2310.01889)
