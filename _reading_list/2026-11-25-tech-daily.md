---
layout: reading
title: "大模型推理加速与系统优化前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-11-25
---

# 📰 2026-11-25 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 大模型推理加速、KV Cache 管理、稀疏注意力、GPU 显存剖析、后训练调参与分布式算力回收。

---

## 1. 块数无关的精度感知学习率迁移：LLM 后训练的高效调参

**来源**：arXiv/cs.CL
**链接**：https://arxiv.org/abs/2611.31166v1
**标签**：后训练 · 学习率 · SFT · DPO · 超参迁移

LLM 后训练（SFT/DPO/RLVR）对层数（block count）和学习率极度敏感，跨不同模型深度迁移超参代价高昂。本文提出精度感知的学习率跨块数迁移方法，使在小块数上验证的学习率可直接、稳定地迁移到更大块数配置，大幅降低后训练调参成本并保持收敛质量。

**核心要点**：
- 后训练效果对 block count 与学习率组合高度敏感，迁移成本高
- 提出精度感知的学习率迁移公式，跨不同层数配置保持稳定
- 在 SFT/DPO/RLVR 场景下显著降低调参开销且不损收敛质量

---

## 2. Corrective Prefix Attention：并行投机解码的纠错前缀注意力

**来源**：arXiv/cs.CL
**链接**：https://arxiv.org/abs/2611.31112v1
**标签**：投机解码 · 并行解码 · 推理加速 · LLM · 注意力机制

并行投机解码一次性提出整块 token，但早期一个不一致就可能让目标模型拒绝剩余后缀。本文提出 Corrective Prefix Attention (CPA)，通过纠错前缀注意力机制在块内早期不一致时自动修正，避免整块被拒，从而显著提升并行投机解码的接受率与端到端推理吞吐。该方法是纯注意力层的轻量改造，易于集成到现有解码框架。

**核心要点**：
- 并行投机解码中单点错误会导致整块 token 被拒绝，浪费已计算量
- CPA 引入纠错前缀注意力，在块内早期不一致时动态修正而非整体回退
- 作为注意力层轻量改动，可无缝接入现有投机解码管线并提升接受率与吞吐

---

## 3. ANOR：基于 GPU-PNM 计算层级的稀疏注意力扩展

**来源**：arXiv/cs.AR
**链接**：https://arxiv.org/abs/2611.30417v1
**标签**：稀疏注意力 · 长上下文 · KV Cache · GPU · 近内存计算

长上下文 LLM 服务需为每个请求保留 KV Cache，极易超出 GPU 显存。ANOR 提出一种 GPU 与近内存计算（PNM）协同的层级架构，仅读取与查询相关的少量稀疏 KV，将大部分 KV 驻留于 PNM 侧并按需供给，从而在有限显存下支撑超长上下文推理并降低访存瓶颈。

**核心要点**：
- 长上下文服务的 KV Cache 占用常超出单卡 GPU 显存容量
- ANOR 以 GPU-PNM 层级把稀疏 KV 下沉到近内存侧按需供给
- 在有限显存下实现稀疏注意力扩展，缓解访存墙、支持超长上下文

---

## 4. 在 NVIDIA GB10 上无 C2C 遥测测量 GPU 内存流量

**来源**：arXiv/cs.AR
**链接**：https://arxiv.org/abs/2611.29915v1
**标签**：GPU · 内存带宽 · Blackwell · NVLink C2C · 性能剖析

NVIDIA GB10（用于 DGX Spark 等桌面 AI 系统）通过 NVLink C2C 将 Arm CPU 与 Blackwell GPU 互联，GPU 无本地显存，每次访问都需跨 C2C。本文提出无需 C2C 遥测即可测量 GPU 内存流量的方法，为这类统一内存架构的性能剖析与瓶颈定位提供实用工具。

**核心要点**：
- GB10 的 GPU 无本地显存，所有访问都经 NVLink C2C，流量难以直接观测
- 提出无需 C2C 遥测的 GPU 内存流量测量方法
- 为统一内存 AI 桌面系统的性能剖析与瓶颈定位提供实用手段

---

## 5. Contexture：面向高吞吐 Agent 服务的运行时上下文驱逐

**来源**：arXiv/cs.OS
**链接**：https://arxiv.org/abs/2611.26980v1
**标签**：Agent 服务 · KV Cache · 上下文驱逐 · HBM · 高吞吐

Hermes、OpenClaw、Codex 等 LLM Agent 反复重发不断增长的执行历史，推高共享 GPU 的 KV Cache 需求。Contexture 提出运行时上下文驱逐机制，在 HBM 压力下智能丢弃/重载低频上下文片段，使多 agent 高并发服务在有限显存下维持高吞吐，避免缓存膨胀导致的内存溢出与降级。

**核心要点**：
- Agent 反复重发增长历史，共享 KV Cache 在 HBM 中持续膨胀
- Contexture 在运行时按压力智能驱逐与重载低频上下文片段
- 在有限显存下维持多 agent 高并发服务的高吞吐与稳定性

---

## 6. TripleKV：面向 AI Agent 的 KV Cache 预测-预取-保护管理

**来源**：arXiv/cs.OS
**链接**：https://arxiv.org/abs/2611.25112v2
**标签**：KV Cache · Agent · 推理服务 · 预取 · 显存管理

LLM Agent 在推理与工具调用间交替，跨调用反复复用不断增长的上下文。高并发下其 KV Cache 工作集可达数百 GB。TripleKV 提出预测、预取、保护三位一体的 KV Cache 管理策略，精准复用 agent 历史上下文、提前预取热点段并隔离保护关键缓存，显著降低显存压力与尾延迟。

**核心要点**：
- Agent 反复复用增长上下文，高并发下 KV 工作集可达数百 GB
- 预测+预取+保护三重机制精准复用历史上下文并提前加载热点
- 在显存受限场景下降低压力与尾延迟，提升 agent 服务吞吐

---

## 7. 回收专家并行气泡用于微调：GPU 闲置算力的再利用

**来源**：arXiv/cs.OS
**链接**：https://arxiv.org/abs/2611.24496v1
**标签**：专家并行 · GPU 利用率 · 微调 · MoE · 算力回收

GPU 昂贵，但延迟敏感的 LLM 推理会让部分算力空闲。本文提出回收专家并行（expert-parallel）中的气泡，将空闲容量用于运行尽力而为的任务（如微调），使推理与微调在同一批 GPU 上安全共置，在不影响在线推理 SLA 的前提下提升整体利用率。

**核心要点**：
- 延迟敏感 LLM 推理在专家并行中留下可利用的空闲算力气泡
- 提出回收气泡机制，将微调等尽力而为任务共置到同一 GPU 集群
- 在不破环在线推理 SLA 的前提下显著提升 GPU 整体利用率

---


## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | 块数无关的精度感知学习率迁移：LLM 后训练的高… | arXiv/cs.CL | 推理加速 |
| 2 | Corrective Prefix Attent… | arXiv/cs.CL | 后训练调参 |
| 3 | ANOR：基于 GPU-PNM 计算层级的稀疏注… | arXiv/cs.AR | GPU显存 |
| 4 | 在 NVIDIA GB10 上无 C2C 遥测测… | arXiv/cs.AR | Agent服务 |
| 5 | Contexture：面向高吞吐 Agent 服… | arXiv/cs.OS | 稀疏注意力 |
| 6 | TripleKV：面向 AI Agent 的 K… | arXiv/cs.OS | 算力回收 |
| 7 | 回收专家并行气泡用于微调：GPU 闲置算力的再利… | arXiv/cs.OS | 上下文管理 |


*自动生成 · 2026-11-25 · jeffinchen daily tech reading list*
