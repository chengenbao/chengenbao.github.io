---
layout: reading
title: "FlashInfer 编译式算子生成、AI 编排器与 MoE 融合内核"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-04-18
---

# 📰 2026-04-18 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖 GPU 算子自动生成、LLM 推理调度、MoE 融合内核与编译器基础设施。

---

## 1. FlashInfer：面向动态服务场景的高效 GPU 内核库

**来源**：arXiv cs.LG（FlashInfer 系统论文）
**链接**：https://arxiv.org/abs/2502.01025
**标签**：GPU 内核 · 推理服务 · 注意力 · JIT 编译

FlashInfer 是面向 LLM 服务的注意力与融合算子内核库，通过 JIT 编译与块稀疏化统一处理 PagedAttention、Grouped-Query Attention 与长上下文解码等动态场景。作为 MLSys/FlashInfer 生态的基石工程，它展示了"内核库 + 运行时适配"路线在算子供给上的工程化范式，是理解现代推理栈内核层的必读文献。

**核心要点**：
- JIT 编译按需生成适配动态 shape 的注意力内核，覆盖 prefill/decode/跨请求共享前缀
- 块稀疏计算格式兼容 Paged KV Cache 的分页访存模式
- 在不同批负载下显著优于手写内核与通用模板库

---

## 2. AlpaServe：面向统计复用的模型编排与服务系统

**来源**：arXiv cs.DC（OSDI 系统论文）
**链接**：https://arxiv.org/abs/2302.13265
**标签**：模型编排 · 推理服务 · 统计复用 · 资源放置

DeepLearning Compiler 与服务化基础设施的交叉方向：AlpaServe 研究如何在多个深度学习模型之间做计算放置与编排，通过模型并行与统计复用（statistical multiplexing）提升加速器利用率。其"把编排当编译问题求解"的思路，对今天多模型共置（multi-model serving）的 GPU 集群管理仍有直接参考价值。

**核心要点**：
- 将模型放置形式化为装箱与调度联合优化问题
- 利用推理负载的统计复用特性提升 GPU 利用率
- 讨论模型并行度与复用度之间的权衡曲线

---

## 3. FlashMoE：将混合专家计算、通信与调度融合进单一持久 GPU 内核

**来源**：arXiv cs.DC（arXiv:2506.04667）
**链接**：https://arxiv.org/abs/2506.04667
**标签**：MoE · 持久内核 · 通信计算重叠 · GPU

MoE 层的计算-通信交错是推理瓶颈的核心来源之一。FlashMoE 将路由、专家计算与 All-to-All 通信全部融合进单个 persistent GPU kernel，摆脱 CPU 对细粒度调度的微观管理，让 GPU 自己掌控 kernel 内的任务流水，显著降低 MoE 推理的气泡时间。

**核心要点**：
- 单一持久内核融合 MoE 全流程，避免多次 kernel 启动与 CPU 介入
- 通信与计算在 warp 级别重叠，隐藏 All-to-All 延迟
- 对小批量、低延迟推理场景收益尤为明显

---

## 4. MLIR：面向异构计算的模块化编译器基础设施

**来源**：arXiv cs.PL（MLIR 论文）
**链接**：https://arxiv.org/abs/2002.11054
**标签**：编译器 · MLIR · 异构计算 · 中间表示

随着 AI 加速器种类爆炸式增长，编译器栈的可复用性成为核心问题。MLIR（Multi-Level Intermediate Representation）提出多层渐进式 IR 设计，让不同抽象层级（张量操作、循环、向量、硬件指令）共享一套基础设施。这是理解 Triton、IREE、torch.compile 等现代 AI 编译栈的共同底座的经典文献。

**核心要点**：
- 多层 IR（dialect）设计，连接框架高层图与硬件底层指令
- 支持增量式优化 pass 复用，降低新硬件接入成本
- 已成为 PyTorch/TensorFlow/Triton 等生态的共同编译基础

---

## 5. Google DeepMind Gemma 技术报告：开放模型系

**来源**：arXiv cs.CL（Gemma 技术报告）
**链接**：https://arxiv.org/abs/2403.08295
**标签**：开源模型 · 模型架构 · 技术报告 · 训练配方

Gemma 系列沿用 Gemini 的架构与数据配方，但以开放权重发布，成为开源社区重要的基线模型。技术报告详细披露了多查询注意力（MQA）、RoPE、GeGLU 激活等架构选型，以及数据清洗与蒸馏训练策略，对理解 2026 年主流开源小模型（Gemma 4、Qwen 3.5 等后继者）的演化起点很有价值。

**核心要点**：
- 与 Gemini 共享架构基因：MQA + RoPE + GeGLU 的小型化组合
- 开放权重配套完整技术报告，成为社区微调生态基座
- 报告中数据配比与安全对齐流程对模型复现极具参考意义

---

## 6. vLLM PagedAttention：LLM 推理的高吞吐内存管理

**来源**：arXiv cs.LG（SOSP'23 论文）
**链接**：https://arxiv.org/abs/2309.06180
**标签**：KV Cache · 内存管理 · 推理服务 · PagedAttention

vLLM 的 PagedAttention 借鉴操作系统虚拟内存分页思想管理 KV Cache，消除内外碎片，实现跨请求共享前缀。作为过去三年推理服务领域被引用最多的系统论文之一，它定义了"KV Cache 当作内存子系统管理"的设计范式，2026 年的 FlashInfer、SGLang 等内核库仍在沿这条路线深化。

**核心要点**：
- 分页式 KV Cache 管理，内存利用率逼近满载
- Copy-on-Write 机制支持并行采样与前缀共享
- 吞吐较早期方案提升 2-4 倍，成为事实标准

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | FlashInfer：面向动态服务场景的高效 GPU 内核库 | arXiv cs.LG | GPU 内核 |
| 2 | AlpaServe：面向统计复用的模型编排与服务系统 | arXiv cs.DC | 模型编排 |
| 3 | FlashMoE：MoE 融合进单一持久 GPU 内核 | arXiv cs.DC | MoE |
| 4 | MLIR：面向异构计算的模块化编译器基础设施 | arXiv cs.PL | 编译器 |
| 5 | Google DeepMind Gemma 技术报告 | arXiv cs.CL | 开源模型 |
| 6 | vLLM PagedAttention：LLM 推理的高吞吐内存管理 | arXiv cs.LG | KV Cache |

---

*自动生成 · 2026-04-18 · jeffinchen daily tech reading list*
