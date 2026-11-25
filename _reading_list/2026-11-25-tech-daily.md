---
layout: reading
title: "2026-11-25 技术速递（7篇）"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-11-25
---

# 📰 2026-11-25 · 每日技术速递

> 今日精选 7 篇技术文章，覆盖 大模型生态 · 深度学习框架 · 自然语言处理。

---

## 1. SkillPrivus: Uncovering PRIVacy of USe Harms in Agentic Skills

**来源**：cs.CL · 自然语言处理
**链接**：https://arxiv.org/abs/2611.26747

Agent skills extend AI assistants through installable instructions and code that can access users' files, accounts, and data. Even when performing their advertised tasks, skills may infer sensitive attributes, retain unnecessary information, or disclose data to unexpected recipients. We study these potential privacy-of-use harms through SkillPrivus, a static audit framework that analyzes skills before installation without executing them. SkillPrivus extracts data operations from code and natural

**核心要点**：
- Agent skills extend AI assistants through installable instructions and code that can access users' files, accounts, and data
- Even when performing their advertised tasks, skills may infer sensitive attributes, retain unnecessary information, or disclose data to unexpected recipients
- We study these potential privacy-of-use harms through SkillPrivus, a static audit framework that analyzes skills before installation without executing them

## 2. ExecuTorch-TensorRT: PyTorch Robot Policies at NVIDIA TensorRT Speed

**来源**：PyTorch · 深度学习框架
**链接**：https://pytorch.org/blog/executorch-tensorrt-pytorch-robot-policies-at-nvidia-tensorrt-speed/

TL;DR With Torch-TensorRT 2.15 and ExecuTorch 1.6 you can compile a PyTorch model with TensorRT, save it as an ExecuTorch program, and run it from C++ or Python without loading... The post ExecuTorch-TensorRT: PyTorch Robot Policies at NVIDIA TensorRT Speed appeared first on PyTorch.

**核心要点**：
- TL;DR With Torch-TensorRT 2.15 and ExecuTorch 1.6 you can compile a PyTorch model with TensorRT, save it as an ExecuTorch program, and run it from C++ or Python without loading..
- The post ExecuTorch-TensorRT: PyTorch Robot Policies at NVIDIA TensorRT Speed appeared first on PyTorch.

## 3. Microduck Obstacle Avoidance: A Sim2Real Workflow with Isaac Lab and Newton

**来源**：HuggingFace · 大模型生态
**链接**：https://huggingface.co/blog/nvidia/microduck-obstacle-avoidance

（详见原文链接）

**核心要点**：
- 来自 HuggingFace 的最新研究成果，关注 大模型生态 方向

## 4. Reinforcement Learning Unlearns Corrupted SFT Reasoning

**来源**：cs.CL · 自然语言处理
**链接**：https://arxiv.org/abs/2611.26756

We show that reinforcement learning (RL) in reasoning tasks can succeed even when the supervised fine-tuning (SFT) data is severely corrupted. In a mathematical reasoning dataset (GSM8K), we systematically corrupt training trajectories across 361 configurations by injecting realistic error types, varying error locations, trace-answer consistency, and corruption ratios up to 100%. While aggregated SFT-only performance steadily deteriorates under corruption (dropping from 43.61% to 13.80%), Group 

**核心要点**：
- We show that reinforcement learning (RL) in reasoning tasks can succeed even when the supervised fine-tuning (SFT) data is severely corrupted
- In a mathematical reasoning dataset (GSM8K), we systematically corrupt training trajectories across 361 configurations by injecting realistic error types, varying error locations, trace-answer consistency, and corruption ratios up to 100%
- While aggregated SFT-only performance steadily deteriorates under corruption (dropping from 43.61% to 13.80%), Group Relative Policy Optimization (GRPO) post-training consistently restores model accuracy back to gold-baseline levels (approximately 60%)

## 5. Celebrating the 2026 PyTorch Foundation Contributor Award Winners and Nominees

**来源**：PyTorch · 深度学习框架
**链接**：https://pytorch.org/blog/2026-pytorch-foundation-contributor-awards/

The PyTorch Foundation is proud to celebrate the winners and nominees of the fourth annual PyTorch Foundation Contributor Awards! Presented on the keynote stage at PyTorch Conference 2026 in San... The post Celebrating the 2026 PyTorch Foundation Contributor Award Winners and Nominees appeared first on PyTorch.

**核心要点**：
- The PyTorch Foundation is proud to celebrate the winners and nominees of the fourth annual PyTorch Foundation Contributor Awards! Presented on the keynote stage at PyTorch Conference 2026 in San..
- The post Celebrating the 2026 PyTorch Foundation Contributor Award Winners and Nominees appeared first on PyTorch.

## 6. RoboTrack: Point tracking for real-world robotic manipulation

**来源**：HuggingFace · 大模型生态
**链接**：https://huggingface.co/blog/allenai/robotrack

（详见原文链接）

**核心要点**：
- 来自 HuggingFace 的最新研究成果，关注 大模型生态 方向

## 7. Know Your Context Bank: Repairing Inconsistency and Measuring Generalization in Context-Aware Text-to-SQL

**来源**：cs.CL · 自然语言处理
**链接**：https://arxiv.org/abs/2611.26775

Text-to-SQL systems increasingly rely on context banks: curated collections of example queries, metric definitions, and domain knowledge that tell a model how an organization writes its queries. In the enterprise, a bank grows over years as many analysts add entries, checked one at a time for correctness (does the SQL answer its own question?) and rarely for consistency with one another. Yet correctness is often loosely defined: a question seldom fixes how an entity is counted or which table def

**核心要点**：
- Text-to-SQL systems increasingly rely on context banks: curated collections of example queries, metric definitions, and domain knowledge that tell a model how an organization writes its queries
- In the enterprise, a bank grows over years as many analysts add entries, checked one at a time for correctness (does the SQL answer its own question?) and rarely for consistency with one another
- Yet correctness is often loosely defined: a question seldom fixes how an entity is counted or which table defines it, so two entries can each pass review yet resolve the same decision in opposite ways

---

## 📊 今日速览

| # | 标题 | 来源 | 方向 |
|---|------|------|------|
| 1 | SkillPrivus: Uncovering PRIVacy of | cs.CL | 自然语言处理 |
| 2 | ExecuTorch-TensorRT: PyTorch Robot | PyTorch | 深度学习框架 |
| 3 | Microduck Obstacle Avoidance: A Si | HuggingFace | 大模型生态 |
| 4 | Reinforcement Learning Unlearns Co | cs.CL | 自然语言处理 |
| 5 | Celebrating the 2026 PyTorch Found | PyTorch | 深度学习框架 |
| 6 | RoboTrack: Point tracking for real | HuggingFace | 大模型生态 |
| 7 | Know Your Context Bank: Repairing  | cs.CL | 自然语言处理 |

---

*自动生成 · 2026-11-25 · jeffinchen daily tech reading list*

