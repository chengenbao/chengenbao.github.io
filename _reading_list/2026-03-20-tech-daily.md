---
layout: reading
title: "技术速递 2026-03-20：优化器革新 × 推理加速 × 多Token预测"
category: tech
tags: [Tech, arXiv, LLM, 推理优化, 量化, 优化器, 多语言, RAG]
date: 2026-03-20
---

# 📰 技术速递 · 2026-03-20

> 今日 AI/系统前沿：优化器革新（MUD）× KV Cache 压缩（CARE）× 端侧量化（RAMP）× 无训练多Token加速 × 多语言Scaling Laws × LLM 溯源与幻觉缓解

---

## 精选论文

### 1. Beyond Muon: MUD — 更快的 Transformer 训练优化器

**[arXiv:2603.17970](https://arxiv.org/abs/2603.17970)** | arXiv cs.LG

MUD（MomentUm Decorrelation）在 Muon 基础上引入动量去相关机制，加速大规模 Transformer 训练收敛。对预训练优化器研究有直接价值。

`#优化器` `#Transformer训练` `#MUD`

---

### 2. CARE — 协方差感知分解，支持 Multi-Head Latent Attention

**[arXiv:2603.17946](https://arxiv.org/abs/2603.17946)** | arXiv cs.LG

针对 MLA 架构的 KV Cache 压缩新方法，秩增强分解降低推理内存压力，对 DeepSeek 类模型推理优化有重要意义。

`#MLA` `#KVCache` `#推理优化` `#注意力机制`

---

### 3. RAMP — 强化学习驱动混合精度量化，端侧 LLM 高效推理

**[arXiv:2603.17891](https://arxiv.org/abs/2603.17891)** | arXiv cs.LG

RL 搜索每层最优量化位宽，在端侧设备保持精度的同时显著降低内存和延迟。

`#量化` `#混合精度` `#端侧推理` `#RL`

---

### 4. ShapleyLaw — 多语言 Scaling Laws 的博弈论框架

**[arXiv:2603.17945](https://arxiv.org/abs/2603.17945)** | arXiv cs.CL

用 Shapley Value 量化各语言对多语言模型性能的贡献，为预训练数据配比优化提供理论支撑。

`#ScalingLaws` `#多语言` `#Shapley` `#预训练`

---

### 5. 无训练多 Token 预测：嵌入空间探针加速推理

**[arXiv:2603.17942](https://arxiv.org/abs/2603.17942)** | arXiv cs.CL

无需修改模型结构，用嵌入空间轻量探针实现多 Token 预测，增强 speculative decoding 效果。

`#推理加速` `#MultiTokenPrediction` `#SpeculativeDecoding`

---

### 6. DebugLM — 可追溯训练数据来源的 LLM 框架

**[arXiv:2603.17884](https://arxiv.org/abs/2603.17884)** | arXiv cs.CL

让 LLM 输出可溯源至训练数据，支持幻觉溯源、数据影响分析和合规审计。

`#数据溯源` `#可解释性` `#幻觉` `#LLM审计`

---

### 7. 领域基础分层检索系统性缓解 LLM 幻觉

**[arXiv:2603.17872](https://arxiv.org/abs/2603.17872)** | arXiv cs.CL

文档 → 段落 → 句子三层检索精准注入领域知识，系统性降低 LLM 在专业领域的幻觉率。

`#RAG` `#幻觉缓解` `#分层检索` `#领域知识`

---

*由 OpenClaw 自动生成 · 2026-03-20*
