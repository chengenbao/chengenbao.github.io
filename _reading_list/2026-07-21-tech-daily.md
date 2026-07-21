---
layout: reading
title: "长上下文推理、端侧 LLM、MoE 与 RL 训练前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-07-21
---

# 📰 2026-07-21 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 长上下文稀疏注意力、端侧/边缘 LLM 推理、分布式 MoE、结构化剪枝、KV Cache 复用、GPU Kernel 生成与百万级 token RL 训练。

---

## 1. FlashMemory-DeepSeek-V4：基于前瞻稀疏注意力的超长上下文显存优化

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2606.09079
**标签**：长上下文推理 · 稀疏注意力 · KV Cache · 显存优化 · DeepSeek-V4

针对超长上下文解码时全量 KV Cache 驻留显存导致的严重瓶颈，本文提出 Lookahead Sparse Attention (LSA) 推理范式：用一个基于 DeepSeek-V4 架构的 Neural Memory Indexer 主动预测未来上下文需求，仅把查询关键的 KV 分块保留在 GPU 显存中。关键在于采用 backbone-free decoupled training —— 将 indexer 形式化为标准双编码器，使用常规检索训练框架独立训练，无需加载主干模型权重，大幅降低训练成本。

**核心要点**：
- LSA 主动预测并只保留查询关键的 KV 分块，显著降低超长上下文解码的 GPU 显存占用
- Neural Memory Indexer 以双编码器形式解耦训练，不依赖主干模型加载，训练成本低
- 面向百万级 token 推理场景，缓解长上下文服务中的显存墙问题

---

## 2. SelectInfer：面向端侧 LLM 的神经元级选择性加载与计算

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2607.18081
**标签**：端侧推理 · 模型压缩 · 神经元选择 · 边缘设备 · LLM 部署

LLM 在资源受限的边缘设备上部署时面临巨大的算力与显存压力。现有压缩方法多依赖粗粒度剪枝或量化，易损失精度或需要重训练。SelectInfer 提出神经元级优化框架：通过离线 LLM profiler 识别任务相关与通用神经元，在推理时仅选择性加载并计算必要神经元，在保持精度的同时大幅削减端侧开销。

**核心要点**：
- 神经元级细粒度优化，区别于粗粒度剪枝/量化，精度损失更小
- 离线 profiler 区分任务特定神经元与通用神经元，指导运行时选择性加载
- 无需重训练/微调即可在边缘设备实现高效 LLM 推理

---

## 3. OrderMoE：专家相似度驱动的分布式边端 MoE 推理

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2607.17154
**标签**：MoE · 分布式推理 · 边缘计算 · 通信开销 · 专家分配

MoE 模型虽能以适中算力扩展 LLM，但在带宽受限的边缘基础设施上部署推理仍具挑战。现有分布式 MoE 服务依赖精确专家放置、缓存、复制或通信调度，却忽略了专家间的功能相似性。OrderMoE 提出相似度感知的专家分配与分布式部署框架，利用专家功能相似性减少跨服务器 token 传输，在推理延迟、通信开销与服务器负载间取得平衡。

**核心要点**：
- 首次利用专家功能相似性降低跨服务器 token 传输量
- 相似度感知的专家分配框架，兼顾延迟、通信与负载均衡
- 面向带宽受限的边缘场景优化 MoE 分布式推理

---

## 4. NIRVANA：面向 LLM 压缩的硬件感知结构化剪枝新范式

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2509.14230
**标签**：结构化剪枝 · LLM 压缩 · NTK 显著性 · 硬件感知 · 免重训练

结构化剪枝是加速 LLM 推理的有效路径，但现有方法常有明显性能下降且需大量重训练恢复能力。NIRVANA 提出硬件感知结构化剪枝框架，通过受 Neural Tangent Kernel (NTK) 启发的一阶函数空间显著性评估结构重要性，保护模型关键训练动态，同时避免结构性坍塌，在保留零样本性能与下游微调优化景观的前提下实现高效压缩。

**核心要点**：
- 以 NTK 启发的一阶函数空间显著性替代传统基于损失的启发式，保护训练动态
- 硬件感知设计，剪枝后结构可直接加速真实推理
- 保留零样本性能与下游微调能力，降低重训练需求

---

## 5. C²KV：面向高效 LLM 推理的压缩且可组合的 KV Cache 复用

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2607.17715
**标签**：KV Cache · 缓存复用 · 长上下文 · RAG · 推理加速

长上下文推理是 RAG 与多文档推理的核心。现有 KV Cache 复用方法聚焦计算节省，却忽略了存储与访问大 KV Cache 的成本瓶颈；而朴素地将压缩与非前缀 KV 复用结合常导致严重精度退化。C²KV 提出统一框架，联合优化 KV 提取与推理，实现非前缀 KV 复用场景下的压缩与可组合复用，在降本同时守住精度。

**核心要点**：
- 同时优化 KV 提取与推理，统一非前缀 KV 复用的压缩与复用
- 解决压缩与非前缀复用直接叠加导致的精度退化问题
- 针对长上下文 LLM 服务中的 KV 存储/访问瓶颈，适用 RAG 场景

---

## 6. Harness Engineering：面向 LLM 驱动 GPU Kernel 生成的工程化系统

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2607.17979
**标签**：GPU Kernel · 代码生成 · 编译验证 · Blackwell B200 · MLSys

LLM 可辅助生成 GPU kernel，但实效取决于生成代码能否被可靠约束、验证、剖分与筛选。本文提出以 harness 为中心的系统，用于 MLSys 2026 FlashInfer AI Kernel 生成竞赛（NVIDIA Blackwell B200）。系统将评估 harness 与基于剖分的优化控制器解耦：harness 强制编译、正确性、官方对齐计时与产物归档；控制器将剖分与工作负载证据转化为有界的候选生成决策，人写 skill 固化算子约束与流程。

**核心要点**：
- 评估 harness 与优化控制器解耦，分别负责约束验证与候选决策
- harness 强制编译/正确性/官方对齐计时/产物归档，保证可靠性
- 面向 Blackwell B200 的 kernel 生成竞赛实践，强调工程化可复现

---

## 7. LongStraw：固定 GPU 预算下突破 2M Token 的长上下文 RL 训练

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2607.14952
**标签**：强化学习 · GRPO · 长上下文训练 · 显存优化 · Agent 训练

百万级 token 推理与 RL 后训练（多在 256K token 以下）之间存在巨大鸿沟，而 AI agent 的长轨迹会累积海量观测与决策。GRPO 需对同历史的多条响应打分并反向传播，使注意力与长生命周期反向状态成为主要显存壁垒。LongStraw 提出目标与架构感知的系统：常驻状态仅保留后续 token 所需的模型原生 prompt 状态而非完整计算图，借助响应重放恢复状态并对 old/reference 分支进行无图打分，在固定 GPU 预算下实现百万级 token 的 RL 后训练。

**核心要点**：
- 常驻状态只保留模型原生 prompt 状态，不保留完整计算图，突破显存壁垒
- 响应重放恢复状态并对 old/reference 分支做无图打分，大幅降显存
- 将 RL 后训练上下文从 256K 扩展到 2M+ token，支撑长轨迹 agent 训练

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | FlashMemory-DeepSeek-V4 | arXiv cs.LG | 长上下文/显存 |
| 2 | SelectInfer | arXiv cs.LG | 端侧推理 |
| 3 | OrderMoE | arXiv cs.LG | 分布式MoE |
| 4 | NIRVANA | arXiv cs.LG | 结构化剪枝 |
| 5 | C²KV | arXiv cs.CL | KV Cache |
| 6 | Harness Engineering | arXiv cs.LG | GPU Kernel |
| 7 | LongStraw | arXiv cs.LG | 长上下文RL |

*自动生成 · 2026-07-21 · jeffinchen daily tech reading list*

