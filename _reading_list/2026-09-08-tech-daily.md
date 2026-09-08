---
layout: reading
title: "LLM 推理加速与量化前沿：MoE 解码、KV Cache、存储 Roofline 与混合精度"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-09-08
---

# 📰 2026-09-08 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 LLM 推理加速、量化压缩、KV Cache 与分布式 serving 系统。

---

## 1. MonoMoE：面向量化 MoE 解码的高效融合 Mega-kernel

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2609.04244
**标签**：MoE 解码 · 融合 Kernel · 量化 GEMM · 显存带宽 · 自回归

MoE 层在不等比例增加算力的前提下扩大模型容量，但其稀疏专家计算在自回归解码中难以高效执行。现有 grouped/batched GEMM 以 token 为主维度，当每个专家命中 token 极少时会产生 tile 填充与预处理开销、启动短生命周期的 kernel 导致显存带宽利用率低下，并暴露量化、激活与归约的额外成本。MonoMoE 提出一个融合 mega-kernel，将量化 MoE 解码的专家计算统一调度，减少 kernel 启动与 padding 浪费，提升廉价硬件上的解码吞吐。

**核心要点**：
- 指出现有 token-major GEMM 在稀疏专家场景下带宽利用率低、tile 填充严重的根本瓶颈
- 提出融合 mega-kernel，将量化、激活、归约与专家计算统一进单次调度
- 面向自回归解码优化，显著降低短生命周期 kernel 启动与预处理开销

---

## 2. 通过低秩注意力适配修复量化 KV Cache 的质量损失

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2609.04263
**标签**：KV Cache · 低比特量化 · 低秩适配 · 蒸馏 · 困惑度恢复

低比特 KV Cache 能大幅降低自回归解码的内存占用，但质量损失因模型与量化器而异。本文固定量化器不变，将浮点 cache 模型的行为蒸馏进低秩 Q/K/V 投影更新，同时学生在物理打包的增量 cache 上执行。在三个随机种子下，4-bit affine-cache 适配器在 TinyLlama-1.1B 上恢复 54.24%±2.47% 的留存困惑度差距，在 Gemma-4-12B 上恢复 75.96%±4.04%，并给出验证集上的收敛结果。

**核心要点**：
- 保持量化器冻结，仅用低秩 Q/K/V 投影更新弥补量化导致的质量退化
- 在物理打包增量 cache 上蒸馏浮点 teacher 行为，兼顾显存与精度
- 在 TinyLlama/Gemma 上量化给出可复现的困惑度恢复比例

---

## 3. Budgeting Bytes：面向存储受限 LLM 解码的窗口化存储 Roofline 与双预算架构消融

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2609.04238
**标签**：存储 Roofline · 字节预算 · 预取调度 · 内存层级 · 解码瓶颈

在廉价硬件上的自回归解码瓶颈并非 FLOPs，而是每个生成 token 必须跨越最慢内存层搬运的字节数。本文把 bytes-per-token 作为一等设计轴，用地址确定性分类法按参数在前向中地址何时可知（A0 采样时、A1 注意力前、A2 层内数据相关、A3 始终读取）对参数分类，将预取调度化简为带释放时间的单机可行性问题，并据此做双预算架构消融。

**核心要点**：
- 提出以「每 token 字节数」为核心的设计维度，重定义廉价硬件解码瓶颈
- 用地址确定性四分类法 (A0–A3) 刻画参数读取时机，指导预取调度
- 将预取调度建模为带释放时间的单机可行性问题，可形式化求解

---

## 4. FlexPosit：面向 LLM 推理加速器的可调分数精度

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2609.04724
**标签**：混合精度 · 分数位宽 · 推理加速器 · 量化粒度 · 能效

大模型能力强大但算力与能耗代价高昂，量化在粒度与位宽之间权衡精度与硬件效率。细粒度（如 group-wise）精度高但带来缩放与控制开销，粗粒度（如 channel-wise）开销低却在低位宽下精度下滑；混合精度量化算法上空间丰富，但现有加速器难以原生支持。FlexPosit 提出可调的分数精度表示，让推理加速器在精度-能效曲面上灵活取点。

**核心要点**：
- 剖析量化粒度/位宽/混合精度在精度与硬件开销间的权衡三角
- 提出可调分数精度 (fractional precision) 表示，适配加速器硬件
- 面向 LLM 推理加速器，支持在精度-能效 Pareto 前沿上灵活配置

---

## 5. Vertumnus：面向生产环境 LLM Serving 的自适应上下文并行

**来源**：arXiv cs.OS
**链接**：https://arxiv.org/abs/2609.04774
**标签**：上下文并行 · LLM Serving · 自适应调度 · 长序列 · 分布式

随着上下文窗口与输入序列变长，serving 系统面临越来越大的计算与内存压力。上下文并行 (CP) 将输入序列切分到多个 rank 以并行化计算，但现有 CP 系统要么依赖静态配置，要么仅对活跃请求/批次调整 CP 度。本文提出 Vertumnus，一个为异构生产环境设计的自适应 CP serving 系统，按负载动态决定并行度。

**核心要点**：
- 指出静态 CP 配置与仅按活跃请求调整两种现有方案的局限
- 提出 Vertumnus 自适应上下文并行系统，面向异构生产 serving
- 按实时负载动态抉择 CP 度，缓解长序列的计算与内存压力

---

## 6. 面向推理加速器芯片的硬件感知软件训练以恢复制造偏差导致的精度退化

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2609.04259
**标签**：硬件偏差 · 推理加速器 · 软件训练 · 芯片良率 · 鲁棒性

DNN 已被广泛用于各行各业，专用芯片以更低功耗、更高吞吐为目标。但芯片制造过程中的硬件偏差 (hardware variation) 是影响推理精度的主要原因。本文提出硬件感知软件训练 (HCST) 方法，使模型在硬件偏差影响下仍保持高推理精度，从软件侧补偿制程波动。

**核心要点**：
- 定位制造工艺偏差是专用推理芯片精度退化的主因
- 提出 HCST（硬件感知软件训练），在训练阶段纳入硬件偏差建模
- 软件侧补偿制程波动，提升低功耗推理芯片的精度鲁棒性

---

## 7. NeoMME：高效的多模态原生、多语言编码器

**来源**：HuggingFace Blog
**链接**：https://huggingface.co/blog/Hcompany/neomme
**标签**：多模态编码器 · 多语言 · 双向Transformer · 扩散目标 · 检索

NeoMME 是一族 260M 与 800M 参数的多语言多模态编码器。与多数生成式视觉语言模型不同，它不使用独立预训练视觉 tower 或因果语言模型，而是用单一双向 Transformer 同时处理文本 token 与原始图像 patch，并以掩码离散扩散目标从零训练。团队用 ColPali 的页面图像方法微调 NeoMME-Retriever，单次前向即可输出稠密与 late-interaction 两种嵌入，两个尺寸均落在 ViDoRe v3 榜单上。

**核心要点**：
- 单一双向 Transformer 原生融合文本与图像 patch，无独立视觉 tower
- 以掩码离散扩散目标从零训练，得到多语言多模态编码器
- NeoMME-Retriever 单次前向产出稠密+late-interaction 嵌入，用于视觉文档检索

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | MonoMoE：面向量化 MoE 解码的高效融合 Mega-kernel | arXiv cs.AR | MoE 解码 |
| 2 | 通过低秩注意力适配修复量化 KV Cache 的质量损失 | arXiv cs.AR | KV Cache |
| 3 | Budgeting Bytes：面向存储受限 LLM 解码的窗口化存储 Roofline 与双预算架构消融 | arXiv cs.AR | 存储 Roofline |
| 4 | FlexPosit：面向 LLM 推理加速器的可调分数精度 | arXiv cs.AR | 混合精度 |
| 5 | Vertumnus：面向生产环境 LLM Serving 的自适应上下文并行 | arXiv cs.OS | 上下文并行 |
| 6 | 面向推理加速器芯片的硬件感知软件训练以恢复制造偏差导致的精度退化 | arXiv cs.AR | 硬件偏差 |
| 7 | NeoMME：高效的多模态原生、多语言编码器 | HuggingFace Blog | 多模态编码器 |

*自动生成 · 2026-09-08 · jeffinchen daily tech reading list*
