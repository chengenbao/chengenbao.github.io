---
layout: reading
title: "技术速递 2026-03-05：推理加速 · KV Cache 量化 · 安全对齐 · 多模态推理"
category: tech
tags: [Tech, 推理加速, LLM, KV-Cache, 安全对齐, 多模态]
date: 2026-03-05
---

本期聚焦推理加速与 LLM 安全两大主题，兼顾算法创新（Saguaro SSD）、工程实践（KV Cache 量化）与对齐安全（激活引导鲁棒性）。

## 1. Speculative Speculative Decoding（Saguaro）

**来源**：arXiv 2603.03251 · [论文链接](https://arxiv.org/abs/2603.03251)

标准推测解码（Speculative Decoding）通过小模型草稿+大模型并行验证加速推理，但草稿与验证之间仍存在串行依赖。本文提出 **Speculative Speculative Decoding（SSD）**，通过并行化草稿生成与验证过程来消除这一瓶颈。核心思路：验证进行的同时，草稿模型预先预测可能的验证结果并准备好对应草稿，若实际结果命中预测集合，则直接返回草稿，完全消除草稿等待开销。实现方案 Saguaro 在开源推理引擎上比优化版 Speculative Decoding **快 2x**，比自回归解码 **快 5x**。

**关键词**：推理加速 · 投机解码 · 并行化 · 自回归解码

---

## 2. KV Cache 量化：解锁更长生成长度

**来源**：HuggingFace Blog · [文章链接](https://huggingface.co/blog/kv-cache-quantization)

随着 LLM 上下文长度不断增长，KV Cache 显存占用成为主要瓶颈。通过将 Key/Value 向量降精度存储（FP16 → INT8/INT4），在长文本生成场景下大幅降低显存需求，对生成质量影响极小。文章系统介绍了 KV Cache 工作原理、量化参数选择策略，以及不同精度下的质量/速度/内存权衡。

**关键词**：KV Cache · 量化 · 长上下文 · 推理优化

---

## 3. LLM Steering 数据集污染的理解与缓解

**来源**：arXiv 2603.03206 · [论文链接](https://arxiv.org/abs/2603.03206)

首次研究对比引导（Contrastive Steering）对训练数据污染/对抗性修改的鲁棒性。发现适度噪声时相当鲁棒，但恶意篡改超过一定比例时副作用显现。关键贡献：用**鲁棒均值估计器**替换高维均值计算步骤，可大幅缓解恶意污染影响。

**关键词**：LLM 对齐 · 激活引导 · AI 安全 · 鲁棒估计

---

## 4. 无自回归抽样的 LLM 数值预测分布（ICLR 2026）

**来源**：arXiv 2603.02913 · [论文链接](https://arxiv.org/abs/2603.02913)

训练轻量「回归探针」直接从 LLM 内部表示预测统计泛函（均值、分位数等），无需重复采样。LLM 嵌入中蕴含丰富的数值不确定性信号，为无采样不确定性感知数值预测提供新思路。

**关键词**：不确定性估计 · 分布预测 · 内部表示 · 无采样推理

---

## 5. Phi-4-Reasoning：多模态推理模型的训练教训

**来源**：Microsoft Research Blog · [文章链接](https://www.microsoft.com/en-us/research/blog/phi-4-reasoning-vision-and-the-lessons-of-training-a-multimodal-reasoning-model/)

深入探讨如何在小参数量模型上实现高质量多模态推理：合成数据配方设计、视觉与语言推理协同优化、RLHF 奖励建模挑战、多模态 CoT 训练技巧，以及视觉信息与文字推理的对齐难题。

**关键词**：多模态推理 · Phi-4 · RLHF · CoT · 合成数据

---

## 6. Teaching LLMs to Reason Like Bayesians

**来源**：Google Research Blog · [文章链接](https://research.google/blog/teaching-llms-to-reason-like-bayesians/)

训练 LLM 进行贝叶斯式概率推理，将先验知识注入训练目标，在 Chain-of-Thought 中引入分布式推理步骤，改善科学问答与数学推理中的不确定性处理。

**关键词**：贝叶斯推理 · 不确定性 · LLM 推理 · 概率建模

---

*iWiki 技术速递：[https://iwiki.woa.com/p/4018503088](https://iwiki.woa.com/p/4018503088)*
