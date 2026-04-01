---
layout: reading
title: "技术速递 2026-04-01：LLM训练效率 · 推理加速 · Diffusion LM"
category: tech
tags: [Tech, arXiv, 大模型, 推理加速, 优化器, DiffusionLM]
date: 2026-04-01
---

今日精选 6 篇前沿技术论文，聚焦 LLM 训练系统优化、推理调度、两阶段优化器理论，以及新兴的 Diffusion 语言模型技术。

---

## 1. LLM 大规模训练的吞吐量优化策略

**论文：** [Throughput Optimization as a Strategic Lever in Large-Scale AI Systems](https://arxiv.org/abs/2603.26823)

大规模基础模型训练受计算和内存瓶颈制约。本文将 **DataLoader 设计**与**流水线调度**视为 LLM 训练基础设施中的一等工程手段，实证证明在相同硬件预算下可显著提升训练效率。

> 关键贡献：量化了 DataLoader 策略对整体吞吐量的影响比例，并提出针对大规模集群的 pipeline 调度建议。

---

## 2. LLM 批量级请求路由：成本与容量双约束优化

**论文：** [Robust Batch-Level Query Routing for LLMs](https://arxiv.org/abs/2603.26796)

传统 per-query 路由忽略了**批次级别**的负载分布效应。本文提出鲁棒批量路由框架，在 GPU 资源与并发约束下最优分配请求至不同规模 LLM，显著降低推理服务成本。

> 工程意义：可直接应用于 vLLM/TensorRT-LLM 等推理服务的调度层。

---

## 3. 高维理论视角下的两阶段优化器

**论文：** [High Dimensional Theory of Two-Phase Optimizers](https://arxiv.org/abs/2603.26954)

随着训练规模增大，**部分异步两阶段优化器**（本地优化 + 全局同步）重获关注。本文提供高维收敛性与泛化特性的理论分析，为 DiLoCo 等分布式训练范式提供理论背书。

> 核心洞察：高维场景下两阶段同步比全同步 SGD 具有更优的方差-偏差权衡。

---

## 4. LogicDiff：逻辑引导的 Diffusion 语言模型推理增强

**论文：** [LogicDiff: Logic-Guided Denoising in Masked Diffusion Language Models](https://arxiv.org/abs/2603.26771)

掩码扩散语言模型（MDLM）支持并行 token 生成，但多步逻辑推理能力偏弱。**LogicDiff** 通过在去噪过程中注入逻辑约束，在保持并行效率的同时大幅提升推理准确率。

---

## 5. GeoBlock：基于依赖几何的并行解码块粒度自动推断

**论文：** [GeoBlock: Inferring Block Granularity from Dependency Geometry](https://arxiv.org/abs/2603.26675)

块扩散（Block Diffusion）的性能高度依赖块大小，传统方法需人工调参。**GeoBlock** 通过分析 token 依赖结构的几何形态自动推断最优块粒度，消除人工超参调节负担。

---

## 6. TED：免训练的多模态推理经验蒸馏

**论文：** [TED: Training-Free Experience Distillation for Multimodal Reasoning](https://arxiv.org/abs/2603.26778)

传统知识蒸馏需要额外训练步骤。**TED** 从大模型的推理轨迹中直接提取"经验"，无需微调即可迁移至小模型，在多模态推理 benchmark 上取得接近教师模型的表现。

---

*来源：arXiv cs.LG / cs.CL | 2026-04-01*
