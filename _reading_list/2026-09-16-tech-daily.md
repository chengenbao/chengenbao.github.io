---
layout: reading
title: "KV 缓存压缩、显存分层与 MoE 高效推理"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-09-16
---

# 📰 2026-09-16 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 KV 缓存压缩、显存分层并发、自适应跳层、万亿参数 MoE 供给、移动端投机解码、GPU 模拟器与矩阵核心数值建模。

---

## 1. Grouped Value Attention：通过按需重建 Key 压缩 KV 缓存

**来源**：arXiv cs.AR (2609.13285)
**链接**：https://arxiv.org/abs/2609.13285
**标签**：KV Cache · GQA · 推理加速 · Transformer · 显存优化

KV 缓存是 Transformer 解码阶段的主要瓶颈，其显存占用与读取流量随序列长度增长。GQA 虽通过共享 KV 头降低成本，但每一步仍需同时存储 key 与 value。本文提出 Grouped Value Attention (GVA)：只存储分组后的 value，并用一个可学习的线性映射按需重建 content key；推理时该映射可被吸收进 query，从而无需在解码路径中实体化 content keys。一个轻量的解耦 RoPE 通道单独缓存位置 key 以保留位置信息。在研究的配置下，GVA 相比匹配的 GQA 将持久缓存标量减少约 45–47%，且接近 GQA 的精度。

**核心要点**：
- 提出 GVA：仅存分组 value，用线性映射按需重建 key，推理时可吸收进 query 省去 key 实体化
- 解耦 RoPE 通道单独缓存位置 key，保持位置编码信息不丢失
- 持久缓存标量相对 GQA 降低约 45–47%，350M 规模五任务平均精度 44.18（GQA 44.36、MLA 43.88）

---

## 2. BOOST：并发访问主机内存与 HBM 以加速 LLM 推理

**来源**：arXiv cs.AR (2609.13592)
**链接**：https://arxiv.org/abs/2609.13592
**标签**：LLM 推理 · 显存分层 · HBM · 主机内存 · vLLM

GPU 显存带宽与容量限制了 LLM 推理吞吐。现有服务系统将 HBM 与主机内存视为层级结构：数据放得下就只用 HBM，放不下再预取到 HBM，主机内存带宽始终未被充分利用；预取虽扩展了容量，却占用 HBM 写入带宽。BOOST 是首个无需改动 kernel 即可让两层显存并发、按比例访问的运行时系统，核心思路是利用 kernel 访问模式做 wave-aware 的页分配与数据管理。在 Grace Hopper 上集成进 vLLM，等批大小下 TPOT 比纯 HBM 提升 4.3%（预取法反而下降 6%），高吞吐下平均吞吐提升 31%。

**核心要点**：
- 指出预取浪费主机内存带宽、挤占 HBM 写入带宽的问题，提出并发按比例访问两层显存
- wave-aware 页分配：静态权重用模运算页放置消除访问比例方差，KV 页池随 GPU wave 动态分配
- 集成 vLLM，Grace Hopper 上吞吐平均提升 31%，无需修改 kernel

---

## 3. LayerRoute：自适应跳层 + LoRA 保质的参数高效 LLM 推理加速

**来源**：arXiv cs.CL (2609.13682)
**链接**：https://arxiv.org/abs/2609.13682
**标签**：层跳过 · 推理加速 · LoRA · 路由 · 小模型

LayerRoute 提出一种参数高效的 Transformer 自适应跳层方法：用直通估计器训练逐层硬门控路由，并与 LoRA 微调联合优化。在 Qwen2.5-0.5B-Instruct 的 24 个 block 上各加轻量 router（约 21.5K 参数）和 rank-8 LoRA（约 1.08M 参数）。10 次独立随机训练中，路由收敛到一致结构（中间 8–16 层共 9 层可跳过），每次都获得真实墙钟加速（1.02x–1.06x，均值 1.04x），且质量不降反升。router 对 87–100% 的留出样本做出真正随输入变化的跳/跑决策，证明是真实输入依赖路由而非固定剪枝。

**核心要点**：
- 逐层硬门控路由 + 联合 LoRA 微调，仅增约 1.1M 参数，单张 A100 训练不到 7 分钟
- 10 次随机种子均收敛到一致可跳层结构，验证为输入依赖路由而非固定模式
- 真实墙钟加速 1.04x 均值，困惑度在所有种子上优于原 backbone

---

## 4. Trillion-Parameter MoE in a Box：用高带宽闪存解耦万亿参数内存供给

**来源**：arXiv cs.AR (2609.15636)
**链接**：https://arxiv.org/abs/2609.15636
**标签**：MoE · 万亿参数 · 高带宽闪存 · 内存供给 · 系统设计

面向低并发下在单节点托管 TB 级权重的万亿参数 MoE 设备，本文结合两个万亿参数 MoE 的算子分析、真实专家路由 trace 与多轮 agent 服务 trace，探索高带宽闪存（HBF）与 DRAM 配置、带宽暴露与近数据计算的设计空间。核心发现：state 带宽与 HBF 传输形成两个基本正交的拐点。Q1：权重迁到 HBF 后 DRAM 所需带宽-容量比仅需 1.4–4.0 s⁻¹（比 HBM3e 的 33.3 低约一个数量级）；Q2：6 个 HBF 封装对主机暴露 384 GB/s/封装（聚合 2.30 TB/s，低于全暴露 62.5%），封装越多则单封装所需带宽越低。

**核心要点**：
- 给出万亿参数 MoE 单机供给的设计空间：HBF/DRAM 配置、带宽暴露、近数据计算
- state 带宽与 HBF 传输是两个正交拐点，各自解决不同的供给问题
- DRM 带宽-容量比需求仅 1.4–4.0 s⁻¹，远低于 HBM3e；多 HBF 封装降低单封装带宽压力

---

## 5. BigMoMo：移动端用投机解码实现大规模 MoE 高效推理

**来源**：arXiv cs.AR (2609.14643)
**链接**：https://arxiv.org/abs/2609.14643
**标签**：MoE · 移动端 · 投机解码 · NPU · 权重卸载

MoE 模型在手机上扩展容量，但专家卸载受限于有限的 DRAM 与高昂的数据搬运。顺序 token 路由把专家执行与碎片化 flash 读取、多阶段 NPU 准备耦合，使稀疏计算卡在权重传输上。BigMoMo 利用投机解码的多 token 验证窗口，将专家搬运与单 token 执行解耦，实现权重复用、连续 flash 读取与加载-计算重叠；并按接受率、路由影响与搬运成本剪枝投机分支与专家激活，依据运行时共加载模式重组 on-flash 专家。四个 MoE 模型、五个基准、两个移动平台下，相对按需自回归卸载平均加速 4.83x，相对最佳投机 MoE 基线 1.82x，支持至 30B 参数。

**核心要点**：
- 利用投机解码多 token 验证窗口解耦专家搬运与单 token 执行，实现权重复用与计算重叠
- 按接受率/路由影响/搬运成本剪枝分支，依运行时模式重组 on-flash 专家以批量就绪
- 移动端平均 4.83x 加速（vs 按需卸载），支持最大 30B 参数 MoE

---

## 6. FlashGPU-sim：面向现代架构与 AI 负载的开源周期精确 GPU 模拟器

**来源**：arXiv cs.AR (2609.15311)
**链接**：https://arxiv.org/abs/2609.15311
**标签**：GPU 模拟器 · 软硬协同 · Triton · 周期精确 · 微架构

现代 AI 系统由紧密的软硬协同设计驱动，后期 GPU 暴露异步数据移动、tensor core 流水线、细粒度同步等特性，而最新开源 NVIDIA GPU 模拟器还停留在约六年前的架构与软件栈，无法支撑 Triton 等现代编译器生成的 SOTA kernel。FlashGPU-sim 是开源、执行驱动、周期精确的现代 AI 负载 GPU 模拟器，忠实建模异步数据移动、细粒度同步、tensor core 执行与分布式共享内存；Triton 提取前端可直接仿真优化算子而无需手工移植。在 RTX 5090、H100、B200 的 131 个负载配置上周期级 MAPE 仅 5.24%，16 线程多线程仿真提速 7.86x。

**核心要点**：
- 填补开源 GPU 模拟器与现代架构（异步移动、tensor core 流水线）之间的鸿沟
- Triton 提取前端直接仿真，免去手工移植；多线程提速 7.86x
- 131 个负载周期级 MAPE 5.24%，并给出 H100 微架构探索案例

---

## 7. Accurate Models of AMD Matrix Cores：跨架构矩阵核心数值行为建模

**来源**：arXiv cs.AR (2609.14845)
**链接**：https://arxiv.org/abs/2609.14845
**标签**：矩阵核心 · AMD GPU · 数值精度 · 浮点 · 可复现性

近期 GPU 的矩阵乘不符合 IEEE 754，不同厂商乃至同厂商不同架构在累加器位宽、舍入、归一化点、中间上下溢、次正规与特殊输入处理上差异巨大，导致小矩阵结果无法跨设备复现且无法用软件控制。本文刻画 AMD 三代架构（CDNA1/2/3，对应 MI100、MI210/250、MI300A/300X）矩阵乘的数值行为，设计针对各输入格式的测试向量以反推每个数值特征；并为每代构建 MATLAB 软件模型，经 1000 万组随机向量随机测试迭代精修至位级可复现。作为示例，进一步量化了 AMD 矩阵核心与 NVIDIA 在应用级精度上的差异。

**核心要点**：
- 揭示 GPU 矩阵核心偏离 IEEE 754 导致的跨设备不可复现性问题
- 设计定向测试向量反推 CDNA1/2/3 各数值特征，构建位级可复现 MATLAB 模型
- 1000 万组随机测试迭代精修，并量化 AMD 与 NVIDIA 应用级精度差异

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | Grouped Value Atte | arXiv cs.AR | KV Cache |
| 2 | BOOST | arXiv cs.AR | LLM 推理 |
| 3 | LayerRoute | arXiv cs.CL | 层跳过 |
| 4 | Trillion-Parameter | arXiv cs.AR | MoE |
| 5 | BigMoMo | arXiv cs.AR | MoE |
| 6 | FlashGPU-sim | arXiv cs.AR | GPU 模拟器 |
| 7 | Accurate Models of | arXiv cs.AR | 矩阵核心 |

*自动生成 · 2026-09-16 · jeffinchen daily tech reading list*
