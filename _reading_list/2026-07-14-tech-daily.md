---
layout: reading
title: "2026-07-14 技术速递 · MoE推理加速与量化前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-07-14
---

# 📰 2026-07-14 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖 MoE推理加速、量化技术、激活稀疏化、HBM近存储计算、扩散LLM服务。

---

## 1. 少比特整数的有符号对称量化

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2607.08779  
**标签**：量化 · LLM压缩 · 少比特推理 · 整数量化 · 模型部署

The signed integer alphabet contains one more negative representable value than positive. Yet, by convention, quantization schemes ignore this asymmetry and clip the extra negative value. This paper revisits this design choice and shows that exploiting the full signed range via signed symmetric quantization can reduce quantization error, especially at 4-bit and lower precision, with no runtime overhead.

**核心要点**：
- 有符号整数天然多一个负数值，传统量化方案将其裁剪浪费了精度空间
- 提出有符号对称量化，充分利用全范围负值，在 4-bit 及更低精度下显著降低量化误差
- 方案引入零运行时开销，可直接嵌入现有量化框架
- 对大模型权重量化与激活量化均有效，部署友好

---

## 2. 粘性路由：面向内存高效推理的 MoE 模型训练

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2607.08780  
**标签**：MoE · 推理优化 · 内存效率 · 路由机制 · 分布式推理

Mixture-of-Experts (MoE) models activate only a sparse subset of experts per token, yet consecutive tokens typically load different experts, causing substantial memory thrashing. Sticky Routing trains MoE models to consistently route similar tokens to the same experts across layers, dramatically reducing expert swaps during autoregressive generation and enabling large MoE models to run on fewer accelerators without accuracy loss.

**核心要点**：
- MoE 推理中相邻 token 频繁切换专家导致大量内存换入换出，拖慢推理速度
- 提出 Sticky Routing 训练策略，让相似 token 在各层一致路由到同一批专家
- 显著减少自回归生成阶段的专家换页次数，降低显存峰值需求
- 在保持模型准确率的同时，大幅减少运行 MoE 所需的加速器数量

---

## 3. Director：基于在线主动专家放置的分布式 MoE 推理加速

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2607.08782  
**标签**：MoE推理 · 专家并行 · 分布式推理 · 负载均衡 · GPU集群

Expert parallelism serves MoE models across multiple GPUs, but static expert placement leads to severe load imbalance and idle GPU time. Director introduces an online proactive expert placement controller that monitors per-expert load statistics in real time and migrates hot experts to reduce cross-device communication overhead, achieving up to 2× throughput improvement on large MoE models.

**核心要点**：
- 静态专家放置导致 MoE 分布式推理中 GPU 负载严重失衡，存在大量空转
- Director 实时监控各专家的请求负载，动态迁移热点专家平衡压力
- 通过主动放置减少跨设备通信开销，显著提升集群吞吐
- 在大规模 MoE 模型上实现最高 2× 推理吞吐量提升

---

## 4. BlockServe：面向高吞吐扩散型大模型推理的块级连续批处理

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2607.08930  
**标签**：扩散LLM · 推理服务 · 连续批处理 · 高吞吐 · dLLM

Diffusion large language models (dLLMs) perform generation through iterative denoising rather than autoregressive decoding, but their variable convergence time per token creates severe batching inefficiency. BlockServe proposes block-grained continuous batching that groups tokens by their denoising progress, enabling high-throughput serving of dLLMs without waiting for the slowest token in a batch.

**核心要点**：
- 扩散型 LLM 基于迭代去噪生成，不同 token 收敛步数不一，导致批处理效率低下
- 提出块级连续批处理（Block-Grained Continuous Batching），按去噪进度分组调度
- 无需等待批内最慢 token，显著提升服务吞吐量和 GPU 利用率
- 为扩散 LLM 的生产部署提供了高效推理服务框架

---

## 5. 面向 LLM 激活稀疏化的敏感度感知阈值与 Token 路由

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2607.08991  
**标签**：激活稀疏化 · LLM推理加速 · Token路由 · 计算跳过 · FFN优化

Activation sparsification skips computation for near-zero activations in LLM FFN layers, but uniform thresholds ignore layer-wise sensitivity differences, causing accuracy degradation. This work introduces a sensitivity-aware per-layer threshold calibration and a token routing mechanism that preserves important activations while maximizing sparsity, achieving significant speedup with negligible quality loss.

**核心要点**：
- 激活稀疏化跳过接近零的 FFN 激活值以减少计算量，但统一阈值忽视层间敏感度差异
- 提出逐层敏感度感知阈值校准，精确识别可安全跳过的计算
- 引入 Token 路由机制，优先保留对输出质量影响大的关键激活
- 在保持模型质量的前提下实现显著推理加速，适用于生产环境部署

---

## 6. StreamDQ：面向可扩展 AI 推理的 HBM 近存储权重反量化

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2607.08993  
**标签**：HBM · 近存储计算 · 权重反量化 · 推理扩展 · 硬件架构

As LLMs scale, memory bandwidth becomes the dominant bottleneck for inference. StreamDQ proposes embedding weight dequantization logic directly in custom High Bandwidth Memory (HBM) stacks, allowing quantized weights to be streamed at full HBM bandwidth and dequantized on the fly before reaching the compute die, effectively decoupling memory bandwidth from dequantization overhead and enabling near-linear scaling of inference throughput.

**核心要点**：
- LLM 推理中内存带宽已成为主要瓶颈，权重搬运与反量化开销制约吞吐扩展
- 提出将反量化逻辑嵌入定制 HBM 堆栈内部，实现近存储计算
- 量化权重以全 HBM 带宽流式传输，到达计算芯片前完成实时反量化
- 有效解耦内存带宽与反量化开销，实现推理吞吐近线性扩展

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | 少比特整数的有符号对称量化 | arXiv cs.LG | 量化 |
| 2 | 粘性路由：面向内存高效推理的 MoE 模型训练 | arXiv cs.LG | MoE推理 |
| 3 | Director：基于在线主动专家放置的分布式 MoE 推理加速 | arXiv cs.LG | MoE推理 |
| 4 | BlockServe：面向高吞吐扩散型大模型推理的块级连续批处理 | arXiv cs.LG | 推理加速 |
| 5 | 面向 LLM 激活稀疏化的敏感度感知阈值与 Token 路由 | arXiv cs.LG | HBM架构 |
| 6 | StreamDQ：面向可扩展 AI 推理的 HBM 近存储权重反量化 | arXiv cs.AR | 推理加速 |

---

*自动生成 · 2026-07-14 · jeffinchen daily tech reading list*
