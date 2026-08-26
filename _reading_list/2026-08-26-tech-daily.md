---
layout: reading
title: "LLM 推理加速与 GPU/TPU 架构编译前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-08-26
---

# 📰 2026-08-26 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 LLM 推理与近存储加速 · GPU/TPU 架构与编译 · 编译器工具链 · 端侧量化部署 · FP8 分布式训练。

---

## 1. FlashAccel：用高带宽闪存(HBF)实现高通量 LLM 推理

**来源**：cs.AR
**链接**：https://arxiv.org/abs/2607.10186
**标签**：LLM推理 · 高带宽闪存 · 近存储计算 · 容量瓶颈 · KV Cache

LLM 推理正被 GPU 上 HBM 的容量限制所束缚——模型权重与 KV Cache 快速增长。高带宽闪存(HBF)容量远超 HBM 且带宽接近，是理想的大容量推理载体，但其高访问延迟、低带宽利用率和缺乏异构资源管理使其难以直接融入 GPU。FlashAccel 提出一套协同设计系统，让 HBF 在 LLM 推理中高效可用，缓解容量墙问题。

**核心要点**：
- 指出现代 LLM 推理的核心瓶颈已从算力转向 HBM 容量，HBF 提供更高容量且带宽可媲美 HBM
- 针对 HBF 高延迟、低带宽利用率、缺异构资源调度三痛点做系统级协同设计
- 通过软硬协同让大容量介质承载权重/KV Cache，提升整体推理吞吐而非单纯堆 HBM

---

## 2. 面向 AI 时代的下一代异步分布式 GPU 架构与周期级仿真

**来源**：cs.AR
**链接**：https://arxiv.org/abs/2608.22602
**标签**：GPU架构 · MCM多芯粒 · 异步执行 · 周期级仿真 · 多相位Kernel

机器学习负载推动 GPU 走向多芯粒(MCM)拓扑、异步执行原语与持久化多相位 Kernel，但周期级仿真基础设施落后，难以同时刻画现代 GPU 的物理非均匀性与大规模 AI 负载。本文提出一个周期级仿真框架，精确建模 Ampere 等现代 GPU 世代的非均匀物理特性与 AI 负载规模。

**核心要点**：
- 梳理 GPU 架构向 MCM 拓扑、异步执行、持久多相位 Kernel 的演进趋势
- 指出现有周期级仿真器无法同时刻画物理非均匀性与 SOTA AI 负载规模
- 提出新一代仿真框架，覆盖 Ampere 等现代 GPU 世代，支撑架构研究

---

## 3. NOVA：面向 Attention-SSM-MoE 混合 LLM 的近内存处理协同设计

**来源**：cs.AR
**链接**：https://arxiv.org/abs/2608.22613
**标签**：近内存计算 · 混合LLM · MoE · SSM · 存算一体

混合 LLM 交错 GQA 注意力、状态空间模型(SSM)与混合专家(MoE)层，给近内存处理(NMP)架构带来双重挑战：技术墙(传统 6F² DRAM 在 10nm 级逼近物理缩放极限，难以满足数百专家的容量需求)与架构墙(现有 NMP 仅针对窄算术强度区间)。NOVA 以技术-架构协同设计应对混合 LLM 推理。

**核心要点**：
- 归纳混合 LLM 给 NMP 带来的'技术墙'与'架构墙'两类根本挑战
- 技术墙：DRAM 单元缩放逼近极限，MoE 数百专家的容量需求难以满足
- 架构墙：传统 NMP 只覆盖窄 Op/B 区间，无法高效支持 GQA/SSM/MoE 混合算子

---

## 4. mold：一款大规模并行的链接器

**来源**：cs.OS
**链接**：https://arxiv.org/abs/2608.23228
**标签**：链接器 · 编译器工具链 · 数据并行 · 构建加速 · C++

链接是软件构建的关键步骤，却长期是编辑-编译-调试循环的瓶颈，尤其在大型 C++ 程序中。现有链接器并行度有限，链接时大量 CPU 核心闲置。mold 是一款 Unix/Linux 链接器，在整个链接流水线中系统性应用数据并行，并首先剖析了阻碍现有链接器扩展的架构约束。

**核心要点**：
- 指出链接阶段在大型 C++ 构建中仍是显著瓶颈，现有工具并行度不足
- mold 在整个链接流水线系统性引入数据并行，充分利用多核
- 论文先剖析了限制传统链接器横向扩展的架构约束再给出方案

---

## 5. 实测研究：哪些 LLM 真正跑在 Apple Neural Engine 上，又为何变快

**来源**：cs.AR
**链接**：https://arxiv.org/abs/2608.22110
**标签**：Apple ANE · 端侧推理 · 量化部署 · 算子支持 · 测量研究

本文通过三组测量回答'什么能让 LLM 上 Apple Neural Engine(ANE)以及什么让它变快'：扫描 64 形态 LLM 原语矩阵记录逐算子设备支持；训练跨尺寸/精度的匹配模型(量化检查点结构与 fp16 字节一致)以测真实训练产物；读取 ANE 内存控制器字节计数器建立推理期访存事实。

**核心要点**：
- 用 64 形态原语矩阵实测 ANE 对各类 LLM 算子的实际支持情况
- 量化检查点与 fp16 结构字节一致，确保每次部署测量都基于真实训练制品
- 通过 ANE 内存控制器字节计数器揭示推理期真实访存行为，破除经验假设

---

## 6. PyTorch：TorchTitan + TorchAO 在 AMD GPU 上的 FP8 训练优化已上游合入

**来源**：PyTorch
**链接**：https://pytorch.org/blog/fp8-training-on-amd-gpus-with-torchtitan-and-torchao-upstreaming-performance-improvements/
**标签**：FP8训练 · AMD Instinct · TorchTitan · TorchAO · 千卡线性扩展

PyTorch 团队曾在 2025 大会展示基于 AMD Primus-Turbo 在 Instinct 集群上超 1000 卡的线性扩展，现已将这些 AMD 优化上游合入，使 TorchTitan 直接支持 AMD Instinct GPU 且开箱即用具备有竞争力的 FP8 性能。所有改动均已合入主线。

**核心要点**：
- 将 AMD Primus-Turbo 的千卡级线性扩展优化上游合入 TorchTitan
- TorchTitan 现原生支持 AMD Instinct GPU，FP8 训练开箱即用
- 所有贡献已合并主线，降低在 AMD 集群做大规模 LLM 训练门槛

---

## 7. Helion 登陆 TPU：面向异构硬件的可移植内核编写 DSL

**来源**：PyTorch
**链接**：https://pytorch.org/blog/helion-on-tpu-towards-hardware-heterogeneous-kernel-authoring/
**标签**：Helion · TPU · Pallas · 内核DSL · 性能可移植

Helion 是 PyTorch 用于编写性能可移植 ML 内核的高层 DSL。PyTorch 与 Google 合作构建了 TPU 后端，将 Helion 内核编译到 Pallas，提供对 PyTorch 用户友好的方式在 TPU 上编写高性能算子，迈向硬件异构的内核编写。

**核心要点**：
- Helion 是 PyTorch 的高层 ML 内核 DSL，主打性能可移植性
- 新 TPU 后端将 Helion 内核编译为 Pallas，贴合 PyTorch 用户习惯
- 打通 GPU/TPU 异构后端，降低跨硬件手写高性能算子成本

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | FlashAccel：用高带宽闪存(HBF)实现高通量 LLM 推理 | cs.AR | LLM推理加速 |
| 2 | 面向 AI 时代的下一代异步分布式 GPU 架构与周期级仿真 | cs.AR | GPU架构仿真 |
| 3 | NOVA：面向 Attention-SSM-MoE 混合 LLM 的近内存处理协同设计 | cs.AR | 近内存计算 |
| 4 | mold：一款大规模并行的链接器 | cs.OS | 编译工具链 |
| 5 | 实测研究：哪些 LLM 真正跑在 Apple Neural Engine 上，又为何变快 | cs.AR | 端侧部署 |
| 6 | PyTorch：TorchTitan + TorchAO 在 AMD GPU 上的 FP8 训练优化已上游合入 | PyTorch | FP8训练 |
| 7 | Helion 登陆 TPU：面向异构硬件的可移植内核编写 DSL | PyTorch | 编译DSL |

*自动生成 · 2026-08-26 · jeffinchen daily tech reading list*
