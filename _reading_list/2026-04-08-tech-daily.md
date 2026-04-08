---
layout: reading
title: "技术速递 2026-04-08：LLM压缩、KV Cache注入与注意力机制优化"
category: tech
tags: [Tech, arXiv, LLM, 推理加速, KV-Cache, 注意力机制, 量化压缩]
date: 2026-04-08
---

本期技术速递精选 6 篇 arXiv 最新论文，聚焦 LLM 压缩部署、KV Cache 优化、注意力机制改进与长上下文建模。

---

## 1. [SoLA: Leveraging Soft Activation Sparsity and Low-Rank Decomposition for LLM Compression](https://arxiv.org/abs/2604.03258)

**来源：** arXiv cs.CL | **标签：** `LLM` `量化压缩` `推理加速` `模型部署`

大规模语言模型的十亿级参数带来了部署挑战。SoLA 结合软激活稀疏性与低秩分解，在保持模型质量的同时大幅压缩 LLM，为边缘推理部署提供新思路。

---

## 2. [Knowledge Packs: Zero-Token Knowledge Delivery via KV Cache Injection](https://arxiv.org/abs/2604.03270)

**来源：** arXiv cs.CL | **标签：** `KV Cache` `RAG` `推理优化` `Transformer`

RAG 会浪费大量 token。Knowledge Packs 提出预计算 KV Cache 的方式，在零 token 代价下注入知识，彻底改变传统 RAG 的信息传递路径。对 KV Cache 优化和推理效率有重要意义。

---

## 3. [Why Attend to Everything? Focus is the Key](https://arxiv.org/abs/2604.03260)

**来源：** arXiv cs.CL | **标签：** `注意力机制` `稀疏注意力` `Transformer架构` `推理加速`

提出 Focus 方法——通过可学习质心将 token 分组，只计算关键 token pair 的注意力，而非近似所有。这是对注意力机制本身的结构性优化，与稀疏注意力方向高度相关。

---

## 4. [LPC-SM: Local Predictive Coding and Sparse Memory for Long-Context Language Modeling](https://arxiv.org/abs/2604.03263)

**来源：** arXiv cs.CL | **标签：** `长上下文` `稀疏记忆` `LLM架构` `序列建模`

当前长上下文模型仍用注意力处理本地和长程状态。LPC-SM 将局部预测编码与稀疏记忆结合，在保持长程理解的同时显著降低计算开销，是长上下文建模的新架构探索。

---

## 5. [On the Geometric Structure of Layer Updates in Deep Language Models](https://arxiv.org/abs/2604.02459)

**来源：** arXiv cs.LG | **标签：** `LLM训练` `几何分析` `低秩结构` `优化器`

研究深层语言模型中层更新的几何结构。从低秩结构、曲率、流形视角理解层更新动态，为训练优化和模型理解提供新框架。

---

## 6. [Self-Execution Simulation Improves Coding Models](https://arxiv.org/abs/2604.03253)

**来源：** arXiv cs.CL | **标签：** `代码生成` `LLM推理` `程序合成` `代码模型`

通过让 LLM 模拟程序执行过程（自执行模拟），使其能更准确地估计代码运行结果，显著提升代码生成正确率。这是将模型内部推理与代码执行语义对齐的重要进展。
