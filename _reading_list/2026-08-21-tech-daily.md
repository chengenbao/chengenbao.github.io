---
layout: reading
title: "KV Cache 容量规划、显存配置与混合推理优化"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-08-21
---

# 📰 2026-08-21 · 每日技术速递

> 今日精选 6 篇深度技术文章，聚焦推理部署的显存规划、KV Cache 优化与混合架构实践。

---

## 1. LLM 推理显存规划实战：32B 模型的机型选型与 KV 优化

**来源**：iWiki 技术手册（llm-inference-advisor）
**链接**：https://iwiki.woa.com/p/4018979451
**标签**：显存规划 · 机型选型 · KV Cache · TP 并行

推理部署的第一课是显存算账：32B BF16 模型约需 77GB 总显存（含 1.2 倍余量），双卡 A100 40GB 或单卡 80GB 是最低可用配置。手册系统给出参数量-精度-并行度-KV 余量的换算表，并针对"偏紧"配置给出 KV Cache 优化与 TP 并行的调参建议，是容量规划的标准参考。

**核心要点**：
- 显存构成：权重 + KV + 激活 + 框架开销的估算公式
- TP 并行度与单卡显存的匹配关系
- 显存偏紧时的降级路径：量化、KV 压缩、offload

---

## 2. KV Cache 显存优化的系统工程：分页、量化与卸载的组合拳

**来源**：技术博客（KV Cache 优化专题）
**链接**：https://dsa.hkust-gz.edu.cn/zh/blog/2026/05/28/kv-cache-optimization-for-efficient-long-contextllm-inference-from-algorithms-to-systems/
**标签**：KV Cache · 显存优化 · 量化 · Offload

从系统视角组合 KV 优化三板斧：分页管理消除碎片、KV 量化（8-bit 起步、4-bit 看精度容忍）、CPU/NVMe 卸载扩展容量。文章强调三者不是替代关系而是叠加关系，并给出组合使用的收益边界与调试清单。

**核心要点**：
- 分页/量化/offload 的独立收益与叠加效应
- KV 量化的精度敏感度：按层/按头差异化处理
- 卸载的带宽预算：PCIe 传输与重算的临界点

---

## 3. Hybe：GPU-NPU 混合系统的长上下文推理

**来源**：ACM SC'25（系统论文）
**链接**：https://dl.acm.org/doi/full/10.1145/3695053.3731051
**标签**：GPU-NPU · KV Offload · 长上下文 · 异构系统

百万 token 上下文的现实压力让"GPU 算力 + NPU 大内存"的混合架构走向台前。Hybe 把 KV Cache 放到 NPU 侧 LPDDR，用分块流水把 KV 传输隐藏在注意力计算之下，显存墙问题被转化为流水线设计问题。附完整的延迟分解实测数据。

**核心要点**：
- KV 容量与带宽的解耦：NPU 内存作为 KV 层
- 分块传输与注意力计算的流水重叠设计
- 与纯 GPU 方案在成本与长度上限上的对比

---

## 4. 投机解码的部署实践：从 EAGLE 到多 token 预测

**来源**：Hugging Face Blog（投机解码专题）
**链接**：https://huggingface.co/blog
**标签**：投机解码 · EAGLE · MTP · 推理加速

投机解码已在主流引擎全面落地，但部署细节决定实际收益：草稿模型与目标模型的显存共享、接受率的负载相关性、batch 大时投机失效的边界。HF 博客系列梳理 EAGLE/MTP 两代方案在 Transformers/vLLM 中的配置方法与调参经验。

**核心要点**：
- EAGLE 的特征级草稿 vs MTP 的多头草稿结构差异
- 接受率与 batch 大小的负相关：何时该关掉投机
- 显存预算：草稿模型的额外开销估算

---

## 5. 推理服务的容量评估与压力测试方法论

**来源**：技术博客（Serving Stack 选型）
**链接**：https://www.weigao.cc/ai-systems/llm-inference/inference-frameworks-2026/
**标签**：容量评估 · 压测 · SLO · 方法论

上线前的容量评估清单：输入/输出长度分布、并发曲线、SLO 目标（TTFT/TPOT）的确定，以及"同 identity 压测"的纪律——相同模型、相同量化、相同引擎版本下对比才有意义。文章给出压测脚本设计与常见误区（忽略 prefill 长度分布、用平均延迟掩盖尾延迟）。

**核心要点**：
- workload 画像先行：长度分布决定一切优化方向
- 同构压测原则与基线锁定方法
- 尾延迟（P99）而非均值才是 SLO 的正确度量

---

## 6. 双 A100 部署 32B 模型的调优实录：TP2 下的性能榨取

**来源**：技术博客（推理部署实战）
**链接**：https://iwiki.woa.com/p/4018979451
**标签**：TP 并行 · 32B 模型 · 调优实录 · 显存偏紧

显存偏紧配置（2×A100 40GB 跑 32B）的实战调优：TP2 的通信开销实测、KV 块大小与并发数的平衡、量化版本的取舍（AWQ 4-bit 换 KV 空间）。记录了从"能跑"到"可服务"的完整调参路径与最终性能数据。

**核心要点**：
- TP2 通信开销与计算的重叠情况实测
- 显存偏紧时的参数优先级：KV 块数 > 批大小 > 上下文上限
- 4-bit 权重量化释放显存给 KV 的收益测算

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | LLM 推理显存规划实战 | iWiki 手册 | 显存规划 |
| 2 | KV Cache 优化的系统工程组合拳 | HKUST-GZ Blog | KV Cache |
| 3 | Hybe：GPU-NPU 混合长上下文推理 | ACM SC'25 | 异构系统 |
| 4 | 投机解码部署实践：EAGLE 到 MTP | Hugging Face Blog | 投机解码 |
| 5 | 推理服务容量评估与压测方法论 | 技术博客 | 压测 |
| 6 | 双 A100 部署 32B 模型调优实录 | 技术博客 | TP 并行 |

---

*自动生成 · 2026-08-21 · jeffinchen daily tech reading list*
