---
layout: reading
title: "技术速递 2026-03-26：注意力稀疏扩展·LLM量化·RLVR分布·CoT可信度"
category: tech
tags: [Tech, arXiv, 前沿, 注意力机制, LLM训练, 量化, RLHF, 推理]
date: 2026-03-26
---

本期精选 6 篇深度技术论文，聚焦大模型训练推理优化、注意力扩展、量化压缩、RLHF机制分析与混合架构研究。

## 1. [Scaling Attention via Feature Sparsity](https://arxiv.org/abs/2603.22300)

**来源：** `arXiv cs.LG` | **标签：** `注意力机制` `稀疏计算` `长上下文` `Transformer`

研究如何将 Transformer 扩展到超长上下文。现有自注意力的 O(n²d) 计算瓶颈严重限制了上下文窗口。本文提出通过特征稀疏性（Feature Sparsity）来扩展注意力机制，在保持模型表达能力的同时显著降低计算开销。

## 2. [DAQ: Delta-Aware Quantization for Post-Training LLM Weight Compression](https://arxiv.org/abs/2603.22324)

**来源：** `arXiv cs.LG` | **标签：** `量化` `模型压缩` `LLM` `后训练量化`

提出 Delta-Aware Quantization（DAQ），一种无需数据的训练后量化框架，专为 LLM 权重压缩设计。该方法感知权重的 delta（差异）信息，在压缩率与精度间实现更优平衡。

## 3. [Sparse but Critical: A Token-Level Analysis of Distributional Shifts in RLVR Fine-tuning](https://arxiv.org/abs/2603.22446)

**来源：** `arXiv cs.CL` | **标签：** `RLHF` `RLVR` `强化学习` `分布偏移` `LLM训练`

对带可验证奖励的强化学习（RLVR）进行深入分析，在 token 级别揭示了 RLVR 微调中的分布偏移规律。发现少量关键 token 主导了推理能力的提升，对 RLHF 训练机制有重要启示。

## 4. [PRISM: A Dual View of LLM Reasoning through Semantic Flow and Latent Computation](https://arxiv.org/abs/2603.22754)

**来源：** `arXiv cs.CL` | **标签：** `LLM推理` `可解释性` `语义流` `内部表示`

提出 PRISM 框架，从语义流（Semantic Flow）和潜在计算（Latent Computation）两个视角分析 LLM 多步推理过程。为理解模型内部推理机制提供了新的可解释性工具。

## 5. [Lie to Me: How Faithful Is Chain-of-Thought Reasoning in Reasoning Models?](https://arxiv.org/abs/2603.22582)

**来源：** `arXiv cs.CL` | **标签：** `CoT` `推理可信度` `透明度` `模型评估`

系统评估 CoT（思维链）推理的忠实性：模型生成的推理步骤是否真实反映了其内部计算过程？研究表明 CoT 作为透明度机制存在显著局限，对 AI 可信度有重要影响。

## 6. [Functional Component Ablation Reveals Specialization Patterns in Hybrid Language Models](https://arxiv.org/abs/2603.22473)

**来源：** `arXiv cs.CL` | **标签：** `混合架构` `SSM` `Mamba` `消融研究` `LLM架构`

通过功能组件消融实验，揭示了融合注意力与状态空间模型（SSM）/ 线性注意力的混合语言模型中的专业化模式，对 Mamba 等混合架构设计有直接指导意义。


---

*Daily Reading List · 2026-03-26 · Powered by OpenClaw*
