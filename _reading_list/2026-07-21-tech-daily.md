---
layout: reading
title: "LLM 推理压缩与加速、MLIR 编译器、GEMM 性能与 CXL 共享内存"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-07-21
---

# 📰 2026-07-21 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 KV 缓存压缩、扩散模型解码加速、推理蒸馏、MLIR 编译器、GEMM 性能分析与 CXL 分布式共享内存。

---

## 1. VarRate：免训练的可变速率 KV 缓存压缩

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2607.15498
**标签**：KV Cache · 长上下文推理 · 低秩压缩 · 免训练 · 内存优化

KV 缓存是长上下文 LLM 推理的主要显存瓶颈。现有免训练方案各有结构性缺陷：token 选择法（SnapKV、Ada-KV）一旦误删重要 token 准确率会暴跌 11–15 点，而均匀低秩编码则对所有 token 一视同仁浪费预算。VarRate 提出按 query 显著性给每个 token 分配可变低秩预算、保留非零秩但不丢弃任何 token 的思路——这种按秩分配而非按删除的思路是两类失败的共同解药。在 LongBench 16 个任务上，匹配 20% 预算时 VarRate 在 Llama-3.1-8B 与 Qwen2.5-7B 上仅距无损模型 0.8 点，且预填充开销约为 KVzip 的八分之一。

**核心要点**：
- 指出 token 选择法误删与均匀低秩编码浪费两类失效可统一归结为「应分配秩而非删除 token」
- 按 query 显著性为每个 token 动态分配可变低秩预算，全程免训练
- 匹配 20% 缓存预算下接近无损，预填充开销仅为专用方法 KVzip 的约 1/8

---

## 2. AdaLook：扩散语言模型的自适应多步前瞻解码

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2607.15655
**标签**：扩散语言模型 · 推理加速 · 前瞻解码 · 并行生成 · 解码轨迹

掩码扩散语言模型（DLM）通过迭代细化被遮掩 token 实现并行文本生成，是自回归解码的有力替代。近期前瞻式解码在提交 token 更新前探索未来解码状态以改善精度—效率权衡，但主流方法依赖浅层单步前瞻，对长程解码轨迹次优；而朴素加深前瞻又会引入多余计算且无法适配异构的中间状态。AdaLook 提出自适应前瞻框架：依据候选分数方差动态决定是否继续 rollout，并在需要额外探索的中间状态启用分支扩展，从而既避免无谓的深 rollout，又能在信息丰富的中间态重新触发前瞻。多个基准与模型上，AdaLook 取得了优于现有单步前瞻解码的精度—步数权衡。

**核心要点**：
- 揭示单步前瞻对长程轨迹次优、朴素深前瞻计算浪费的缺陷
- 以候选分数方差驱动自适应 rollout 深度与分支扩展
- 在 DLM 上获得更优的精度—解码步数权衡

---

## 3. BIRD：基于自推理蒸馏的紧凑推理模型训练

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2607.15736
**标签**：推理蒸馏 · 自蒸馏 · Chain-of-Thought · 推理压缩 · Qwen3

大型推理模型常以冗长 CoT 求解，但大量算力耗散在冗余推导与重复自验上。现有 on-policy 自蒸馏仅对采样到的前缀施加监督，从冗长基模型起步时 KL 损失落在噪声、冗余甚至已跑偏的上下文上，形成初始化瓶颈。BIRD（Bootstrapped Iterative Self-Reasoning Distillation）提出两阶段方法：先以简洁指令采样正确答案轨迹并做轻量 prompt-switch SFT，把「指令诱导的简洁」转成默认推理行为；再以此 warm 模型做 on-policy 反向 KL 蒸馏。在 Qwen3-8B 上，MATH-500 准确率从 86.2% 提升到 92.0%，平均回复长度从 3099 降至 1115 token。

**核心要点**：
- 指出 on-policy 自蒸馏的初始化瓶颈：监督落在噪声/冗余前缀上
- 两阶段：先简洁指令采样+prompt-switch SFT 做预热，再 on-policy 反向 KL 蒸馏
- Qwen3-8B 上精度提升且回复长度压缩至约 1/3

---

## 4. 基于 MLIR 的大模型编译方法（TPU-MLIR）

**来源**：arXiv cs.CL
**链接**：https://arxiv.org/abs/2607.15865
**标签**：MLIR · 编译器 · TPU · 推理调度 · 量化部署

LLM 已是 AI 加速器上的核心负载，但在专用硬件上部署仍面临两大挑战：如何将训练好的模型导入编译器友好的中间表示，以及在有限片上内存下高效调度自回归推理循环。本文提出基于 MLIR 的大模型编译方法，用 TopOp 与 TpuOp 两个方言表达：TopOp 作为与源框架和目标芯片均无关的高层图方言承载模型语义，TpuOp 作为目标硬件方言承载量化、层分组、内存布局等芯片相关决策；模型先表示为 TopOp，再逐层 lowering 到 TpuOp 并生成可部署二进制。每个 Transformer 层被静态拆分为 prefill、prefill_kv、decode 三阶段以适配提示并行与逐 token 生成的差异。该方法已在 TPU-MLIR 中实现，支持 Qwen、Llama、InternVL、MiniCPM-V 等及 GPTQ/AWQ/AutoRound 多种量化形式。

**核心要点**：
- 双方言设计：TopOp 表达语义、TpuOp 承载芯片相关决策，逐层 lowering
- Transformer 层拆为 prefill/prefill_kv/decode 三阶段适配不同计算特征
- 已落地 TPU-MLIR，支持多系列生成模型与多种量化部署形态

---

## 5. LLA：循环 Transformer 的跨循环 KV 压缩

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2607.15456
**标签**：KV 压缩 · 循环 Transformer · 后训练编解码 · 低秩 · 长上下文

循环权重共享 Transformer 通过复用同一 block 缩减参数量，但解码时仍为每个递归步保存独立 K/V 缓存。本文发现该 loop-indexed 缓存高度结构化：固定 token、层与头时，K/V 向量在循环维度上呈短低秩轨迹，而头与层轴则平坦得多。据此提出 Looped Latent Attention（LLA）——一种后训练缓存编解码器，只存储紧凑的 K/V 潜变量，仅在 attention 读取时重建循环特定的 K/V 向量。默认 per-head 编解码压缩递归，LLA-2D 进一步把头也折叠进单一潜变量以进入极限压缩区间。在匹配缓存预算下，per-head LLA 优于 head-axis MLA、跨层共享、KV 量化与最终循环复用。单张 H200 上，潜变量存储路径将 Ouro-1.4B 在 4k 上下文的批容量从 32 提升到 768 条序列（21.3x 压缩）。

**核心要点**：
- 揭示循环 KV 缓存的低秩结构：循环维轨迹短、头/层轴平坦
- 后训练潜变量编解码，仅在使用时重建 K/V，缓存缩减精确无损
- 单卡 H200 上 4k 上下文批容量提升 21.3 倍

---

## 6. 从 Roofline 到 Ruggedness：GEMM 性能曲面分解与平滑

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2605.29752
**标签**：GEMM · GPU 性能 · 性能分析 · Kernel 优化 · Roofline

相邻仅差 128 元素步长的 GEMM 问题吞吐可相差 30%——这种普遍存在的「性能崎岖度」被 roofline 分析与峰值 FLOPs 直觉完全忽略，却主导一切非峰值负载。本文提出性能崎岖度分析，作为 roofline 的补充框架：不以标量上界概括 GPU，而是把整个多维性能曲面作为研究对象，将其纹理分解为可归因于具体机制的成分，并区分软件可消除与硬件绑定损失。在 Intel Battlemage（Arc B580）BF16 NN GEMM 上以 32768 组 (M,N,K) 配置扫描实例化，提出 roughness 指标（平均逐步吞吐变化），并通过两阶段栈（best-of-six 动态 tile 选择 + 动态规划 padding/splitting 优化器）将 roughness 削减 70%、均值吞吐提升 30%。最终仅凭 datasheet 整数从第一性原理推导最优可达曲面，给出 Kernel Optimality Levels 评级。

**核心要点**：
- 定义 GEMM 性能崎岖度，揭示 roofline 无法捕捉的离散硬件导致的吞吐波动
- 两阶段栈（动态 tile 选择 + DP padding/splitting）削减 roughness 70%、提升均值吞吐 30%
- 仅凭 datasheet 从第一性原理推导最优曲面，给出 Kernel 最优性评级

---

## 7. xDSM：基于弹性 CXL 的分布式共享内存扩展多线程应用

**来源**：arXiv cs.OS
**链接**：https://arxiv.org/abs/2607.15569
**标签**：CXL · 分布式共享内存 · 操作系统 · 弹性页 · 近线性扩展

CXL 为分布式共享内存（DSM）提供了有前景的硬件基底，但跨多节点无缝扩展多线程应用仍是难题：现有方案需手动改代码共享非堆数据、采用刚性数据放置策略、并在亚微秒页错误环境下承受高昂处理开销。xDSM 提出构建于 CXL 之上的全空间、弹性 DSM 系统，可透明扩展未修改的多线程应用。其 OS-runtime 协同设计建立全局协调地址空间以无缝共享所有内存段；用动态、延迟驱动策略替代静态放置以在本地 DRAM 与 CXL 间主动平衡数据；并以空间局部性感知的弹性机制动态合并/拆分页来摊销页错误成本。15 种系统配置评估下，xDSM 较纯 CXL 基线快 1.5x–2.2x、较前沿混合 DSM 快 1.1x–2.2x，且近乎线性可扩展。

**核心要点**：
- OS-runtime 协同建立全局地址空间，透明共享所有内存段免改代码
- 延迟驱动的动态数据放置替代静态策略，弹性页合并/拆分摊销页错误开销
- 较 CXL 基线快 1.5–2.2x，近乎线性可扩展

---


## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | VarRate：免训练的可变速率 KV 缓存压缩 | arXiv cs.CL | KV压缩 |
| 2 | AdaLook：扩散语言模型的自适应多步前瞻解码 | arXiv cs.CL | 推理加速 |
| 3 | BIRD：基于自推理蒸馏的紧凑推理模型训练 | arXiv cs.CL | 推理蒸馏 |
| 4 | 基于 MLIR 的大模型编译方法（TPU-MLIR） | arXiv cs.CL | 编译器 |
| 5 | LLA：循环 Transformer 的跨循环 KV 压缩 | arXiv cs.LG | KV压缩 |
| 6 | 从 Roofline 到 Ruggedness：GEMM 性能曲面分解与平滑 | arXiv cs.AR | GPU性能 |
| 7 | xDSM：基于弹性 CXL 的分布式共享内存扩展多线程应用 | arXiv cs.OS | OS/内存 |

*自动生成 · 2026-07-21 · jeffinchen daily tech reading list*
