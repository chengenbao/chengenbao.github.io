---
layout: reading
title: "技术速递 2026-05-02：LLM推理加速 · KV Cache优化 · GPU编译器 · 量化加速器"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-05-02
---

## 1. SMG：LLM 推理服务中 CPU/GPU 解耦架构的探讨

**来源：** [PyTorch Blog](https://pytorch.org/blog/lightseek-smg/)  
**原标题：** *SMG: The Case for Disaggregating CPU from GPU in LLM Serving*

LightSeek 提出 SMG（Split Memory-compute Graph）方案，将 LLM 推理中的 CPU 前处理与 GPU 计算解耦，通过异步流水线消除 prefill/decode 之间的 CPU 瓶颈，实测可将吞吐量提升 2× 以上。

🔗 [阅读原文](https://pytorch.org/blog/lightseek-smg/)

---

## 2. AutoSP：基于编译器的序列并行，解锁超长上下文 LLM 训练

**来源：** [arXiv cs.LG](https://arxiv.org/abs/2604.27089)  
**原标题：** *AutoSP: Unlocking Long-Context LLM Training Via Compiler-Based Sequence Parallelism*

提出 AutoSP 框架，通过编译器自动分析计算图，将序列维度并行化切分到多卡，无需手动标注，支持 128K+ token 长上下文训练，并在 attention/FFN 层实现通信与计算重叠。

🔗 [阅读原文](https://arxiv.org/abs/2604.27089)

---

## 3. 大规模 GPU 推理中 KV Cache 的预测式多级内存管理

**来源：** [arXiv cs.AR](https://arxiv.org/abs/2604.26968)  
**原标题：** *Predictive Multi-Tier Memory Management for KV Cache in Large-Scale GPU Inference*

针对 LLM 推理时 KV Cache 显存溢出问题，提出基于访问模式预测的多级内存（HBM→DRAM→NVMe）动态调度策略，在保持低延迟的同时将有效显存容量扩展 3-5 倍。

🔗 [阅读原文](https://arxiv.org/abs/2604.26968)

---

## 4. VitaLLM：超紧凑三值量化 LLM 加速器与依赖感知调度

**来源：** [arXiv cs.AR](https://arxiv.org/abs/2604.27396)  
**原标题：** *VitaLLM: A Versatile, Ultra-Compact Ternary LLM Accelerator with Dependency-Aware Scheduling*

设计专用于三值权重（-1/0/+1）LLM 的 ASIC 加速器，引入依赖感知调度器消除层间数据竞争，芯片面积较 FP16 基线缩小 8×，在边缘设备上实现 10 TOPS/W 能效。

🔗 [阅读原文](https://arxiv.org/abs/2604.27396)

---

## 5. CuLifter：将 GPU 二进制提升为带类型 IR

**来源：** [arXiv cs.AR](https://arxiv.org/abs/2604.27486)  
**原标题：** *CuLifter: Lifting GPU Binaries to Typed IR*

提出 CuLifter 工具链，可将 CUDA/PTX 二进制逆向提升为带类型中间表示（LLVM IR），支持跨架构移植与编译优化，为闭源 GPU kernel 的分析与再优化提供基础。

🔗 [阅读原文](https://arxiv.org/abs/2604.27486)

---

## 6. Path-Lock Expert：架构级推理模式分离的混合思维模型

**来源：** [arXiv cs.CL](https://arxiv.org/abs/2604.27201)  
**原标题：** *Path-Lock Expert: Separating Reasoning Mode in Hybrid Thinking via Architecture-Level Separation*

针对 LLM 混合推理（快速直觉 vs 深度链式思维）模式干扰问题，在架构层设计独立的推理路径锁定专家，通过路由机制将 fast/slow thinking 完全解耦，推理准确率提升 4.2%。

🔗 [阅读原文](https://arxiv.org/abs/2604.27201)

---

