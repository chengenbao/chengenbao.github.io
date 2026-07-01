---
layout: reading
title: "LLM 推理系统 · 存内计算 · FPGA 量化 · RL Post-Training"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-07-01
---

# 📰 2026-07-01 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖 LLM 异构推理、CUDA 内核仿真、PIM 编译、铁电存算一体、FPGA 量化剪枝、大规模 RL Post-Training。

---

## 1. HBM 不是唯一答案：跨异构内存加速器的高效 LLM 分离式推理

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2606.29986  
**标签**：LLM推理 · 分离式部署 · 内存异构 · KV缓存 · 吞吐优化

LLM 推理包含计算密集的 prefill 和内存密集的 decode 两个阶段，现有分离式系统仍假设所有加速器均配备 HBM，造成大量内存带宽浪费。本文提出一套内存异构的分离式推理系统，将 HBM 和 DRAM 加速器协同调度，在保持吞吐的同时将推理成本降低高达 40%。核心创新在于细粒度的 KV cache 迁移策略与动态任务路由机制。

**核心要点**：
- 针对 prefill/decode 不同计算特性设计异构资源调度策略
- KV cache 在 HBM/DRAM 加速器间动态迁移，最大化内存带宽利用率
- 在相同吞吐目标下推理成本降低 40%，对大规模生产部署有直接价值

---

## 2. KernelSight-LM：内核级 LLM 推理性能仿真器

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2606.28565  
**标签**：LLM推理 · 性能建模 · CUDA内核 · 仿真器 · 硬件感知

生产环境中评估 LLM 推理性能需要在不同硬件、模型架构和批处理策略间进行大量实验，成本极高。KernelSight-LM 是一个内核粒度的仿真器，对 Transformer 推理中每个 CUDA 内核进行建模，实现跨硬件/模型/批次配置的精确性能预测，无需实际部署即可快速评估优化方案。

**核心要点**：
- 以 CUDA 内核为最小建模单元，精确捕捉 attention、FFN、softmax 等算子的性能特征
- 支持多种硬件配置（A100、H100、L40S 等）与量化策略的快速横向对比
- 可作为推理优化工具的性能预测后端，大幅减少实验迭代周期

---

## 3. DCC：面向存内计算架构的以数据为中心的 ML 内核编译器

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2511.15503  
**标签**：PIM · 编译器 · 存内计算 · 内核映射 · 嵌入查找

Processing-In-Memory（PIM）可显著加速内存密集型 ML 内核，但手动映射复杂且不可移植。DCC 是一套以数据为中心的编译框架，通过分析数据移动模式自动将 ML 内核映射到 PIM 设备，对嵌入查找、KV cache 操作等内存瓶颈任务实现显著加速。

**核心要点**：
- 以数据流分析驱动编译决策，自动识别适合 PIM 卸载的内核
- 支持主机 CPU 与 PIM 设备的协同调度，避免不必要的数据搬运开销
- 在 embedding lookup 和内存密集型 Transformer 算子上超越 GPU 基线

---

## 4. FCDC：基于 HZO 铁电电容的非易失性电荷域 Attention 计算

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2605.28208  
**标签**：存算一体 · KV缓存 · 铁电器件 · Transformer · 近存计算

随着上下文长度增长，KV cache 的持续读写成为 Transformer 解码的主要瓶颈。FCDC 利用 HZO 铁电电容实现非易失性电荷域存算一体设计，将 attention 计算直接嵌入存储阵列，大幅降低 KV cache 带宽压力和能耗，为超长上下文推理提供新的硬件路径。

**核心要点**：
- HZO 铁电电容兼具非易失性存储与模拟计算能力，天然适合 attention score 计算
- 电荷域计算避免 ADC/DAC 转换开销，能效优于传统 SRAM/HBM 方案
- 为长序列推理场景提供可行的近存计算硬件架构

---

## 5. RQP：面向 FPGA 神经网络的资源导向量化器剪枝

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2606.30382  
**标签**：量化 · 剪枝 · FPGA · 边缘推理 · 资源效率

HGQ（高粒度量化）通过权重级量化和剪枝设计资源高效的 FPGA 加速器，但比特宽度搜索空间庞大。RQP 引入资源导向的量化器剪枝，在 FPGA 资源约束下联合优化比特宽度选择和权重剪枝，在边缘硬件上实现最先进的精度-效率权衡。

**核心要点**：
- 将 FPGA DSP/LUT 资源使用量显式纳入量化搜索目标，避免传统方法的资源浪费
- 权重剪枝与量化比特宽度联合优化，两者互补而非独立
- 在 MNIST、CIFAR 等视觉任务上以极低资源占用达到竞争性精度

---

## 6. Miles：面向大规模 LLM RL Post-Training 的 PyTorch 原生框架

**来源**：PyTorch Blog  
**链接**：https://pytorch.org/blog/miles-a-pytorch-native-stack-for-large-scale-llm-rl-post-training/  
**标签**：RLHF · Post-Training · 分布式训练 · SGLang · Megatron-LM

Miles 是 RadixArk 开源的大规模 LLM RL Post-Training 框架，将 SGLang（rollout）、NVIDIA Megatron-LM（训练）、Ray（编排）与 PyTorch 原生可扩展性融合为统一 API。它面向 RLHF/RLAIF 大规模工作流，支持跨数千 GPU 的 rollout 与 training 阶段高效协同调度。

**核心要点**：
- 模块化设计：rollout、reward、training 三阶段各自可插拔替换
- 基于 Ray 的分布式编排，支持 rollout/training 异步流水线，消除 GPU 空闲等待
- 与 PyTorch 生态深度集成，降低 RLHF 基础设施搭建门槛

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | HBM 不是唯一答案：异构内存 LLM 分离推理 | cs.AR | LLM 推理系统 |
| 2 | KernelSight-LM：内核级 LLM 推理仿真器 | cs.AR | 性能建模 |
| 3 | DCC：以数据为中心的 PIM 编译器 | cs.AR | 存内计算编译 |
| 4 | FCDC：铁电电容非易失存算一体 Attention | cs.AR | 近存计算硬件 |
| 5 | RQP：资源导向 FPGA 量化器剪枝 | cs.AR | 量化与剪枝 |
| 6 | Miles：PyTorch 原生大规模 RL Post-Training | PyTorch | 分布式训练 |

---

*自动生成 · 2026-07-01 · jeffinchen daily tech reading list*
