---
layout: reading
title: "Agentic Kernel 优化、MLSys 竞赛与 CUDA 自动调优"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-07-09
---

# 📰 2026-07-09 · 每日技术速递

> 今日精选 6 篇深度技术文章，聚焦 Agent 驱动的内核优化、CUDA 自动调优与算子工程实践。

---

## 1. 全栈自研编译器：从 PyTorch 到 AI 芯片的 Triton 级适配

**来源**：技术博客（AI 编译器实践）
**链接**：https://zhuanlan.zhihu.com/p/1969431459241648245
**标签**：Agentic RL · CUDA · 内核生成 · 大规模训练

ICLR'26 GPU kernel 生成方向投稿论文合集的导读：编译器自动调优与 LLM 生成两条路线在 kernel 生成任务上正面竞争，LLM 路线在 KernelBench 上达到 97.6% 正确率与 1.68 倍平均加速。文章梳理两条路线的适用边界与融合趋势，是理解 2026 年算子生成格局的高效入口。

**核心要点**：
- ICLR'26 kernel 生成投稿的方向分布
- LLM 生成 vs 编译器自动调优的优劣对比
- 融合趋势：搜索空间剪枝 + 语义先验

---

## 2. 让 Agent 自己优化 CUDA Kernel：KDA 开源工作流

**来源**：知乎专栏（KDA 框架解析）
**链接**：https://zhuanlan.zhihu.com/p/2044459666327999866
**标签**：KDA · Agent 工作流 · FlashInfer · 开源

KDA 的核心哲学是把 kernel 优化拆成 Agent 可执行的原子动作：读 profile、提假设、改代码、跑基准、对比分析。文章详解其 prompt 工程、工具链设计与失败恢复机制，并给出在 MLSys 2026 FlashInfer 赛道的名次数据。想自建"AI 算子工程师"的团队可直接抄作业。

**核心要点**：
- 优化假设库：tile 大小、流水深度、访存合并等原子动作
- 工具链：编译器诊断、Nsight 数据的结构化注入
- 失败恢复与预算控制：何时回滚何时止损

---

## 3. KernelBench 实战：主流模型生成 CUDA 内核的横向评测

**来源**：GitHub（ScalingIntelligence）
**链接**：https://github.com/ScalingIntelligence/KernelBench
**标签**：评测基准 · 内核生成 · 加速比 · 实测

基于 KernelBench 的横向实测：不同模型在 250 个算子上的通过率与加速比分布，头部模型在简单算子已接近 98% 正确率、平均 1.68 倍于 PyTorch 基线，但在 reduce 类与复杂访存模式算子仍有明显断层。实测数据为"哪些算子值得交给 AI"提供了直接答案。

**核心要点**：
- 正确率 97.6% 背后的失败模式：边界与数值稳定
- 加速比断层：元素级算子易、归约/扫描类难
- 人工复核成本：AI 内核的可信度分级策略

---

## 4. Triton autotune 与 CUDA Graph：PyTorch 性能优化的两条路径

**来源**：PyTorch 官方教程
**链接**：https://docs.pytorch.ac.cn/tutorials/recipes/torch_compile_user_defined_triton_kernel_tutorial.html
**标签**：Triton · autotune · CUDA Graph · torch.compile

torch.compile 已支持用户自定义 Triton kernel 的一等公民接入。教程演示如何把 Triton autotune 的多配置搜索包装进编译图，以及 CUDA Graph 捕获如何消除 kernel 启动开销。这是把"手写高性能算子"无缝融入 PyTorch 生产栈的标准姿势。

**核心要点**：
- 自定义 Triton kernel 接入 torch.compile 的规范写法
- autotune 缓存与动态 shape 的配合技巧
- CUDA Graph 捕获的适用条件与陷阱（动态控制流、显存地址）

---

## 5. 面向 Decoding 的 Split-K 与变长 batch 内核设计

**来源**：arXiv cs.LG（解码内核研究）
**链接**：https://arxiv.org/abs/2401.09670
**标签**：Split-K · 解码内核 · 负载均衡 · GEMV

decode 阶段的 GEMV 形 GEMM 计算强度低、靠带宽吃性能，单 SM 数量远吃不饱。Split-K 把 K 维切块并行归约，配合变长 batch 的动态调度填满 SM。文章给出 decode 内核"低计算强度、高带宽需求"下的占用率设计公式，是自研 attention/GEMV 内核的必修课。

**核心要点**：
- decode 阶段的瓶颈迁移：算力受限 → 带宽受限
- Split-K 分块与跨块归约的两种实现路径
- 变长 batch 下的 SM 负载均衡策略

---

## 6. 推理系统面经：从 KV Cache 到集群调度的知识体系

**来源**：技术博客（推理系统知识地图）
**链接**：https://www.weigao.cc/ai-systems/llm-inference/inference-frameworks-2026/
**标签**：知识体系 · 推理系统 · 学习路径 · Serving Stack

从 Engine 到 Serving Stack 的完整知识地图：内核层（attention/GEMM）、引擎层（调度/缓存/投机）、集群层（PD 分离/路由/降级），并给出每层的"选型合同"——先描述 workload 与 SLO，再检查候选栈数据路径，最后同构压测。适合作为推理方向系统学习的总纲。

**核心要点**：
- 三层知识体系：内核/引擎/集群的职责边界
- 选型合同方法论：SLO → 数据路径 → 同构压测
- 各层开源项目的成熟度与发展方向

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | GPU kernel 生成方向论文合集导读（ICLR'26） | 知乎专栏 | Kernel 生成 |
| 2 | 让 Agent 自己优化 CUDA Kernel：KDA 工作流 | 知乎专栏 | Agent 工作流 |
| 3 | KernelBench 实战：内核生成横向评测 | GitHub | 评测基准 |
| 4 | Triton autotune 与 CUDA Graph 双路径优化 | PyTorch 官方 | 性能优化 |
| 5 | Split-K 与变长 batch 的解码内核设计 | arXiv cs.LG | 解码内核 |
| 6 | 推理系统知识体系：从 KV Cache 到集群调度 | 技术博客 | 知识地图 |

---

*自动生成 · 2026-07-09 · jeffinchen daily tech reading list*
