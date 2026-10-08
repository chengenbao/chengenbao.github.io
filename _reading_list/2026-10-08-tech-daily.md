---
layout: reading
title: "大模型训练与服务中的显存、KV Cache 与推理内核优化"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-10-08
---

# 📰 2026-10-08 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 分布式训练缩容、KV Cache 压缩与放置、位置编码改进、Flash Attention 与推理内核优化。

---

## 1. TRANSIT：面向多节点大模型训练的透明缩容框架

**来源**：arXiv (cs.AR)
**链接**：https://arxiv.org/abs/2610.07593
**标签**：分布式训练 · GPU显存 · CPU DRAM · 大模型训练

TRANSIT 是一个透明的缩容（scale-in）框架，通过把 CPU DRAM 当作 GPU 显存的扩展，让多节点大模型训练能够在更少 GPU 上保持训练效率。它在用户态通过拦截层实现，无需修改应用、训练框架、集群调度器、设备驱动或操作系统，并借助零拷贝数据通路提升 CPU-GPU 传输效率。在稠密与 MoE 模型上、规模达 64 张 NVIDIA H100 的实测验证了该方案的可扩展性。

**核心要点**：
- 透明缩容：用 CPU DRAM 扩展 GPU 显存，减少训练所需卡数
- 用户态拦截层实现，对现有训练栈零侵入
- 零拷贝 CPU-GPU 数据通路提升传输效率；最高验证到 64×H100

---

## 2. AttSVD：基于注意力引导 SVD 的提示自适应低秩 KV Cache 压缩

**来源**：arXiv (cs.LG)
**链接**：https://arxiv.org/abs/2610.06927
**标签**：KV Cache · 推理加速 · 低秩压缩 · 长上下文

自回归 Transformer 的 KV Cache 随上下文长度线性增长，在长上下文场景下成为内存瓶颈。现有无训练方法多在序列轴上驱逐低重要性 token，属于不可逆选择；AttSVD 转而保留每个 token、沿“特征轴”更廉价地存储——它基于每个 prompt 自身的注意力几何进行在线、按提示的截断 SVD，只保留注意力真正读取的方向，从而按注意力集中度成比例削减每头持久 KV 内存。

**核心要点**：
- 从“丢弃 token”转向“降维存储”，避免不可逆的驱逐决策
- 按 prompt 在线计算截断 SVD，基向量来自注意力几何
- 持久 KV 内存随注意力集中度成比例下降，无需训练

---

## 3. Lachesis：面向智能体服务的 KV Cache 生命周期感知放置（HBM 与高带宽闪存）

**来源**：arXiv (cs.AR)
**链接**：https://arxiv.org/abs/2610.08378
**标签**：LLM 服务 · KV Cache · 智能体 · 高带宽闪存

LLM 服务正日益被智能体工作负载主导，智能体及其子代理跨多次请求累积 KV Cache，消耗大量显存。高带宽闪存（HBF）以接近 HBM 的读带宽提供约一个数量级的更大容量，但其有限的写入寿命是关键限制。Lachesis 的核心洞见是按生命周期在 HBM 与 HBF 间放置 KV Cache：将短生命周期数据放入 HBM，吸收更多写入，从而减少对 HBF 的写压力。

**核心要点**：
- 智能体工作负载使 KV Cache 跨请求累积，显存压力剧增
- HBF 容量大但写入寿命有限，需按生命周期分级放置
- 短生命周期数据进 HBM，降低 HBF 写磨损，延长寿命

---

## 4. 评估生成式 AI 的推理算力：企业级工作负载框架

**来源**：arXiv (cs.AR)
**链接**：https://arxiv.org/abs/2610.07094
**标签**：推理硬件 · 智能体 · 成本模型 · 加速器

LLM 部署正从单轮补全转向智能体轨迹——模型在测试时规划、调用工具、读取结果并推理后再行动。这反转了推理硬件的经济学：对话服务可通过大批量摊销权重读取，而智能体轨迹串行依赖、实际批量约为 1，使每 token 解码延迟（TPOT）成为任务完成时间的主导项。作者用 roofline 分析与闭式回合延迟模型说明，该 regime 更偏好把权重放在片上 SRAM 的加速器（如 Cerebras）。

**核心要点**：
- 智能体轨迹反转推理经济学：TPOT 成为主导成本项
- roofline + 闭式回合延迟模型量化硬件选型
- 片上 SRAM 权重驻留型加速器更适配 agentic 推理

---

## 5. WavePrune：RoPE 一个周期往往已经足够

**来源**：arXiv (cs.CL)
**链接**：https://arxiv.org/abs/2610.06963
**标签**：RoPE · 位置编码 · 注意力 · 长上下文

旋转位置编码（RoPE）按通道特定频率旋转查询与键向量，使注意力对数相对平移不变，但其旋转具有周期性，会导致位置混叠——相隔一个完整旋转周期的相对位置难以区分。WavePrune 将每个通道限制在其首个旋转周期内，去除位置混叠在注意力图中引入的干扰，从而改善整体长上下文表现。

**核心要点**：
- RoPE 周期性带来位置混叠，损害长程相对位置判别
- 限制通道到首个旋转周期即可消除混叠干扰
- 无需重新训练即可提升长上下文表现

---

## 6. 用 Helion 构建高性能可移植的 vLLM 线性层后端

**来源**：PyTorch Blog
**链接**：https://pytorch.org/blog/building-a-high-performance-and-portable-vllm-linear-backend-with-helion/
**标签**：vLLM · 推理内核 · 编译器 DSL · Helion

本文介绍将 Helion（高层内核 DSL）集成进 vLLM 的线性层后端，探索自动调优、高抽象层级的内核 DSL 如何在提升 LLM 推理性能的同时降低内核实现复杂度。一条 Helion 通用内核描述即可覆盖多种形状与后端，兼顾性能与可移植性。

**核心要点**：
- Helion 高层 DSL 集成进 vLLM 线性后端
- 自动调优 + 单一通用内核实现，降低手工内核成本
- 在保持可移植性的同时逼近手写内核性能

---

## 7. 用 TLX 优化 Jagged Flash Attention：迈向 Blackwell 上的 SOTA FA4

**来源**：PyTorch Blog
**链接**：https://pytorch.org/blog/optimizing-jagged-flash-attention-with-tlx-the-road-toward-sota-fa4-on-blackwell/
**标签**：Flash Attention · Blackwell · GPU 内核 · 变长序列

本文介绍在 NVIDIA Blackwell（B200）上实现 Jagged Flash Attention（JFA）——Meta 生成式广告模型（GEM）背后的注意力内核。JFA 针对变长（jagged）序列设计，借助 TLX 在 Blackwell 架构上优化，向 SOTA 的 Flash Attention 4 迈进，展示了新一代 GPU 上注意力内核的效率边界。

**核心要点**：
- JFA 是面向变长序列的 Flash Attention，支撑 Meta GEM
- 基于 TLX 在 Blackwell B200 上优化
- 向 FA4 SOTA 性能迈进，刷新注意力内核效率

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | TRANSIT：面向多节点大模型训练的透明缩容框架 | arXiv | 分布式训练 |
| 2 | AttSVD：基于注意力引导 SVD 的提示自适应低秩 KV Cache 压缩 | arXiv | KV Cache 压缩 |
| 3 | Lachesis：面向智能体服务的 KV Cache 生命周期感知放置（HBM 与高带宽闪存） | arXiv | KV Cache 放置 |
| 4 | 评估生成式 AI 的推理算力：企业级工作负载框架 | arXiv | 推理硬件 |
| 5 | WavePrune：RoPE 一个周期往往已经足够 | arXiv | 位置编码 |
| 6 | 用 Helion 构建高性能可移植的 vLLM 线性层后端 | PyTorch | 推理内核 |
| 7 | 用 TLX 优化 Jagged Flash Attention：迈向 Blackwell 上的 SOTA FA4 | PyTorch | GPU 内核 |

*自动生成 · 2026-10-08 · jeffinchen daily tech reading list*
