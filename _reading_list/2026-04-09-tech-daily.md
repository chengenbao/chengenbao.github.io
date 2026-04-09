---
layout: reading
title: "技术速递 2026-04-09：LLM训练/推理优化与架构探索"
category: tech
tags: [Tech, arXiv, 大模型, 推理优化, MoE, 混合架构, 微调]
date: 2026-04-09
---

今日精选 5 篇 arXiv 前沿论文，聚焦大模型训练效率、推理加速、MoE 架构分析与非 Transformer 混合架构探索。

## 1. MegaTrain：单卡全精度训练 100B+ 参数 LLM

**来源：** [arXiv:2604.05091](https://arxiv.org/abs/2604.05091)

以内存为中心的系统设计，通过参数卸载（CPU/NVMe）+ 按需流式加载 + 梯度压缩 + 流水线 IO，在单块消费级 GPU 上实现 100B+ 模型全精度训练。对独立研究者和小团队意义重大，大幅降低前沿模型研究门槛。

---

## 2. Multi-Drafter Speculative Decoding with Alignment Feedback

**来源：** [arXiv:2604.05417](https://arxiv.org/abs/2604.05417)

在投机解码框架中引入多个草稿模型，通过对齐反馈动态调整各草稿置信权重。相比单草稿方案接受率更高、延迟更低，同时减少草稿模型引入的分布偏移，提升输出质量。

---

## 3. Scalable Variational Bayesian Fine-Tuning via Orthogonalized Low-Rank Adapters

**来源：** [arXiv:2604.03388](https://arxiv.org/abs/2604.03388)

将变分贝叶斯推断与正交化 LoRA 结合，使 LLM 能够在安全关键场景输出可靠置信度估计。正交化约束确保不同秩分量独立更新，显著降低变分近似误差，计算开销与标准 LoRA 相当。

---

## 4. Do Domain-specific Experts Exist in MoE-based LLMs?

**来源：** [arXiv:2604.05267](https://arxiv.org/abs/2604.05267)

系统分析 MoE 模型中专家网络的功能分工：浅层专家负责语法/格式，深层才现语义/领域特化；大量专家存在功能重叠，暗示路由机制冗余，为 MoE 剪枝和蒸馏提供新分析维度。

---

## 5. Olmo Hybrid：线性循环模型与 Transformer 的工程化融合

**来源：** [arXiv:2604.03444](https://arxiv.org/abs/2604.03444)

OLMo Hybrid 将线性循环层（Mamba/SSM）与标准 Transformer 注意力层交替堆叠，在长序列效率与复杂推理精度之间取得平衡。详细分析了混合比例与插入位置的影响，全套代码与权重开源。

---

*由 OpenClaw 自动生成 · 2026-04-09*
