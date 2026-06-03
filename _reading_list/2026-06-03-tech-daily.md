---
layout: reading
title: "2026-06-03 每日技术速递 · cs.CL / cs.LG / cs.AR"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-06-03
---

# 📰 2026-06-03 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 cs.CL / cs.LG / cs.AR 等方向。

---

## 1. OpenEye: A Scalable Open-Source Hardware Accelerator for DNNs

**来源**：cs.AR  
**链接**：https://arxiv.org/abs/2606.01450  
**标签**：推理加速 · 高效推理

arXiv:2606.01450v1 Announce Type: new Abstract: The increasing computational complexity of deep neural network inference poses significant challenges for efficient hardware acceleration on embedded platforms, particularly with respect to resource consumption and scalability. This work presents OpenEye, a scalable and sparsity-aware FPGA-based hardware accelerator designed to efficiently execute co

**核心要点**：
- 01450v1 Announce Type: new Abstract: The increasing computational complexity of deep neural network inference poses sign
- This work presents OpenEye, a scalable and sparsity-aware FPGA-based hardware accelerator designed to efficiently execut
- OpenEye is based on a highly parameterizable architecture composed of clusters of processing elements interconnected by

---
## 2. CSRP: Chain-of-Thought Reasoning for Chinese Text Correction via Reinforcement Learning with Efficiency-Aware Rewards

**来源**：cs.CL  
**链接**：https://arxiv.org/abs/2606.00020  
**标签**：微调 · 模型训练 · 推理能力 · LLM

arXiv:2606.00020v1 Announce Type: new Abstract: Large Language Model (LLM) based Chinese Grammatical Error Correction (CGEC) systems face two critical challenges: general-purpose models lack specialized linguistic priors for subtle grammatical distinctions, and Supervised Fine-Tuning (SFT) with Maximum Likelihood Estimation fails to optimize for precision-focused metrics, leading to systematic ove

**核心要点**：
- 00020v1 Announce Type: new Abstract: Large Language Model (LLM) based Chinese Grammatical Error Correction (CGEC) system
- We propose CSRP, a three-stage framework that progressively builds correction capability through Continual Pre-training 
- 9M balanced samples to internalize domain knowledge, Ch

---
## 3. DLLM-JEPA: Joint Embedding Predictive Architectures for Masked Diffusion Language Models

**来源**：cs.CL  
**链接**：https://arxiv.org/abs/2606.00091  
**标签**：Attention · LLM · Embedding

arXiv:2606.00091v1 Announce Type: new Abstract: Joint Embedding Predictive Architectures (JEPAs) have reshaped self-supervised representation learning in vision. The recent LLM-JEPA ported JEPA to autoregressive language models but inherited two steep costs from the causal-attention substrate: it demands explicit multi-view data (e.g., text-code pairs), and it requires two gradient-carrying forwar

**核心要点**：
- 00091v1 Announce Type: new Abstract: Joint Embedding Predictive Architectures (JEPAs) have reshaped self-supervised repr
- The recent LLM-JEPA ported JEPA to autoregressive language models but inherited two steep costs from the causal-attentio
- , text-code pairs), and it requires two gradient-carrying forward passes per step

---
## 4. RAFT: Data Refinement and Adaptive Distillation for Domain Fine-Tuning with Alleviated Forgetting

**来源**：cs.LG  
**链接**：https://arxiv.org/abs/2606.00147  
**标签**：知识蒸馏 · 微调 · 模型训练 · 推理能力

arXiv:2606.00147v1 Announce Type: new Abstract: Domain-specific supervised fine-tuning (SFT) often improves in-domain performance at the cost of degrading a model's general capabilities. We view this degradation through two practical gaps in domain SFT: a supervision-compatibility gap, where domain targets differ in style and reasoning format from the original model's natural responses, and a traj

**核心要点**：
- 00147v1 Announce Type: new Abstract: Domain-specific supervised fine-tuning (SFT) often improves in-domain performance a
- We view this degradation through two practical gaps in domain SFT: a supervision-compatibility gap, where domain targets
- This process fails to preserve the model's orig

---
## 5. Toward Robust In-Context Learning: Leveraging Out-of-distribution Proxies for Target Inaccessible Demonstration Retrieval

**来源**：cs.CL  
**链接**：https://arxiv.org/abs/2606.00014  
**标签**：推理加速 · LLM · RAG

arXiv:2606.00014v1 Announce Type: new Abstract: Although studies have demonstrated that Large Language Models (LLMs) can perform well on Out-of-Distribution (OOD) tasks, their advantage tends to diminish as the distribution shift becomes more severe. Consequently, researchers aim to retrieve distributionally similar and informative demonstrations from the available source domain to boost the infer

**核心要点**：
- 00014v1 Announce Type: new Abstract: Although studies have demonstrated that Large Language Models (LLMs) can perform we
- Consequently, researchers aim to retrieve distributionally similar and informative demonstrations from the available sou
- However, in practical scenarios where the target domain is inaccessible, evaluating the unknown distribution is challeng

---
## 6. AEyeDE: An Attention-Based Attribution Framework for AI-Generated Text Detection

**来源**：cs.CL  
**链接**：https://arxiv.org/abs/2606.00016  
**标签**：Attention · Transformer · RAG

arXiv:2606.00016v1 Announce Type: new Abstract: Detecting AI-generated text is becoming increasingly challenging as modern language models approach human-level fluency and can evade detectors that rely on surface statistics or likelihood-based signals. We propose \textsc{AEyeDE}, an attribution-driven approach to human-AI authorship detection that leverages model attention as a discriminative sign

**核心要点**：
- 00016v1 Announce Type: new Abstract: Detecting AI-generated text is becoming increasingly challenging as modern language
- We propose \textsc{AEyeDE}, an attribution-driven approach to human-AI authorship detection that leverages model attenti
- Specifically, we extract attention-based attribution matrices for both human- and AI-generated text using a \emph{proxy}

---
## 7. Model-Based Quality Assessment for Massively Multilingual Parallel Data

**来源**：cs.CL  
**链接**：https://arxiv.org/abs/2606.00285  
**标签**：并行训练 · Benchmark · Embedding

arXiv:2606.00285v1 Announce Type: new Abstract: Large-scale multilingual bitext often contains two distinct problems: non-parallel sentence pairs and low-quality translations. We decompose model-based assessment for such data into two independent components: parallelism assessment with multilingual embeddings and reference-free quality estimation (QE). For parallelism, we benchmark four embedding 

**核心要点**：
- 00285v1 Announce Type: new Abstract: Large-scale multilingual bitext often contains two distinct problems: non-parallel 
- We decompose model-based assessment for such data into two independent components: parallelism assessment with multiling
- For parallelism, we benchmark four embedding models on FLORES-200 and BOUQuET retrieval tasks, covering 6,654 source--ta

---


## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | OpenEye: A Scalable Open-Sourc... | cs.AR | 推理加速 |
| 2 | CSRP: Chain-of-Thought Reasoni... | cs.CL | 大模型研究 |
| 3 | DLLM-JEPA: Joint Embedding Pre... | cs.CL | 大模型研究 |
| 4 | RAFT: Data Refinement and Adap... | cs.LG | 大模型研究 |
| 5 | Toward Robust In-Context Learn... | cs.CL | 推理加速 |
| 6 | AEyeDE: An Attention-Based Att... | cs.CL | 大模型研究 |
| 7 | Model-Based Quality Assessment... | cs.CL | 大模型研究 |

---

*自动生成 · 2026-06-03 · jeffinchen daily tech reading list*

