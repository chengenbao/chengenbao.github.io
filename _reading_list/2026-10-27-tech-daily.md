---
layout: reading
title: "LLM 推理与训练系统：KV-Cache 存储、异构加速、注意力内核与量化"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-10-27
---

# 📰 2026-10-27 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 LLM 服务 KV-Cache 存储优化、GPU-NPU 异构推理加速、FlashAttention 长向量内核、LLM 强化学习数值稳定性与边缘三元量化。

---

## 1. LLM 服务中 KV-Cache SSD 卸载的可预测性与延迟权衡（以 LMCache 为例）

**来源**：cs.OS (arXiv)  
**链接**：http://arxiv.org/abs/2610.21249v1  
**标签**：KV-Cache · SSD卸载 · LLM服务 · LMCache · 延迟优化

本文首次在块设备层级对 LMCache 的 KV-Cache SSD 卸载行为进行系统刻画。作者通过覆盖『相同/不同上下文 × 相同/不同提示词』的真实多会话负载（每类超 100 个无状态请求），对比了 LMCache 0.3.0（缓冲 I/O + 内核页缓存）与 0.3.16（前缀去重 + O_DIRECT）两种可落地部署快照，给出了关于可预测性与延迟之间权衡的关键发现，为大规模 LLM 服务落地 KV 卸载提供了实测依据。

**核心要点**：
- 首次在块设备层面量化 LMCache 的 KV-Cache 卸载行为，而非仅看上层命中率
- 通过真实多会话工作负载揭示不同上下文/提示组合下的延迟与可预测性差异
- 对比新旧版本（缓冲 I/O vs O_DIRECT + 前缀去重）给出明确部署收益指引

---
## 2. Range：解耦 KV-Cache 复用粒度与存储开销

**来源**：cs.OS (arXiv)  
**链接**：http://arxiv.org/abs/2610.21241v1  
**标签**：KV-Cache · NVMe · TTFT · 存储栈 · 前缀复用

LLM 服务通过复用缓存的 KV 状态跳过 prefill，但增长中的上下文把状态推过 GPU HBM 与 CPU DRAM、落到 NVMe SSD，使前缀恢复成为首 token 延迟（TTFT）的关键路径。论文发现：在配置良好的主机下发路径上，SSD 并非瓶颈——同样 6 GiB 的 KV，以 192 个对象恢复需 2.1s，以 6144 个对象恢复则需 3.0s 且分层加载下更慢，因为存储栈每一层（查找、分配、I/O 提交、GPU 投递、存储、索引）都按逻辑对象计数收费而字节数不变。Range 据此解耦复用粒度与存储工作。

**核心要点**：
- 揭示 KV-Cache 前缀恢复耗时随逻辑对象数而非字节数增长的根因
- 证明在优化路径上 NVMe 带宽并非 TTFT 瓶颈，瓶颈在存储栈的对象开销
- 提出解耦复用粒度与存储工作的设计，降低大上下文下的首 token 延迟

---
## 3. HAWK：面向 GPU-NPU 异构分阶段 LLM 推理的 KV-Cache 传输优化

**来源**：cs.AR (arXiv)  
**链接**：http://arxiv.org/abs/2610.29873v2  
**标签**：GPU-NPU · 异构推理 · KV-Cache · 推理加速 · 阶段分离

LLM 推理的两个阶段特性迥异：prefill 计算密集，decode 更偏内存带宽受限。GPU 擅长 prefill 却在 decode 阶段利用率不足，而 NPU 在 decode 阶段能更高效利用算力。HAWK 据此探索将 prefill 映射到 GPU、decode 映射到 NPU 的异构架构，并提出异构感知的 KV-Cache 传输机制，在不同加速器间高效搬移中间状态，从而提升整体推理吞吐与资源利用率。

**核心要点**：
- 指出 prefill/decode 两阶段对 GPU 与 NPU 的算力利用率差异
- 提出 GPU 做 prefill、NPU 做 decode 的阶段分离异构部署方案
- 设计异构感知的 KV-Cache 传输策略以降低跨设备状态搬移开销

---
## 4. 重构 FlashAttention 以适配长向量处理器

**来源**：cs.AR (arXiv)  
**链接**：http://arxiv.org/abs/2610.29489v1  
**标签**：FlashAttention · 向量化 · RISC-V · Transformer · 注意力内核

Transformer 的核心注意力开销随序列长度平方增长，FlashAttention 通过分块并更新在线 softmax 将其变得可控，但此前几乎所有工作都面向 GPU。在长向量处理器上，一个向量寄存器可容纳多达 512 个单精度数，向量指令的节省正比于寄存器被使用的比例，因此『对哪个循环做向量化』成为一阶决策。本文将 PyTorch 的 CPU FlashAttention 内核移植到 RISC-V 向量扩展（RVV），通过重排向量轴来重构注意力计算，使其适配长向量架构。

**核心要点**：
- 指出长向量处理器上向量化循环方向对性能的决定性影响（一阶优化）
- 将 PyTorch CPU FlashAttention 内核移植到 RISC-V 向量扩展并重构向量轴
- 为 GPU 之外的长向量架构提供高效注意力内核实现参考

---
## 5. 舍入位置对齐：稳定 LLM 强化学习训练的关键

**来源**：cs.LG (arXiv)  
**链接**：http://arxiv.org/abs/2610.32951v1  
**标签**：LLM训练 · 强化学习 · BF16 · 数值精度 · 推理-训练对齐

LLM 的强化学习常使用相互独立的推理与训练系统。即便权重与 BF16 张量完全一致，实现细节差异仍可能把本应 on-policy 的更新变成 off-policy，从而 destabilize 训练。现有方案要么对齐整个推理与训练栈、要么改全局精度、要么改学习算法。本文质疑：仅对齐执行栈的选定部分能否在保留其他实现差异的同时稳定 BF16 RL？作者将训练侧在 RoPE、SwiGLU 与某个区域的舍入位置与推理对齐，验证了局部舍入位置对齐即可显著提升训练稳定性。

**核心要点**：
- 揭示推理/训练栈实现差异（舍入位置）会悄悄把 on-policy 变成 off-policy
- 提出仅对齐 RoPE、SwiGLU 等关键区域的舍入位置即可稳定 BF16 RL
- 相比对齐全栈或改精度/算法，局部对齐更轻量且保留工程灵活性

---
## 6. TernScope：面向边缘医疗分割的三元量化模型-硬件协同设计

**来源**：cs.AR (arXiv)  
**链接**：http://arxiv.org/abs/2610.29617v1  
**标签**：三元量化 · 模型-硬件协同 · 边缘推理 · 量化感知训练 · 医疗分割

边缘医疗分割必须在小模型体积、高精度与快速推理之间取得平衡。TernScope 通过模型-硬件协同设计应对这一权衡：学习一个紧凑的分割网络，其低位宽算子同时简化硬件执行；split-channel 大核块在有限计算下保留空间上下文，分支融合消除冗余推理路径。进而用三元渐进式量化感知训练（TPQ）从深层到浅层、经解码器再到输入/预测层逐步适配，使全部 25 个卷积层具备三元高效性，兼顾精度与边缘可部署性。

**核心要点**：
- 以模型-硬件协同设计同时压缩模型并简化低位宽算子执行
- split-channel 大核块 + 分支融合在低成本下保留空间上下文
- 三元渐进式量化感知训练（TPQ）分层适配，全部 25 个卷积层达三元高效

---
## 7. uGDS：面向高性能 GPU 直连存储的用户态开发套件

**来源**：cs.OS (arXiv)  
**链接**：http://arxiv.org/abs/2610.30868v2  
**标签**：GPU存储 · GDS · NVMe · 用户态 · PCIe DMA

数据密集的 GPU 应用越来越依赖 NVMe SSD 扩展显存。NVIDIA GPU Direct Storage（GDS）通过 PCIe 对等 DMA 实现 SSD 到 GPU 的直接传输、省去主机中转，但存在三大局限：控制路径每次 I/O 都穿越内核带来软件开销；流有序路径即便只需批级有序也强制严格全序；闭源用户态引擎阻碍诊断与自定义调度。本文提出开源的 uGDS（https://github.com/ScaleX-IO/uGDS），让用户态应用精细化控制 GPU 直连存储的 I/O 路径。

**核心要点**：
- 剖析 GDS 在控制路径开销、过度有序、闭源三方面的关键局限
- 开源 uGDS 将 GPU 直连存储的 I/O 控制下放到用户态
- 支持批级有序与自定义调度，降低 SSD-GPU 直传的软件开销

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | LLM 服务中 KV-Cache SSD 卸载的可预测性与延迟权衡（以 LMCache 为例） | cs.OS (arXiv) | KV-Cache/SSD卸载 |
| 2 | Range：解耦 KV-Cache 复用粒度与存储开销 | cs.OS (arXiv) | KV-Cache/NVMe |
| 3 | HAWK：面向 GPU-NPU 异构分阶段 LLM 推理的 KV-Cache 传输优化 | cs.AR (arXiv) | GPU-NPU/异构推理 |
| 4 | 重构 FlashAttention 以适配长向量处理器 | cs.AR (arXiv) | FlashAttention/向量化 |
| 5 | 舍入位置对齐：稳定 LLM 强化学习训练的关键 | cs.LG (arXiv) | LLM训练/强化学习 |
| 6 | TernScope：面向边缘医疗分割的三元量化模型-硬件协同设计 | cs.AR (arXiv) | 三元量化/模型-硬件协同 |
| 7 | uGDS：面向高性能 GPU 直连存储的用户态开发套件 | cs.OS (arXiv) | GPU存储/GDS |

---

*自动生成 · 2026-10-27 · jeffinchen daily tech reading list*

