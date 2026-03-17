---
layout: reading
title: "技术速递 2026-03-17：LLM推理加速三路进击 · ActTail稀疏化、vLLM语义路由、激活引导"
category: tech
tags: [Tech, ArXiv, LLM推理, 激活稀疏, vLLM, MoE, 强化学习]
date: 2026-03-17
---

今日精选 5 篇前沿论文，聚焦 LLM 推理加速（ActTail 激活稀疏、vLLM 语义路由器 98× 提速）、激活引导可控生成（跨层一致性约束）、MoE+LoRA 多任务专家系统，以及 RL 课程学习的热力学理论。

## 精选文章

### 1. ActTail：LLM 全局激活稀疏性推理加速
[arxiv.org/abs/2603.12272](https://arxiv.org/abs/2603.12272)

现有方法对所有 Projection 均匀施加稀疏度，忽略各层统计异质性。ActTail 通过激活尾部特征实现全局自适应稀疏，在保持精度的同时减少计算和内存带宽。

### 2. 98× 提速：无专用 GPU 的 vLLM 语义路由器
[arxiv.org/abs/2603.12646](https://arxiv.org/abs/2603.12646)

结合 Flash Attention、Prompt Compression 和近流式处理，实现系统级安全/域路由器 98× 加速，无需独立 GPU。

### 3. Global Evolutionary Steering：跨层一致激活引导
[arxiv.org/abs/2603.12298](https://arxiv.org/abs/2603.12298)

用跨层一致性约束克服激活引导的高维噪声和层间语义漂移问题，无需微调即可精确控制 LLM 输出。

### 4. Expert Pyramid Tuning：MoE+LoRA 多任务专家系统
[arxiv.org/abs/2603.12577](https://arxiv.org/abs/2603.12577)

引入专家金字塔结构，按任务复杂度层次化分配低秩专家，极少参数实现精细多任务路由。

### 5. 强化学习课程的热力学
[arxiv.org/abs/2603.12324](https://arxiv.org/abs/2603.12324)

借鉴非平衡热力学为 RL 课程学习建立正式理论框架，揭示课程设计与优化轨迹深层关系。
