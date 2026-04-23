---
layout: reading
title: "技术速递 2026-04-23：MoE推理加速 · 早退优化 · 知识蒸馏 · 扩散LLM · SAE安全"
category: tech
tags: [Tech, arXiv, 前沿, MoE, 推理加速, 知识蒸馏]
date: 2026-04-23
---

今日精选 6 篇深度技术论文，聚焦 LLM 推理加速、MoE 架构优化、知识蒸馏、扩散语言模型生成机制、模型安全与硬件感知 KG 检索。

---

## 1. MoE LLM 在 Apple Silicon NPU 上的高效推理

**标题**：Efficient Mixture-of-Experts LLM Inference with Apple Silicon NPUs

**来源**：arXiv cs.LG | [论文链接](https://arxiv.org/abs/2604.18788)

Apple Neural Engine（ANE）是集成在每颗 Apple Silicon 芯片中的专用 NPU。本文研究如何将 MoE-LLM 推理卸载至 ANE，通过算子映射与 Core ML 编译优化，实现比纯 GPU 路径更低的延迟与更高的吞吐，首次在消费级设备上高效运行 MoE-LLM，为端侧 AI 部署开辟新路径。

`MoE` `推理加速` `NPU` `Apple Silicon` `边缘部署`

---

## 2. LLM 推理的二维早退优化

**标题**：Two-dimensional early exit optimisation of LLM inference

**来源**：arXiv cs.CL | [论文链接](https://arxiv.org/abs/2604.18592)

提出二维早退（2D Early Exit）策略，同时协调逐层退出（layer-wise）与逐句退出（sentence-wise）两个维度。动态跳过不必要的 Transformer 层与样本计算，推理 FLOPs 减少 30%~50%，同时保持与全量推理相当的精度。

`推理加速` `Early Exit` `Transformer` `效率优化`

---

## 3. LLM 可蒸馏性的校准旋钮：蒸馏陷阱与守卫

**标题**：Distillation Traps and Guards: A Calibration Knob for LLM Distillability

**来源**：arXiv cs.LG | [论文链接](https://arxiv.org/abs/2604.18963)

揭示"蒸馏陷阱"现象：教师模型某些能力天然难以蒸馏（如推理链、多步骤计划）。提出 Guard 机制作为可调节旋钮，在蒸馏过程中保护这些关键能力，显著提升学生模型在困难任务上的效果与稳定性。

`知识蒸馏` `LLM压缩` `模型优化`

---

## 4. 掩码扩散语言模型的 Token-to-Mask 精炼

**标题**：Remask, Don't Replace: Token-to-Mask Refinement in Masked Diffusion Language Models

**来源**：arXiv cs.CL | [论文链接](https://arxiv.org/abs/2604.18738)

针对掩码扩散语言模型（如 LLaDA2.1）提出"重掩码而非替换"策略：将错误 token 重新转为 [MASK] 而非直接替换，在随后去噪步骤中重新生成，避免错误传播，在多个生成基准上提升质量。

`扩散模型` `掩码语言模型` `LLM生成` `推理优化`

---

## 5. 稀疏自编码器鲁棒性：LLM 安全机制探索

**标题**：Towards Understanding the Robustness of Sparse Autoencoders

**来源**：arXiv cs.LG | [论文链接](https://arxiv.org/abs/2604.18756)

系统研究稀疏自编码器（SAE）在对抗攻击下的鲁棒性，发现 SAE 特征在对抗样本下的失效模式（特征崩塌与激活漂移），为构建更鲁棒的 LLM 安全防御机制提供理论依据。

`LLM安全` `稀疏自编码器` `可解释性` `对抗攻击`

---

## 6. LogosKG：硬件优化的可扩展知识图谱检索

**标题**：LogosKG: Hardware-Optimized Scalable and Interpretable Knowledge Graph Retrieval

**来源**：arXiv cs.CL | [论文链接](https://arxiv.org/abs/2604.18913)

将知识图谱（KG）与 LLM 集成提供结构化可验证推理。LogosKG 通过硬件感知的图索引与量化检索优化，在保持完整解释性的同时大幅提升检索速度，在标准 RAG 基准上达到 SOTA 水平。

`知识图谱` `RAG` `LLM推理` `硬件优化`

---

*来源：arXiv cs.LG / cs.CL · 2026-04-23*
