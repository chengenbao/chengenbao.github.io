---
layout: reading
title: "技术速递 2026-03-15：大模型推理加速、序列并行与 MoE 架构前沿"
category: tech
tags: [Tech, 多源, 前沿, 推理加速, MoE, 序列并行, RLHF, 稀疏注意力]
date: 2026-03-15
---

## 本期导读

聚焦大模型训练推理优化、序列并行扩展与后训练对齐三大方向，来自 arXiv 最新论文与 HuggingFace Blog 实践报告。

---

### 1. IndexCache：跨层索引复用加速稀疏注意力
**来源：** arXiv 2603.12201  
**链接：** <https://arxiv.org/abs/2603.12201>

长上下文 Agentic 工作流已成为大模型核心场景。IndexCache 发现相邻 Transformer 层的稀疏注意力索引高度相似，提出**跨层索引复用**机制，用轻量 Lightning Indexer 选出 top-k 相关 token 后在相邻层间共享该索引，同时引入自适应缓存策略动态决定复用还是刷新，大幅削减索引计算开销，推理吞吐显著提升。

---

### 2. 超越 Cross-Entropy：能量基特征匹配微调
**来源：** arXiv 2603.12248  
**链接：** <https://arxiv.org/abs/2603.12248>

CE 训练在 teacher forcing 下优化 next-token prediction，与模型实际 rollout 分布存在本质差距。本文提出**特征匹配微调目标**，对 completion 分布的序列级统计量进行对齐，提供无需奖励模型的稠密语义反馈，序列级生成质量显著提升。

---

### 3. Reasoning LLMs-as-Judges 在 Post-Training 中的有效性
**来源：** arXiv 2603.12246  
**链接：** <https://arxiv.org/abs/2603.12246>

系统研究发现：尽管 Reasoning Judge 在静态 benchmark 上表现更优，在实际策略训练中**并不总是优于普通 LLM Judge**，为 RLHF/RLAIF pipeline 设计提供重要参考。

---

### 4. Ulysses 序列并行：百万 Token 上下文的分布式训练
**来源：** HuggingFace Blog  
**链接：** <https://huggingface.co/blog/ulysses-sp>

**Ulysses Sequence Parallelism** 将序列维度切分到多张 GPU，通过高效的 all-to-all 通信并行计算注意力，与 Tensor Parallelism 正交可叠加，支持百万级 token 上下文训练。

---

### 5. 16 个开源 RL 框架实践：如何让 GPU 不闲置
**来源：** HuggingFace Blog  
**链接：** <https://huggingface.co/blog/async-rl-training-landscape>

调研 16 个开源 RL 训练框架，聚焦异步 RL 训练中 GPU 利用率低问题（通常不足 40%），梳理 rollout-generation 与 training 的解耦方式、同步/异步策略权衡、KV Cache 复用等优化手段。

---

### 6. MoE in Transformers：混合专家架构深度解析
**来源：** HuggingFace Blog  
**链接：** <https://huggingface.co/blog/moe-transformers>

系统梳理 MoE 在 Transformer 中的应用，覆盖专家路由算法、负载均衡、专家并行通信（EP）、推理时专家卸载等关键技术，分析 MoE 相比 Dense 模型的 FLOP 效率优势。
