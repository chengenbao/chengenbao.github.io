---
layout: reading
title: "KV Cache 压缩 · MoE 安全 · 存内计算 · vLLM RL 工程（2026-05-07）"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-05-07
---

# 📰 2026-05-07 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖 KV Cache 压缩、MoE 安全、存内计算架构、多模态 Embedding 训练、vLLM RL 工程实践。

---

## 1. eOptShrinkQ：基于最优谱降噪与量化的近无损 KV Cache 压缩

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2605.02905  
**标签**：KV Cache · 量化 · 低秩分解 · Transformer 推理 · 内存优化

该研究揭示了 Transformer 注意力头中 KV Cache 存在天然的低秩分解结构：共享上下文分量（低秩）加上全秩的 per-token 残差，符合 spiked random matrix 模型。基于此，论文提出 eOptShrinkQ 框架，通过最优谱收缩去噪后再进行量化，实现近无损的 KV Cache 压缩。与现有方法相比，该方案在相同压缩比下显著降低了推理精度损失，可直接应用于长上下文推理场景的内存优化。

**核心要点**：
- 发现 KV Cache 可分解为低秩共享上下文与全秩残差两部分，符合 spiked 随机矩阵模型
- 提出先谱降噪再量化的两阶段压缩框架 eOptShrinkQ，理论上最优
- 在多个 LLM 基准上验证，压缩比相同时精度损失极小（near-lossless）
- 可直接用于长上下文推理的显存瓶颈优化，无需修改模型架构

---
## 2. RouteHijack：针对 MoE 大语言模型路由机制的定向攻击

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2605.02946  
**标签**：MoE · 安全对齐 · 对抗攻击 · 路由机制 · LLM 安全

随着 Mixture-of-Experts（MoE）架构被广泛用于扩展 LLM 规模，其安全鲁棒性问题日益凸显。本文提出 RouteHijack，一种路由感知的对抗攻击方法：通过构造特定输入，强制将 token 路由到安全对齐较弱的专家子网络，从而绕过安全机制产生有害输出。实验表明现有 MoE 模型在路由层面存在系统性安全漏洞，对齐训练无法完全覆盖所有专家路径。

**核心要点**：
- 提出路由感知攻击框架 RouteHijack，通过操控 token 路由绕过安全对齐
- 揭示 MoE 架构中不同专家的安全对齐强度不均匀，是系统性漏洞根源
- 攻击成功率在主流 MoE-LLM 上显著高于传统越狱方法
- 为 MoE 模型的安全评估和强化对齐提供了新的研究方向

---
## 3. DARTH-PUM：融合模拟内存计算与数字逻辑的混合 PIM 架构

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2602.16075  
**标签**：存内计算 · PIM · 矩阵乘法 · 硬件架构 · 内存带宽

模拟存内计算（PUM/in-memory computing）通过在存储阵列内直接完成矩阵-向量乘法来减少数据搬运开销，但许多实际算子需要在 MVM 之外执行非线性运算，纯模拟方案难以高效处理。DARTH-PUM 提出了一种混合架构：模拟 PUM 核处理 bulk MVM，数字逻辑单元处理其余运算，两者紧密耦合以最小化中间数据搬运。在 AI 推理负载上相比纯数字方案能耗和延迟均大幅降低。

**核心要点**：
- 模拟 PUM 负责 bulk MVM，数字单元处理非线性激活及控制逻辑，两者紧耦合
- 混合设计克服了纯模拟方案无法高效处理非 MVM 算子的瓶颈
- 在 AI 推理工作负载上能耗和延迟显著优于纯数字基线
- 为下一代 AI 芯片在存储带宽瓶颈下的算力扩展提供了可行路径

---
## 4. 随机逻辑存内计算：最大化内存级并行度的新型架构

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2604.23146  
**标签**：存内计算 · 内存级并行 · 随机计算 · 数据密集型负载 · 芯片架构

随着单核性能提升放缓而数据密集型负载快速增长，数据搬运的延迟与能耗已成为高性能计算的核心瓶颈。本文提出将随机逻辑（stochastic logic）集成到内存阵列中，在不改变存储单元的前提下实现大规模内存级并行计算（MLP）。该架构通过随机比特流在存储阵列内完成算术运算，可显著降低数据移动开销，适用于深度学习推理等高 MLP 需求场景。

**核心要点**：
- 将随机逻辑电路集成于内存阵列，无需额外计算核心即可实现大规模 MLP
- 随机比特流计算模型天然容忍噪声，与模拟存储特性高度匹配
- 在数据密集型工作负载（含 DNN 推理）上大幅降低数据移动能耗
- 提供了芯片级内存-计算融合的新范式，对 AI 加速器设计具有参考价值

---
## 5. 用 Sentence Transformers 训练多模态 Embedding 与 Reranker 模型

**来源**：HuggingFace Blog  
**链接**：https://huggingface.co/blog/train-multimodal-sentence-transformers  
**标签**：多模态 Embedding · Reranker · Sentence Transformers · 微调 · 检索增强

HuggingFace 官方博客介绍了 Sentence Transformers 库对多模态 Embedding 和 Reranker 模型训练/微调的完整支持。涵盖图文对比学习、跨模态检索排序的训练流程，提供了从数据准备、损失函数选择到评估的端到端实践指南，可直接用于构建图文混合的 RAG 系统或语义搜索应用。

**核心要点**：
- Sentence Transformers 新增对多模态（图文）Embedding 模型的训练和微调支持
- 提供跨模态 Reranker 训练流程，提升图文混合检索的排序质量
- 从数据准备到损失函数到评估的完整端到端教程，开箱即用
- 可直接应用于多模态 RAG 系统、视觉语言搜索等场景

---
## 6. vLLM V0 到 V1 迁移：强化学习训练中先保证正确性再优化

**来源**：HuggingFace Blog  
**链接**：https://huggingface.co/blog/ServiceNow-AI/correctness-before-corrections  
**标签**：vLLM · 强化学习 · LLM 训练 · V1 迁移 · RLHF

ServiceNow AI 团队记录了将 RL 训练流程从 vLLM V0 迁移至 V1 的完整经验，核心教训是：在追求 RL 奖励优化之前，必须首先确保 rollout 生成的正确性。文章详述了 V0 到 V1 的架构差异、迁移中遇到的 correctness bugs 及调试方法，为工程团队在 vLLM V1 上构建 RLHF/GRPO 等 RL 训练流程提供了实战参考。

**核心要点**：
- 详述 vLLM V0 到 V1 迁移中 RL 训练的 correctness 问题及根因分析
- 提出先保正确性、再追优化的工程原则，避免因 rollout bug 导致 RL 训练发散
- 对比 V0/V1 架构差异（如 scheduler、采样逻辑变更）对 RL 流程的影响
- 提供可复用的调试清单，适用于 RLHF、GRPO 等强化学习训练框架

---
## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | eOptShrinkQ KV Cache 压缩 | arXiv cs.LG | KV Cache 压缩 |
| 2 | RouteHijack MoE 路由攻击 | arXiv cs.LG | MoE 安全 |
| 3 | DARTH-PUM 混合 PIM 架构 | arXiv cs.AR | 存内计算架构 |
| 4 | 随机逻辑存内计算 MLP | arXiv cs.AR | 内存级并行 |
| 5 | 多模态 Embedding 训练 | HuggingFace Blog | 多模态检索 |
| 6 | vLLM V1 迁移 RL 工程 | HuggingFace Blog | vLLM RL 工程 |

---

*自动生成 · 2026-05-07 · jeffinchen daily tech reading list*