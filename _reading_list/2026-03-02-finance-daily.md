---
layout: reading
title: "财经精选 2026-03-02：DeFi 代币货币视角 / 深度学习选股基准 / Gibbs 后验组合"
category: finance
tags: [Finance, 财经, 宏观, DeFi, 量化投资, 风险管理]
date: 2026-03-02
---

# 💰 财经精选（2026-03-02）

> 精选来源：arXiv econ.GN / q-fin｜更新时间：2026-03-02

---

## 1. 代币层级全解：去中心化金融的"货币观"

**原文：** [Tokens All the Way Down: A Money View of Decentralized Finance](http://arxiv.org/abs/2603.01803v2) · econ.GN · 2026-03-02 · Wenbin Wu

传统银行体系中，存款—贷款的循环让 1 美元准备金支撑数美元债权。DeFi 用代币复刻了类似结构：本文构建覆盖 200 条区块链、10,200 个代币的"代币图"（Token Graph），刻画其层级体系与信用创造机制。

**关键发现：**
- 截至 2025 年末，每 1 美元底层资产支撑约 **4.7 美元**总债权，形成 DeFi 版"货币乘数"
- 代币层级图揭示了类似传统金融的"安全资产—风险资产"金字塔结构
- 该结构内嵌脆弱性放大机制，为 DeFi 系统性风险评估提供新工具

**[→ 阅读原文](http://arxiv.org/abs/2603.01803v2)**

---

## 2. 深度学习金融时间序列：大规模风险调整收益基准

**原文：** [Deep Learning for Financial Time Series: A Large-Scale Benchmark of Risk-Adjusted Performance](http://arxiv.org/abs/2603.01820v1) · q-fin.TR · 2026-03-02 · Adir Saly-Kaufmann, Kieran Wood & Jan Peter-Calliess

本文对现代深度学习架构在金融时间序列预测与仓位管理任务上做了大规模基准测试，核心指标为夏普比率。评估对象涵盖线性模型、循环网络、Transformer、状态空间模型及新兴序列表示方法，在跨资产日频期货数据上做样本外检验。

**核心发现：**
- 没有单一架构全面占优：不同市场状态下领先模型轮动
- 复杂模型相对良好调优的线性基线的夏普优势有限且不稳定
- 为量化行业"模型选型"提供了可复现的公开基准

**[→ 阅读原文](http://arxiv.org/abs/2603.01820v1)**

---

## 3. Gibbs 后验与参数化组合选择

**原文：** [The Gibbs Posterior and Parametric Portfolio Choice](http://arxiv.org/abs/2603.02455v2) · q-fin.PM · 2026-03-02 · Christopher G. Lamoureux

参数化组合策略面临估计风险。本文发展广义贝叶斯框架，在无需设定收益生成过程模型的前提下，对特征倾斜（characteristic tilts）与样本外收益给出后验分布——这是与投资者效用函数一致的唯一信念更新规则。

**关键发现：**
- Gibbs 后验是与效用一致的最接近分布，规避了传统贝叶斯对收益分布的强假设
- 直接输出特征策略后验，便于样本外收益的不确定性量化
- 为"因子投资 + 估计误差管理"提供了严谨的统计基础

**[→ 阅读原文](http://arxiv.org/abs/2603.02455v2)**

---

## 4. 分位数建模收益尺度动态：VaR 与 ES 预测

**原文：** [Quantile-based modeling of scale dynamics in financial returns for Value-at-Risk and Expected Shortfall forecasting](http://arxiv.org/abs/2603.02357v2) · econ.EM / q-fin.RM · 2026-03-02 · Xiaochun Liu & Richard Luger

本文提出半参数方法预测 VaR 与期望损失（ES）：将收益的条件尺度定义为两个分位数之差，用受限分位数回归建模。下行风险侧，VaR 由缩放后收益的左尾分位数导出，ES 用左尾以下分位数平均近似。

**关键发现：**
- 尺度差（分位数差）比传统波动率更稳健地捕捉收益分布尾部变化
- 在 VaR 与 ES 回测中表现稳定，不需要参数化收益分布假设
- 计算轻量，适合高频风险监控场景

**[→ 阅读原文](http://arxiv.org/abs/2603.02357v2)**

---

*生成时间：2026-03-02 | 数据来源：arxiv.org (econ.GN / q-fin)*
