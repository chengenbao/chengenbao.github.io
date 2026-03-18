---
layout: reading
title: "技术速递 2026-03-18：Transformer 内部机制 × MXFP8 训练加速 × 多智能体协调"
category: tech
tags: [Tech, arXiv, PyTorch, LLM, 多智能体, 训练加速, 注意力机制]
date: 2026-03-18
---

本期聚焦 Transformer 内部工作机制的最新研究、无训练多智能体协调框架，以及 PyTorch 在 MoE 训练加速和 FlashAttention-4 上的重要进展。

## 精选文章

### 1. [How Transformers Reject Wrong Answers: Rotational Dynamics of Factual Constraint Processing](https://arxiv.org/abs/2603.13259)
**来源**：arXiv cs.CL

当语言模型接收到错误答案时，内部网络发生了什么？研究揭示了注意力头中的旋转动态机制，这些机制能够主动抑制与事实冲突的输入，将真实性理解从静态表征提升为动态过程。

---

### 2. [Training-Free Agentic AI: Probabilistic Control and Coordination in Multi-Agent LLM Systems](https://arxiv.org/abs/2603.13256)
**来源**：arXiv cs.CL

多智能体 LLM 系统的概率控制框架 REDEREF：无需微调即可实现智能路由和降噪协调，显著降低多智能体系统的部署成本和交互复杂性。

---

### 3. [Steering at the Source: Style Modulation Heads for Robust Persona Control](https://arxiv.org/abs/2603.13249)
**来源**：arXiv cs.CL

通过激活引导控制 LLM 人格风格，无需微调。定位到特定的风格调制注意力头，实现稳健的人格控制同时保持输出连贯性。

---

### 4. [Continual Fine-Tuning with Provably Accurate and Parameter-Free Task Retrieval](https://arxiv.org/abs/2603.13235)
**来源**：arXiv cs.LG

解决持续微调中的灾难性遗忘问题。提出无参数任务检索机制，具备理论准确性保证，支持顺序任务适配而无需专用检索参数。

---

### 5. [Your Code Agent Can Grow Alongside You with Structured Memory](https://arxiv.org/abs/2603.13258)
**来源**：arXiv cs.LG

为代码智能体引入结构化记忆，捕获软件项目的时序演化历史，让代码 Agent 能够理解意图导向编程历史并随代码库共同演进。

---

### 6. [MXFP8 Training for MoEs: 1.3x Training Speedup vs BF16 for Llama4 Scout on GB200](https://pytorch.org/blog/mxfp8-training-moes/)
**来源**：PyTorch Blog

PyTorch 发布 MXFP8 混合精度 MoE 训练加速报告：Llama4 Scout 在 NVIDIA GB200 集群上实现相比 BF16 **1.3x 训练提速**，详述量化策略与算子优化细节。

---

### 7. [FlexAttention + FlashAttention-4: Fast and Flexible](https://pytorch.org/blog/flexattention-flashattention-4/)
**来源**：PyTorch Blog

将 FlexAttention 可编程掩码模式与 FlashAttention-4 IO 最优 CUDA 核融合，在 H100/H200/GB200 GPU 上实现接近硬件峰值 FLOP/s 的自定义注意力性能。

---

*由 OpenClaw 自动整理 · 2026-03-18*
