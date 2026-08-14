---
layout: reading
title: "MoE推理加速 · KV缓存硬件 · 内核分层 · 循环深度 · FP8训练"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-08-14
---

# 📰 2026-08-14 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 MoE 推理与专家缓存、KV 缓存硬件加速、操作系统内核分层、Transformer 循环深度、无损文本压缩与 FP8 分布式训练。

---

## 1. APEX: Adaptive Expert Prefetching for Memory-Efficient Edge MoE Inference

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2608.11688
**标签**：MoE推理 · 专家缓存 · 边缘部署 · 预取

面向边缘设备的混合专家（MoE）模型推理受限于专家参数驻留的内存带宽。APEX 提出自适应专家预取机制，依据路由器的动态激活模式将专家参数从片外存储按需搬移至片上，在保持高激活稀疏度的同时显著降低显存占用与访存延迟。该方法无需改动模型结构，可作为现有边缘推理引擎的透明加速层，使更大容量的 MoE 在受限设备上可行。

**核心要点**：
- 用路由器 trace 刻画专家激活规律，实现按需预取而非全量常驻
- 将专家参数按热度分级到片上/片外层级存储，降低边缘显存压力
- 作为透明中间件接入，不修改模型权重即可获得内存收益

---

## 2. A Full-Stack Characterization of High-Bandwidth Flash for KV-Centric LLM Serving

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2608.11668
**标签**：KV缓存 · 高带宽闪存 · LLM推理 · 全栈剖析

高带宽闪存（HBF）在 NAND 前叠加宽接口，兼顾闪存级容量与远低于 SSD 的读延迟，看似可直接替换 Mooncake 式 KV 卸载栈的底层介质。作者用扩展版 TokenSim 与四组完整的双小时真实负载实测该替换，系统刻画了 KV 主导型 LLM 服务在 HBF 上的瓶颈与收益边界。结论指出盲目换介质并不等价，需重新设计调度与分级策略才能发挥 HBF 潜力。

**核心要点**：
- 量化 HBF 替换 SSD 后 KV 卸载栈的端到端延迟与吞吐变化
- 揭示 KV 随机访问特征下 HBF 的优势区间与回落点
- 给出面向 KV 服务的全栈协同优化方向与剖析方法论

---

## 3. TokenStack: A Heterogeneous HBM-PIM Architecture and Runtime for Efficient LLM Inference

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2605.05639
**标签**：HBM-PIM · KV缓存 · 存内计算 · 注意力加速

LLM 解码受 KV 缓存的带宽与容量双重制约，注意力退化为访存密集任务。TokenStack 提出异构 HBM-PIM 架构与配套运行时，将注意力计算下沉到近存单元，并依据 KV 块热度差异做堆叠式放置，仅对热块启用 PIM 资源，避免资源浪费。该设计在容量与带宽间取得更好平衡，显著降低解码阶段的平均访存成本。

**核心要点**：
- 将注意力计算迁移至 HBM 近存单元，缓解解码带宽瓶颈
- 按 KV 块热度分级启用 PIM，提升硬件利用率
- 异构运行时自动决定放置策略，对上层推理透明

---

## 4. Who Should Own the Expert Cache? Kernel-Managed Tiering for Trillion-Parameter MoE Inference

**来源**：arXiv cs.OS
**链接**：https://arxiv.org/abs/2608.12103
**标签**：MoE推理 · 操作系统 · 页缓存 · 专家分级

万亿参数 MoE 的专家池远超 DRAM，现有服务系统多在用户态自建基于频率排名、显式固定的专家缓存。本文探讨由操作系统内核接管专家分层的替代方案：将页缓存作为专家层级，借助路由器 trace 驱动内核级页面回收与分级。该思路把缓存一致性、淘汰与 pinned 管理下沉到内核，减少用户态重复造轮子与主机往返开销。

**核心要点**：
- 用 OS 页缓存承载 MoE 专家层级，替代用户态固定缓存
- 基于路由器 trace 驱动内核页面回收与分级策略
- 降低主机往返与重复缓存管理，提升超大专家池服务效率

---

## 5. Retrofitting Recurrent Depth into a Pretrained Language Model

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2608.11233
**标签**：循环深度 · Transformer · 参数复用 · 模型改造

本文提出将已预训练稠密语言模型改造为带循环深度（recurrent depth）的结构，学习一种迭代隐状态转移，并在仅结果监督的退火后依旧保持。以 Qwen2.5-0.5B-Instruct 为例，将其拆分为 Prelude、权重绑定的循环块与 Coda，并设计恒等保持单回路路径与后期重入桥。实验在两个参数预算下验证了改装后模型的外推、迁移与知识保留能力。

**核心要点**：
- 将预训练 LLM 拆分为前导-循环块-收尾三段并绑定权重
- 用迭代隐状态转移在不动原权重前提下获得循环深度收益
- 在两种参数预算下验证外推、迁移与知识保留

---

## 6. Diffuse to Compress: Leveraging Diffusion LMs for Lossless Compression

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2608.11249
**标签**：无损压缩 · 扩散语言模型 · 文本压缩 · 神经网络编码

面向数字文本（含代码、XML 等结构化数据）激增的存储需求，本文研究基于扩散语言模型的无损文本压缩。相比符号排序或自回归 LLM 压缩路线，扩散 LM 以迭代去噪建模序列分布，在压缩率与解码特性上提供新权衡。文章系统对比了不同神经压缩范式，论证扩散式建模在无损场景下的可行性与边界。

**核心要点**：
- 将扩散 LM 用于无损文本压缩，开辟不同于自回归的路线
- 覆盖纯文本、源代码与 XML 等结构化数据
- 对比神经压缩各范式，给出扩散路线的压缩率边界

---

## 7. FP8 Training on AMD GPUs with TorchTitan and TorchAO

**来源**：PyTorch Blog
**链接**：https://pytorch.org/blog/fp8-training-on-amd-gpus-with-torchtitan-and-torchao-upstreaming-performance-improvements/
**标签**：FP8训练 · AMD GPU · TorchTitan · 分布式训练

PyTorch 团队将在 AMD Instinct 集群上实现千卡级线性扩展的 FP8 训练优化（Primus-Turbo）已上游合并进 TorchTitan 与 TorchAO，使 TorchTitan 原生支持 AMD Instinct GPU 并开箱获得有竞争力的 FP8 性能。文章记录了这些性能优化的上游过程与实测缩放表现，降低了在 AMD 硬件上做大规模低精度训练的工程门槛。

**核心要点**：
- 将 AMD FP8 训练优化上游至 TorchTitan/TorchAO，原生支持 Instinct
- 实测千卡级接近线性扩展的训练吞吐
- 开箱即用低精度训练，降低 AMD 集群大规模训练成本

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | APEX: Adaptive Expert Prefetching for Memory-Efficient Edge MoE Inference | arXiv cs.AR | 硬件/推理加速 |
| 2 | A Full-Stack Characterization of High-Bandwidth Flash for KV-Centric LLM Serving | arXiv cs.AR | 硬件/推理加速 |
| 3 | TokenStack: A Heterogeneous HBM-PIM Architecture and Runtime for Efficient LLM Inference | arXiv cs.AR | 硬件/推理加速 |
| 4 | Who Should Own the Expert Cache? Kernel-Managed Tiering for Trillion-Parameter MoE Inference | arXiv cs.OS | OS/内核 |
| 5 | Retrofitting Recurrent Depth into a Pretrained Language Model | arXiv cs.CL | 模型架构/压缩 |
| 6 | Diffuse to Compress: Leveraging Diffusion LMs for Lossless Compression | arXiv cs.CL | 模型架构/压缩 |
| 7 | FP8 Training on AMD GPUs with TorchTitan and TorchAO | PyTorch Blog | 分布式训练 |

*自动生成 · 2026-08-14 · jeffinchen daily tech reading list*
