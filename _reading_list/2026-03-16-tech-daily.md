---
layout: reading
title: "技术速递 2026-03-16：训练算法革新与大规模 RL 后训练基础设施"
category: tech
tags: [Tech, ArXiv, PyTorch, HuggingFace, 训练算法, 分布式训练, RLHF, Mamba]
date: 2026-03-16
---

本期聚焦：新型 on-policy 训练范式（EBFT）、持续后训练遗忘定量理论、Mamba/SSM 安全防御、torchforge+Weaver 大规模 RL 框架、Ulysses 序列并行与统一神经架构原语。

---

## 1. Energy-Based Fine-Tuning of Language Models

**来源：** [arXiv:2603.12248](https://arxiv.org/abs/2603.12248)

提出 EBFT，以序列级特征匹配替代 cross-entropy，通过分块并行采样并发生成多个 rollout 完成 on-policy policy-gradient 更新。无需 verifier/preference model，在 Q&A、代码、翻译任务上媲美 RLVR，且验证 CE 更低。

**核心贡献：** 无 reward model 的 on-policy 训练新范式 | `#训练算法` `#RLHF替代`

---

## 2. A Quantitative Characterization of Forgetting in Post-Training

**来源：** [arXiv:2603.12163](https://arxiv.org/abs/2603.12163)

在双模态混合框架下，将遗忘分为 mass forgetting 和 old-component drift 两类，证明 forward-KL 驱使旧权重归零而 reverse-KL 收敛至真实目标，量化了 replay 与两类 KL 目标的交互。

**核心贡献：** 持续微调遗忘的可证明理论边界 | `#大模型训练` `#持续学习`

---

## 3. Defending Hybrid LLMs Against Hidden State Poisoning Attacks

**来源：** [arXiv:2603.12206](https://arxiv.org/abs/2603.12206)

首个针对 Mamba/SSM 混合架构的 HiSPA 防御框架 CLASP，利用 BOE 异常模式以 XGBoost 轻量检测恶意 token，文档级 F1=99.3%。

**核心贡献：** 轻量级 SSM 安全防御，计算开销极小 | `#Mamba` `#SSM` `#模型安全`

---

## 4. Supercharging LLMs: Scalable RL with torchforge and Weaver

**来源：** [PyTorch Blog](https://pytorch.org/blog/supercharging-llms-scalable-rl-with-torchforge-and-weaver/)（Stanford × Meta × CoreWeave）

torchforge + Weaver 解决 LLM RL 后训练的 actor/critic/rollout 异步调度难题，动态平衡生成与训练 throughput，已在 Meta 内部大规模验证。

**核心贡献：** 工业级可扩展 LLM RL 后训练基础设施 | `#大模型训练` `#强化学习` `#分布式系统`

---

## 5. Ulysses Sequence Parallelism: Training with Million-Token Contexts

**来源：** [Hugging Face Blog](https://huggingface.co/blog/ulysses-sp)（2026-03-09）

Accelerate 集成 Ulysses SP，将序列切分到多 GPU，all-to-all 重排 attention head 维度，支持百万 token 上下文训练，通信效率优于 Ring Attention。

**核心贡献：** 开箱即用的百万级上下文分布式训练 | `#分布式训练` `#序列并行` `#长上下文`

---

## 6. Separable Neural Architectures as a Primitive for Unified Intelligence

**来源：** [arXiv:2603.12244](https://arxiv.org/abs/2603.12244)

将加法、二次和张量分解神经模型统一在 SNA 框架下，发现混沌时空动力学与语言自回归的结构类比，实现连续物理系统与离散序列的统一表达。

**核心贡献：** 架构层面统一生成与预测的新框架 | `#神经网络架构` `#生成模型`

---

*生成时间：2026-03-16 09:00 CST*
