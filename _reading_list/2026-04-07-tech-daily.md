---
layout: reading
title: "技术速递 2026-04-07：MoE-PEFT、过程奖励推理与 WebGPU LLM 性能剖析"
category: tech
tags: [Tech, arXiv, LLM, MoE, Inference, RL, Diffusion]
date: 2026-04-07
---

今日精选来自 arXiv cs.LG / cs.CL 的 6 篇前沿论文，聚焦大模型训练效率、推理优化与系统性能。

## 1. LiME：轻量级 MoE × PEFT 多任务学习

将 Mixture of Experts 与参数高效微调结合，在多模态多任务场景下自适应路由，参数量大幅压缩。  
🔗 [arXiv 2604.02338](https://arxiv.org/abs/2604.02338)

## 2. 基于过程奖励的 LLM 推理

引入 Process Reward Model（PRM）对每一步推理打分，减少幻觉步骤，数学推理精度显著提升。  
🔗 [arXiv 2604.02341](https://arxiv.org/abs/2604.02341)

## 3. 前沿模型真的需要来验证数学证明吗？

探讨 inference-time compute 的合理分配：专门化小模型 vs 通用大模型的验证能力比较。  
🔗 [arXiv 2604.02450](https://arxiv.org/abs/2604.02450)

## 4. WebGPU 调度开销：跨 GPU 厂商 LLM 推理性能实测

对 NVIDIA、AMD、Intel、Apple 四大厂商进行系统化 dispatch 延迟测量，为 Web 端 LLM 部署提供优化依据。  
🔗 [arXiv 2604.02344](https://arxiv.org/abs/2604.02344)

## 5. RL 驱动的知识蒸馏：LLM-as-a-Judge

以大模型为 Judge 提供 RL 奖励信号，学生模型推理能力超越同参数量 SFT 蒸馏结果。  
🔗 [arXiv 2604.02621](https://arxiv.org/abs/2604.02621)

## 6. Masked Diffusion 模型推理加速

发现去噪步骤贡献差异悬殊，自适应调度策略使 MDLM 生成速度提升 2-3×。  
🔗 [arXiv 2604.02340](https://arxiv.org/abs/2604.02340)
