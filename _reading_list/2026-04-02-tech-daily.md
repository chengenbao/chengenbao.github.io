---
layout: reading
title: "技术速递 2026-04-02：LLM 压缩 · GPU 内核优化 · Transformer 长程依赖分析"
category: tech
tags: [Tech, arXiv, LLM, GPU优化, 模型压缩, 训练优化, 二阶优化]
date: 2026-04-02
---

今日精选 6 篇 arXiv 前沿深度技术论文，覆盖大模型压缩、优化器改进、GPU 内核自动优化等方向。

## 1. OneComp：一行代码统一 LLM 压缩

**论文**：[arXiv:2603.28845](https://arxiv.org/abs/2603.28845)

OneComp 提出统一 API，一行代码完成量化/剪枝/蒸馏，在主流 LLM 上实现 <1% 性能损失的高效压缩，显著降低推理内存占用。

## 2. Beta-Scheduling：临界阻尼动量调度

**论文**：[arXiv:2603.28921](https://arxiv.org/abs/2603.28921)

将物理临界阻尼原理引入动量参数调度，自适应调整 Adam/SGD 的 β 参数，有效诊断并纠正训练中动量过积累问题，大批量场景收敛提速显著。

## 3. 可微初始化加速的 CPU-GPU 混合组合调度

**论文**：[arXiv:2603.28943](https://arxiv.org/abs/2603.28943)

可微松弛初始化加速组合求解器，将异构集群 ML 训练任务图调度决策时延降低数倍，适用于分布式训练任务图动态分配优化。

## 4. 用 DSL 提升 GPU 内核优化 Agent 效率

**论文**：[arXiv:2603.29010](https://arxiv.org/abs/2603.29010)

为 LLM-based GPU 内核优化 Agent 设计 DSL，将搜索空间从原始 CUDA C++ 压缩到结构化表达，算子融合任务上 Agent 效率提升 2-4x。

## 5. Transformer 长程依赖的幻觉：整数乘法视角

**论文**：[arXiv:2603.29069](https://arxiv.org/abs/2603.29069)

以整数乘法为 benchmark，揭示注意力机制在结构化长程任务上的系统性盲区，为 RoPE、ALiBi 等位置编码改进提供实证分析基础。

## 6. KFAC 超梯度：高效双层优化

**论文**：[arXiv:2603.29108](https://arxiv.org/abs/2603.29108)

将 KFAC 近似曲率引入双层优化超梯度计算，效率提升 3-5x，适配 MAML、DARTS、超参搜索等标准双层任务。
