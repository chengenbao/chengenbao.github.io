---
layout: reading
title: "技术速递 2026-04-24：LLM 推理加速与训练效率全景"
category: tech
tags: [Tech, arXiv, KV-Cache, MoE, 推理加速, 量化, 知识蒸馏]
date: 2026-04-24
---

今日精选 6 篇 arXiv 深度技术论文，聚焦 **LLM 推理加速、MoE 架构优化、量化压缩与知识蒸馏**四大方向。

---

## 1. TTKV：面向长上下文 LLM 推理的时序分层 KV Cache

**arXiv cs.CL** · [2604.19769](https://arxiv.org/abs/2604.19769)

提出时序分层 KV Cache 机制（TTKV），通过将历史 Token 的 KV 按时间分层压缩，将 KV Cache 显存占用从线性增长降为次线性，在保持推理精度的同时显著提升长上下文（128K+）场景下的吞吐量。是 PagedAttention/StreamingLLM 之后 KV Cache 管理的新一轮探索。

`KV Cache` `LLM推理` `长上下文` `显存优化`

---

## 2. Expert Upcycling：突破 MoE 计算效率前沿

**arXiv cs.LG** · [2604.19835](https://arxiv.org/abs/2604.19835)

研究如何将稠密模型的权重升级回收为 MoE 架构，在接近原始计算成本的前提下大幅扩展模型容量。实验表明，相比从头训练 MoE，Upcycling 策略可以在相同 FLOPs 预算下获得更好的下游任务性能，为大规模 MoE 模型的成本优化提供了新路径。

`MoE` `混合专家` `模型架构` `训练效率`

---

## 3. 投机解码加速生产级 LLM Agent：PayPal 实践报告

**arXiv cs.LG** · [2604.19767](https://arxiv.org/abs/2604.19767)

PayPal 在真实商业 Agent 系统上使用 EAGLE3 投机解码的工业落地报告。在不牺牲输出质量的前提下，推理延迟降低 30–40%，Token 吞吐量提升约 2×，详细分析了草稿模型选择与业务指标的权衡关系，是投机解码工业化落地的重要参考。

`推理加速` `投机解码` `EAGLE3` `工业落地`

---

## 4. 避免过度/不足思考：LLM 推理的课程感知预算调度

**arXiv cs.CL** · [2604.19780](https://arxiv.org/abs/2604.19780)

针对 Test-Time Compute 扩展中简单问题浪费 Token、难问题思考不足的矛盾，提出课程感知的推理预算动态分配方法，按题目难度自适应分配 Thinking Token。在数学推理基准上以约 50% 的 Token 消耗达到或超过固定预算方法。

`Test-Time Compute` `推理预算` `思维链` `计算效率`

---

## 5. PTQ 的两种失效模式：信号衰减与计算崩溃

**arXiv cs.CL** · [2604.19884](https://arxiv.org/abs/2604.19884)

系统分析大模型训练后量化（PTQ）失效的根本原因，发现低比特量化存在"激活信号衰减"和"注意力计算崩溃"两种独立失效模式，明确了 4-bit vs 2-bit 量化的质量边界，为 GPTQ、AWQ 等混合精度量化策略设计提供理论依据。

`量化` `PTQ` `低比特推理` `模型压缩`

---

## 6. LLM 混合策略蒸馏

**arXiv cs.CL** · [2604.20244](https://arxiv.org/abs/2604.20244)

提出混合策略知识蒸馏框架，结合序列级和 Token 级蒸馏损失，克服传统 KD 模式覆盖不足或梯度不稳定的问题。在多项生成任务上相较于单一蒸馏策略提升显著，尤其在 Teacher 70B 到 Student 7B 的大参数差距场景。

`知识蒸馏` `模型压缩` `LLM训练` `小模型`

---

*由 OpenClaw 技术速递 Bot 自动生成 · 2026-04-24*
