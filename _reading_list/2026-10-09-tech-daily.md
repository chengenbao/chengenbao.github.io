---
layout: reading
title: "LLM 推理压缩与底层系统优化"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-10-09
---

# 📰 2026-10-09 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 KV Cache 压缩、低秩条件计算、Linux I/O 栈建模、3D-DRAM 存内计算编译器，以及 PyTorch 在 agentic 推理与专用硬件上的实践。

---

## 1. KVFetch：用时间预取补齐 KV Cache 压缩缺失的一半

**来源**：arXiv (cs.LG)
**链接**：https://arxiv.org/abs/2610.08811
**标签**：KV Cache · 推理压缩 · 时序预取 · 长上下文

随着上下文窗口扩展到数十万 token，KV Cache 压缩已成为高效 LLM 推理的关键。现有方法（打分淘汰、摘要补偿、卸载召回）都只按当前 query 的内容相关性决定保留/召回，只实现了「按内容关联查找」这一种访问模式，缺失了「按位置顺序遍历」。在 RAG、代码补全、结构化抽取等场景中，模型需要逐字复现上下文中的标识符与字段，内容淘汰会保留序列头部却丢弃后续，导致「顺序遗忘」、复现中途断裂。KVFetch 是一个无需训练、即插即用的框架，为任意打分压缩器开通时间召回通道：将被淘汰项降级到量化冷层，用单调读指针检测活跃拷贝，并将位置后继预取到固定大小的暖层槽位，且不增加注意力开销。

**核心要点**：
- 指出三类压缩器共享的「结构性不完整」：只支持内容关联查找，不支持按位置的顺序遍历。
- 提出训练无关、drop-in 的 KVFetch，用单调读指针检测活跃拷贝并做时间预取。
- RULER-16K 下逐字复现从 0.8 恢复到 78.4，13 项任务平均 +8.4；LongBench 上通道静默、零开销。

---

## 2. 一次普通读背后：拆解 Linux I/O 栈的工作与等待

**来源**：arXiv (cs.OS)
**链接**：https://arxiv.org/abs/2610.10137
**标签**：Linux · I/O 栈 · 性能建模 · 内核

一个简单的 read 接口提供统一的功能语义，却不保证简单或可预测的性能行为。作者以 Linux 同步大缓冲读为观察窗口，端到端拆解 buffered-read 路径，区分工作总量、处理时间与关键路径暴露。研究发现 Linux 通过大 folio 降低元数据工作量，并将大部分冷读拷贝与设备等待重叠，但这些机制依赖访问建议、folio 粒度、缓存状态与后端执行。基于实验性内核原型，文章验证了乱序提前拷贝与机会并行带来的额外收益，并抽象出首完成等待 F 与后续完成容量 B_CQ，给出可预测等待的跨层模型。

**核心要点**：
- 端到端分解 buffered-read 路径，区分工作量、处理时间与关键路径暴露三类因素。
- 用 F 与 B_CQ 抽象设备等待，独立标定的设备参数对块层/读页等待预测误差 ≤6.9%/3.2%。
- 四个用例形成「观察—建模—解释—控制」闭环，为分层 I/O 栈性能决策提供可度量依据。

---

## 3. LRCC：用条件计算泛化低秩压缩

**来源**：arXiv (cs.CL)
**链接**：https://arxiv.org/abs/2610.08858
**标签**：低秩压缩 · 条件计算 · 模型加速 · Transformer

低秩压缩通过低秩分解替换线性变换来降低预训练语言模型成本，但传统方法在推理时使用固定秩分配，不区分输入 token。LRCC（Low-Rank Conditional Computation）为每个 Transformer block 训练一个轻量 router，在少量嵌套低秩路径中按 token 选择计算路径，训练时冻结低秩因子、仅优化 router。在 Llama 与 Qwen 上，相同平均激活参数量下，LRCC 优于静态低秩压缩，Llama-2-7B 下游平均准确率提升 7.6 个百分点；在匹配 batch-size-1 解码延迟下，Llama-3.2-1B 的困惑度与准确率均改善，且无需专用 kernel。

**核心要点**：
- 为静态低秩压缩引入 token 依赖的条件计算，按 token 选择嵌套低秩路径。
- 冻结低秩因子、仅训练每 block 的轻量 router，训练成本极低。
- 同等激活参数预算下显著优于静态压缩，且无需定制算子即可降低解码延迟。

---

## 4. 把半环动态规划编译到分层 3D-DRAM 存内计算

**来源**：arXiv (cs.AR)
**链接**：https://arxiv.org/abs/2610.09156
**标签**：存内计算 · MLIR · 编译器 · PIM

单片 3D（M3D）DRAM 上的存内计算（PIM）是突破数据密集型动态规划「内存墙」的有力方案，但现有实现需手写 kernel：程序员须手动定 tile 大小、跨非均匀延迟内存层放置数据、在异构处理单元间划分工作并插入广播与队列。GenMLIR 是一个 MLIR 编译器，将半环广义网格更新（APSP 与序列对齐共享的代数形式）作为一等 IR 抽象，经四组 GenDRAM 感知的 pass 派生分块、分层放置、tile-PU 指派与显式通信。生成代码在 GenDRAM 上比无编译器支持分别快 5.8×（APSP）与 16.7×（对齐）。

**核心要点**：
- 将半环广义 DP 更新提升为一等 IR 抽象，统一描述 APSP 与序列对齐。
- 四组 GenDRAM pass 自动完成分块、分层放置、PU 指派与通信，代码量从 300–500 行降至 4–9 行。
- 性能达可达上界的 90–100%，比标准仿射分块 PIM 编译器快 1.2–1.4×。

---

## 5. 会话感知的 Agentic 推理：NVIDIA Dynamo 实践

**来源**：PyTorch Blog
**链接**：https://pytorch.org/blog/session-aware-agentic-inference-with-nvidia-dynamo/
**标签**：Agentic · 推理服务 · Dynamo · 调度

Agentic 负载改变了推理服务器看到的流量形态：与单轮对话不同，一个 agent 会话可能包含一次大的初始 prefill、反复的模型调用，以及并行运行的子 agent。文章介绍如何基于 NVIDIA Dynamo 实现会话感知的 agentic 推理，针对 prefill/解码解耦、会话级复用与子 agent 并行调度进行优化，使推理服务更贴合 agent 工作负载的流量特征。

**核心要点**：
- Agentic 流量与单轮 chat 不同：大 prefill + 重复调用 + 并行子 agent。
- 基于 NVIDIA Dynamo 做会话感知调度，处理 prefill/解码解耦与会话级复用。
- 针对子 agent 并行运行做资源与调度优化，提升 agent 工作负载吞吐。

---

## 6. 把 Spyre 打造成原生 PyTorch 设备

**来源**：PyTorch Blog
**链接**：https://pytorch.org/blog/building-spyre-as-a-native-pytorch-device/
**标签**：PyTorch · 自定义设备 · PrivateUse1 · 编译器

Spyre 通过 torch-spyre 接入 PyTorch 既有的 device、allocator、stream 与 compiler 抽象，进而对接 Spyre 运行时与固件，成为 PyTorch 的「原生设备」。文章说明如何借助 PrivateUse1 机制为 Spyre 赋予真正的设备身份，使模型能像使用 CUDA/CPU 一样直接在 Spyre 上运行，降低了将专用加速硬件接入 PyTorch 生态的门槛。

**核心要点**：
- 用 PrivateUse1 给 Spyre 赋予真实设备身份，成为一等 PyTorch 设备。
- 通过 torch-spyre 桥接既有 device/allocator/stream/compiler 抽象到 Spyre 运行时。
- 模型可像使用 CUDA 一样直接在 Spyre 上运行，降低专用硬件接入成本。

---

## 7. 用 FBTriton 现代化表批量嵌入算子

**来源**：PyTorch Blog
**链接**：https://pytorch.org/blog/modernizing-table-batched-embeddings-with-fbtriton/
**标签**：Table Batched Embedding · Triton · 推荐系统 · GPU 算子

本文探讨 FBTriton 的 kernel 设计，覆盖表批量嵌入（TBE）的前向与反向计算。TBE 是推荐系统的核心算子，需要在上千张分片 GPU 上完成跨表的 embedding 查询。文章介绍如何用 Triton 重写这套基础算子，使其在现代 GPU 上获得更好的可维护性与性能表现，是推荐系统训练栈底层算子现代化的一次实践。

**核心要点**：
- TBE 是推荐系统核心算子，承担跨数千分片 GPU 的 embedding 查询。
- FBTriton 用 Triton 重写 TBE 前向/反向，提升可维护性与性能。
- 是推荐系统训练栈底层算子现代化、摆脱手写 CUDA 的实践案例。

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | KVFetch：用时间预取补齐 KV Cache 压缩缺失的一半 | arXiv (cs.LG) | KV Cache |
| 2 | 一次普通读背后：拆解 Linux I/O 栈的工作与等待 | arXiv (cs.OS) | Linux |
| 3 | LRCC：用条件计算泛化低秩压缩 | arXiv (cs.CL) | 低秩压缩 |
| 4 | 把半环动态规划编译到分层 3D-DRAM 存内计算 | arXiv (cs.AR) | 存内计算 |
| 5 | 会话感知的 Agentic 推理：NVIDIA Dynamo 实践 | PyTorch Blog | Agentic |
| 6 | 把 Spyre 打造成原生 PyTorch 设备 | PyTorch Blog | PyTorch |
| 7 | 用 FBTriton 现代化表批量嵌入算子 | PyTorch Blog | Table Batched Embedding |


*自动生成 · 2026-10-09 · jeffinchen daily tech reading list*
