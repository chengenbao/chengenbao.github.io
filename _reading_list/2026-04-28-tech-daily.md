---
layout: reading
title: "技术速递 2026-04-28：LLM 推理加速 · 分布式训练 · 编译器优化前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-04-28
---

### [Guess-Verify-Refine: Blackwell GPU 上基于时序相关性的稀疏注意力 Top-K 解码加速](https://arxiv.org/abs/2604.22312)

> **来源：** cs.AR / arXiv

提出 Guess-Verify-Refine（GVR）框架，利用 LLM 解码过程中注意力的时序相关性进行推测性 Top-K 筛选，在 NVIDIA Blackwell GPU 上实现稀疏注意力解码的显著加速，适用于长上下文场景。


### [Kernel Contracts：跨异构硬件 ML 算子正确性规范语言](https://arxiv.org/abs/2604.22032)

> **来源：** cs.LG / arXiv

提出 Kernel Contracts——一种 ML 算子正确性规范语言，将隐式的算子契约显式化，实现跨 GPU、TPU 等异构硅的算子验证与一致性检测，有助于发现 matmul 等核心算子的静默错误。


### [LayerBoost：面向 LLM 高效推理的层级感知注意力压缩](https://arxiv.org/abs/2604.22050)

> **来源：** cs.LG / arXiv

针对 Transformer 二次复杂度注意力瓶颈，提出 LayerBoost——基于层级重要性的动态注意力缩减方案，在保持精度的同时大幅降低推理计算量，尤其适合超长序列场景。


### [LLM 自我纠错机制：内部置信信号如何驱动错误检测与修正](https://arxiv.org/abs/2604.22271)

> **来源：** cs.LG / arXiv

研究大语言模型在无外部反馈下自主检测并修正错误的机理，揭示内部置信信号在 CoT 推理中的关键作用，为可靠推理和自主 Agent 设计提供理论依据。


### [Ulysses 序列并行：百万 Token 上下文的分布式训练实践](https://huggingface.co/blog/ulysses-sp)

> **来源：** HuggingFace Blog

HuggingFace 团队详解 DeepSpeed Ulysses 序列并行方案在百万 Token 超长上下文训练中的工程实践，涵盖通信优化、显存管理及与 ZeRO 的协同策略。


### [Keep the Tokens Flowing：16 个开源 RL 训练库的经验总结](https://huggingface.co/blog/async-rl-training-landscape)

> **来源：** HuggingFace Blog

横向对比 16 个主流开源强化学习训练框架（含 TRL、OpenRLHF、veRL 等），从吞吐量、可扩展性、易用性多维度总结各库的设计取舍与工程经验，为 RLHF/GRPO 工程选型提供参考。


### [CGRA 编译器的多面体变换预优化内核复用技术](https://arxiv.org/abs/2604.22297)

> **来源：** cs.AR / arXiv

研究如何在粗粒度可重构阵列（CGRA）编译器中结合多面体变换复用预优化的 mmul 内核，减少编译搜索空间并提升矩阵运算吞吐，对 AI 加速器后端编译优化有直接借鉴意义。


