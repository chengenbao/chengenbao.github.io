---
layout: reading
title: "FP8训练·编译器·端侧推理·序列并行"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-08-17
---

# 📰 2026-08-17 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 FP8分布式训练、编译器与DSL、端侧推理、推理引擎加速、序列并行等方向。

---

## 1. AMD GPU 上的 FP8 训练：TorchTitan 与 TorchAO 性能优化合入上游

**来源**：PyTorch
**链接**：https://pytorch.org/blog/fp8-training-on-amd-gpus-with-torchtitan-and-torchao-upstreaming-performance-improvements/
**标签**：FP8训练 · AMD Instinct · TorchTitan · 分布式训练 · 量化

文章介绍了 AMD 将其在 PyTorch Conference 2025 上展示的、基于 Primus-Turbo 优化库在 1000+ 张 AMD Instinct GPU 上实现近乎线性扩展的 FP8 训练能力合入 TorchTitan 上游的过程。核心是将 AMD 的 FP8 量化与优化路径直接集成到 TorchTitan 训练框架中，使开发者无需额外补丁即可在 AMD 集群上获得高性能大规模训练支持。

**核心要点**：
- 基于 Primus-Turbo 的优化在 1000+ 张 AMD Instinct GPU 上实现近线性扩展
- FP8 训练路径与 TorchAO 量化能力已合入 TorchTitan 上游主干
- 降低 AMD 硬件上大规模 LLM 训练的工程门槛，开箱即用

---

## 2. Helion 登陆 TPU：面向异构硬件的内核编写 DSL

**来源**：PyTorch
**链接**：https://pytorch.org/blog/helion-on-tpu-towards-hardware-heterogeneous-kernel-authoring/
**标签**：Helion · TPU · 编译器 · Pallas · ML内核

Helion 是 PyTorch 的高层 DSL，用于编写性能可移植的 ML 算子内核。本文介绍 PyTorch 与 Google 合作构建的 TPU 后端：Helion 内核可被编译到 Google 的 Pallas 底层，为 TPU 用户提供熟悉的 PyTorch 风格编程接口，同时保留底层性能控制能力。

**核心要点**：
- Helion DSL 新增 TPU 后端，编译目标为 Google Pallas
- 用户可用统一 PyTorch 风格 API 编写跨 GPU/TPU 的可移植内核
- 降低在 TPU 上手工优化算子的门槛，兼顾易用性与性能

---

## 3. 端侧 Agentic AI：ExecuTorch 上的 Muse Glimmer 30B 模型

**来源**：PyTorch
**链接**：https://pytorch.org/blog/fast-ondevice-agentic-ai-with-executorch/
**标签**：端侧推理 · ExecuTorch · 模型蒸馏 · NVIDIA · Agent

Meta 发布 Muse Glimmer——一个从 Muse Spark 蒸馏得到的 300 亿参数开放权重模型，面向端侧 Agent 工作流。文章介绍 ExecuTorch 如何为此新增端到端运行支持，使 30B 级别模型能够在 NVIDIA 等端侧硬件上高效执行 Agent 任务。

**核心要点**：
- Muse Glimmer 为 30B 参数蒸馏模型，专注端侧 Agent 场景
- ExecuTorch 新增端到端推理支持，覆盖 NVIDIA 等硬件
- 推动大模型从云端向端侧设备迁移，降低延迟与隐私风险

---

## 4. Triton 插件扩展：开箱即用的自定义编译 Pass 与方言

**来源**：PyTorch
**链接**：https://pytorch.org/blog/triton-plugin-extensions-enabling-tlx-and-custom-compiler-passes-out-of-the-box/
**标签**：Triton · 编译器 · 插件系统 · 自定义Pass · DSL

PyTorch-Triton 3.7 引入 Triton Plugin Extensions 系统，允许开发者将自定义编译 Pass、方言（含其算子）以及 DSL 扩展动态加载到上游 Triton 中。这改变了以往必须 fork Triton 才能实验新编译技术的局面，降低了编译器创新的迭代成本。

**核心要点**：
- 3.7 版本新增插件机制，支持动态加载自定义编译 Pass 与方言
- 无需 fork Triton 即可实验新编译器特性（如 TLX）
- 为编译器生态的模块化与社区贡献打开通道

---

## 5. FBTriton 基础设施：自定义 GPU 编译器的上游同步与分层验证

**来源**：PyTorch
**链接**：https://pytorch.org/blog/fbtriton-infra-upstream-ingestion-hierarchical-validation-ideals-vs-realities/
**标签**：GPU编译器 · Triton · 自动化验证 · TLX · 工程实践

文章揭秘 Meta 的 FBTriton 基础设施如何支撑 TLX、autoWS 等自定义 GPU 编译器创新，同时保持与上游 Triton 的同步。核心是一套 agentic ingestion 流程与分层的 L1/L2/L3 验证框架，在理想设计与现实约束之间取得平衡，确保自定义优化可安全合入主干。

**核心要点**：
- FBTriton 通过 agentic ingestion 自动同步上游 Triton 变更
- L1/L2/L3 分层验证框架平衡创新速度与稳定性
- 为大规模 GPU 编译器工程提供可复用的协同范式

---

## 6. 原生速度 vLLM Transformers 建模后端

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/native-speed-vllm-transformers-backend
**标签**：vLLM · 推理加速 · Transformers · 推理引擎 · 吞吐

Hugging Face 推出原生速度的 vLLM Transformers 建模后端，使 🤗 Transformers 模型能够以接近 vLLM 原生的性能运行推理。该后端桥接了 Transformers 生态的模型丰富度与 vLLM 的高吞吐推理能力，研究者可在不牺牲易用性的前提下获得生产级推理性能。

**核心要点**：
- Transformers 模型可直接以 vLLM 原生速度运行推理
- 打通 Hugging Face 模型库与 vLLM 高性能推理引擎
- 兼顾模型覆盖广度与生产级吞吐需求

---

## 7. Ulysses 序列并行：百万 Token 上下文训练

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/ulysses-sp
**标签**：序列并行 · 长上下文 · 分布式训练 · 注意力 · Ulysses

Ulysses 序列并行是一种将长序列切分到多设备上的分布式训练策略，使模型能够在百万级 Token 的超长上下文上高效训练。文章详述其通信模式与注意力计算分解方式，相比朴素数据并行显著降低了单卡显存峰值，为长文档、长代码等场景提供可扩展方案。

**核心要点**：
- 序列维度切分，支持百万 Token 级超长上下文训练
- 注意力计算按序列分片，降低单卡显存峰值
- 相比纯数据并行，长序列训练的可扩展性显著提升

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | AMD GPU 上的 FP8 训练 | PyTorch | FP8训练 |
| 2 | Helion 登陆 TPU | PyTorch | 编译器DSL |
| 3 | 端侧 Agentic AI | PyTorch | 端侧推理 |
| 4 | Triton 插件扩展 | PyTorch | 编译器 |
| 5 | FBTriton 基础设施 | PyTorch | GPU编译工程 |
| 6 | 原生速度 vLLM Transforme | HuggingFace | 推理加速 |
| 7 | Ulysses 序列并行 | HuggingFace | 序列并行 |

---

*自动生成 · 2026-08-17 · jeffinchen daily tech reading list*
