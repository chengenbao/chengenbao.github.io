---
layout: reading
title: "长上下文稀疏注意力与 KV Cache 压缩 · 端侧投机解码 · 推理解聚与 GPU 网络通信 · 异构内存页面放置"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-11-07
---

# 📰 2026-11-07 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖长上下文稀疏注意力与 KV Cache 压缩、端侧投机解码、推理服务解聚、GPU 原生网络通信、异构内存页面放置、Agent token 成本度量。

---

## 1. Recursive Sparse Attention: 百万 Token 长度外推的对数级代价

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2611.07232
**标签**：稀疏注意力 · 长上下文 · KV Cache · 长度外推 · 推理加速

在约 1M token 的上下文长度下，稠密注意力在 prefill 和 decoding 阶段都代价高昂，且训练所需的长文档随目标长度增加愈发稀缺。大多数稀疏注意力方法需要对所有历史 block 或 token 打分，查询选择代价为 O(L)；层次化选择可将代价降到 O(log L)，但此前可训练的层次方法仅在训练长度内报告精度。本文提出 RecSA（递归稀疏注意力），在 KV cache 上构建树结构，每个节点用一个 key 和均值池化 value 总结子树，各层池化器共享并蒸馏自稠密注意力。每个层级仅选择至多 k 个候选，使每个查询的选择与读取代价均为 O(log L)。

**核心要点**：
- 构建 KV cache 树，每层节点以单一 key + 均值池化 value 总结子树，查询仅需 O(log L) 代价
- 每层共享的池化器由稠密注意力蒸馏而来，支持超出训练长度的长度外推
- 在不牺牲精度的前提下，将长上下文推理的注意力开销从线性降到对数级

---

## 2. HyFlash: 面向端侧部署的前缀记忆先验并行投机解码

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2611.07263
**标签**：投机解码 · 端侧推理 · Draft 模型 · 显存优化 · 推理加速

并行 block drafting 通过单次主干前向预测多个未来位置来加速投机解码，但端侧部署同时约束了 drafter 容量与上下文存储。在严格显存预算下，drafter 必须在有限容量内建模长程依赖与细粒度 draft 位置交互，而不产生随上下文增长的 KV-cache 开销。本文提出 HyFlash，一种紧凑的并行 drafter，引入前缀记忆先验将长程上下文建模与细粒度 draft 位置交互解耦：局部注意力捕捉近期上下文与位置交互，循环记忆将历史 target 特征压缩为固定大小状态，从而在受限显存下实现有效的局部与长程建模。

**核心要点**：
- 前缀记忆先验解耦长程上下文建模与 draft 位置交互，避免 full-context 注意力带来的 KV-cache 膨胀
- 循环记忆将历史特征压缩为固定大小状态，实现有界的 draft 侧显存占用
- 在端侧严格显存预算下取得更高的 draft 接受率，加速投机解码

---

## 3. LowRAM: 测试时可组合 KV Cache 压缩的低秩 Key 与摊还 Value

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2611.07025
**标签**：KV Cache 压缩 · 低秩 · 测试时优化 · 显存 · 推理吞吐

自回归解码依赖 KV cache，其显存占用随上下文长度线性增长，限制了 batch size 与吞吐。现有训练后方法利用 token、层和特征维度的冗余或降低数值精度。本文发现一个互补维度：在每个注意力头内，value 很大程度上可由其对应 key 预测。由此提出 value cache 摊还——在测试时拟合轻量逐头预测器，仅存储其重建最差 token 的残差，实现约 4× 近无损 value 压缩。预测器可由元学习初始化以降低 prefill 开销，或从零拟合于允许一次性成本的场景（如多查询推理）。再结合跨层低秩 key 压缩，形成 LowRAM 方案。

**核心要点**：
- 利用注意力头内 value 可由 key 预测的特性，以逐头预测器 + 残差存储实现 4× 近无损 value 压缩
- 预测器支持元学习初始化（低 prefill 开销）或从零拟合（一次性成本场景）
- 跨层低秩 key 压缩与 value 摊还组合，系统性降低 KV cache 显存占用、提升吞吐

---

## 4. Should You Split Decode? 跨未来模型与加速器的推理解聚分析

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2611.08299
**标签**：推理解聚 · P/D 分离 · 服务架构 · 加速器 · 部署

大模型推理中，prefill/decode 解聚（P/D）已在生产环境部署以隔离不同资源需求的阶段。注意力/前馈网络解聚（AFD）进一步拆分 decode，但引入了通信、放置约束和额外的资源池。本文探讨：随着模型与加速器路线图演进，角色专用放置是否能收回成本。基础设施在 future 模型与硬件可基准测试前就已确定。作者提出 SiliconForge，一个可独立变化服务分解与硬件分配的标定模型，在四种模型方向、四类锚定当前与已发布零件的加速器、两种互联 fabric 上比较 P/D 与 P/D+AFD，并以相同规则做流水线重叠与功耗预算协同优化。

**核心要点**：
- 提出 SiliconForge 标定模型，可独立变化服务分解与硬件分配，前瞻评估解聚收益
- 覆盖四种模型方向、四类加速器（当前与已发布）与两种互联，统一功耗预算协同优化
- 为『是否进一步拆分 decode（AFD）』给出面向未来模型/硬件的部署决策依据

---

## 5. OpenRDMA: GPU 原生的多路径可靠 RDMA 传输架构

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2611.07559
**标签**：RDMA · GPU 通信 · AI 集群 · 网络 · FPGA NIC

随着大规模 AI 集群算力密度上升，网络通信日益成为系统性能瓶颈。GPU 发起的通信需要低开销的请求提交以重叠通信与计算。然而 GPUDirect RDMA（GDR）控制路径仍依赖 CPU，而 InfiniBand GPUDirect Async（IBGDA）要求 GPU 组装 Work Queue Entries（WQE），消耗额外 SM 资源与显存带宽。本文提出 OpenRDMA，一个开源、面向 GPU 友好的多路径可靠 RDMA 传输架构：提供极简 GPU doorbell 机制（GPU 发出 8 字节 token，专用 FPGA NIC 从预配置上下文重建完整 WQE）；并基于多路径可靠连接（MRC）支持逐包多路径传输与乱序交付。

**核心要点**：
- 极简 GPU doorbell：GPU 仅发 8 字节 token，FPGA NIC 重建完整 WQE，消除 CPU 控制路径依赖
- 基于 MRC 支持逐包多路径传输与乱序交付，提升大规模 AI 集群网络利用率
- 开源 GPU 原生 RDMA 架构，降低通信提交开销、更好重叠通信与计算

---

## 6. Memory Fit: 通过学习行为模型支持页面放置的体系结构抽象

**来源**：arXiv cs.OS
**链接**：https://arxiv.org/abs/2611.07506
**标签**：异构内存 · CXL · 页面放置 · 操作系统 · 体系结构

服务器主存日益异构，包含 zswap、CXL 内存扩展器与新兴相变存储器（PCM）等。为决定页面放置位置，操作系统需在高效的粗粒度访问位、带外 profiling 与昂贵的运行时插桩之间取舍。本文提出 memory fit——一个向操作系统提供精确页面放置信息的硬件抽象，衡量一个页面与系统中各类内存的『契合度』，并将『测量页面-内存亲和性』的机制与『选择移动哪些页面』的 OS 策略解耦。论文描述的软trace匹配（STM）是一种计算与空间高效的算法，可通过采样估计 memory fit。

**核心要点**：
- 提出 memory fit 硬件抽象，为异构内存（CXL/PCM/zswap）提供精确的页面契合度度量
- 将『测量亲和性』的机制与『选择移动页面』的 OS 策略解耦，提升策略灵活性
- 软 trace 匹配（STM）以采样方式高效估计 memory fit，避免昂贵运行时插桩

---

## 7. How Do AI Agents Waste Your Money? 智能体 Token 浪费的统一度量框架

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2611.07238
**标签**：AI Agent · Token 成本 · 推理效率 · LLM-as-Judge · 可观测

AI 智能体在各类任务中可能消耗海量 token，但其中多少是真正必要的？尽管已有工作表明智能体可能产生意料之外的高 token 成本，仍缺乏系统化识别与度量 token 浪费的方法。本文提出一个跨异构智能体系统度量 token 浪费的统一框架：记录并归一化完整 agent 轨迹，使用 LLM-as-a-Judge（LLMaJ）识别可移除片段，并将识别出的文本片段映射回原始轨迹做 token 级度量。通过直接干预（移除标记片段、恢复执行并测量任务结果与后续动作变化）验证浪费。在线剪枝实验中，移除 41-46% 的推理 token 而基本保持任务性能。

**核心要点**：
- 提出统一框架，以 LLM-as-a-Judge 识别智能体轨迹中可移除的 token 浪费片段
- 将识别片段映射回原始轨迹做 token 级度量，并通过干预实验验证有效性
- 在线剪枝可移除 41-46% 推理 token 而基本保持任务性能，直接降本

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | Recursive Sparse Attention: 百万 Token 长度外推的对数级代价 | arXiv cs.CL | 长上下文推理 |
| 2 | HyFlash: 面向端侧部署的前缀记忆先验并行投机解码 | arXiv cs.CL | 端侧推理 |
| 3 | LowRAM: 测试时可组合 KV Cache 压缩的低秩 Key 与摊还 Value | arXiv cs.LG | KV Cache 压缩 |
| 4 | Should You Split Decode? 跨未来模型与加速器的推理解聚分析 | arXiv cs.AR | 推理服务 |
| 5 | OpenRDMA: GPU 原生的多路径可靠 RDMA 传输架构 | arXiv cs.AR | GPU 网络 |
| 6 | Memory Fit: 通过学习行为模型支持页面放置的体系结构抽象 | arXiv cs.OS | 异构内存 |
| 7 | How Do AI Agents Waste Your Money? 智能体 Token 浪费的统一度量框架 | arXiv cs.CL | Agent 成本 |


*自动生成 · 2026-11-07 · jeffinchen daily tech reading list*
