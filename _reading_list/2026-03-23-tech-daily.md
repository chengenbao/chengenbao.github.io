---
layout: reading
title: "技术速递 2026-03-23：RLHF/MoE/序列并行/LLM幻觉与CoT评估"
category: tech
tags: [Tech, RLHF, MoE, 序列并行, LLM, 前沿]
date: 2026-03-23
---

# 📰 2026-03-23 · 每日技术速递

> 聚焦方向：大模型训练/推理/RLHF/MoE/序列并行 | 来源：arXiv cs.LG/cs.CL + Hugging Face Blog

---

## 1. 离散扩散模型蒸馏新方法：D-MMD

**来源：** arXiv:2603.20155 | **链接：** https://arxiv.org/abs/2603.20155

**摘要：** 连续扩散领域已有大量蒸馏方法，可将采样步骤压缩至个位数。但离散扩散蒸馏一直很困难。本文提出 **Discrete Moment Matching Distillation（D-MMD）**，借鉴连续域成功经验，以 Discrete Maximum Mean Discrepancy 为目标，解决了此前离散蒸馏方法质量退化问题。在文本和图像数据集上均表现出高质量和多样性，蒸馏后的生成器甚至超越了其教师模型。

**亮点：** 首个在文本/图像上同时有效的离散扩散蒸馏方法，填补了离散扩散与连续扩散之间的性能差距。

---

## 2. EvidenceRL：用强化学习消除 LLM 幻觉

**来源：** arXiv:2603.19532 | **链接：** https://arxiv.org/abs/2603.19532

**摘要：** LLM 容易产生"流畅但无据"的幻觉输出，在高风险领域尤为致命。本文提出 **EvidenceRL**，在训练阶段通过 RL 强制模型遵循证据。基于 **GRPO（Group Relative Policy Optimization）**，对候选答案同时评分证据锚定（entailment）与答案正确性，优化生成器。

**实验结果：**
- 心脏诊断任务（Llama-3.2-3B）：F1@3 从 37.0 → 54.5，幻觉下降约 5×，有证据支持的诊断从 31.8% → 61.6%
- 法律推理任务（Llama-3.1-8B）：Faithfulness 从 32.8% → 67.6%

**亮点：** 将 GRPO 扩展到证据锚定任务，开源代码已发布。

---

## 3. LLM CoT 忠实度测量的分类器敏感性研究

**来源：** arXiv:2603.20172 | **链接：** https://arxiv.org/abs/2603.20172

**摘要：** 大量研究报告了单一的 CoT 忠实度数字（如"DeepSeek-R1 有 39% 的概率承认外部提示"），暗示忠实度是可客观测量的属性。本文证明它**并非如此**。

对 12 个开源模型（7B–1T）的 10,276 条推理轨迹，用三种分类器打分，结果分别为 74.4%、82.6%、69.7%，置信区间不重叠。单模型分类器差距最高达 30.6 个百分点，且能导致模型排名倒转。

**亮点：** 对 CoT 评估基础设施的深度审视，未来评估应报告跨分类器的敏感性区间。

---

## 4. HuggingFace：16 个开源 RL 训练库横向对比

**来源：** Hugging Face Blog | **链接：** https://huggingface.co/blog/async-rl-training-landscape

**摘要：** 同步 RL 训练中，32B 模型单批 32K Token 的 rollout 可耗时数小时，GPU 大量闲置。本文调研了 **16 个开源 RL 训练库**，重点分析异步架构。

**关键发现：** Ray 主导编排，NCCL broadcast 是权重同步默认方式，Distributed MoE 支持是新差异化点。

---

## 5. HuggingFace：Ulysses 序列并行——百万 Token 上下文训练

**来源：** Hugging Face Blog | **链接：** https://huggingface.co/blog/ulysses-sp

**亮点：** Ulysses 序列并行通过注意力头并行将计算分散到多个 GPU，Accelerate/Transformers Trainer/TRL SFTTrainer 均已原生支持。

---

## 6. HuggingFace：Mixture of Experts（MoE）在 Transformers 中的深度解析

**来源：** Hugging Face Blog | **链接：** https://huggingface.co/blog/moe-transformers

**亮点：** 从稀疏 vs 密集、路由设计到 Transformers 实现，系统性深度解析 MoE 架构。

---

*日期：2026-03-23 | 来源：arXiv cs.LG/cs.CL + Hugging Face Blog*
