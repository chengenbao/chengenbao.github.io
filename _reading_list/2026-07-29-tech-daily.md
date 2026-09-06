---
layout: reading
title: "arXiv 论文速报、线性注意力与推理系统基准评测"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-07-29
---

# 📰 2026-07-29 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖论文速报、注意力机制、推理系统评测与异构推理。

---

## 1. arXiv 论文速报 2026-07-29：cs.IR / cs.LG / cs.AI 三分类扫描

**来源**：iWiki 论文速报
**链接**：https://iwiki.woa.com/p/4028301568
**标签**：论文速报 · arXiv · 日报 · cs.LG

当日 arXiv 速报：cs.IR / cs.LG / cs.AI 三分类双通路初筛 + 精读，保留 68 篇论文并附评分。覆盖 LLM 系统、Agent 与检索方向的最新工作，适合作为当日技术圈的"论文雷达"快速扫描。

**核心要点**：
- 三分类 68 篇保留论文的主题分布
- LLM 推理与 Agent 基础设施方向的持续升温
- 大厂工业化系统论文的识别与精读建议

---

## 2. LLM 系统论文清单：2026 上半年精选（AmberLJC/LLMSys-PaperList）

**来源**：GitHub（LLM Systems Paper List）
**链接**：https://github.com/AmberLJC/LLMSys-PaperList
**标签**：论文清单 · LLM 系统 · 推理 · 训练

维护中的 LLM 系统方向论文清单，按推理服务、KV Cache、调度、通信、训练系统等主题分类，覆盖 OSDI/SOSP/NSDI/MLSys/FAST 顶会与 arXiv 预印本。做系统方向研究或调研时，这是一份省去大量检索时间的高质量入口。

**核心要点**：
- 推理服务系统的论文按问题域组织（调度/缓存/解耦）
- 顶会与预印本的收录标准与短评
- 快速定位"某子问题目前有哪些代表工作"

---

## 3. 2026 LLM 推理论文合集：FP4、KV 量化与 GPU 内核

**来源**：GPUHunter（论文精选）
**链接**：https://www.gpuhunter.io/research/2026-llm-inference-papers
**标签**：量化 · KV Cache · GPU 内核 · 论文合集

2026 年推理方向的论文精选集：FP4 量化、KV Cache 量化、本地推理控制器、GPU 内核与 AMD 生态等主题。合集按技术栈分层组织并附一句话点评，是把握"推理优化当下前沿"的高效索引。

**核心要点**：
- FP4/FP8 量化的内核成熟度与精度实测
- KV 量化从 8-bit 到 4-bit 的精度边界
- 非 NVIDIA 生态（AMD/自研 ASIC）的推理栈进展

---

## 4. GPU-NPU 混合系统：百万 token 上下文的推理方案

**来源**：ACM（SC'25 系统论文 Hybe）
**链接**：https://dl.acm.org/doi/full/10.1145/3695053.3731051
**标签**：GPU-NPU 混合 · 长上下文 · KV Cache offload · 异构

Hybe 面向百万 token 上下文场景，把 KV Cache offload 到 NPU 的大内存与专用带宽，GPU 专注计算：利用 NPU 的 LPDDR 带宽承载 KV 访存，通过细粒度流水隐藏传输。在异构算力（GPU+NPU）日益普遍的 2026 年，这是"按数据流特性分配存储"的代表方案。

**核心要点**：
- KV offload 的带宽-容量双收益：NPU 侧 LPDDR 的角色
- 细粒度流水：KV 分块传输与注意力计算重叠
- 百万上下文的 prefill/decode 延迟实测

---

## 5. AdaSpec：投机解码器的选择性知识蒸馏

**来源**：NeurIPS'25 论文（Georgia Tech）
**链接**：https://iwiki.woa.com/p/4021459825
**标签**：投机解码 · 知识蒸馏 · 草稿模型 · 加速

AdaSpec 改进投机解码的草稿模型训练：不是全量蒸馏目标模型，而是按目标模型在不同领域的接受率选择性蒸馏，让草稿模型在"高接受率域"更激进、低接受率域更保守。以更小的草稿模型取得更高的平均接受长度，直接降低投机解码的算力开销。

**核心要点**：
- 接受率引导的选择性蒸馏目标设计
- 草稿模型容量与接受长度的帕累托改进
- 领域自适应：同一服务多负载的动态适配

---

## 6. Q-Infer：GPU-CPU 协同的高效 LLM 推理

**来源**：ACM TURC（系统论文）
**链接**：https://dl.acm.org/doi/10.1145/3764589
**标签**：GPU-CPU 协同 · 显存扩展 · 推理系统 · 异构计算

Q-Infer 重新审视 GPU-CPU 协同推理：PCIe 带宽不再是唯一瓶颈，通过量化 KV 的 CPU 侧存储、GPU 侧按需拉取与计算-传输重叠，在消费级设备上扩展上下文长度。为显存受限场景（私有化部署、边缘服务器）提供了低成本方案。

**核心要点**：
- CPU 侧量化 KV 存储与 GPU 按需访存的协同设计
- 传输-计算流水线的带宽利用率优化
- 与纯 GPU offload 方案的成本-性能对比

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | arXiv 论文速报 2026-07-29 | iWiki | 论文速报 |
| 2 | LLM 系统论文清单 2026 上半年 | GitHub | 论文清单 |
| 3 | 2026 LLM 推理论文合集：FP4/KV 量化 | GPUHunter | 量化 |
| 4 | Hybe：GPU-NPU 混合系统百万上下文推理 | ACM SC'25 | 异构推理 |
| 5 | AdaSpec：投机解码的选择性知识蒸馏 | NeurIPS'25 | 投机解码 |
| 6 | Q-Infer：GPU-CPU 协同推理 | ACM | 异构计算 |

---

*自动生成 · 2026-07-29 · jeffinchen daily tech reading list*
