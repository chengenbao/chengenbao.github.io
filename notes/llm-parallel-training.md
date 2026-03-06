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

![Megatron 序列并行 + 张量并行架构图](/assets/notes/parallel/megatron-sp-tp.png)
*图：序列并行（SP）与张量并行（TP）的协作区域划分，g/ḡ 为 All-Reduce/All-Gather 算子*



![张量并行 GEMM 切分示意：Column Parallel（左）与 Row Parallel（右）](/assets/notes/parallel/tp-gemm.png)
*图：Megatron-LM 张量并行中的列切分（ColumnParallel）与行切分（RowParallel），来源：HuggingFace Transformers Docs*

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

![流水线并行 bubble 示意：朴素 GPipe vs 1F1B](/assets/notes/parallel/pp-bubble.png)
*图：GPipe 朴素流水线（上）存在大量 bubble；1F1B 交替调度（下）大幅降低空闲比例，来源：HuggingFace Transformers Docs*


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


![ZeRO 三阶段显存切分示意](/assets/notes/parallel/zero-stages.png)
*图：ZeRO-1/2/3 分别对 Optimizer States、Gradients、Parameters 进行切分，来源：HuggingFace Transformers Docs*

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

## 附录：参考实现

### A. 数据并行（PyTorch DDP）

```python
# 标准 DDP 最简实现
import torch
import torch.distributed as dist
import torch.nn as nn
from torch.nn.parallel import DistributedDataParallel as DDP

def setup(rank, world_size):
    dist.init_process_group("nccl", rank=rank, world_size=world_size)
    torch.cuda.set_device(rank)

def cleanup():
    dist.destroy_process_group()

def train(rank, world_size):
    setup(rank, world_size)

    model = nn.Transformer(d_model=512, nhead=8).to(rank)
    # DDP 包装：自动在反向传播后 All-Reduce 梯度
    ddp_model = DDP(model, device_ids=[rank])

    optimizer = torch.optim.Adam(ddp_model.parameters(), lr=1e-4)

    # DistributedSampler 保证每卡看到不同数据
    from torch.utils.data.distributed import DistributedSampler
    sampler = DistributedSampler(dataset, num_replicas=world_size, rank=rank)
    loader = DataLoader(dataset, batch_size=32, sampler=sampler)

    for batch in loader:
        loss = ddp_model(batch)
        loss.backward()   # 自动触发 All-Reduce
        optimizer.step()
        optimizer.zero_grad()

    cleanup()

# 启动：torchrun --nproc_per_node=8 train.py
```

---

### B. 张量并行（列切分 Linear）

```python
import torch
import torch.distributed as dist
import torch.nn as nn
import torch.nn.functional as F

class ColumnParallelLinear(nn.Module):
    """
    将 Linear(in_features, out_features) 按列切分到 tp_world_size 张卡。
    每卡持有 out_features // tp_world_size 列。
    
    前向：本地 matmul（无通信）
    反向：All-Reduce 输入梯度（1次）
    """
    def __init__(self, in_features: int, out_features: int):
        super().__init__()
        self.tp_rank = dist.get_rank()
        self.tp_world = dist.get_world_size()
        
        assert out_features % self.tp_world == 0
        self.local_out = out_features // self.tp_world
        
        # 每卡只存一列分片
        self.weight = nn.Parameter(torch.empty(self.local_out, in_features))
        self.bias   = nn.Parameter(torch.zeros(self.local_out))
        nn.init.xavier_uniform_(self.weight)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x: [B, T, in_features]，本地计算
        return F.linear(x, self.weight, self.bias)
        # 输出: [B, T, local_out]，各卡持有不同列分片

class RowParallelLinear(nn.Module):
    """
    将 Linear(in_features, out_features) 按行切分。
    每卡持有 in_features // tp_world_size 行，
    前向后需 All-Reduce 聚合各卡的部分和。
    """
    def __init__(self, in_features: int, out_features: int):
        super().__init__()
        self.tp_rank = dist.get_rank()
        self.tp_world = dist.get_world_size()
        
        assert in_features % self.tp_world == 0
        self.local_in = in_features // self.tp_world
        
        self.weight = nn.Parameter(torch.empty(out_features, self.local_in))
        # bias 只在 rank 0 上加（避免重复累加）
        self.bias = nn.Parameter(torch.zeros(out_features)) if self.tp_rank == 0 else None
        nn.init.xavier_uniform_(self.weight)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x: [B, T, local_in]（上游 ColumnParallelLinear 的输出分片）
        out = F.linear(x, self.weight)  # [B, T, out_features]（部分和）
        dist.all_reduce(out, op=dist.ReduceOp.SUM)  # 聚合各卡部分和
        if self.bias is not None:
            out = out + self.bias
        return out

# 使用示例（Megatron 风格 MLP）
class TensorParallelMLP(nn.Module):
    def __init__(self, hidden: int, ffn: int):
        super().__init__()
        self.fc1 = ColumnParallelLinear(hidden, ffn)   # 按列切：无通信
        self.fc2 = RowParallelLinear(ffn, hidden)       # 按行切：前向后 All-Reduce

    def forward(self, x):
        return self.fc2(F.gelu(self.fc1(x)))
```

---

### C. 流水线并行（GPipe 风格 micro-batch）

```python
import torch
import torch.distributed as dist
from typing import List

class PipelineStage(torch.nn.Module):
    """单个流水线阶段，持有模型的若干层"""
    def __init__(self, layers: List[torch.nn.Module]):
        super().__init__()
        self.layers = torch.nn.ModuleList(layers)

    def forward(self, x):
        for layer in self.layers:
            x = layer(x)
        return x

def gpipe_forward(stages: List[PipelineStage], micro_batches: List[torch.Tensor],
                  rank: int, world_size: int):
    """
    GPipe 前向：先让所有 micro-batch 流过所有阶段（存激活），再统一反向。
    bubble ratio = (p-1) / (m + p - 1)，其中 p=阶段数，m=micro-batch数
    """
    activations = []
    
    for mb in micro_batches:
        if rank == 0:
            x = stages[rank](mb)
        else:
            # 从上游接收激活
            shape = torch.zeros(1, dtype=torch.long)
            dist.recv(shape, src=rank - 1)
            x = torch.zeros(*shape.tolist())
            dist.recv(x, src=rank - 1)
            x = stages[rank](x)
        
        if rank < world_size - 1:
            # 发送激活给下游（先发 shape）
            shape = torch.tensor(list(x.shape), dtype=torch.long)
            dist.send(shape, dst=rank + 1)
            dist.send(x.detach(), dst=rank + 1)
        
        activations.append(x)
    
    return activations

# 实践推荐：直接用 torch.distributed.pipeline.sync.Pipe
from torch.distributed.pipeline.sync import Pipe

model = torch.nn.Sequential(
    torch.nn.Linear(512, 1024),  # stage 0
    torch.nn.ReLU(),
    torch.nn.Linear(1024, 512),  # stage 1（自动分配到下一个 rank）
)
# 按 chunks 切分 micro-batch，balance 指定每阶段层数
pipe_model = Pipe(model, chunks=8)  # 8 个 micro-batch，减少 bubble
```

---

### D. ZeRO-3 最简演示（手动 All-Gather / Reduce-Scatter）

```python
import torch
import torch.distributed as dist

def zero3_forward_hook(module, input):
    """ZeRO-3 前向 hook：在使用前 All-Gather 完整参数"""
    for param in module.parameters(recurse=False):
        if hasattr(param, '_z3_shard'):
            # 从所有 rank 聚合完整参数
            full_param = torch.zeros(
                param._z3_full_shape, device=param.device, dtype=param.dtype
            )
            dist.all_gather_into_tensor(full_param, param.data)
            param._full_data = full_param  # 临时存整个参数，计算后丢弃

def zero3_backward_hook(module, grad_input, grad_output):
    """ZeRO-3 反向 hook：计算后 Reduce-Scatter 梯度，丢弃完整参数"""
    for param in module.parameters(recurse=False):
        if hasattr(param, '_z3_shard') and param.grad is not None:
            # Reduce-Scatter：每个 rank 只保留自己负责的梯度分片
            grad_shard = torch.zeros_like(param.data)
            dist.reduce_scatter_tensor(grad_shard, param.grad)
            param.grad = grad_shard
            # 丢弃临时聚合的完整参数，释放显存
            if hasattr(param, '_full_data'):
                del param._full_data

# 实践：直接用 DeepSpeed ZeRO-3 / PyTorch FSDP
# FSDP 是 PyTorch 官方的 ZeRO-3 等价实现，更易用

from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
from torch.distributed.fsdp import MixedPrecision
import torch

model = MyLargeModel()
fsdp_model = FSDP(
    model,
    mixed_precision=MixedPrecision(
        param_dtype=torch.bfloat16,
        reduce_dtype=torch.float32,
    ),
    # auto_wrap_policy 按 Transformer 层粒度切分
    auto_wrap_policy=transformer_auto_wrap_policy,
)
```

---

### E. DeepSpeed 完整训练脚本骨架

```python
import deepspeed
import torch
import torch.nn as nn

class MyModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.layers = nn.ModuleList([
            nn.TransformerEncoderLayer(d_model=4096, nhead=32, dim_feedforward=16384)
            for _ in range(32)
        ])

    def forward(self, x):
        for layer in self.layers:
            x = layer(x)
        return x

# DeepSpeed 配置（ZeRO-2 + 混合精度）
ds_config = {
    "train_batch_size": 256,
    "train_micro_batch_size_per_gpu": 4,
    "gradient_accumulation_steps": 8,
    "fp16": {"enabled": True, "loss_scale": 0, "initial_scale_power": 16},
    "zero_optimization": {
        "stage": 2,                        # ZeRO-2：切 grad + optimizer states
        "allgather_partitions": True,
        "reduce_scatter": True,
        "overlap_comm": True,              # 通信与计算 overlap
        "contiguous_gradients": True,
    },
    "optimizer": {
        "type": "AdamW",
        "params": {"lr": 1e-4, "betas": [0.9, 0.95], "weight_decay": 0.1}
    },
    "scheduler": {
        "type": "WarmupDecayLR",
        "params": {"warmup_min_lr": 0, "warmup_max_lr": 1e-4, "warmup_num_steps": 2000}
    },
}

model = MyModel()
model_engine, optimizer, _, scheduler = deepspeed.initialize(
    model=model,
    config=ds_config,
)

for batch in dataloader:
    inputs, labels = batch
    outputs = model_engine(inputs)
    loss = criterion(outputs, labels)
    
    model_engine.backward(loss)   # DeepSpeed 接管反向传播
    model_engine.step()           # 更新参数 + 处理 ZeRO 通信

# 保存检查点（ZeRO-3 需合并分片）
model_engine.save_checkpoint("./checkpoints")

# ZeRO-3 Zero-to-FP32 转换（推理用）
# deepspeed --num_gpus 1 zero_to_fp32.py ./checkpoints ./model_fp32.pt
```

---

### F. 组合配置：Megatron-DeepSpeed 3D 并行

```bash
# 启动命令示例：128卡，TP=8，PP=4，DP=4，ZeRO-1
torchrun     --nproc_per_node 8     --nnodes 16     --node_rank ${NODE_RANK}     --master_addr ${MASTER_ADDR}     pretrain_gpt.py     --tensor-model-parallel-size 8     --pipeline-model-parallel-size 4     --num-layers 96     --hidden-size 12288     --num-attention-heads 96     --seq-length 2048     --global-batch-size 1536     --micro-batch-size 3     --train-iters 500000     --lr 1e-4     --min-lr 1e-5     --lr-warmup-iters 2000     --fp16     --use-flash-attn     --recompute-activations  # 激活重计算，以时间换显存
```

```python
# Megatron 核心并行组初始化逻辑（简化版）
def initialize_model_parallel(
    tensor_model_parallel_size: int = 1,
    pipeline_model_parallel_size: int = 1,
):
    world_size = torch.distributed.get_world_size()
    data_parallel_size = world_size // (tensor_model_parallel_size * pipeline_model_parallel_size)

    # TP 组：同机 NVLink 内，连续 rank
    # 例：world=8, TP=4 → 组 [0,1,2,3], [4,5,6,7]
    for i in range(pipeline_model_parallel_size * data_parallel_size):
        ranks = range(i * tensor_model_parallel_size, (i+1) * tensor_model_parallel_size)
        group = torch.distributed.new_group(ranks)

    # PP 组：步长 TP 的等差序列
    # 例：world=8, TP=2, PP=2 → 组 [0,2], [1,3], [4,6], [5,7]
    for i in range(tensor_model_parallel_size * data_parallel_size):
        ranks = range(i, world_size, tensor_model_parallel_size)
        group = torch.distributed.new_group(ranks)
```

---

## 参考资料

- [Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism](https://arxiv.org/abs/1909.08053)
- [ZeRO: Memory Optimizations Toward Training Trillion Parameter Models](https://arxiv.org/abs/1910.02054)
- [DeepSpeed ZeRO++: A collection of memory space and communication efficiency optimizations](https://arxiv.org/abs/2306.10209)
- [Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM](https://arxiv.org/abs/2104.04473)
- [Ring Attention with Blockwise Transformers for Near-Infinite Context](https://arxiv.org/abs/2310.01889)
