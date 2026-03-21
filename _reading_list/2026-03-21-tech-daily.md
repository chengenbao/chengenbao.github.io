---
layout: reading
title: "技术速递 2026-03-21：推理加速 × 后训练优化 × MoE 架构"
category: tech
tags: [Tech, ArXiv, 推理加速, SpeculativeDecoding, MoE, PostTraining, TARo]
date: 2026-03-21
---

今日精选 6 篇深度技术论文，聚焦推理加速框架、后训练 Pipeline 自动化、MoE 稀疏激活蒸馏与 Token 级测试时对齐等前沿方向。

## SpecForge：推测解码开源训练框架

**来源：** arxiv cs.LG · [2603.18567](https://arxiv.org/abs/2603.18567)

大语言模型自回归解码带来高推理延迟。**SpecForge** 提供了灵活高效的开源推测解码训练框架，支持多种 draft model 训练策略，显著降低 LLM 推理延迟。

**关键词：** 推理加速 · 推测解码 · 开源框架

---

## Nemotron-Cascade 2：级联 RL 与多域在线蒸馏后训练

**来源：** arxiv cs.CL · [2603.19220](https://arxiv.org/abs/2603.19220)

NVIDIA 发布 **Nemotron-Cascade 2**，30B MoE 模型仅激活 3B 参数，通过 Cascade RL + 多域在线策略蒸馏训练，在效率和效果上达到同级别最优。

**关键词：** MoE · 强化学习 · On-Policy 蒸馏 · 后训练

---

## LLM 后训练 Pipeline 自动配置

**来源：** arxiv cs.LG · [2603.18773](https://arxiv.org/abs/2603.18773)

SFT + RL 组合训练 Pipeline 配置复杂。本文提出自动配置方法，减少人工调参开销，提升后训练可复现性与效率。

**关键词：** Post-Training · 自动化 · SFT · RLHF Pipeline

---

## TARo：LLM 测试时对齐的 Token 级自适应路由

**来源：** arxiv cs.CL · [2603.18411](https://arxiv.org/abs/2603.18411)

**TARo** 在推理时进行 Token 级自适应路由，无需完整再训练即可实现高效测试时对齐，兼顾性能与计算效率。

**关键词：** Test-time Alignment · Token Routing · 推理效率

---

## 从混合数据到专域：语言模型最优分割策略

**来源：** arxiv cs.CL · [2603.19149](https://arxiv.org/abs/2603.19149)

研究如何将混合数据训练的语言模型"最优分割"为特定专业域模型，推进 MoE 式专业化与域自适应策略的理论框架。

**关键词：** MoE 专业化 · 域自适应 · 模型分割

---

## 日语小语言模型领域适配：规模、架构与量化

**来源：** arxiv cs.LG · [2603.18037](https://arxiv.org/abs/2603.18037)

系统梳理构建日语领域 SLM 的方法论——规模选择、架构设计与量化部署，对边缘侧 SLM 工程实践有参考价值。

**关键词：** 小语言模型 · 量化 · 域专化 · 边缘部署
