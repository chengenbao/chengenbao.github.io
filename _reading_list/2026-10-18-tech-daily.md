---
layout: reading
title: "预训练数据、端侧推理与分布式训练前沿"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-10-18
---

# 📰 2026-10-18 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖 预训练数据工程、端侧/边缘推理、分布式训练、硬件/GPU 生态、媒体数据预处理。

---

## 1. NVIDIA 与 IIT Madras 联合发布 2.8 万亿 Token 的 Indic 预训练语料与开源数据生成管线

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/nvidia/scaling-indic-pretraining
**标签**：预训练数据 · 多语言语料 · GPU 数据管线 · 合成数据

NVIDIA 联合 IIT Madras 的 AI4Bharat 实验室发布 Nemotron-pretraining-Indic-v1，覆盖 22 种印度语言的约 2.8 万亿 Token 预训练语料，并同步开源了用于生成该语料的 GPU 加速数据管线。管线完整复现了从文档级翻译、质量评估到定向修复的生成配方（含 prompt、LLM 工作流与质量检查），每条数据都可溯源到原始文档并与输入交叉校验以保持事实一致性。语料包含 22 种语言的原生文字与 12 种语言的罗马化文本，将高质量英文数学、代码与问答内容迁移到印度语言。

**核心要点**：
- 约 2.8 万亿 Token、覆盖 22 种印度语言（含 12 种语言的罗马化），34 种语言-文字配置
- 开源 GPU 加速数据生成管线：文档翻译 + 质量评估 + 定向修复，保留公式/表格/代码结构
- 每条合成数据可溯源至原始文档并做输入交叉校验，保证事实接地而非凭空生成

---

## 2. LiquidAI 发布 LFM2.5-24B-A2B：24B 参数仅激活 2B 的端侧推理智能体模型

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/LiquidAI/lfm25-24b-a2b
**标签**：MoE · 端侧推理 · 智能体 · 推理加速

LFM2.5-24B-A2B 是一个总参 24B、每 token 仅激活 2B 的 MoE 推理模型，专为端侧/本地快速、隐私友好的智能体场景设计。它延续 40 层、每 MoE 块 64 专家 top-4 路由与门控短卷积+GQA 混合架构，上下文从 32K 扩展到 131K，词表由 65K 扩至 128K。在 Apple M5 Max 上可达 184 tok/s，单张 NVIDIA B300 上最高 21K 输出 tok/s。后训练分 SFT、教师专精、非智能体/智能体两阶段 MOPD 蒸馏、智能体 RL 五个阶段，在 LiveClaw 工具调用基准上以 48.09 超过参数量更大的 Qwen3.6-35B-A3B（38.19）。

**核心要点**：
- 24B 总参 / 2B 激活的 MoE，每 token 推理成本低，可本地持续运行智能体
- 上下文 32K→131K，词表 65K→128K，强化非拉丁语言 token 化效率
- 五阶段后训练（含 Agentic RL），在端侧硬件实现 184 tok/s 的高效推理

---

## 3. Zephon 为 TorchTitan 带来弹性确定性数据加载

**来源**：PyTorch
**链接**：https://pytorch.org/blog/bringing-elastically-deterministic-data-loading-to-torchtitan-with-zephon/
**标签**：分布式训练 · 数据加载 · 确定性 · TorchTitan

PyTorch 团队介绍 Zephon 如何为 TorchTitan 训练框架引入「弹性确定性数据加载」。训练时模型所见的样本顺序对最终性能影响巨大，而大规模分布式训练中数据分片、弹性扩缩容常破坏可复现性。Zephon 在支持节点弹性伸缩的同时，保证每个 step 的数据加载顺序确定可复现，使研究人员能稳定地做实验对照与故障排查，而无需牺牲训练集群的资源利用率。

**核心要点**：
- 在弹性扩缩容下保持数据加载顺序的确定性，提升大规模训练可复现性
- 解耦资源弹性与实验可复现，便于做对照实验与故障定位
- 面向 TorchTitan 原生集成，降低分布式训练调试成本

---

## 4. ExecuTorch 登陆 Arm Cortex-M 微控制器

**来源**：PyTorch
**链接**：https://pytorch.org/blog/bringing-executorch-to-arm-cortex-m-cpus/
**标签**：端侧推理 · ExecuTorch · 微控制器 · Arm Cortex-M

PyTorch 宣布 ExecuTorch（边缘端推理运行时）正式支持 Arm Cortex-M 系列微控制器。过去在 MCU 上部署 PyTorch 模型需要转换为其他框架格式、维护独立工具链并调试跨框架数值不一致。ExecuTorch 让开发者可以用统一的 PyTorch 工具链直接把模型部署到资源极度受限的 Cortex-M 设备，消除格式转换与数值偏差带来的工程负担，拓展了端侧 AI 的硬件边界。

**核心要点**：
- 统一 PyTorch 工具链直接部署到 Cortex-M，免去框架格式转换
- 缓解跨框架数值不匹配问题，降低 MCU 端推理调试成本
- 将端侧 AI 推进到资源极度受限的微控制器级别

---

## 5. PyTorch 硬件适配进展：加速器集成工作组更新

**来源**：PyTorch
**链接**：https://pytorch.org/blog/pytorch-hardware-enablement-updates-from-the-acceleration-integration-working-group/
**标签**：硬件适配 · 加速器 · GPU · 生态标准化

PyTorch 加速器集成工作组（Accelerator Integration Working Group）分享了最新进展，介绍其如何标准化新硬件架构接入开源 AI 生态的方式。随着训练/推理计算平台（各类 GPU、NPU、专用加速器）日益多样化，工作组致力于统一设备接入、算子与编译路径，降低新硬件对接 PyTorch 的门槛，推动异构算力在训练与推理中的可用性与互操作性。

**核心要点**：
- 标准化新硬件架构接入 PyTorch 的方式，降低加速器对接门槛
- 覆盖 GPU、NPU 等多样化计算平台的设备/算子/编译路径统一
- 提升异构算力在训练与推理场景的可用性与互操作性

---

## 6. PyTorch 媒体处理体系演进：TorchCodec 与 TorchVision 分工明确

**来源**：PyTorch
**链接**：https://pytorch.org/blog/evolution-of-the-pytorch-media-processing-landscape/
**标签**：媒体处理 · TorchCodec · 编解码 · 数据预处理

PyTorch 梳理了媒体处理（图像/视频/音频）工具链的演进方向：解码与编码优先使用 TorchCodec，图像与视频的变换使用 TorchVision，音频侧亦给出对应推荐路径。文章厘清了各类媒体负载在 PyTorch 生态中的最佳处理组件，帮助开发者在多模态训练数据预处理与推理管线中选用更高效的原生方案，减少外部依赖与格式摩擦。

**核心要点**：
- 媒体编解码统一交由 TorchCodec，变换交给 TorchVision（图像/视频）
- 为多模态训练/推理数据预处理提供清晰的组件选型指引
- 减少外部依赖与格式转换摩擦，提升媒体管线效率

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | NVIDIA 与 IIT Madras 联合发布 2.8 万亿 Token 的 Indic 预训练语料与开源数据生成管线 | HuggingFace | 预训练数据工程 |
| 2 | LiquidAI 发布 LFM2.5-24B-A2B：24B 参数仅激活 2B 的端侧推理智能体模型 | HuggingFace | 端侧/边缘推理 |
| 3 | Zephon 为 TorchTitan 带来弹性确定性数据加载 | PyTorch | 分布式训练 |
| 4 | ExecuTorch 登陆 Arm Cortex-M 微控制器 | PyTorch | 端侧/边缘推理 |
| 5 | PyTorch 硬件适配进展：加速器集成工作组更新 | PyTorch | 硬件/GPU 生态 |
| 6 | PyTorch 媒体处理体系演进：TorchCodec 与 TorchVision 分工明确 | PyTorch | 数据预处理/媒体 |

---

*自动生成 · 2026-10-18 · jeffinchen daily tech reading list*
