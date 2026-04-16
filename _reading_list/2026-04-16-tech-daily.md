---
layout: reading
title: "技术速递 2026-04-16：SFT层级分析、LoRA高阶扩展与推测解码加速"
category: tech
tags: [Tech, arXiv, LLM, 推理加速, 分布式训练, RLHF]
date: 2026-04-16
---

本期精选 6 篇 arXiv 最新论文，聚焦大模型 SFT 分析、参数高效微调改进、多 Token 训练目标、低带宽分布式训练、RLHF 自蒸馏以及推测解码加速方向。

## [A Layer-wise Analysis of Supervised Fine-Tuning](https://arxiv.org/abs/2604.11838)

**来源：** arXiv cs.LG · **标签：** SFT / Fine-Tuning / LoRA / LLM对齐

深入分析 SFT 各层行为：前层冻结有助减少过拟合，后层更新是对齐效果的关键所在。揭示了 LoRA 等参数高效方法在 SFT 中的机制。

## [Polynomial Expansion Rank Adaptation: Enhancing Low-Rank Fine-Tuning with High-Order Interactions](https://arxiv.org/abs/2604.11841)

**来源：** arXiv cs.LG · **标签：** LoRA / PEFT / 参数高效微调 / LLM

PolyLoRA：通过多项式展开引入高阶交互，在同等参数量下显著提升 LoRA 的表达能力，GLUE/MMLU 多项基准超越标准 LoRA。

## [How Transformers Learn to Plan via Multi-Token Prediction](https://arxiv.org/abs/2604.11912)

**来源：** arXiv cs.LG · **标签：** Multi-Token Prediction / 训练目标 / Transformer / 推理规划

多 token 预测训练目标（MTP）使模型隐式学会前瞻规划能力，优于 NTP。分析了 MTP 在解码树搜索中的优势以及对 CoT 推理的增益。

## [ResBM: Residual Bottleneck Models for Low-Bandwidth Pipeline Parallelism](https://arxiv.org/abs/2604.11947)

**来源：** arXiv cs.LG · **标签：** 分布式训练 / Pipeline并行 / 通信优化 / LLM训练

提出 ResBM 架构，专为低带宽网络下的去中心化 pipeline 并行训练设计，大幅减少跨节点通信量，支持大规模分布式 LLM 训练。

## [Self-Distillation Zero: Self-Revision Turns Binary Rewards into Dense Supervision](https://arxiv.org/abs/2604.12002)

**来源：** arXiv cs.CL · **标签：** RLHF / Self-Distillation / 奖励模型 / 强化学习

SD-Zero：将稀疏二元奖励通过自蒸馏转化为稠密监督信号，在可验证的 RLHF 场景中填补 PPO 与 DPO 之间的空白，大幅提升数学推理性能。

## [SpecBound: Adaptive Bounded Self-Speculation with Layer-wise Confidence Calibration](https://arxiv.org/abs/2604.12247)

**来源：** arXiv cs.CL · **标签：** 推测解码 / 推理加速 / Speculative Decoding / LLM推理

SpecBound：无需独立草稿模型的自推测解码方案，通过逐层置信度校准自适应确定推测深度，在多类模型上实现 1.5–2.2x 推理加速。
