---
layout: reading
title: "技术速递 2026-04-25：超网络弹性推理、语义缓存、扩散模型量化、推理早停与长期记忆"
category: tech
tags: [Tech, arXiv, 前沿, 大模型, 推理加速]
date: 2026-04-25
---

本期精选 6 篇来自 arXiv cs.LG / cs.CL 的深度技术论文，涵盖大模型架构、推理加速、模型量化、长期记忆等核心方向。

---

## 1. Super Apriel: One Checkpoint, Many Speeds

**来源**：arXiv cs.LG · [2604.19877](https://arxiv.org/abs/2604.19877)

提出 **Super Apriel**，一个 15B 参数的超网络（supernet），每个解码器层可在全注意力（FA）、滑动窗口注意力、线性注意力和 MLP-Mixer 四种混合器之间动态切换。通过弹性子网络采样，同一个检查点可在不同算力预算下直接复用，无需重新训练。对多场景部署具有重要意义，显著降低多场景维护成本。

**🏷️** `MoE` `注意力机制` `推理加速` `模型压缩`

---

## 2. Continuous Semantic Caching for Low-Cost LLM Serving

**来源**：arXiv cs.LG · [2604.20021](https://arxiv.org/abs/2604.20021)

提出**连续语义缓存（CSC）**方案：利用向量相似度检索语义等价的历史请求，复用已计算的响应，替代传统精确字符匹配的 KV Cache。支持流式输出场景下的渐进缓存更新。在高并发 LLM 服务场景中，缓存命中率提升至传统方案的数倍，有效降低 GPU 计算负载。

**🏷️** `KV Cache` `LLM 推理` `语义缓存` `低成本服务`

---

## 3. On the Quantization Robustness of Diffusion Language Models in Coding Benchmarks

**来源**：arXiv cs.LG · [2604.20079](https://arxiv.org/abs/2604.20079)

系统评估扩散语言模型（DLM）在 INT4/INT8 量化下的代码生成鲁棒性，与自回归 LLM 的量化行为横向对比。揭示 DLM 独特的量化敏感性分布，为边缘设备部署扩散语言模型提供量化策略实证依据。

**🏷️** `模型量化` `扩散语言模型` `代码生成` `推理优化`

---

## 4. TRACES: Tagging Reasoning Steps for Adaptive Cost-Efficient Early-Stopping

**来源**：arXiv cs.CL · [2604.21057](https://arxiv.org/abs/2604.21057)

针对语言推理模型（LRM）的过度思考（overthinking）问题，提出 TRACES 框架：为推理链中的每个中间步骤打置信度标签，当累积置信度达到阈值时触发自适应早停。在保持精度的同时将推理 token 消耗降低 **30%-50%**。

**🏷️** `推理加速` `早停` `推理效率` `LRM`

---

## 5. EngramaBench: Evaluating Long-Term Conversational Memory with Structured Graph Retrieval

**来源**：arXiv cs.CL · [2604.21229](https://arxiv.org/abs/2604.21229)

构建 EngramaBench 基准，专门评估 LLM 助手跨多轮对话的长期记忆能力。引入结构化图检索机制，将对话历史组织为实体关系图，在多跳推理和时序理解任务上超越基于向量检索（RAG）的方法。

**🏷️** `长期记忆` `图检索` `RAG` `多轮对话`

---

## 6. Differentiable Conformal Training for LLM Reasoning Factuality

**来源**：arXiv cs.LG · [2604.20098](https://arxiv.org/abs/2604.20098)

将保角预测（Conformal Prediction）引入 LLM 训练过程，提出可微分保角训练（DCT）目标。在训练阶段直接优化覆盖率保证约束，相比推理时后处理方案，在分布外（OOD）场景下更鲁棒，LLM 幻觉率显著下降。

**🏷️** `幻觉缓解` `训练优化` `LLM 可靠性` `不确定性`
