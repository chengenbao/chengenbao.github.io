---
layout: reading
title: "技术速递 2026-03-31：注意力机制优化、推理加速与二阶优化器进展"
category: tech
tags: [Tech, arXiv, 前沿, Attention, 推理加速, 优化器]
date: 2026-03-31
---

今日精选 6 篇来自 arXiv 的深度技术论文，聚焦大模型注意力机制改进、扩散模型推理加速、BitNet 量化训练、二阶优化器、KV Cache 压缩以及多智能体协作效率。

## 精选文章

### 1. [Switch Attention: Towards Dynamic and Fine-grained Hybrid Transformers](https://arxiv.org/abs/2603.26380)

**来源**：arXiv cs.CL | **标签**：Attention, Transformer, 推理加速, 架构优化

提出 Switch Attention 机制，动态切换全注意力与局部/稀疏注意力，在保持模型表达力的同时将注意力计算复杂度从 O(n²) 降至近线性。在 LLM 推理加速方向具有重要意义。

---

### 2. [DRiffusion: Draft-and-Refine Process Parallelizes Diffusion Models with Ease](https://arxiv.org/abs/2603.25872)

**来源**：arXiv cs.LG | **标签**：Diffusion, 推理加速, 并行化, 生成模型

借鉴 Speculative Decoding 思想，提出 Draft-and-Refine 流程将扩散模型的迭代采样步骤并行化，显著降低生成延迟。类比 vLLM 对 LLM 推理的加速思路，迁移到扩散模型领域。

---

### 3. [MAGNET: Autonomous Expert Model Generation via Decentralized Autoresearch and BitNet Training](https://arxiv.org/abs/2603.25813)

**来源**：arXiv cs.LG | **标签**：BitNet, 量化训练, MoE, 自动化训练

提出去中心化的自动化系统，自主生成、训练并部署领域专家语言模型，结合 BitNet 1-bit 量化训练方案，大幅降低专家模型的训练与部署成本。

---

### 4. [Second-Order, First-Class: A Composable Stack for Curvature-Aware Training](https://arxiv.org/abs/2603.25976)

**来源**：arXiv cs.LG | **标签**：优化器, 二阶方法, 训练加速, K-FAC

系统性解决二阶优化方法在深度学习中实现复杂、调参困难的痛点，提出可组合的曲率感知训练框架，使 K-FAC、Shampoo 等二阶优化器像 Adam 一样易用，加速大模型训练收敛。

---

### 5. [Density-aware Soft Context Compression with Semi-Dynamic Compression Ratio](https://arxiv.org/abs/2603.25926)

**来源**：arXiv cs.CL | **标签**：KV Cache, 上下文压缩, 长上下文, 推理优化

针对 LLM 长上下文处理的 KV Cache 压缩问题，提出密度感知的软压缩方法，根据上下文信息密度动态调整压缩率，在显著减少计算量的同时维持性能。

---

### 6. [AgentCollab: A Self-Evaluation-Driven Collaboration Paradigm for Efficient LLM Agents](https://arxiv.org/abs/2603.26034)

**来源**：arXiv cs.CL | **标签**：LLM Agents, 多智能体, 推理效率, 自评估

提出基于自评估的 LLM 多智能体协作范式，通过智能体间的动态分工与自我评估机制提升复杂任务完成效率，减少冗余推理调用，降低整体 Token 消耗。

---

*来源：arXiv cs.LG / cs.CL | 更新于 2026-03-31*
