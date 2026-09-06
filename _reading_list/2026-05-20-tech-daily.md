---
layout: reading
title: "GPU 算子自动生成综述、Shader 编译器与 Transformer 推理引擎"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-05-20
---

# 📰 2026-05-20 · 每日技术速递

> 今日精选 6 篇深度技术文章，覆盖 LLM4Kernel 综述、推理引擎架构、分布式推理系统与算子层优化。

---

## 1. Harness + Workflow 实现 GPU Kernel 自动生成 Coding Agent 框架

**来源**：iWiki 技术笔记（LLM4Kernel 方向综述）
**链接**：https://iwiki.woa.com/p/4020603537
**标签**：GPU Kernel · Coding Agent · 综述 · 算子生成

系统性梳理 LLM4Kernel 方向的问题定义与工作流设计：高性能 GPU kernel 需要同时具备算法语义、硬件微架构、CUDA/Triton 编程与性能调优四类知识，Agent 框架的核心是用 Harness 把这四类知识工程化地注入生成-验证循环。对算子团队引入 AI 辅助开发是很好的入门综述。

**核心要点**：
- Kernel 生成 = 算法语义 + 微架构知识 + DSL 语法 + 调优经验的组合问题
- 生成→编译→正确性→性能的验证闭环是框架核心
- 数值精度（softmax 稳定性、归约顺序）是 Agent 最易翻车处

---

## 2. vLLM vs SGLang vs TensorRT-LLM：2026 推理引擎全景对比

**来源**：技术博客（推理引擎对比）
**链接**：https://charlestar.github.io/2026/05/11/2026%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%8E%A8%E7%90%86%E5%BC%95%E6%93%8E%E5%85%A8%E6%99%AF%E5%AF%B9%E6%AF%94/
**标签**：推理引擎 · 选型 · 性能对比 · Serving

从引擎架构、功能矩阵与性能数据三个维度对比 2026 年三大主流推理栈：vLLM 的生态广度、SGLang 的前缀缓存与调度灵活性、TensorRT-LLM 的 NVIDIA 深度优化。文章给出"按 workload 与 SLO 选引擎，而非按跑分选引擎"的实用结论。

**核心要点**：
- 三大引擎的设计哲学差异：生态 vs 调度 vs 峰值性能
- 前缀缓存、PD 分离、量化支持的功能矩阵
- 选型方法论：先定义 workload 与 SLO，再压测同构场景

---

## 3. Deepspeed-MII：低延迟大模型推理的民主化

**来源**：arXiv cs.CL（DeepSpeed Inference 论文）
**链接**：https://arxiv.org/abs/2207.00032
**标签**：推理加速 · 量化 · Kernel 注入 · 微软

DeepSpeed Inference 展示了"模型并行 + 算子注入 + 量化"三位一体的推理加速路线：自动替换 Transformer 层为融合内核，配合 INT8 量化与多卡切分，在延迟与成本之间取得平衡。其 kernel 注入机制是理解今天各推理框架"图改写 + 算子替换"思想的早期范本。

**核心要点**：
- 自动 kernel 注入：改写 Transformer 图而不改模型代码
- INT8 量化的显存减半与带宽收益分析
- 推理并行切分策略与延迟-吞吐权衡

---

## 4. Orca：面向 Transformer 的细粒度迭代级调度

**来源**：OSDI'22 经典论文（Orca）
**链接**：https://www.usenix.org/conference/osdi22/presentation/yu
**标签**：迭代级调度 · Continuous Batching · 推理服务

Orca 首创 iteration-level scheduling：不等整批请求完成，每一步解码后即可加入/退出请求，配合 selective batching 解决变长 batch 的算子执行问题。这个"continuous batching"机制是所有现代推理引擎（vLLM/SGLang/TGI）的调度基石。

**核心要点**：
- 迭代级调度消除请求级批处理的队头阻塞
- Selective batching：按算子特性选择批处理维度
- KV Cache 的请求级管理设计与 SDK 接口

---

## 5. FlashAttention-2：更快的注意力与更好的并行度

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2307.08691
**标签**：注意力内核 · Tensor Core · 并行化 · GPU

FlashAttention-2 重构了并行维度（沿序列维并行而非 batch/头维），改进 warp 间分工减少非矩阵乘运算，将 Tensor Core 利用率从 FA1 的 25-40% 提升到 50-73%。它是训练与推理内核从"能用"到"主力"的关键一步，也是理解内核级 occupancy 调优的教科书案例。

**核心要点**：
- 序列维并行提升长序列小 batch 场景的 GPU 占用率
- 减少 warp 间通信与非 GEMM 指令占比
- 前向/反向的 Kernel 级流水细节与 Hopper 适配

---

## 6. 编译器如何看你的代码：从 AST 到机器码的现代编译管线

**来源**：LLVM 官方文档 / Kaleidoscope 教程
**链接**：https://llvm.org/docs/tutorial/
**标签**：编译器 · LLVM · IR · 优化

AI 编译器（Triton/MLIR/XLA）的底层都是 LLVM 生态。Kaleidoscope 教程用最小可运行的语言实现，串起词法分析、AST、IR 生成、优化 pass 与 JIT 的完整管线。对想做"算子编译器"或理解 torch.compile 降级路径的工程师，这是最好的入门材料。

**核心要点**：
- 前端-IR-后端的分层解耦：优化 pass 的可组合性
- SSA 形式与数据流分析为何是优化的基础
- JIT 编译在动态语言与 AI 运行时中的角色

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | Harness + Workflow 实现 GPU Kernel 自动生成框架 | iWiki 笔记 | Kernel Agent |
| 2 | vLLM vs SGLang vs TensorRT-LLM 全景对比 | 技术博客 | 推理引擎 |
| 3 | Deepspeed-MII：低延迟大模型推理的民主化 | arXiv cs.CL | 推理加速 |
| 4 | Orca：面向 Transformer 的细粒度迭代级调度 | OSDI'22 | 调度 |
| 5 | FlashAttention-2：更快的注意力与更好的并行度 | arXiv cs.LG | 注意力内核 |
| 6 | 从 AST 到机器码：LLVM 编译管线入门 | LLVM 官方 | 编译器 |

---

*自动生成 · 2026-05-20 · jeffinchen daily tech reading list*
