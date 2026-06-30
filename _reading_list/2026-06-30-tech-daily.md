---
layout: reading
title: "LLM 推理加速 · 注意力优化 · 混合精度 · Transformer 架构创新 · RLHF 对齐"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-06-30
---

# 📰 2026-06-30 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 LLM 推理加速 · 注意力优化 · 混合精度 · Transformer 架构创新 · RLHF 对齐。

---

## 1. EntMTP：基于熵引导的多 Token 预测加速 LLM 推理

**来源**：cs.CL / arXiv  
**链接**：https://arxiv.org/abs/2606.27550  
**标签**：LLM推理加速 · 多Token预测 · 投机解码 · Entropy引导 · 推理效率

Multi-token prediction has been shown to increase data density during training, improve downstream text-generation quality, and serves as the defacto approach for speculative decoding. EntMTP introduces entropy-guided scheduling to decide when to predict multiple tokens at once, dynamically balancing speed vs. quality trade-offs.

**核心要点**：
- 提出基于熵值动态决策的多Token预测策略，熵低时扩展预测步长，熵高时回退单步精确预测
- 与传统 speculative decoding 相比，无需草稿模型即可实现推理加速
- 训练和推理双阶段均有收益：训练数据密度提升 + 推理吞吐提升
- 在多个主流 LLM benchmark 上验证效果，延迟降低且质量不损失

---
## 2. NLL 引导的全注意力层选择：免训练滑动窗口适配

**来源**：cs.CL / arXiv  
**链接**：https://arxiv.org/abs/2606.27791  
**标签**：长上下文推理 · 混合注意力 · 滑动窗口 · 免训练 · LLM高效化

Hybrid attention models mixing full and sliding-window attention offer efficient long-context inference, but selecting which layers get full attention has required costly retraining. This paper proposes NLL (Negative Log-Likelihood) guided selection to identify critical full-attention layers without any training, enabling training-free conversion of dense models to hybrid attention.

**核心要点**：
- 通过 NLL 差异度量每层从全注意力切换为滑动窗口时的信息损失，自动识别关键层
- 无需任何微调或重训练，直接对已有密集模型做免训练混合注意力转换
- 在长文本任务（128K+ tokens）中保持性能接近全注意力，推理 KV 缓存显著减少
- 为长上下文模型部署提供零成本优化路径

---
## 3. 校准引导 LLM 压缩中的输出空间分配代价研究

**来源**：cs.CL / arXiv  
**链接**：https://arxiv.org/abs/2606.27785  
**标签**：LLM压缩 · 模型量化 · 校准数据 · 无训练压缩 · ROCKET

Training-free compression methods for LLMs often use calibration data to guide compression decisions. This paper studies ROCKET and related methods, revealing that output-space token allocation during calibration creates hidden costs affecting compression quality. The study identifies systematic biases and proposes corrections.

**核心要点**：
- 揭示校准引导压缩中输出空间分配的隐藏代价，ROCKET 等方法受此影响产生系统偏差
- 分析 token 分配不均对压缩质量的影响，提出修正方案
- 实验表明修正后在多个模型（Llama、Mistral 等）上压缩质量显著提升
- 对工业界无训练量化/剪枝流程具有直接指导价值

---
## 4. Phase Matters：移动端 SoC 上异构视觉语言模型推理特性分析

**来源**：cs.AR / arXiv  
**链接**：https://arxiv.org/abs/2606.27906  
**标签**：移动端推理 · VLM · NPU · SoC架构 · 异构计算

Recent phone-class mobile SoCs expose NPU execution paths for on-device VLM inference, but developers lack phase-level understanding of compute patterns. This paper characterizes VLM inference across CPU/GPU/NPU on mobile SoCs, revealing phase-specific bottlenecks and providing architectural optimization guidance.

**核心要点**：
- 首次系统分析手机级 SoC 上 VLM（视觉语言模型）推理的各阶段（prefill/decode）计算特性
- 揭示 CPU/GPU/NPU 在不同推理阶段的性能瓶颈差异，提供阶段感知调度建议
- 指导开发者针对 prefill 和 decode 阶段选择最优执行单元
- 为端侧大模型部署优化提供实测数据支撑

---
## 5. SEADA：面向多精度空间架构的混合精度 DNN 优化方法

**来源**：cs.AR / arXiv  
**链接**：https://arxiv.org/abs/2606.27884  
**标签**：混合精度 · DNN加速器 · 空间架构 · 延迟优化 · 神经网络量化

Mixed-precision computation reduces latency, energy, and memory footprint of DNNs. SEADA proposes an efficient design-space exploration methodology to automatically assign optimal precision to each DNN layer on multi-precision spatial accelerators, achieving better latency-accuracy tradeoffs.

**核心要点**：
- 针对支持多精度（INT2/INT4/INT8 等）的空间加速器，自动搜索逐层最优精度配置
- 引入高效的设计空间裁剪策略，将搜索代价从指数级降至可接受范围
- 在保持精度损失 <1% 的前提下，延迟和能耗分别优于均匀精度基线
- 方法通用，适配不同 DNN 网络架构和硬件平台

---
## 6. Prism Transformer：渐进式多头调度实现层次化注意力处理

**来源**：cs.LG / arXiv  
**链接**：https://arxiv.org/abs/2606.27449  
**标签**：Transformer架构 · 多头注意力 · 架构创新 · 预训练LLM · 零开销优化

Standard multi-head attention uses identical head counts across all layers. Prism Transformer replaces this with a progressive head schedule—fewer, wider heads in early layers to capture complex local patterns, and more, narrower heads in deep layers for specialized decomposition. This is parameter-neutral and compute-neutral yet consistently outperforms uniform baselines.

**核心要点**：
- 提出渐进式多头调度：浅层使用少量宽头捕捉局部复杂模式，深层使用多量窄头做特征分解
- 参数量和 FLOP 完全不变，零额外训练开销，纯架构层面的改进
- 在 124M/354M/757M 三个规模下均持续优于均匀多头基线（PIQA、HellaSwag、ARC-Easy 等）
- 揭示非均匀子空间分配是挖掘 Transformer 容量的有效手段

---
## 7. PEBS：RLHF 奖励模型的逐标注者经验贝叶斯收缩校准

**来源**：cs.LG / arXiv  
**链接**：https://arxiv.org/abs/2606.27578  
**标签**：RLHF · 奖励模型 · 贝叶斯 · 标注者偏差 · 对齐训练

RLHF reward models pool preferences across thousands of annotators and fit one global affine calibration, ignoring individual rater biases. PEBS applies per-rater empirical Bayes shrinkage to calibrate reward models individually, improving the alignment between human preferences and model rewards.

**核心要点**：
- 发现 RLHF 中全局统一校准忽略了个体标注者偏差，导致奖励模型校准偏差
- 提出逐标注者经验贝叶斯收缩（PEBS），对每个 rater 单独估计偏移和缩放参数
- 在数据稀疏的 rater 上自动收缩到全局均值，避免过拟合
- 在多个 RLHF 数据集上验证，奖励校准误差显著降低，对齐质量提升

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | EntMTP：基于熵引导的多 Token 预测加速... | cs.CL | 推理加速 |
| 2 | NLL 引导的全注意力层选择：免训练滑动窗口适配... | cs.CL | 注意力机制 |
| 3 | 校准引导 LLM 压缩中的输出空间分配代价研究... | cs.CL | 模型压缩 |
| 4 | Phase Matters：移动端 SoC 上异构... | cs.AR | 端侧推理/硬件 |
| 5 | SEADA：面向多精度空间架构的混合精度 DNN ... | cs.AR | 硬件加速/量化 |
| 6 | Prism Transformer：渐进式多头调度... | cs.LG | 模型架构 |
| 7 | PEBS：RLHF 奖励模型的逐标注者经验贝叶斯收... | cs.LG | RLHF/对齐 |

---

*自动生成 · 2026-06-30 · jeffinchen daily tech reading list*

