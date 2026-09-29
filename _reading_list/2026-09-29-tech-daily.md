---
layout: reading
title: "今日 7 篇：KV Cache 管理、PTQ/INT4 量化、MoE 剪枝与推理加速"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-09-29
---

# 📰 2026-09-29 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 KV Cache 管理、训练后量化（PTQ）、MoE 专家剪枝、低比特量化、推理 early-stopping 与 Prefill/Decode 服务优化。

---

## 1. ActKV：面向 LLM 智能体的动作引导 KV Cache 管理

**来源**：arXiv
**链接**：https://arxiv.org/abs/2609.31395
**标签**：KV Cache · Agent 推理 · 显存优化 · 推理加速

Agent 类 LLM 推理在「观察-思考-行动」循环中不断累积超长 KV Cache，带来巨大显存开销并严重限制服务吞吐。ActKV 提出以动作（action）为引导的 KV Cache 管理策略，根据动作执行对历史上下文的实际依赖来动态裁剪与保留 token，在不显著损失任务质量的前提下大幅降低显存占用、提升并发服务能力。

**核心要点**：
- 指出现有 KV 压缩方法以整体输出质量为优化目标，忽略了 Agent 循环对上下文的局部依赖结构
- 以 action 作为信号区分关键/可丢弃 token，实现细粒度、动态化的缓存淘汰
- 在 Agent 长程任务上同时降低显存压力并提升 serving 吞吐

---

## 2. G²PTQ：用广义梯度补偿改进 LLM 训练后量化（PTQ）

**来源**：arXiv
**链接**：https://arxiv.org/abs/2609.31009
**标签**：量化 · PTQ · GPTQ · 低比特推理

训练后量化（PTQ）是无需重训即可压缩 LLM 显存与计算量的实用方案，GPTQ 类方法已成事实标准，但在极低位宽下仍有明显精度损失。G²PTQ 提出广义梯度补偿机制，在量化求解过程中更充分地补偿因舍入引入的层间误差传播，从而在高低位宽下都获得更接近全精度的结果。

**核心要点**：
- 剖析 GPTQ 类方法在标准流程下的失效来源（误差补偿不充分）
- 引入广义梯度补偿以更稳妥地抵消量化舍入带来的层间误差
- 在低位宽场景（如 3/4-bit）相比基线取得更好的困惑度/下游精度

---

## 3. 超越均值注意力：面向 KV Cache 淘汰的多样性感知逐层打分

**来源**：arXiv
**链接**：https://arxiv.org/abs/2609.30738
**标签**：KV Cache · 注意力 · 长上下文 · 推理加速

SnapKV、PyramidKV 等方法仅用观察窗口内的「平均注意力」对 token 排序以做 KV 淘汰，忽略了注意力的分布结构。本文提出统一打分 μ_i + λ₁σ_i + λ₂·corr(i,S)，在均值之外引入注意力离散度与和关键集合的相关性，实现逐层、多样性感知的 KV 淘汰，在长上下文任务上以更少缓存保留更好质量。

**核心要点**：
- 指出仅用平均注意力排序会丢弃具有分散但重要注意力的 token
- 提出融合方差与相关性项的统一打分函数
- 逐层自适应地淘汰冗余 token，兼顾长上下文质量与显存

---

## 4. RAZOR：剪除 LLM 中可被替代的 MoE 专家

**来源**：arXiv
**链接**：https://arxiv.org/abs/2609.30465
**标签**：MoE · 专家剪枝 · 模型压缩 · 稀疏模型

Mixture-of-Experts 模型每个 token 仅激活少数专家，却仍需存储全部专家池，存储开销巨大。RAZOR 在固定剪枝预算下，以「保留原模型输出分布」为目标识别并剪除可被其余专家替代的冗余专家，在不显著改变模型行为的前提下降低 MoE 存储与推理成本。

**核心要点**：
- 明确 MoE 存储瓶颈：全量专家池常驻显存而单 token 激活极少
- 以输出分布保真度为导向选择可替代（冗余）专家进行剪除
- 在给定剪枝预算上比随机/启发式剪枝更好地维持模型质量

---

## 5. 量化循环Transformer：反馈暴露与校准盲区

**来源**：arXiv
**链接**：https://arxiv.org/abs/2609.30820
**标签**：INT4 量化 · 循环Transformer · 低比特 · 校准

循环Transformer（looped transformer）跨递归步复用权重，使低位量化极具吸引力。本文在 Huginn-3.5B 上指出标准 PTQ 的两类失效模式：反馈暴露（feedback exposure，跨步权重复用放大量化误差）与校准盲区（calibration blindness，单步校准无法覆盖多步展开），并据此给出更稳健的量化思路。

**核心要点**：
- 揭示循环结构下权重复用会放大、而非均摊量化误差
- 定义 feedback exposure 与 calibration blindness 两个具体失效模式
- 以 INT4 为例说明标准 per-channel 量化在 Huginn-3.5B 上显著失效

---

## 6. 无需「学会停止」即可学会停止：自监督置信度训练提升推理效率

**来源**：arXiv
**链接**：https://arxiv.org/abs/2609.31619
**标签**：推理加速 · Early-Stopping · 自监督 · Test-Time Compute

推理模型常生成极长的思维链，使推理计算代价高昂。已有方法多在推理时做 early-stopping 或显式训练停止策略。本文提出自监督置信度训练，让模型在无需额外「停止」标签的情况下学会在置信时提前终止生成，从而在保持精度的同时显著压缩推理长度与开销。

**核心要点**：
- 指出现有推理效率方法依赖显式停止策略或推理时启发式
- 用自监督方式训练 token 级置信度，实现「该停就停」
- 在不牺牲准确性的前提下缩短推理轨迹、降低算力消耗

---

## 7. 并发请求下的 Prefill 与 Decode 分离：LLM 性能优化实践

**来源**：HuggingFace
**链接**：https://huggingface.co/blog/tngtech/llm-performance-prefill-decode-concurrent-requests
**标签**：Prefill/Decode · 推理服务 · 并发优化 · vLLM

LLM 在线服务中 prefill（处理提示）与 decode（逐 token 生成）两个阶段的计算特征差异巨大，混部会互相拖累吞吐与延迟。本文系统讲解在并发请求场景下如何对 prefill 与 decode 进行分离调度，并通过批处理、连续批处理与资源隔离等手段最大化 GPU 利用率与服务吞吐。

**核心要点**：
- 剖析 prefill（算力密集）与 decode（访存密集）的计算特性差异
- 说明混部导致长尾延迟与 GPU 利用率下降的根因
- 给出分离/连续批处理等工程手段以在并发下提升吞吐与降低延迟

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | ActKV：面向 LLM 智能体的动作引导 KV Cache 管理 | arXiv | KV Cache / Agent |
| 2 | G²PTQ：用广义梯度补偿改进 LLM 训练后量化（PTQ） | arXiv | 量化 / PTQ |
| 3 | 超越均值注意力：面向 KV Cache 淘汰的多样性感知逐层打分 | arXiv | KV Cache 压缩 |
| 4 | RAZOR：剪除 LLM 中可被替代的 MoE 专家 | arXiv | MoE 剪枝 |
| 5 | 量化循环Transformer：反馈暴露与校准盲区 | arXiv | INT4 量化 |
| 6 | 无需「学会停止」即可学会停止：自监督置信度训练提升推理效率 | arXiv | 推理加速 |
| 7 | 并发请求下的 Prefill 与 Decode 分离：LLM 性能优化实践 | HuggingFace | 推理服务 |

*自动生成 · 2026-09-29 · jeffinchen daily tech reading list*

