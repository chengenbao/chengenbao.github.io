---
layout: reading
title: "MoE压缩·KV Cache·可解释训练·端侧推理"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-08-12
---

# 📰 2026-08-12 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 MoE 专家压缩与剪枝、KV Cache 压缩、可解释语言模型、推理时算力分配、OS 内存翻译代码合成、端侧智能体推理。

---

## 1. Shape Mutating Expert Compression：LorExperts 与 BTExperts

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2608.07814  
**标签**：MoE · 专家压缩 · 低秩分解 · 模型部署 · 权重共享

Mixture-of-Experts（MoE）语言模型以低每 token 算力提供高容量，但部署需压缩大量专家权重矩阵。现有方法（REAP 剪枝、D²-MoE 低秩分解）要么牺牲精度、需重训路由，要么因专家权重近正交而随专家数增长急剧退化。本文提出 LorExperts 与 BTExperts 两类"形状可变"专家压缩方法，利用专家权重的近正交性进行更紧凑的分解，在保持全部专家与路由结构的同时显著降低存储与算力开销。

**核心要点**：
- 指出 D²-MoE 类共享低秩组件在专家数增多时因无法逼近近正交专家而精度骤降
- 提出形状可变的专家分解（LorExperts / BTExperts），无需重训路由器
- 在保持 MoE 容量前提下大幅压缩部署成本，利于大模型落地

---

## 2. CommitKV：面向多轮 Agent 的生命周期感知 KV Cache 压缩

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2608.07855  
**标签**：KV Cache · 推理加速 · 多轮 Agent · 内存优化 · ReAct

多轮 Reasoning-and-Acting（ReAct）智能体会累积不断增长的推理、工具调用与观测轨迹，其 KV Cache 随之膨胀，推高显存占用与注意力计算成本。现有 KV 压缩按当前轮注意力分数驱逐状态，但当前低注意力不代表未来无 relevant——暂 inactive 的信息可能后续变关键。本文提出 CommitKV，基于"提交转移（commit transitions）"识别生命周期边界，做生命周期感知的 KV 压缩。

**核心要点**：
- 揭示"当前轮低注意力 ≠ 未来无关"的 KV 误驱逐问题
- 用 commit transition 标记轨迹生命周期，区分可丢弃与需保留状态
- 针对多轮 Agent 场景显著降低 KV 内存与推理开销

---

## 3. 轻量微调下的路由敏感度识别 MoE 可剪枝专家

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2608.07890  
**标签**：MoE · 专家剪枝 · 参数高效微调 · 模型压缩 · 路由分析

MoE 部署仍需存储全部专家。近期理论表明：在微调中路由范数变化最小的专家可被剪枝且保持精度，但该结论假设全量微调。本文验证轻量适配能否恢复该信号——用参数高效适配器做简短微调，按诱导的 ℓ₂ 路由变化对专家排序，一次性剪掉变化最小的专家。在 Mixtral-8×7B-Instruct 上保留 44.83% 的 MMLU 表现同时大幅削减专家数。

**核心要点**：
- 将"路由范数变化最小即可剪枝"的理论从全量微调推广到轻量适配
- 用 PEFT 适配器短暂微调即可获得可靠的专家重要性排序信号
- 在 Mixtral-8×7B 上实现一次性、无需重训的显著专家剪枝

---

## 4. Scaling Inherently Interpretable Language Models

**来源**：arXiv cs.CL  
**链接**：https://arxiv.org/abs/2608.07594  
**标签**：可解释性 · 语言模型训练 · 训练约束 · 自回归 · 扩散 LM

可解释性常被视为能力的"税"：先训练不透明模型，再做事后解释，且解释可靠性难以验证。本文挑战这一前提——不逆向工程模型，而是把可解释性作为训练流程的约束，与语言建模目标联合优化。横跨三个数量级算力、在自回归与扩散语言模型上均验证：可解释性随能力一同扩展，而非相互制约。

**核心要点**：
- 将可解释性从"事后解释"前移为训练期联合优化约束
- 在自回归与扩散两类 LM 上验证可解释性随算力扩展而增强
- 提供比事后归因更可靠、可验证的内在可解释方案

---

## 5. Thinking Hard, Not Smart：推理模型无法跨题合理分配测试时算力

**来源**：arXiv cs.CL  
**链接**：https://arxiv.org/abs/2608.07968  
**标签**：推理模型 · 测试时算力 · 算力分配 · 评测框架 · LLM 推理

推理语言模型越来越多用测试时算力提升表现，但现有评测大多逐题独立考察。当多个问题共享端到端成本/延迟约束时，模型必须决定如何在题间分配有限推理算力。本文提出"考试式"评测框架：模型须将一份共享 token 预算按难度与分值分配到不同题目以最大化总分。实验显示多个开源推理模型在跨题算力配给上表现糟糕。

**核心要点**：
- 指出当前评测忽略"多题共享算力预算"的真实约束场景
- 设计考试式框架量化模型跨题分配推理算力的能力
- 揭示主流推理模型偏好"每题用力"而非"按需分配"的系统性缺陷

---

## 6. Velosiraptor：面向内存翻译的代码合成

**来源**：arXiv cs.OS  
**链接**：https://arxiv.org/abs/2608.07966  
**标签**：操作系统 · 内存翻译 · 代码合成 · 安全隔离 · 硬件配置

安全是 OS 开发者的首要关切，安全运行时依赖 OS 正确配置内存硬件以提供隔离与完整性保障。但平台内存硬件配置并非一次性工作——设计者不断推出带不同特性与配置方式的新翻译/保护机制。本文提出 Velosiraptor，用代码合成方法自动生成与验证内存翻译相关配置代码，降低人为出错与攻击面。

**核心要点**：
- 面向 OS 内存翻译/保护硬件的代码自动合成
- 应对不断演进的内存机制带来的配置复杂性与安全挑战
- 通过合成+验证保障隔离完整性，减少手动配置漏洞

---

## 7. Fast, On Device Agentic AI with Muse Glimmer on ExecuTorch

**来源**：PyTorch Blog  
**链接**：https://pytorch.org/blog/fast-ondevice-agentic-ai-with-executorch/  
**标签**：端侧推理 · ExecuTorch · 模型蒸馏 · 智能体 · NVIDIA

Meta 发布 Muse Glimmer：一个从 Muse Spark 蒸馏而来的 300 亿参数开放权重模型，面向端侧智能体工作流。与此同时，ExecuTorch 正加入端到端支持，使 Muse Glimmer 可在 NVIDIA 等端侧硬件上高效运行，将智能体能力下沉到设备本地，降低延迟与云端依赖。

**核心要点**：
- Muse Glimmer 为 30B 参数、从 Muse Spark 蒸馏的开放权重端侧模型
- ExecuTorch 新增端到端支持，可在 NVIDIA 等设备上原生运行
- 推动低延迟、隐私友好的本地智能体推理部署

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | Shape Mutating Expert Compression (LorExperts/BTExperts) | arXiv cs.LG | MoE 专家压缩 |
| 2 | CommitKV：生命周期感知 KV Cache 压缩 | arXiv cs.LG | 推理加速/内存 |
| 3 | 轻量微调路由敏感度识别可剪枝专家 | arXiv cs.LG | MoE 剪枝 |
| 4 | Scaling Inherently Interpretable LMs | arXiv cs.CL | 可解释训练 |
| 5 | 推理模型跨题算力分配缺陷 | arXiv cs.CL | 测试时算力 |
| 6 | Velosiraptor 内存翻译代码合成 | arXiv cs.OS | OS/安全 |
| 7 | Muse Glimmer 端侧智能体推理 | PyTorch | 端侧推理 |

---

*自动生成 · 2026-08-12 · jeffinchen daily tech reading list*
