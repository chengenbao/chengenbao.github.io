---
layout: reading
title: "技术速递 2026-04-22：LLM 推理加速 / Test-Time Scaling / 训练优化"
category: tech
tags: [Tech, arXiv, LLM, 推理加速, RLHF]
date: 2026-04-22
---

今日精选 6 篇 arXiv 深度技术论文，涵盖 LLM 推理加速（投机解码）、奖励引导解码、Test-Time Scaling、预训练数据混合策略、GRPO 后训练及训练显存优化。

## 今日文章

### 1. 跨家族投机解码在 Apple Silicon 上的实证研究
**[Cross-Family Speculative Decoding for Polish Language Models on Apple Silicon](https://arxiv.org/abs/2604.16368)**  
投机解码加速 LLM 推理：草稿模型提案 token + 目标模型验证。聚焦跨架构家族组合在 Apple Silicon CPU/GPU 统一内存下的推理延迟与吞吐量实测。

### 2. 无训练奖励引导解码：SMC 框架
**[Sampling for Quality: Training-Free Reward-Guided LLM Decoding via Sequential Monte Carlo](https://arxiv.org/abs/2604.16453)**  
基于序列蒙特卡洛的无训练奖励引导解码框架，推理阶段直接将奖励模型融入采样权重，无需微调即可提升输出质量。

### 3. SCATR：校准良好的 Test-Time 排序
**[SCATR: Simple Calibrated Test-Time Ranking](https://arxiv.org/abs/2604.16535)**  
针对 Best-of-N 中评分函数校准问题，提出简单高效的 test-time 候选排序方法，极低额外计算开销显著提升数学推理准确率。

### 4. LLM 预训练数据混合策略综述
**[Data Mixing for Large Language Models Pretraining: A Survey and Outlook](https://arxiv.org/abs/2604.16380)**  
系统综述训练数据配比对 LLM 能力的决定性影响，涵盖静态/动态混合、Proxy 实验预测、多语言代码混合比例研究。

### 5. S-GRPO：VLM 统一后训练框架
**[S-GRPO: Unified Post-Training for Large Vision-Language Models](https://arxiv.org/abs/2604.16557)**  
融合 GRPO 强化学习与 SFT 的统一 LVLM 后训练范式，在多模态推理与指令跟随任务同时获益，避免两阶段训练信号冲突。

### 6. BASIS：显存高效的激活压缩反向传播
**[BASIS: Balanced Activation Sketching with Invariant Scalars for Ghost Backpropagation](https://arxiv.org/abs/2604.16324)**  
通过低秩激活压缩（sketching）显著降低大模型训练显存占用，引入不变标量保证梯度方向无偏性，兼容 Activation Checkpointing。

---

📖 [iWiki 完整版](https://iwiki.woa.com/p/4019984973)
