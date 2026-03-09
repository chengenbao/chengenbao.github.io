---
layout: reading
title: "技术速递 2026-03-08：内存分层/LLM多租户推理/内核ML/机密计算"
category: tech
tags: [Tech, LLM, Linux, Memory, TEE, CUDA]
date: 2026-03-08
---

# 📰 每日技术速递 · 2026-03-08

> 精选方向：LLM 推理优化 · Linux 内核 · 内存系统 · 机密计算  
> 来源：arXiv cs.OS / cs.LG / cs.DC

---

## 1. OBASE：面向内存分层的地址空间重组

**论文**：Object-Based Address-Space Engineering to Improve Memory Tiering  
**链接**：[arXiv:2603.00378](https://arxiv.org/abs/2603.00378)  
**机构**：Google（基于 Meta/Twitter 生产 trace 验证）

**核心问题**：内存分层架构中，热页利用率极低——Google 生产负载分析显示，活跃页中高达 **97%** 的字节是"冷数据"，根源在于分配器按 size 而非访问模式放置对象（热/冷对象混杂同一页，单个热对象使整页无法迁移）。

**方案**：编译器 + 运行时系统（OBASE）作为页感知 OS 后端的对象感知前端，轻量级指针插桩追踪访问，无锁协议安全迁移对象，不修改任何 OS backend（kswapd/TMO/TPP/Memtis）。

**结果**：页利用率提升 **2–4×**，内存占用降低最高 **70%**，开销仅 2–5%。

---

## 2. Token Pools：多租户 AI 推理平台容量控制抽象

**论文**：Token Management in Multi-Tenant AI Inference Platforms  
**链接**：[arXiv:2603.00356](https://arxiv.org/abs/2603.00356)

**核心问题**：传统速率限制不感知推理执行代价，导致专用端点浪费或 admission control 与 autoscaling 不一致。

**方案**：Token Pools 控制面抽象，以 token 吞吐量、KV cache 用量、并发度表示容量权益，同一容量模型同时驱动 admission 和 autoscaling，支持 priority-aware 分配、差异化 SLO、debt-based 公平性，不修改底层推理运行时。

**实验（Kubernetes + vLLM backend）**：过载场景下，高优先级负载 P99 延迟保持有界；无 admission control 的 baseline 所有负载延迟无界退化。

---

## 3. ML Library in Linux Kernel：内核空间 ML 推理架构

**论文**：Machine Learning (ML) library in Linux kernel  
**链接**：[arXiv:2603.02145](https://arxiv.org/abs/2603.02145)

**核心挑战**：Linux 内核禁止 FPU 操作，ML 模型潜在引入严重性能退化。本文提出内核 ML 库基础架构，解决 FPU 限制问题，设计内核空间模型代理与用户空间模型线程的交互接口，为调度器/内存管理等子系统引入自适应 ML 决策铺路。

---

## 4. vmcacheⁿ：多层内存的虚拟内存辅助缓冲池

**论文**：Virtual-Memory Assisted Buffer Management In Tiered Memory  
**链接**：[arXiv:2603.03271](https://arxiv.org/abs/2603.03271)

**方案**：vmcacheⁿ 基于虚拟内存子系统与 OS 调用实现跨层页迁移，引入 `move_pages2` 系统调用提供细粒度迁移控制。TPC-C 负载下，相比 vmcache 实现最高 **4×** 查询吞吐提升。

---

## 5. Mica：可证明的机密计算工作流架构

**论文**：Sharing is caring: Attestable and Trusted Workflows out of Distrustful Components  
**链接**：[arXiv:2603.03403](https://arxiv.org/abs/2603.03403)

**方案**：Mica 将机密性与信任解耦，租户可显式定义并证明组件间所有通信路径，策略语言控制 Realm 间通信，基于 Arm CCA 实现，仅对 TCB 做最小改动。

---

*本期聚焦：内存系统底层优化 + LLM 推理基础设施 + 系统软件前沿*

[📄 iWiki 版本](https://iwiki.woa.com/p/4018555951) | [返回 Reading List](/reading-list/)
