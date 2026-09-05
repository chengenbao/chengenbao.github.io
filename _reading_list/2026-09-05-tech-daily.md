---
layout: reading
title: "大模型训练、推理加速与系统优化前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-09-05
---

# 📰 2026-09-05 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 大模型训练 / 推理加速 / 编译器 / GPU / 分布式系统。

---

## 1. The Geometry of Ignorance: LLMs Know When to Temper Bayesian Priors

**来源**：cs.LG
**链接**：https://arxiv.org/abs/2609.02959
**标签**：embedding

arXiv:2609.02959v1 Announce Type: new Abstract: What does a language model predict when it has few clues? The answer lurks in its unembedding geometry: a single direction of the unembedding matrix encodes the unigram distribution of the training corpus, which serves as the Bayesian prior the model falls back on when uncertain. This structure --- which we ter

**核心要点**：
- arXiv:2609.02959v1 Announce Type: new Abstract: What does a language model predict when it has few clues? The answer lurks in its unembeddin
- This structure --- which we term the \emph{direction of ignorance} --- appears in all four model families examined (\texttt{Llama}, \texttt{
- Projecting the final prediction state onto this

---

## 2. Equation Recast for Canonical Operator Learning Across Parametric PDEs

**来源**：cs.LG
**链接**：https://arxiv.org/abs/2609.02982
**标签**：前沿 · 技术 · 研究

arXiv:2609.02982v1 Announce Type: new Abstract: Learning solution operators across broad parameter ranges can require substantial coverage of both input functions and physical parameters, particularly for purely data-driven parametric models. In addition, the resulting models may fail silently outside the training distribution. We introduce equation recast, 

**核心要点**：
- arXiv:2609.02982v1 Announce Type: new Abstract: Learning solution operators across broad parameter ranges can require substantial coverage o
- In addition, the resulting models may fail silently outside the training distribution.
- We introduce equation recast, which reformulates parametric operator learning as the learning of a single canonical operator.

---

## 3. Modern Transformers Are Implicit Hybrids: From Functional Differentiation to Principled Hybrid Architecture Design

**来源**：cs.LG
**链接**：https://arxiv.org/abs/2609.02986
**标签**：Transformer

arXiv:2609.02986v1 Announce Type: new Abstract: Hybrid architectures combining Full Attention (FA) and Linear Attention (LA) are increasingly prominent, yet their allocation remains heuristic. We seek an evidence-grounded basis in head-level functional organization learned by RoPE-based Transformers. Behavioral probes do not yield a complete taxonomy, so we 

**核心要点**：
- arXiv:2609.02986v1 Announce Type: new Abstract: Hybrid architectures combining Full Attention (FA) and Linear Attention (LA) are increasingl
- We seek an evidence-grounded basis in head-level functional organization learned by RoPE-based Transformers.
- Behavioral probes do not yield a complete taxonomy, so we propose two intervention metrics: RoPE Frequency Importance Score (RFIS), measurin

---

## 4. Tail-Likelihood Reinforcement Learning

**来源**：cs.LG
**链接**：https://arxiv.org/abs/2609.02987
**标签**：前沿 · 技术 · 研究

arXiv:2609.02987v1 Announce Type: new Abstract: Reinforcement learning typically optimizes average reward. For generative policies, the average can hide an important distinction: two policies can achieve the same mean reward while having very different chances of producing a rare but high-reward rollout. This matters as sampling increases during training and

**核心要点**：
- arXiv:2609.02987v1 Announce Type: new Abstract: Reinforcement learning typically optimizes average reward.
- For generative policies, the average can hide an important distinction: two policies can achieve the same mean reward while having very diff
- This matters as sampling increases during training and inference, since its benefit depends on retaining probability mass on high-reward out

---

## 5. Mesh-Native Physics-Informed Graph Surrogates for TCAD-in-the-Loop Design Space Exploration

**来源**：cs.LG
**链接**：https://arxiv.org/abs/2609.02988
**标签**：前沿 · 技术 · 研究

arXiv:2609.02988v1 Announce Type: new Abstract: High-fidelity TCAD simulation of drift-diffusion transport remains the workhorse of emerging FinFET device design, but it is computationally expensive, especially for 3D structures where runtime escalates steeply with mesh complexity. This sharply limits multi-objective design space exploration. Existing machin

**核心要点**：
- arXiv:2609.02988v1 Announce Type: new Abstract: High-fidelity TCAD simulation of drift-diffusion transport remains the workhorse of emerging
- This sharply limits multi-objective design space exploration.
- Existing machine-learning surrogates map a fixed set of design parameters to a few scalar device metrics, discarding the underlying physics 

---

## 6. TRACE: Spatiotemporal Contact Memory Graph Network Simulator for Granular Dynamics

**来源**：cs.LG
**链接**：https://arxiv.org/abs/2609.02991
**标签**：前沿 · 技术 · 研究

arXiv:2609.02991v1 Announce Type: new Abstract: Learned graph simulators provide an efficient alternative to high-fidelity solvers for granular dynamics. However, granular motion depends strongly on inter-granular contact history, which is difficult to preserve when particle contacts form, break, and rearrange. Existing simulators mainly store temporal infor

**核心要点**：
- arXiv:2609.02991v1 Announce Type: new Abstract: Learned graph simulators provide an efficient alternative to high-fidelity solvers for granu
- However, granular motion depends strongly on inter-granular contact history, which is difficult to preserve when particle contacts form, bre
- Existing simulators mainly store temporal information in node features or node-level memory.

---

## 7. No-Regret Bayesian Optimization with Finite-Library Input-Warped Kernels

**来源**：cs.LG
**链接**：https://arxiv.org/abs/2609.02993
**标签**：前沿 · 技术 · 研究

arXiv:2609.02993v1 Announce Type: new Abstract: Gaussian-process Bayesian optimization (GP-BO) excels at black-box optimization of costly functions, e.g., hyperparameter optimization (HPO) and multi-agent system (MAS) design. Convergence-rate guarantees exist for select methods, notably GP upper confidence bound (GP-UCB), but require a fixed kernel. Critical

**核心要点**：
- arXiv:2609.02993v1 Announce Type: new Abstract: Gaussian-process Bayesian optimization (GP-BO) excels at black-box optimization of costly fu
- Convergence-rate guarantees exist for select methods, notably GP upper confidence bound (GP-UCB), but require a fixed kernel.
- Critically, the kernel encodes how input proximity affects objective value similarity.

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | The Geometry of Ignorance: LLMs Know Whe | cs.LG | 机器学习/大模型 |
| 2 | Equation Recast for Canonical Operator L | cs.LG | 机器学习/大模型 |
| 3 | Modern Transformers Are Implicit Hybrids | cs.LG | 机器学习/大模型 |
| 4 | Tail-Likelihood Reinforcement Learning | cs.LG | 机器学习/大模型 |
| 5 | Mesh-Native Physics-Informed Graph Surro | cs.LG | 机器学习/大模型 |
| 6 | TRACE: Spatiotemporal Contact Memory Gra | cs.LG | 机器学习/大模型 |
| 7 | No-Regret Bayesian Optimization with Fin | cs.LG | 机器学习/大模型 |

*自动生成 · 2026-09-05 · jeffinchen daily tech reading list*

