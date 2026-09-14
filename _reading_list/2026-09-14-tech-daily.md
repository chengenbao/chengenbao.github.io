---
layout: reading
title: "时间序列基础模型、RL训练、基准评测与Agent记忆"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-09-14
---

# 📰 2026-09-14 · 每日技术速递

> 今日精选 5 篇深度技术文章，覆盖 时间序列基础模型、强化学习训练、基准评测、Agent 记忆、安全对齐。

---

## 1. IBM 发布 SOTA 时间序列基础模型 Granite PatchTST-FM-r2（商用友好许可）

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/ibm-research/ibm-releases-sota-granite-time-series
**标签**：时间序列基础模型 · PatchTST · 零样本预测 · Apache2.0 · Granite

IBM 发布 Granite Time Series 家族新模型 PatchTST-FM-r2（约 3.85 亿参数）。它在 PatchTST-FM-r1 基础上更新了架构、扩充了预训练语料，并新增概率预测与缺失值填补能力。该模型以 Apache 2.0 与 OpenMDW 1.0 商用友好开源许可发布，在 GIFT-Eval 零样本时序预测榜单上位列可复现零样本模型第一、总榜第二，权重、架构、推理管线与复现代码全部公开。

**核心要点**：
- 约 3.85 亿参数，支持概率预测与缺失值填补，强化零样本泛化
- 采用 Apache 2.0 + OpenMDW 1.0 商用友好许可，可放心落地生产
- 在 GIFT-Eval 可复现零样本模型中排名第一，适合需求/能源/流量/遥测等预测

---

## 2. 用 TRL + OpenEnv 训练会画水彩的编程模型

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/train-to-paint-with-code
**标签**：强化学习 · TRL · OpenEnv · RL环境 · 代码生成

本文复现并开源了一个用强化学习训练语言模型写 JavaScript（基于 p5.brush 库）来绘制水彩画的完整管线。作者借助 TRL 与 OpenEnv，把参考图数据集、RL 环境、打分模型、训练脚本与最终模型全部开源，整条流水线端到端跑在 Hugging Face 上（Jobs 训练、Spaces 承载环境与打分模型、Inference Providers 提供 pairwise judge）。

**核心要点**：
- 用 RL 环境 + 视觉打分模型作为 reward，引导模型生成可执行的绘图代码
- 训练、环境、打分与产物全部在 HF 平台开源，一条命令即可复现
- 展示了 RLHF/RLAIF 在「代码即创作媒介」上的工程化范式

---

## 3. BenchMIRT：用项目反应理论审计 LLM 基准真正在测什么

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/allenai/benchmirt
**标签**：基准评测 · Item Response Theory · 能力解耦 · AI2 · 可解释性

AI2 提出 BenchMIRT，一种在「单条 prompt」粒度审计 LLM 基准的方法。它借鉴心理测量学的项目反应理论（IRT），通过分析模型在各题目上的表现，估计背后真正驱动得分的潜在能力维度。文章指出，许多基准（如 BBQ、WildJailbreak）内部不同题目组其实在测不同能力，直接求平均会掩盖信号差异。

**核心要点**：
- 在单个 prompt 粒度解耦基准，揭示单条题目实际测的是哪些能力
- 基于 IRT 估计题目难度与能力关联，避免把不同能力混为一个总分
- 配套公开 tech report、数据集与代码，可复现审计流程

---

## 4. funes：给编码 Agent 一套你真正拥有的长期记忆

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/funes
**标签**：Agent记忆 · 检索增强 · 本地优先 · ClaudeCode · Codex

funes 是一个面向编码 Agent（Claude Code、Codex、pi、Hermes）的持久记忆层，由本机已有会话构建。它以单二进制形式提供，默认无 ML 运行时依赖，embedding 与 reranking 都在本地完成；一条命令即可接入并自动增量索引每个完成的回合。需要时记忆可同步到你私有所有的 HF 数据集。

**核心要点**：
- 把 Agent session trace 转化为可检索记忆，解决跨机器/跨 Agent 失忆问题
- 本地优先、无外部 ML 依赖，增量索引而非全量重算
- 一行 `funes add` 接入现有 Agent，并支持同步到私有 HF 数据集

---

## 5. Safety for Whom? 按边界自我蒸馏实现可控的安全拒答

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/MultiverseComputingCAI/safety-for-whom
**标签**：安全对齐 · 拒答校准 · 边界感知 · 自我蒸馏 · LLM安全

作者指出主流安全对齐把危害当作「话题级」属性（如 LlamaGuard-3 的题材分类），导致同一话题内难以表达不同部署策略所需的细致边界。论文提出 Boundary-Aware Self-Distillation：在话题全集中界定需要拒答的有害子集，训练模型在该子集内拒答、在其余部分正常作答，并以清晰阶跃边界度量效果，而非一刀切地拒答整个话题。

**核心要点**：
- 用「边界感知」替代「话题级」安全策略，精准拒绝有害子集而非整类话题
- 通过自我蒸馏训练，使模型在边界内拒答、边界外正常回答
- 面向真实部署的差异化安全需求（如公民教育 vs 公共服务助手）

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | IBM 发布 SOTA 时间序列基础模型 Gran… | HuggingFace | 时序基础模型 |
| 2 | 用 TRL + OpenEnv 训练会画水彩的编程… | HuggingFace | RL/对齐训练 |
| 3 | BenchMIRT：用项目反应理论审计 LLM 基… | HuggingFace | 评测/可解释 |
| 4 | funes：给编码 Agent 一套你真正拥有的长… | HuggingFace | Agent/系统 |
| 5 | Safety for Whom? 按边界自我蒸馏实… | HuggingFace | 安全对齐 |

*自动生成 · 2026-09-14 · jeffinchen daily tech reading list*
