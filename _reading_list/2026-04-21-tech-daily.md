---
layout: reading
title: "技术速递 2026-04-21：KV Cache压缩·FP16精度·LoRA优化·DPO对齐·Qwen3.5-Omni"
category: tech
tags: [Tech, arXiv, 推理优化, 微调, 多模态]
date: 2026-04-21
---

今日精选 6 篇深度技术论文，涵盖大模型推理优化、参数高效微调、对齐算法、知识蒸馏与多模态架构。

## 1. Sequential KV Cache Compression via Probabilistic Language Tries

**来源**：arXiv cs.LG &nbsp;|&nbsp; [论文链接](https://arxiv.org/abs/2604.15356)

本文提出基于概率语言 Trie 结构的 KV Cache 序列化压缩方法，突破 TurboQuant 的 per-vector Shannon 熵下限。通过跨 token 联合建模与压缩，超越逐向量压缩理论上限，显著降低长上下文推理内存占用。

**关键词**：KV Cache · 推理优化 · 熵编码 · Transformer 压缩

---

## 2. The Illusion of Equivalence: Systematic FP16 Divergence in KV-Cached Inference

**来源**：arXiv cs.LG &nbsp;|&nbsp; [论文链接](https://arxiv.org/abs/2604.15409)

揭示 FP16 精度下 KV 缓存与无缓存自回归推理存在系统性数值偏差，长序列生成中偏差可累积放大，影响输出一致性与可重现性，并提出缓解策略。

**关键词**：KV Cache · FP16 精度 · 数值稳定性 · 自回归推理

---

## 3. Aletheia: Gradient-Guided Layer Selection for Efficient LoRA Fine-Tuning

**来源**：arXiv cs.LG &nbsp;|&nbsp; [论文链接](https://arxiv.org/abs/2604.15351)

通过梯度引导的层选择算法，自动识别最值得 LoRA 适配的层子集，在不同架构上实现更高参数效率，保持性能的同时大幅减少可训练参数量。

**关键词**：LoRA · 参数高效微调 · 梯度分析 · LLM 微调

---

## 4. GroupDPO: Memory Efficient Group-wise Direct Preference Optimization

**来源**：arXiv cs.CL &nbsp;|&nbsp; [论文链接](https://arxiv.org/abs/2604.15602)

将多个 DPO 偏好对组合为批量组进行联合优化，显著降低 GPU 内存占用，同时改善对齐质量。可直接替换现有 DPO 训练流程，工程落地成本低。

**关键词**：DPO · 偏好优化 · RLHF · 内存优化 · LLM 对齐

---

## 5. Improving Reasoning in Small Models through Mixture-of-Layers Distillation

**来源**：arXiv cs.CL &nbsp;|&nbsp; [论文链接](https://arxiv.org/abs/2604.15701)

提出混合层蒸馏结合逐步监督信号，将 Chain-of-Thought 推理能力高效传递给小模型，在数学和逻辑推理基准上取得显著提升，为边缘端可推理小模型提供新方案。

**关键词**：知识蒸馏 · Chain-of-Thought · 推理小模型 · 逐步监督

---

## 6. Qwen3.5-Omni Technical Report

**来源**：arXiv cs.CL &nbsp;|&nbsp; [论文链接](https://arxiv.org/abs/2604.15804)

Qwen-Omni 系列最新多模态大模型技术报告，详细介绍多模态统一架构、训练策略、多任务对齐方法，以及语音、图像、视频和文本理解的综合性能。

**关键词**：多模态大模型 · Qwen · Omni 模型 · 多模态训练 · 技术报告

---

*生成时间：2026-04-21 | 数据来源：arXiv cs.LG / cs.CL RSS*
