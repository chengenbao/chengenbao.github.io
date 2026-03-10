---
layout: reading
title: "技术速递 2026-03-10：稀疏注意力 · 序列并行 · 激活引导 · 多模态推理调度"
category: tech
tags: [Tech, arXiv, HuggingFace, 前沿, LLM推理, 分布式训练, 注意力机制, RLHF]
date: 2026-03-10
---

精选 6 篇围绕大模型训练/推理/对齐的前沿论文与技术博客，覆盖序列并行、稀疏注意力、免训练激活引导、不确定性对齐与多模态推理调度。

## 1. Ulysses 序列并行：Million-Token 上下文训练

**来源：** [Hugging Face Blog](https://huggingface.co/blog/ulysses-sp) · 2026-03-09

Snowflake AI Research 提出 Ulysses Sequence Parallelism，通过注意力头并行将注意力计算分布至多卡，支持 million-token 级超长上下文训练。文章详解与 Accelerate、Transformers Trainer 和 TRL SFTTrainer 的集成，对比 Ring Attention 权衡取舍，附基准测试数据。

---

## 2. torchforge + Weaver：可扩展 RL 助力 LLM 后训练

**来源：** [PyTorch Blog](https://pytorch.org/blog/supercharging-llms-scalable-rl-with-torchforge-and-weaver/) · 2026-01-09

Stanford/Meta/CoreWeave 联合发布 torchforge + Weaver，专为 LLM RL 后训练设计。Weaver 统一调度 actor/critic/reward 异步计算；torchforge 提供底层张量并行支持，大幅降低显存和通信瓶颈。

---

## 3. Stem：因果信息流视角下的稀疏注意力

**来源：** [arXiv 2603.06274](https://arxiv.org/abs/2603.06274) · 2026-03-06

初始位置 token 被所有后续 token 递归依赖，但现有 top-k 方法均匀对待各位置。Stem 引入 Token Position-Decay（位置相关 top-k）保留初始 token，配合 Output-Aware Metric 识别高影响 token，即插即用显著降低预填充延迟。

---

## 4. COLD-Steer：免训练 LLM 激活引导（ICLR 2026）

**来源：** [arXiv 2603.06495](https://arxiv.org/abs/2603.06495) · 2026-03-06

COLD-Steer 无需参数更新，在推理时近似模拟少样本微调对激活的效果来引导 LLM。两种实现：单位核近似法 + 仅需 2 次前向传播的有限差分法。以 1/50 样本量达到 95% 引导效果，在多元对齐任务上验证有效性。

---

## 5. RL 后训练让 LLM 学会估计校准不确定性

**来源：** [arXiv 2603.06317](https://arxiv.org/abs/2603.06317) · 2026-03-06

三阶段流水线：① 嵌入空间熵计算细粒度不确定性分数 → ② Platt Scaling 校准 → ③ RL 后训练对齐校准信号。推理时无需额外采样，泛化到未见任务，展现鲁棒的不确定性推理行为。

---

## 6. M-CMAB：多模态 LLM 推理多约束在线调度

**来源：** [arXiv 2603.06403](https://arxiv.org/abs/2603.06403) · 2026-03-06

M-CMAB 三组件：CLS 注意力适配器预测器 + 原对偶 Lagrange 约束器 + 两阶段调度器，在不可逆预算下平衡探索与利用。证明多维背包遗憾界，在异构后端基准上超越 SOTA 最高 14.18%。
