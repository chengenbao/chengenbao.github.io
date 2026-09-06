---
layout: reading
title: "MoE 推理优化全景、路由与专家并行通信"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-06-23
---

# 📰 2026-06-23 · 每日技术速递

> 今日精选 6 篇深度技术文章，聚焦 MoE 推理优化、专家并行通信、路由算法与训练推理一体化。

---

## 1. MoE 推理优化：路由、专家并行与通信全景指南

**来源**：技术博客（MoE 推理优化指南）
**链接**：https://charlestar.github.io/2026/05/13/MoE%E6%8E%A8%E7%90%86%E4%BC%98%E5%8C%96%E5%85%A8%E6%99%AF%E6%8C%87%E5%8D%97/
**标签**：MoE · 专家并行 · All-to-All · 推理优化

从" Dense 矩阵乘变成路由 + 重排 + All-to-All + 大小不一的专家 GEMM"讲起，系统梳理 MoE 推理的五大优化方向：路由 kernel、专家 GEMM 分组、All-to-All 通信重叠、KV 与专家缓存、动态负载均衡。是理解 2026 年 MoE serving 复杂度的最佳单篇。

**核心要点**：
- MoE 推理数据流：gate → dispatch → expert GEMM → combine
- All-to-All 通信与专家计算的四种重叠策略
- 专家热点的缓存与迁移：Expert Offload 的成本模型

---

## 2. Janus：面向可扩展 MoE 推理的注意力-专家解耦

**来源**：arXiv cs.DC（系统论文）
**链接**：https://qzweng.github.io/assets/pdf/2025.arXiv-Janus-Zhang.pdf
**标签**：MoE Serving · 解耦 · 异构资源 · 弹性扩展

Janus 发现 MoE 模型中注意力与专家层的资源画像截然不同（访存 vs 计算、稳定 vs 突发），提出将两者解耦到独立资源池的 disaggregated 架构，各自独立扩缩容。与 PD 分离同属"按阶段解耦"的架构思潮，对大规模 MoE 集群设计极具启发性。

**核心要点**：
- 注意力层与专家层的资源画像差异量化分析
- 解耦架构的资源分配与 KV/激活数据流设计
- 弹性扩缩容：专家池无状态化的部署收益

---

## 3. Megablocks：MoE 训练的高效稀疏内核

**来源**：arXiv cs.LG（MLSys'23 论文）
**链接**：https://arxiv.org/abs/2211.15841
**标签**：MoE · 稀疏计算 · 分组 GEMM · Dropless

Megablocks 用 grouped GEMM + 块稀疏数据结构实现 dropless MoE：不同专家的 token 数不均时不再丢弃 token，而是把专家权重组织成对角块矩阵，一次 GEMM 处理所有专家。它解决了 MoE 训练的负载不均痛点，其 grouped GEMM 内核也是今天 MoE 推理内核（如 vLLM fused_moe）的基础。

**核心要点**：
- 把变长专家 batch 表达为块对角矩阵乘
- Dropless MoE：不丢 token 的负载不均处理
- grouped GEMM 内核的实现细节与 GPU 利用率分析

---

## 4. DeepSeek-V3 技术报告：MoE 架构与负载均衡

**来源**：arXiv cs.CL（技术报告）
**链接**：https://arxiv.org/abs/2412.19437
**标签**：MoE · MLA · 无辅助损失均衡 · 技术报告

DeepSeek-V3 披露了多项被广泛引用的 MoE 工程实践：细粒度专家 + 共享专家的混合路由、无辅助损失的动态偏置均衡策略、MLA 注意力与 FP8 混合训练。其 256 专家、每 token 8 激活的配置成为后续开源 MoE 模型的参照系，报告中的 infra 章节同样是推理优化的必读材料。

**核心要点**：
- 细粒度专家切分：增加组合灵活性同时保持专家容量
- 无辅助损失负载均衡：纯动态偏置的在线调节
- FP8 训练与 DualPipe 流水线的工程细节

---

## 5. LLM 推理优化白皮书： PD 分离架构的吞吐跃迁

**来源**：KM 技术专栏（一念 LLM PD 分离实践）
**链接**：https://km.woa.com/articles/show/648421
**标签**：PD 分离 · SLO · 吞吐优化 · DeepSeek

在 SLO（TPOT=50ms）约束下，PD 分离架构让 DeepSeek-R1 服务吞吐较混部 SGLang 提升 88-90%。文章剖析 prefill 与 decode 的计算特性差异、KV 传输的实现与成本、以及 1P1D/N P N D 部署拓扑的选型逻辑，是国内一线团队的实战复盘。

**核心要点**：
- 混部场景下 prefill 对 decode 的干扰量化
- KV Cache 传输：RDMA 直传 vs 共享存储的取舍
- 部署拓扑（1P1D、N P N D）与集群利用率的权衡

---

## 6. Switch Transformer：千亿参数模型的简单高效稀疏路由

**来源**：arXiv cs.LG（JMLR 经典论文）
**链接**：https://arxiv.org/abs/2101.03961
**标签**：MoE · Top-1 路由 · 训练效率 · 经典

Switch Transformer 将 MoE 路由简化到极致：每 token 只选 1 个专家（Top-1），配合 expert capacity 与 aux loss 稳定训练。它证明了稀疏激活的 scaling law，是所有 MoE 模型的起点，其 top-k 路由与容量因子的设计仍被今天的路由 kernel 沿用。

**核心要点**：
- Top-1 路由的训练稳定性技巧与容量因子分析
- MoE 的预训练 scaling law：同算力下 4-7 倍加速
- 路由的数值实现：softmax 与负载均衡损失

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | MoE 推理优化：路由、专家并行与通信全景 | 技术博客 | MoE |
| 2 | Janus：注意力-专家解耦的可扩展 MoE 推理 | arXiv cs.DC | 解耦架构 |
| 3 | Megablocks：MoE 训练的高效稀疏内核 | arXiv cs.LG | 稀疏计算 |
| 4 | DeepSeek-V3 技术报告：MoE 架构与负载均衡 | arXiv cs.CL | 技术报告 |
| 5 | PD 分离架构的吞吐跃迁实战 | KM 专栏 | PD 分离 |
| 6 | Switch Transformer：简单高效稀疏路由 | arXiv cs.LG | 经典论文 |

---

*自动生成 · 2026-06-23 · jeffinchen daily tech reading list*
