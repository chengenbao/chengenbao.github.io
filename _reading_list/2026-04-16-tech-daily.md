---
layout: reading
title: "📰 2026-04-16 每日技术速递"
date: 2026-04-16
category: tech
tags: [LLM, inference, quantization, pruning, edge-ai]
---

## 📰 2026-04-16 每日技术速递

> 每日精选 AI 系统 / 大模型 / 推理优化前沿论文，来源：arXiv cs.LG / cs.CL

---

### 1. KV Packet: Recomputation-Free Context-Independent KV Caching for LLMs

**来源**：arXiv cs.LG | **链接**：[https://arxiv.org/abs/2604.13226](https://arxiv.org/abs/2604.13226)

提出 KV Packet 方案，通过构建上下文无关的 KV 缓存块，彻底消除 LLM 推理中的 KV 重计算开销。核心思路是将 Prompt 切分为可独立缓存的数据包，每个 Packet 的 KV 值仅依赖自身内容而非全局上下文，从而实现跨请求的高效复用，显著降低首包时延（TTFT）和推理计算量。

---

### 2. MOONSHOT: A Framework for Multi-Objective Pruning of Vision and Large Language Models

**来源**：arXiv cs.LG | **链接**：[https://arxiv.org/abs/2604.13287](https://arxiv.org/abs/2604.13287)

MOONSHOT 是一个统一的多目标剪枝框架，同时优化模型精度、延迟和内存占用三个维度。通过 Pareto 前沿搜索自动找到剪枝比例与性能的最优权衡，支持视觉模型（ViT 系列）和大语言模型，在 ImageNet 及多个 NLP Benchmark 上取得 SOTA 的压缩-精度曲线。

---

### 3. Lossless Prompt Compression via Dictionary-Encoding and In-Context Learning

**来源**：arXiv cs.CL | **链接**：[https://arxiv.org/abs/2604.13066](https://arxiv.org/abs/2604.13066)

提出基于字典编码的无损 Prompt 压缩方法，利用 LLM 自身的 In-Context Learning 能力在推理侧实时解压。与有损压缩方法不同，该方案不改变语义，适用于需要精确复现原始指令的生产场景，压缩率可达 3-5×，兼容主流闭源 API。

---

### 4. Correct Chains, Wrong Answers: Dissociating Reasoning from Output in LLM Logic

**来源**：arXiv cs.CL | **链接**：[https://arxiv.org/abs/2604.13065](https://arxiv.org/abs/2604.13065)

系统研究大模型 Chain-of-Thought 推理链与最终答案的解耦现象：即便中间推理步骤完全正确，模型仍可能输出错误答案，反之亦然。论文深入分析了这一"推理-输出解耦"的机制，对 RLHF 奖励设计及推理验证器的研发有重要参考意义。

---

### 5. Unleashing Implicit Rewards: Prefix-Value Learning for Distribution-Level Optimization

**来源**：arXiv cs.CL | **链接**：[https://arxiv.org/abs/2604.13197](https://arxiv.org/abs/2604.13197)

提出 Prefix-Value Learning 方法，通过从语言模型中提取隐式奖励信号来进行分布级优化，无需显式奖励模型。该方法将奖励信号嵌入前缀序列的值函数中，在对齐任务上超越传统 PPO/DPO 方案，并有效缓解奖励模型偏差问题。

---

### 6. BioTrain: Sub-MB, Sub-50mW On-Device Fine-Tuning for Edge-AI on Biosignals

**来源**：arXiv cs.LG | **链接**：[https://arxiv.org/abs/2604.13359](https://arxiv.org/abs/2604.13359)

面向边缘 AI 场景的超轻量级在设备微调框架，模型更新内存占用不超过 1MB，功耗低于 50mW。针对生物信号（ECG/EEG）的持续学习需求，通过梯度稀疏化和定点训练实现极低资源消耗，为可穿戴设备上的个性化模型更新提供可行路径。

---

*生成时间：2026-04-16 | 数据来源：arXiv RSS*
