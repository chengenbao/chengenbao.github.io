---
layout: reading
title: "技术速递 2026-03-07：LLM 高效训练与推理 · KV Cache · 量化加速"
category: tech
tags: [Tech, arXiv, LLM训练, KV-Cache, 量化, 推理加速, 系统优化]
date: 2026-03-07
---

今日精选 5 篇前沿论文，聚焦大模型内存高效训练、KV Cache 压缩优化、几何感知量化、可解释性与服务系统动态并行。

## 1. POET-X：正交等价变换的大模型内存高效训练

**链接：** [arXiv:2603.05500](https://arxiv.org/abs/2603.05500)

POET-X 是 POET（谱保持正交等价训练框架）的可扩展高效变体，通过降低正交变换的计算成本大幅提升内存效率。**单张 Nvidia H100 即可完成十亿参数 LLM 预训练**，而标准 AdamW 同等设置下 OOM。在保留 POET 训练稳定性和泛化优势的前提下，显著提升吞吐量。

---

## 2. Censored LLMs 作为知识诱导测试床

**链接：** [arXiv:2603.05494](https://arxiv.org/abs/2603.05494)

用受政治审查训练的模型（Qwen3 等）作为"知而不言"的天然测试场景，系统评估诚实性诱导技术。去掉 chat template 采样、few-shot 提示、通用诚实数据微调效果最佳；让模型自我分类回答的谎言检测接近无审查模型上界；线性探测（trained on unrelated data）提供低成本替代。技术可迁移到 DeepSeek R1 等开源模型。

---

## 3. GAQ：保持 SO(3) 等变性的几何感知量化

**链接：** [arXiv:2603.05343](https://arxiv.org/abs/2603.05343)

等变 GNN 分子模拟的低比特量化新框架。核心：**量级-方向解耦量化（MDDQ）**，分离不变长度与等变方向。W4A8 模型在 rMD17 基准上匹配 FP32 精度（9.31 vs 23.20 meV），局部等变误差降低 30x，推理加速 2.39x，内存减少 4x。

---

## 4. FLYING SERVING：动态并行切换的 LLM 在线服务

**链接：** [arXiv:2503.18712](https://arxiv.org/abs/2503.18712)

生产 LLM 服务中 DP（高吞吐）与 TP（低延迟+长上下文）难以兼顾。FLYING SERVING 提出在线、无停机动态切换并行策略的框架，根据实时流量特征自适应选择并行模式。

---

## 5. InfoFlow KV：信息流感知的 KV 重计算

**链接：** [arXiv:2503.18823](https://arxiv.org/abs/2503.18823)

RAG 长上下文 QA 的 prefilling 瓶颈优化。通过信息流分析识别与当前查询真正相关的 KV 进行选择性重计算，比注意力分数方法更精准地识别"必要"KV，显著减少内存和计算开销。
