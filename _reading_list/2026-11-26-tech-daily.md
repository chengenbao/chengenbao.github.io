---
layout: reading
title: "LLM 推理加速与 Agent 工程前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-11-26
---

# 📰 2026-11-26 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 LLM 推理加速、KV 缓存压缩、长程 Agent 训练、开源语音模型与单卡推理部署。

---

## 1. vLLM 新加坡 Meetup：更快 Prefill、Agent 编写 Kernel 与 Vera Rubin 路线图

**来源**：PyTorch Blog
**链接**：https://pytorch.org/blog/vllm-singapore-meetup-faster-prefill-agent-written-kernels-and-the-road-to-vera-rubin/
**标签**：推理加速 · vLLM · Pipeline Parallelism · Agent-Kernel ·  prefill

新加坡 vLLM Meetup 上，来自 Inferact、NVIDIA 与阿里云的工程师展示了开源 AI 推理的最新进展：通过流水线并行（pipeline parallelism）优化显著提升 prefill 吞吐，以及由 agent 自动编写的定制 kernel，并展望了面向 Vera Rubin 架构的下一代推理栈。

**核心要点**：
- 流水线并行优化直接改善长上下文 prefill 阶段的吞吐与延迟
- Agent 自动编写并调优高性能 kernel，降低手写 CUDA 的门槛
- 社区正围绕 Vera Rubin 硬件架构规划下一代开源推理引擎路线

---

## 2. FreDraft：直接最大化 On-policy 期望接受长度以改进投机解码

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2611.31256
**标签**：投机解码 · LLM 推理 · 草稿模型 · On-policy 训练 · 加速

投机解码用轻量草稿模型提议 token、再由目标模型验证以加速 LLM 推理。传统草稿训练优化 token 级分布对齐，却不直接优化被接受前缀的长度——一旦某个草稿 token 被拒，其后所有草稿 token 全部作废。本文提出 FreDraft，一种 on-policy 训练方法，直接最大化期望接受长度。

**核心要点**：
- 指出 token 级对齐并不直接对应更长的接受前缀这一关键缺陷
- 通过 on-policy 训练让草稿模型与目标模型的接受长度直接对齐
- 在推理加速比上优于常规分布对齐训练的草稿模型

---

## 3. SD-Cache：基于状态引导动态缓存的可学习记忆整合

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2611.31265
**标签**：KV Cache · RAG · 内存压缩 · 推理成本 · 可学习缓存

LLM 越来越多作为需推理大量外部知识的持久服务部署。RAG 用检索增强生成把回答锚定到相关文档，但检索内容仍需编码并存入 KV cache，随内容累积线性增长，推高显存占用与推理成本。现有压缩方法难以兼顾保真与效率，本文提出 SD-Cache，以状态引导的动态缓存做可学习记忆整合。

**核心要点**：
- 揭示 RAG 场景下 KV cache 随外部知识线性膨胀的内存瓶颈
- 用状态引导的动态缓存机制学习保留高价值记忆、压缩冗余
- 在控制质量损失的同时降低显存与推理开销

---

## 4. RECAP：用上下文压缩训练长程 LLM Agent

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2611.31258
**标签**：LLM Agent · 上下文压缩 · 强化学习 · 长程任务 · 记忆

上下文压缩让 LLM agent 在有限上下文预算内处理长程任务——用压缩摘要替代交互历史。但让压缩与 agent 策略联合学习很难，因为摘要会影响后续多步交互的成功率。本文提出 RECAP，一个基于“摘要应保留未来任务所需信息”原则的强化学习框架。

**核心要点**：
- 将上下文压缩与 agent 策略的联合训练形式化为 RL 问题
- 以“保留未来有用信息”为奖励信号指导压缩决策
- 在长程任务上提升成功率并压低上下文开销

---

## 5. Rumble-2-Flash：覆盖 40+ 语言的开源 0.5B 语音模型

**来源**：Hugging Face Blog
**链接**：https://huggingface.co/blog/rumble-2-flash
**标签**：语音模型 · 多语言 · 小型模型 · 开源 · 端侧推理

Rumble-2-Flash 是一个开源的 0.5B 参数语音模型，支持 40 多种语言。它在保持极小参数量的同时覆盖广泛语种，适合在资源受限设备与实时场景中部署，为多语言语音理解与生成提供了一个轻量可复现的基线。

**核心要点**：
- 0.5B 小参数规模兼顾多语言覆盖与低部署成本
- 覆盖 40+ 语言，面向端侧与实时语音应用
- 开源权重便于社区复现与二次开发

---

## 6. 在单张 RTX PRO 6000 上做 Inkling 2 Mini 推理的爬坡实验

**来源**：Hugging Face Blog
**链接**：https://huggingface.co/blog/inkling-2-mini-inference-hillclimb
**标签**：单卡推理 · RTX PRO 6000 · 吞吐优化 · vLLM · 部署

本文记录在单张 RTX PRO 6000 上运行 Inkling 2 Mini 的推理爬坡实验：通过逐步调优配置把吞吐推到约 190 tok/s，并对比了 vLLM、SGLang 等方案的取舍，剖析了速度瓶颈所在。

**核心要点**：
- 单卡 RTX PRO 6000 上实现约 190 tok/s 的推理吞吐
- 对比 vLLM / SGLang 等引擎在该模型上的表现与限制
- 剖析并定位了限制进一步提速的工程瓶颈

---

## 7. 如何构建并维护 Agent 化评测（Agentic Evaluation）

**来源**：Hugging Face Blog
**链接**：https://huggingface.co/blog/agentic-evals
**标签**：Agent 评测 · 评估体系 · 可靠性 · 持续维护 · MLOps

随着 agent 系统越来越复杂，传统单次评测难以反映真实表现。本文系统讲解如何设计并长期维护一套 agent 化评测体系，涵盖评测目标设定、环境构建、指标选择与持续回归监控，帮助团队在迭代中保持可靠性。

**核心要点**：
- 从目标定义到环境搭建给出可落地的 agent 评测方法论
- 强调多轮、多运行评估以捕捉 agent 的不稳定行为
- 提供持续回归监控思路，避免迭代引入隐性退化

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | vLLM 新加坡 Meetup | PyTorch Blog | 推理加速 |
| 2 | FreDraft | arXiv cs.CL | 投机解码 |
| 3 | SD-Cache | arXiv cs.CL | KV缓存 |
| 4 | RECAP | arXiv cs.CL | 长程Agent |
| 5 | Rumble-2-Flash | Hugging Face Blog | 多语言语音 |
| 6 | 在单张 RTX PRO 6000 上 | Hugging Face Blog | 单卡推理 |
| 7 | 如何构建并维护 Agent 化评测（ | Hugging Face Blog | Agent评测 |

*自动生成 · 2026-11-26 · jeffinchen daily tech reading list*

