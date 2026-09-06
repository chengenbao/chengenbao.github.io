---
layout: reading
title: "Preemptive 调度、KV Cache 复用与边缘推理系统"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-06-29
---

# 📰 2026-06-29 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖 GPU 集群调度、KV Cache 复用、AI 系统论文与端侧推理。

---

## 1. Llumnix：面向 LLM 服务的动态跨请求调度

**来源**：arXiv cs.DC（OSDI'24 论文）
**链接**：https://arxiv.org/abs/2406.09328
**标签**：调度 · 请求迁移 · 多实例 · SLO

Llumnix 把请求调度从单实例扩展到多实例集群：通过实时请求重调度（rescheduling）在实例间迁移请求以响应负载变化，减少排队与抢占开销，提升 SLO 达标率。它把"调度器感知推理语义（KV Cache、执行阶段）"设计成跨实例的一等能力，是多副本推理集群调度方向的代表作。

**核心要点**：
- 推理语义感知的跨实例请求迁移
- 动态负载下的重调度时机与迁移成本建模
- 多实例场景 SLO 达标率与利用率双提升

---

## 2. CacheGen：面向 LLM 上下文的流式 KV Cache 压缩与传输

**来源**：arXiv cs.CL（SIGCOMM'24 论文）
**链接**：https://arxiv.org/abs/2310.07240
**标签**：KV Cache 压缩 · 上下文传输 · 边缘推理 · 编码

CacheGen 把 KV Cache 视为可压缩、可传输的数据：针对 KV 的张量分布设计专用编码，压缩率 3.5-4 倍，配合流式传输让边缘节点快速"预热"长上下文。它是"KV Cache 成为分布式系统一等公民"路线的代表工作，与 PD 分离、跨节点 KV 复用同属一个方向。

**核心要点**：
- KV 张量的熵特性与专用无损/有损编码器
- 流式加载：边传边算，压缩与传输流水线化
- 跨层/跨 token 的压缩率自适应分配

---

## 3. Mooncake：以 KV Cache 为中心的 disaggregated LLM 集群架构

**来源**：FAST'25 论文（Mooncake）
**链接**：https://www.usenix.org/conference/fast25/presentation/qin
**标签**：KV Cache 中心化 · PD 分离 · 调度器 · 月之暗面

Mooncake 把 KV Cache 当作集群全局资源：KVCache-centric 调度器决定请求落到哪个 prefill/decode 节点，优先最大化 KV 复用（跨请求、跨轮次），通过全局 KV 池化提升命中率。在月产千万级请求的生产负载上验证，是 KV 中心化架构路线最有说服力的系统论文。

**核心要点**：
- KV 亲和性调度：长上下文多轮会话的复用最大化
- prefill/decode 资源池的动态比例调节
- 过载时的 SLO-aware 降级与请求准入控制

---

## 4. EdgeLLM：端侧异构设备上的大模型推理综述

**来源**：arXiv cs.LG（端侧推理综述）
**链接**：https://arxiv.org/abs/2312.13223
**标签**：端侧推理 · NPU · 量化 · 移动设备

端侧推理的约束集与数据中心截然不同：内存带宽瓶颈、功耗墙、温度节流。综述梳理端侧 LLM 的关键技术：权重/激活量化（4-bit 为主）、NPU 图编译、内存映射权重加载与投机解码的移动端适配，并对比 llama.cpp、MLC-LLM、ExecuTorch 三大端侧栈的实现差异。

**核心要点**：
- 端侧瓶颈：DRAM 带宽而非算力，decode 速度由带宽决定
- 4-bit 量化内核与 NPU 算子的编译适配
- 内存映射与按需加载减少冷启动时间

---

## 5. 训练-推理共置：GPU 集群利用率提升的系统方案

**来源**：arXiv cs.DC（共置调度研究）
**链接**：https://arxiv.org/abs/2312.12176
**标签**：共置调度 · GPU 利用率 · 抢占 · MIG

训练集群平均利用率常常不到 50%，共置（colocation）让推理服务填进训练任务的空闲间隙：利用训练的同步屏障间隙跑推理、用 MIG/时间片做硬件隔离。文章分析干扰控制与故障域隔离的设计，是 GPU 集群成本优化的高价值方向。

**核心要点**：
- 训练同步屏障间隙的算力空窗测算
- 硬件隔离（MIG）vs 软件隔离（CUDA MPS）的干扰对比
- 抢占边界：推理请求在训练 step 边界的换入换出

---

## 6. Medusa：多 decoder 头的无损 LLM 解码加速

**来源**：arXiv cs.LG（Princeton 论文）
**链接**：https://arxiv.org/abs/2401.10774
**标签**：投机解码 · 流水线并行 · 长上下文 · 系统组合

Medusa 在目标模型上附加多个轻量 decoder 头并行预测多 token，配合 tree attention 一次验证，无需独立草稿模型即可加速解码。它的"自投机"设计避免了草稿模型与目标模型的分布对齐难题，是 2024-2026 年投机解码工业化的重要分支。

**核心要点**：
- 多 decoder 头的并行多 token 预测结构
- Tree attention 批量验证与树型接受采样
- 免草稿模型：训练成本低、部署拓扑简单

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | Llumnix：面向 LLM 服务的动态跨请求调度 | arXiv cs.DC | 调度 |
| 2 | CacheGen：KV Cache 压缩与流式传输 | arXiv cs.CL | KV 压缩 |
| 3 | Mooncake：KV Cache 为中心的集群架构 | FAST'25 | KV 中心化 |
| 4 | EdgeLLM：端侧异构设备推理综述 | arXiv cs.LG | 端侧推理 |
| 5 | 训练-推理共置的利用率方案 | arXiv cs.DC | 共置调度 |
| 6 | Medusa：多 decoder 头的无损解码加速 | arXiv cs.LG | 投机解码 |

---

*自动生成 · 2026-06-29 · jeffinchen daily tech reading list*
