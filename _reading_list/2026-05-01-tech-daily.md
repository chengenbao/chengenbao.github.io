---
layout: reading
title: "技术速递 2026-05-01：扩散LLM蒸馏 · MoE Serverless推理 · RL后训练加速"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-05-01
---

> 今日精选 6 篇前沿技术论文，聚焦大模型训练、推理系统优化与模型可解释性。

### 1. 系统集成推测解码加速 RL 后训练 Rollout 生成

> 来源：cs.CL | [原文链接](http://arxiv.org/abs/2604.26779v1)

大模型 RL 后训练（如 RLHF）中，自回归 rollout 生成是主要瓶颈。本文提出将推测解码（Speculative Decoding）与 RL 训练系统深度集成，大幅提升 rollout 生成速度，降低训练整体耗时。

### 2. TIDE：跨架构蒸馏训练扩散大语言模型

> 来源：cs.LG | [原文链接](http://arxiv.org/abs/2604.26951v1)

扩散语言模型（dLLM）支持并行解码与双向上下文，但训练代价高昂。本文提出从自回归 LLM 到 dLLM 的跨架构知识蒸馏方法，显著降低扩散语言模型的训练成本，同时保持生成质量。

### 3. FaaSMoE：面向多租户 MoE 模型推理的 Serverless 框架

> 来源：cs.LG | [原文链接](http://arxiv.org/abs/2604.26881v1)

MoE 模型通过稀疏激活专家降低推理成本，但多租户部署面临冷启动、负载均衡和专家路由挑战。FaaSMoE 提出 Serverless 架构解决方案，实现高效弹性的 MoE 模型在线服务。

### 4. 语言扩散模型作为联想记忆：泛化与记忆的边界分析

> 来源：cs.LG | [原文链接](http://arxiv.org/abs/2604.26841v1)

本文分析扩散语言模型的记忆与泛化机制，证明其本质上是一种联想记忆系统，能够检索训练数据之外的组合模式。研究为理解扩散 LLM 的生成能力上界提供了定量框架。

### 5. MoRFI：单调稀疏自编码器的 LLM 特征解释方法

> 来源：cs.LG | [原文链接](http://arxiv.org/abs/2604.26866v1)

LLM 在预训练阶段习得大部分事实知识，但内部表示难以解释。MoRFI 提出单调稀疏自编码器方法，实现对 LLM 特征的可解释性识别，揭示事实知识在 transformer 各层的存储与检索机制。

### 6. 深度 Transformer 训练中的随机缩放极限与噪声同步理论

> 来源：cs.LG | [原文链接](http://arxiv.org/abs/2604.26898v1)

本文从理论上证明了有限深度、有限宽度 Transformer 模型中 token 演化的路径收敛性，建立了深度 Transformer 训练过程中噪声同步效应的数学基础，为大模型训练稳定性分析提供理论工具。

