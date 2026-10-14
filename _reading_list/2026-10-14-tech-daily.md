---
layout: reading
title: "大模型推理加速、量化与分布式训练"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-10-14
---

# 📰 2026-10-14 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 大模型推理加速 / KV 显存优化 / Flash Attention / vLLM 内核 DSL / 异步 GRPO / 推理自蒸馏 / 本地量化部署。

---

## 1. Adaptive DHiSparse：面向分层稀疏 KV 的推测解码与负载自适应驻留

**来源**：arXiv cs.CL
**链接**：http://arxiv.org/abs/2610.13708v1
**标签**：推测解码 · KV Cache · 长上下文推理 · 分层稀疏 · 显存优化

长上下文服务往往在 GPU 吞吐打满之前就用尽了 KV 显存。HiSparse 方案将历史存于主机内存、只拉取被选中条目，但它只使用固定缓存预算且不支持 DSpark、DFlash2 这类 draft-model 推测解码。本文提出 Adaptive DHiSparse，首次在分层稀疏 KV 之上支持推测解码，并让驻留策略随负载自适应调整——既保护必要工作集，又在显存余量内保留更多历史，从而在长上下文场景下同时获得更高吞吐与更低显存占用。

**核心要点**：
- 在分层稀疏 KV 之上打通 draft-model 推测解码（DSpark/DFlash2 兼容）
- 驻留策略随负载自适应，动态保护工作集并弹性保留历史
- 显存未打满前即可维持高吞吐，长上下文服务成本明显下降

---

## 2. 用 TLX 优化 Jagged Flash Attention：迈向 Blackwell 上的 SOTA FA4

**来源**：PyTorch
**链接**：https://pytorch.org/blog/optimizing-jagged-flash-attention-with-tlx-the-road-toward-sota-fa4-on-blackwell/
**标签**：Flash Attention · Blackwell · CUDA 内核 · TLX · 推理加速

本文介绍在 NVIDIA Blackwell(B200) 上实现的 Jagged Flash Attention(JFA)——Meta 生成式广告模型 GEM 背后的注意力内核。作者基于 TLX 对不规则（jagged）序列的变长注意力进行优化，梳理了从基线到 SOTA FA4 的调优路径，包括内存布局、occupancy 与指令级优化，为变长序列场景下的注意力计算树立了新的性能标杆。

**核心要点**：
- JFA 是 GEM 生产级注意力内核，面向变长/不规则序列
- 基于 TLX 在 Blackwell B200 上逐步逼近 SOTA FA4 性能
- 给出可复用的内存布局与指令级调优经验

---

## 3. 用 Helion 为 vLLM 构建高性能可移植的 Linear 后端

**来源**：PyTorch
**链接**：https://pytorch.org/blog/building-a-high-performance-and-portable-vllm-linear-backend-with-helion/
**标签**：vLLM · Helion · 内核 DSL · LLM 推理 · 自动调优

本文把高级内核 DSL Helion 集成进 vLLM 的 linear 后端，探索如何用自动调优的高层描述提升 LLM 推理性能、同时降低手写内核的复杂度。一个统一的 Helion 通用实现即可覆盖多种 shape 与硬件，兼顾性能与可移植性，减少了针对不同后端重复手写 Triton/CUDA 内核的负担。

**核心要点**：
- Helion 自动调优 DSL 替代手写内核，降低实现复杂度
- 单一通用实现覆盖多 shape/多硬件，提升可移植性
- 在 vLLM 推理路径上验证性能与易维护性的平衡

---

## 4. 跨 HF Jobs 的异步 GRPO + LoRA：一个桶、一个代理、没有 NCCL

**来源**：Hugging Face
**链接**：https://huggingface.co/blog/asyncgrpo-lora-hfjobs
**标签**：GRPO · LoRA · 异步训练 · RLHF · 分布式

TRL v1.14 的 AsyncGRPOTrainer 落地了 LoRA 支持（PR #7017）：异步训练器现在训练的是适配器而非全量模型，且只把 LoRA 适配器同步给 vLLM。本文给出一个真实工程——训练与推理不再同机，仅靠一个对象存储桶和一个代理完成参数交换，彻底甩掉 NCCL。由于 rank-1 适配器仅几 MB（对比全量 3GB），每次更新只推送适配器即可，vLLM 还能同时加载多个适配器，旧 rollout 用旧策略、新 rollout 用新策略。

**核心要点**：
- LoRA 适配器仅几 MB，替代全量 3GB 权重同步，省带宽
- 用对象桶+代理替代 NCCL，训练/推理彻底解耦分机
- vLLM 多适配器并发，rollout 按启动时的策略版本执行

---

## 5. 只打分不搜索的特权教师：基于 Critique 的同策略自蒸馏推理

**来源**：arXiv cs.CL
**链接**：http://arxiv.org/abs/2610.13744v1
**标签**：自蒸馏 · 推理训练 · RL · 信用分配 · 对齐

同策略自蒸馏试图用稠密的 token 级信号取代可验证奖励 RL 稀疏的结果信号：一个以参考解为条件的教师对学生自己的 rollout 打分，再把由此产生的分布偏移蒸馏回策略。本文用 rollout 中的金标准错误跨度做探针，检验“打分信号”是否真的包含纠错信用，并对比自蒸馏目标所消费的两类信号（全分布 KL 与局部对齐），厘清了哪些信号真正驱动了推理能力的提升。

**核心要点**：
- 同策略自蒸馏用稠密打分替代稀疏 RL 结果信号
- 借金标准错误跨度验证打分信号中的纠错信用
- 区分全分布 KL 与局部对齐两类信号对推理的贡献

---

## 6. Transformers 现已支持 llama.cpp 量化权重（GGUF）

**来源**：Hugging Face
**链接**：https://huggingface.co/blog/transformers-llama-cpp-quants
**标签**：量化 · GGUF · 本地推理 · llama.cpp · 模型部署

本地推理生态（Ollama、LM Studio、Jan 等）大量依赖 llama.cpp 引擎，而其核心权重格式是 GGUF 量化文件。本文宣布 transformers 现已能直接加载并服务 GGUF 量化权重——用户可以在保持 transformers 熟悉接口的同时，跑起 llama.cpp 风格的量化模型，大幅降低在笔记本等消费级设备上部署大模型的门槛，并可与既有推理/服务界面无缝衔接。

**核心要点**：
- transformers 原生支持加载 GGUF 量化权重
- 复用 llama.cpp 成熟量化格式，降低本地部署门槛
- 与现有 transformers 服务接口无缝对接

---

## 7. 用 FBTriton 现代化 Table Batched Embedding 内核

**来源**：PyTorch
**链接**：https://pytorch.org/blog/modernizing-table-batched-embeddings-with-fbtriton/
**标签**：Triton · Embedding · 推荐系统 · GPU 内核 · 性能优化

Table Batched Embedding(TBE) 是推荐系统的核心算子，负责跨数千张分片 GPU 的 embedding 查表与前反向计算。本文介绍 FBTriton 的 TBE 内核设计，用 Triton 重写前向与反向通路，在保持大规模分布式查表语义的同时提升 kernel 性能与可维护性，为推荐模型训练这一重量级负载提供了更现代、更易演进的内核实现。

**核心要点**：
- TBE 是推荐系统核心算子，跨数千 GPU 做 embedding 查表
- FBTriton 用 Triton 重写前/反向，提升性能与可维护性
- 面向大规模分布式推荐训练的重量级负载优化

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | Adaptive DHiSparse：面向分层稀疏 KV 的推测解码与负载自适应驻留 | arXiv cs.CL | 推测解码/KV Cache |
| 2 | 用 TLX 优化 Jagged Flash Attention：迈向 Blackwell 上的 SOTA FA4 | PyTorch | Flash Attention/Blackwell |
| 3 | 用 Helion 为 vLLM 构建高性能可移植的 Linear 后端 | PyTorch | vLLM/Helion |
| 4 | 跨 HF Jobs 的异步 GRPO + LoRA：一个桶、一个代理、没有 NCCL | Hugging Face | GRPO/LoRA |
| 5 | 只打分不搜索的特权教师：基于 Critique 的同策略自蒸馏推理 | arXiv cs.CL | 自蒸馏/推理训练 |
| 6 | Transformers 现已支持 llama.cpp 量化权重（GGUF） | Hugging Face | 量化/GGUF |
| 7 | 用 FBTriton 现代化 Table Batched Embedding 内核 | PyTorch | Triton/Embedding |


*自动生成 · 2026-10-14 · jeffinchen daily tech reading list*
