---
layout: reading
title: "技术速递 2026-03-13：MoE推理加速·SAM优化器·序列并行·RL训练框架"
category: tech
tags: [Tech, ArXiv, HuggingFace, MoE, RL, 分布式训练]
date: 2026-03-13
---

今日精选 6 篇前沿技术文章，聚焦 MoE 推理优化、优化器改进、LLM 可解释性与大规模分布式训练。

## 🔬 ArXiv 前沿论文

### MoE-SpAc: Efficient MoE Inference Based on Speculative Activation Utility
> [arxiv.org/abs/2603.09983](https://arxiv.org/abs/2603.09983)

提出推测激活效用（Speculative Activation Utility）框架，实现异构边缘场景下 MoE 模型高效推理。在保持精度的同时显著降低推理延迟，对边缘部署具有重要意义。

**标签：** #MoE #推理加速 #边缘计算

---

### Revisiting Sharpness-Aware Minimization: A More Faithful and Effective Implementation
> [arxiv.org/abs/2603.10048](https://arxiv.org/abs/2603.10048)

重新审视 SAM 优化器的理论与实现差距，提出更忠实高效的实现方案，直接提升大模型训练泛化能力。

**标签：** #优化器 #SAM #训练算法

---

### Causally Grounded Mechanistic Interpretability for LLMs
> [arxiv.org/abs/2603.09988](https://arxiv.org/abs/2603.09988)

结合因果推断与神经网络内部机制分析，为理解大模型决策过程提供新视角，生成忠实自然语言解释。

**标签：** #可解释性 #因果推断 #LLM

---

### The Dunning-Kruger Effect in LLMs: Confidence Calibration
> [arxiv.org/abs/2603.09985](https://arxiv.org/abs/2603.09985)

揭示 LLM 系统性置信度偏差：低能力任务过度自信，高能力任务低估自身。对 RLHF 与对齐研究有重要参考价值。

**标签：** #置信度校准 #LLM评估 #对齐

---

## 🤗 HuggingFace Blog

### Ulysses Sequence Parallelism: Training with Million-Token Contexts
> [huggingface.co/blog/ulysses-sp](https://huggingface.co/blog/ulysses-sp)

详解序列并行技术，通过序列切片 + All-to-All 通信优化注意力计算，突破百万 Token 上下文训练的内存瓶颈。

**标签：** #序列并行 #分布式训练 #长上下文

---

### Keep the Tokens Flowing: Lessons from 16 Open-Source RL Libraries
> [huggingface.co/blog/async-rl-training-landscape](https://huggingface.co/blog/async-rl-training-landscape)

对比 TRL、OpenRLHF、veRL 等 16 个 RL 训练框架，总结异步 RL 训练关键设计模式：Token 吞吐量、Actor-Critic 解耦、KV Cache 复用。

**标签：** #RL训练 #RLHF #框架对比 #TRL
