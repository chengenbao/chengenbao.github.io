---
layout: reading
title: "技术速递 2026-04-05：RL推理扩展 · RLHF奖励建模 · LLM机理分析"
category: tech
tags: [Tech, arXiv, 前沿, LLM, RLHF, 推理优化]
date: 2026-04-05
---

今日精选 6 篇来自 arXiv cs.CL 的深度技术论文，聚焦大模型推理扩展、RLHF 奖励建模与模型机理分析。

## 📌 今日主题

**推理即效率**：本周 arXiv 出现多篇探讨如何在推理阶段更聪明地使用计算资源的论文——无论是通过 RL 动态分配 token 预算、自适应停止多轮推理，还是从机理层面理解模型置信度，方向高度一致。

---

## 1. [Scaling Reasoning Tokens via RL and Parallel Thinking](https://arxiv.org/abs/2604.01302)

**来源：** arXiv cs.CL | **标签：** `RL` `推理扩展` `LLM训练`

研究如何通过强化学习（RL）和并行思维来扩展推理 token 预算以应对竞争性编程任务。提出两种互补策略：基于 RL 的自适应预算分配，以及通过并行采样实现的推理扩展，在无需额外训练的前提下显著提升了模型在难题上的 pass@k 指标。

---

## 2. [Procedural Knowledge at Scale Improves Reasoning](https://arxiv.org/abs/2604.01348)

**来源：** arXiv cs.CL | **标签：** `推理优化` `知识增强` `Test-time Scaling`

探索将程序性知识大规模注入语言模型以改善推理能力。从知识类型角度切入，证明结构化程序性知识在规模扩展时能带来持续推理增益。

---

## 3. [Adaptive Stopping for Multi-Turn LLM Reasoning](https://arxiv.org/abs/2604.01413)

**来源：** arXiv cs.CL | **标签：** `推理效率` `多轮对话` `自适应`

针对 LLM 多轮推理场景，提出自适应停止机制，通过学习停止条件在效率与准确性之间取得最优平衡。

---

## 4. [Wired for Overconfidence: A Mechanistic Perspective on LLMs](https://arxiv.org/abs/2604.01457)

**来源：** arXiv cs.CL | **标签：** `模型内部机制` `置信度校准` `可解释性`

从机理层面解析 LLM 的过度自信现象，通过对 attention 模式和内部激活的分析找到结构性成因，为校准方法提供理论基础。

---

## 5. [Preference Learning in Shades of Gray: Bias-Aware Reward Modeling](https://arxiv.org/abs/2604.01312)

**来源：** arXiv cs.CL | **标签：** `RLHF` `奖励建模` `偏好学习`

提出灰度感知奖励模型，在保持可解释性的同时显式建模标注不确定性，显著改善 RLHF 奖励建模的鲁棒性。

---

## 6. [Why Instruction-Based Unlearning Fails in Diffusion Models](https://arxiv.org/abs/2604.01514)

**来源：** arXiv cs.CL | **标签：** `机器遗忘` `扩散模型` `模型安全`

揭示指令级遗忘在扩散模型中失效的机理成因，并提出针对性改进策略。
