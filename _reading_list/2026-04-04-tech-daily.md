---
layout: reading
title: "技术速递 2026-04-04：Test-Time Scaling · GPU Kernel 自动生成 · 扩散模型加速"
category: tech
tags: [Tech, arXiv, LLM, 推理优化, CUDA, 后训练, 扩散模型]
date: 2026-04-04
---

本期技术速递聚焦 **LLM 推理扩展计算策略**、**LLM 驱动的 GPU Kernel 自动生成**、**扩散模型加速**及**后训练优化算法**，均来自 arXiv 最新论文。

## 1. [Test-Time Scaling Makes Overtraining Compute-Optimal](https://arxiv.org/abs/2604.01411)

**来源：** arXiv cs.LG　**标签：** `推理扩展 · 计算优化`

Modern LLMs scale at test-time, e.g. via repeated sampling, where inference FLOPs can be traded for higher accuracy. This paper shows that test-time scaling fundamentally changes the optimal training strategy: models benefit from overtraining beyond the Chinchilla-optimal point when test-time compute is available, creating a new compute-optimal frontier that balances training and inference budgets.

---

## 2. [CuTeGen: An LLM-Based Agentic Framework for Generation and Optimization of High-Performance GPU Kernels](https://arxiv.org/abs/2604.01489)

**来源：** arXiv cs.LG　**标签：** `GPU内核生成 · CUDA优化`

High-performance GPU kernels are critical to modern ML systems. CuTeGen leverages LLM agents to automatically generate and iteratively optimize CUDA kernels using CuTe/CUTLASS abstractions, achieving competitive performance against hand-tuned implementations and demonstrating a new paradigm for hardware-aware code synthesis.

---

## 3. [Matching Accuracy, Different Geometry: Evolution Strategies vs GRPO in LLM Post-Training](https://arxiv.org/abs/2604.01499)

**来源：** arXiv cs.LG　**标签：** `后训练 · RLHF · 优化器`

Evolution Strategies (ES) have emerged as a scalable gradient-free alternative to GRPO for LLM post-training. This paper systematically compares the two approaches, showing they achieve similar accuracy but follow very different optimization trajectories in parameter space, offering new insights into the geometry of RLHF-style training.

---

## 4. [ZEUS: Accelerating Diffusion Models with Only Second-Order Predictor](https://arxiv.org/abs/2604.01552)

**来源：** arXiv cs.LG　**标签：** `扩散模型加速 · 推理优化`

Diffusion models deliver high-fidelity generation but suffer from slow inference due to iterative denoising. ZEUS introduces a second-order predictor that dramatically reduces the number of function evaluations needed, achieving state-of-the-art sample quality with far fewer steps than existing ODE solvers, applicable to both image and video generation.

---

## 5. [LinearARD: Linear-Memory Attention Distillation for RoPE Restoration](https://arxiv.org/abs/2604.00004)

**来源：** arXiv cs.CL　**标签：** `线性注意力 · RoPE · 长上下文`

Extending context windows in LLMs typically requires expensive positional encoding adjustments. LinearARD proposes distilling standard RoPE attention into a linear-memory recurrent form that preserves long-range dependency modeling, enabling efficient long-context inference with O(1) memory per step while restoring the full attention capability through a novel training objective.

---

## 6. [Learning from the Right Rollouts: Data Attribution for PPO-based LLM Post-Training](https://arxiv.org/abs/2604.01597)

**来源：** arXiv cs.LG　**标签：** `PPO · 数据归因 · 后训练`

In PPO-based LLM fine-tuning, not all rollout trajectories contribute equally to learning. This paper introduces data attribution methods to identify high-quality rollouts that drive policy improvement, showing that selectively weighting or filtering training data based on attribution scores leads to significantly better alignment outcomes with fewer samples.

---

