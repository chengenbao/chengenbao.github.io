---
layout: reading
title: "大模型推理加速：超低比特量化、KV-Cache 与弹性服务"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-09-25
---

# 📰 2026-09-25 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 大模型超低比特量化、KV-Cache 管理、LLM 推理服务调度、GPU 显存弹性共享。

---

## 1. 面向 INT2 KV-Cache 量化的输出感知旋转方法

**来源**：arXiv (cs.LG)  
**链接**：https://arxiv.org/abs/2608.02691  
**标签**：INT2量化 · KV-Cache · 注意力读出 · 旋转校准 · 长上下文

KV-Cache 已成为长上下文大模型推理的主要显存与带宽瓶颈，超低位量化愈发关键。现有基于旋转的 INT2 方法在注意力完整读出（经过输出投影 W_O）之前优化缓存统计量，与模型实际受到的误差存在错位。本文提出 OptR，将 W_O 之后的注意力输出误差分解为 key 与 value 两个分量，并在完整的 INT2 量化与注意力路径上学习逐头正交校正，同时引入注意力等价的 key 重参数化以消除大幅通道偏移而不改变 softmax 分布。

**核心要点**：
- 现有旋转式 INT2 量化在注意力读出前优化统计量，与模型真实误差错配。
- OptR 把 W_O 后输出误差拆为 key/value 分量，沿完整量化路径学习逐头正交校正。
- 三个模型、五类推理/代码基准上持续优于 QuaRot 与 OSCAR，保留分页 KV-Cache 格式且推理开销可忽略。

---

## 2. 在 CGLA 可编程阵列上以 Signed-Int4 指令实现 BitNet 推理

**来源**：arXiv (cs.AR)  
**链接**：https://arxiv.org/abs/2609.27453  
**标签**：BitNet · Int4 · CGLA · 矩阵乘累加 · 端侧推理

BitNet b1.58 以三值权重与整数激活表示超低比特模型，但其算术与常规 int8/浮点 GEMM 不匹配，现有加速器多用专用数据通路实现。本文将其映射到 CGLA（带显式 DMA、本地存储与可复用整型通道的可编程 ASIC），新增一条可复用的 OP_SMA4 有符号 int4 乘累加指令而非 BitNet 专用通路。每个三值权重占一个 signed-4bit 通道，int8 激活拆分为两个 int4 片段再移位相加重建。频率从 145MHz FPGA 外推至 28nm 840MHz CGLA，单条 signed-int4 乘积耗时 0.390ns，实测 C++ 执行达 2.52 tokens/s。

**核心要点**：
- 用通用 signed-int4 乘累加指令（OP_SMA4）承载 BitNet 的 1.58-bit 算术，避免专用数据通路。
- int8 激活切片为两段 int4 再移位加重建，三值权重单 lane 存放。
- 28nm/840MHz CGLA 下单乘积 0.390ns，C++ 实测 2.52 tokens/s，证明可编程阵列跑超低比特 LLM 可行。

---

## 3. Crossflow：面向 Agentic LLM 服务的 Prefill-Decode 弹性调度

**来源**：arXiv (cs.LG)  
**链接**：https://arxiv.org/abs/2609.27085  
**标签**：P/D分离 · 弹性调度 · Agentic服务 · 吞吐提升 · KV容量

Prefill-Decode（P/D）分离通过两阶段专业化提升服务效率，但依赖静态资源划分，而实际阶段需求高度动态。作者观察到大型 LLM 集群中未命中缓存的输入/输出 token 比在分钟级峰值-均值比达 4.7x，公开 agentic 轨迹日内中位比达 24.5x，而重分配副本需数十分钟。Crossflow 让边界弹性化：每个 decode 节点发布短期可撤销租约，约束本地 prefill 算力、KV 容量与预期输出。在公开与内部轨迹上，Crossflow 相比静态 P/D 几何均值吞吐提升 16.2–17.4%，高负载下最高 43.4%。

**核心要点**：
- 揭示 P/D 静态划分在 agentic 流量下的严重错配：日内 P/D 比波动达 24.5x，重分配副本却需数十分钟。
- 以 decode 节点的短期可撤销租约实现弹性边界，不改变节点角色。
- 几何均值吞吐提升 16.2–17.4%，高负载下最高 43.4%。

---

## 4. PipeLive：动态 LLM 服务中就地实时流水线并行重配置

**来源**：arXiv (cs.LG)  
**链接**：https://arxiv.org/abs/2604.12171  
**标签**：流水线并行 · 实时重配置 · KV-Cache · PageAttention · 无中断

流水线并行（PP）常用于跨 GPU 切分 LLM 层，但现有系统依赖静态配置，难以适配 serverless 与异构 GPU 等动态环境。停服重部署代价过高，重配置必须就地实时进行且不中断推理。PipeLive 提出重设计的 KV-Cache 布局配合 PageAttention 协同扩展，形成统一的实时 KV 缩放机制，在 GPU 已被权重与 KV 占满时仍能安全重排层放置，并在执行中保持 KV 一致性。

**核心要点**：
- 实时就地重配置的核心难点：GPU 已被占满、KV-Cache 需缩放，且与 vLLM 预分配冲突。
- 重设计 KV-Cache 布局 + PageAttention 扩展，统一实现实时 KV 缩放。
- 在不中断推理的前提下完成 PP 配置变更，适配 serverless/异构 GPU。

---

## 5. LM-CXD：用分块感知 KV-Cache 管理桥接 LLM 服务与 CXL-SSD

**来源**：arXiv (cs.AR)  
**链接**：https://arxiv.org/abs/2609.26828  
**标签**：CXL-SSD · 前缀缓存 · KV-Cache · 字节寻址 · 延迟隐藏

NAND 存储可提供 LLM 前缀缓存所需容量，但其块 I/O 路径带来 CPU 缓存争用与主机 DRAM 中转开销。作者论证即便以 DRAM 为介质这些接口开销仍存在，因而采用 CXL-SSD 实现 NAND 容量的字节寻址访问。但原生 CXL-SSD 仍比本地 DRAM 慢约 3x、与 NVMe 相当，普通预取收效甚微。LM-CXD 将 KV 分块设为设备可见的 I/O 单元，向服务引擎暴露 NAND→DRAM 进度，并用设备 DRAM 作 GPU 可访问缓冲；配合窗口化预取与按层 KV 搬运/计算流水，在 5 个模型上 TTFT 较原生 CXL-SSD 最高降低 2.6x。

**核心要点**：
- 揭示前缀缓存走块 I/O 的接口开销（CPU 缓存争用、DRAM 中转），论证 CXL-SSD 字节寻址的必要性。
- LM-CXD 让 KV 分块成为设备可见 I/O 单元，暴露 NAND→DRAM 进度并复用设备 DRAM 作 GPU 缓冲。
- 窗口化预取 + 按层 KV 搬运与计算流水，TTFT 最高降 2.6x。

---

## 6. MicroQonv：重构卷积张量以实现高效的 Microscaling 量化

**来源**：arXiv (cs.AR)  
**链接**：https://arxiv.org/abs/2609.28358  
**标签**：微缩放量化 · 卷积 · im2col · 内存搬运 · 训练推理

Microscaling 量化用 8bit 及以下表示神经网络参数且近全精度，但在卷积层高效应用并不容易：朴素做法把全精度权重/激活搬到计算单元后各量化两次，且 im2col 使激活张量显著膨胀。MicroQonv 将 microscaling 与卷积前/反向计算结合，每张量仅量化一次，并把激活在改进的「通道-批次优先 im2col」之前量化。权重量化成本降 2x、梯度 2x、激活最高 9x，存储与内存搬运较全精度最多降 7.53x，精度损失可忽略。

**核心要点**：
- 朴素卷积量化需双份量化 + im2col 膨胀，内存搬运远超预期。
- MicroQonv 张量仅量化一次，并先把激活在「通道-批次优先 im2col」前量化。
- 权重量化/梯度/激活成本分别降 2x/2x/9x，内存搬运最多降 7.53x。

---

## 7. EMA：跨 GPU 的弹性且性能透明的显存共享

**来源**：arXiv (cs.LG)  
**链接**：https://arxiv.org/abs/2609.27040  
**标签**：多GPU · 显存弹性 · 远程访问 · 预取 · 性能透明

多 GPU 服务器是现代数据中心标准单元，但 LLM 推理等负载显存需求高度动态，常出现一卡耗尽而它卡闲置。EMA 提出显存共享系统，让同机 GPU 相互借还显存形成弹性容量池，并对借贷双方保证性能透明：借用方通过预取隐藏远程访问开销，使远程与本地的性能难以区分；出借方借出资源可随时回收，保证性能不低于静态划分。评估显示单用户吞吐最高提升 52%，达 2 倍容量系统吞吐的 96%，且延迟接近静态划分。

**核心要点**：
- 针对 LLM 推理显存需求动态错配，构建同机 GPU 间可弹性借还的显存池。
- 借用方用预取隐藏远程访问开销；出借方资源可随时回收，性能不低于静态划分。
- 单用户吞吐最高 +52%，达 2x 容量系统 96% 吞吐，延迟接近静态划分。

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | 面向 INT2 KV-Cache 量化的输出感知旋转（OptR） | arXiv | KV量化 |
| 2 | CGLA 上以 Signed-Int4 实现 BitNet 推理 | arXiv | 超低比特推理 |
| 3 | Crossflow：P/D 弹性调度服务 Agentic LLM | arXiv | 服务调度 |
| 4 | PipeLive：就地实时流水线并行重配置 | arXiv | 并行推理 |
| 5 | LM-CXD：CXL-SSD 分块感知 KV-Cache 管理 | arXiv | 存储缓存 |
| 6 | MicroQonv：高效卷积 Microscaling 量化 | arXiv | 量化训练 |
| 7 | EMA：跨 GPU 弹性且性能透明显存共享 | arXiv | GPU显存 |

---

*自动生成 · 2026-09-25 · jeffinchen daily tech reading list*

