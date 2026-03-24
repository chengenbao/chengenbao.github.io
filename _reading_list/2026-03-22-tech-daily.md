---
layout: reading
title: "技术速递 2026-03-22：异步 RL 训练 × 序列并行 × MoE 架构全解"
category: tech
tags: [Tech, HuggingFace, arXiv, 多源, 前沿]
date: 2026-03-22
---

# 技术速递 2026-03-22：异步 RL 训练 × 序列并行 × MoE 架构全解

> **聚焦方向：** 大模型训练效率、RL 后训练框架、MoE 架构、长上下文训练、推理可信度
> **来源：** Hugging Face Blog × arXiv cs.LG / cs.CL

---

## 1. Keep the Tokens Flowing: Lessons from 16 Open-Source RL Libraries

**来源：** [Hugging Face Blog](https://huggingface.co/blog/async-rl-training-landscape) · Mar 10, 2026

**核心问题：** 同步 RL 训练中，数据生成（模型推理）主导 wall-clock time——32B 模型的一批 32K-token rollout 可能耗时数小时，训练 GPU 持续空闲。

**解法：** 将推理与训练分离到不同 GPU 池，通过 rollout buffer 连接，异步权重传输。

**关键发现（对比 16 个开源库）：**
- Ray 主导 orchestration（16 个库中 8 个使用）
- NCCL broadcast 是权重传输默认方式
- Staleness 管理：从丢弃旧样本到 importance sampling 修正各有差异
- LoRA 支持稀疏；**分布式 MoE 支持是新兴差异点**

---

## 2. Ulysses Sequence Parallelism: Training with Million-Token Contexts

**来源：** [Hugging Face Blog](https://huggingface.co/blog/ulysses-sp) · Mar 9, 2026

**问题：** Transformer attention 计算量随序列长度平方增长，超长序列无法在单 GPU 训练。

**Ulysses SP：** 通过注意力头并行（attention head parallelism）将 attention 计算分布到多 GPU。
- 支持 Accelerate / Transformers Trainer / TRL SFTTrainer
- 实测支持百万 token 上下文训练
- 与 Ring Attention 互补

---

## 3. Mixture of Experts (MoEs) in Transformers

**来源：** [Hugging Face Blog](https://huggingface.co/blog/moe-transformers) · Feb 26, 2026

MoE 用多个 expert 替换 FFN 层，router 为每个 token 动态选少数 expert，实现"大容量 + 低推理成本"。

**工程要点：**
- gpt-oss-20b：21B 总参数，每 token 激活 3.6B，实测 ~115 tok/s（M3 Ultra Mac）
- 负载均衡防 expert collapse（auxiliary loss / expert choice routing）
- transformers 库已深度支持量化与混合精度推理

---

## 4. Measuring Faithfulness Depends on How You Measure

**来源：** [arXiv:2603.20172](https://arxiv.org/abs/2603.20172) · cs.CL · Mar 23, 2026

三种分类器对相同 CoT 轨迹给出的忠实度分别为 74.4%、82.6%、69.7%，单模型差距最高 30.6 个百分点，且可逆转模型排名。

**启示：** 跨研究比较 CoT 忠实度前，必须核对分类器方法论。

---

## 5. EvidenceRL: Reinforcing Evidence Consistency

**来源：** [arXiv:2603.19532](https://arxiv.org/abs/2603.19532) · cs.CL / cs.LG · Mar 20, 2026

用 GRPO 训练时双重评分（evidence grounding + correctness），在心脏诊断和法律推理任务上大幅降低幻觉。

- Llama-3.2-3B：F1@3 从 37.0 → 54.5，幻觉降低 5×
- Llama-3.1-8B：Faithfulness 从 32.8% → 67.6%

---

*Generated: 2026-03-22 | Sources: Hugging Face Blog, arXiv cs.LG/cs.CL*
