---
layout: reading
title: "分布式训练、GPU算子加速与 MoE 软硬件协同设计"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-10-22
---

# 📰 2026-10-22 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 分布式训练/5D并行、GPU算子加速与量化、MoE硬件协同设计与专家剪枝、存内推理与可持续AI、RLVR推理训练、LLM Agent内存安全。

---

## 1. TorchTitan 更新：新 SOTA 模型、大规模性能与 TitanRL 强化学习支持

**来源**：PyTorch
**链接**：https://pytorch.org/blog/torchtitan-update-new-sota-models-performance-at-scale-titanrl/
**标签**：分布式训练 · 5D并行 · 大规模预训练 · RL训练

TorchTitan 是 PyTorch 原生的生成式 AI 大规模预训练/后训练平台。本次更新为 Muse Spark Open、Kimi K2.7/K3、DeepSeek-V4、Qwen3.5 等开放权重架构提供了全量训练栈集成，并贯穿其 5D 并行栈。团队还引入了基于类型化 Python recipe 的可组合配置系统，支持并行策略、模型组件与硬件的灵活覆盖，同时新增 RL 训练支持与通信优化特性。

**核心要点**：
- 原生支持 DeepSeek-V4、Kimi K3、Qwen3.5 等最新开放权重架构，覆盖 5D 并行栈
- 通过类型化 recipe + 模型配置变换实现可组合、可覆盖的训练配置
- 新增强化学习（TitanRL）训练支持与大规模开箱即用性能优化

---

## 2. AMD MI350 GPU 上的 FlyDSL：用统一算子 API 加速 PyTorch

**来源**：PyTorch
**链接**：https://pytorch.org/blog/accelerating-pytorch-on-amd-gpus-with-flydsl/
**标签**：AMD GPU · ROCm · 算子加速 · 量化GEMM · 注意力

FlyDSL 让 AMD MI350 系列 GPU 上的 PyTorch 用户无需调用自定义 kernel，即可通过既有算子 API 获得密集/分组 GEMM、融合多头注意力(FMHA)、MXFP8/MXFP4 量化 GEMM、RMSNorm 与 TopK 加速。注意力 kernel 作为 AOTriton 的附加后端随 0.14b 系列发布。在 vLLM 端到端 A/B 测试中，整体请求速度最高提升 2.04x。

**核心要点**：
- MXFP8/MXFP4 量化 GEMM 相对 ATen 几何平均提速 1.58x / 1.68x
- TopK 在 10 个小 K 用例上相对 ATen 几何平均提速 4.82x
- vLLM 端到端请求级加速 BF16 达 1.13x、量化路径达 2.04x

---

## 3. 3D-MoE：面向 3D 堆叠近存处理的 MoE 解码软硬件协同设计

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2610.26991
**标签**：MoE · 近存计算 · 3D堆叠DRAM · 负载均衡 · 硬件协同设计

Mixture-of-Experts 解码高度受内存带宽限制且专家负载倾斜严重。3D 堆叠 DRAM 近存处理(NMP)能提供所需带宽，但分块组织令负载均衡代价高昂。论文提出 3D-MoE，利用"专家互补性"(领域特化路由使各步负载波动相互抵消)，通过一次性互补性引导分组与局部感知映射建立粗粒度组间均衡，再用轻量协作内存网络(CMN)流式搬运热点专家权重瓦片处理残余热点。

**核心要点**：
- 提出"专家互补性"洞察：领域特化路由使各解码步专家负载相互对冲
- 离线一次性分组 + 局部感知映射建立粗粒度组间负载均衡
- CMN 轻量流式搬运热点专家权重瓦片，缓解残余组内热点

---

## 4. OptEP：基于重建最优的专家剪枝，大幅保留 MoE 模型能力

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2610.27123
**标签**：MoE · 专家剪枝 · 模型压缩 · 重建优化 · 持续训练

MoE 虽靠稀疏激活降低算力，但部署仍需存储大量专家参数。论文将训练后专家剪枝建模为子集选择问题：给定预训练 MoE 层，选出固定数量专家以最佳重建原输出。面对组合爆炸，作者提出 OptEP——可扩展的连续松弛方法，直接优化可微专家选择变量，再对保留专家权重做轻量重建更新。在数学推理与代码生成上大幅领先现有基线。

**核心要点**：
- 将专家剪枝形式化为可微连续松弛的子集选择优化问题
- 在 Qwen3-30B-A3B、GLM-4.5-Air、Mixtral-8x7B 等上绝对精度提升最高 +16.2%
- 激进剪枝下优势尤为明显，重建驱动的选择+权重适配保留更多能力

---

## 5. 面向 Transformer 推理的存内处理(PID)碳足迹建模与替换周期优化

**来源**：arXiv cs.AR
**链接**：https://arxiv.org/abs/2610.27345
**标签**：存内计算 · 推理加速 · 碳排放 · 可持续AI · 硬件评估

Processing-in-DRAM(PID)加速器将计算搬近数据，缓解内存带宽瓶颈并降低能耗，被视为 GPU 的可持续替代。论文提出覆盖隐含碳与运行碳的 PID 推理服务器碳建模框架，横跨 3 种 GPU 基线、6 种 PID 架构与 8 类 Transformer 负载，并提出老化感知的 10 年碳最优替换模型。结论显示引入 PID 最多仅增加 2.5% 隐含碳，却可降低最高 1.14x 总碳。

**核心要点**：
- 构建覆盖隐含碳 + 空闲/运行碳的 PID 推理服务器碳建模框架
- 评测 3 GPU 基线 × 6 PID 架构 × 8 Transformer 负载
- PID 引入最多 +2.5% 隐含碳，总碳最高降 1.14x；配对 GPU 选择至关重要

---

## 6. ASE：激活空间探索，在固定 rollout 预算下提升 RLVR 训练多样性

**来源**：arXiv cs.LG
**链接**：https://arxiv.org/abs/2610.27012
**标签**：强化学习 · RLVR · 推理训练 · 探索 · 激活空间

带可验证奖励的强化学习(RLVR)通过采样响应、自动验证打分来更新策略，但重复采样易收敛到相似推理路径。论文提出 Activation-Space Exploration(ASE)：对同一题正确/错误响应的激活差聚类，构建结构化潜空间探索向量库，在后续 rollout 中施加不同向量以探索替代推理路径。在 Qwen3 基座上以相同 rollout 预算超越 GRPO。

**核心要点**：
- 通过正确/错误响应激活差聚类，构建潜空间探索向量库
- 在后续 rollout 施加不同向量，探索替代推理路径提升多样性
- Qwen3-1.7B/8B 上 Mean@8 相对 GRPO 提升 4.87 / 1.87 个百分点

---

## 7. ClawMMU：为有状态的 LLM Agent 提供保护一致的虚拟内存

**来源**：arXiv cs.OS
**链接**：https://arxiv.org/abs/2610.28093
**标签**：LLM Agent · 内存安全 · 操作系统 · 提示注入 · 访问控制

AI Agent 保存大量来自网页与工具结果的长期记忆，但运行模型的 harness 不记录记忆的归属方，导致隐藏在抓取页面中的指令被当作用户指令执行。论文提出 ClawMMU 保护层，为每条记忆标注拥有者、作用域、来源与权威，并在内容进入上下文、写入/读出存储或驱动工具调用时校验标签。其核心理念"权威只升不自救"将记忆攻击转化为经典保护故障。

**核心要点**：
- 为每条 Agent 记忆标注 owner/scope/origin/authority 元数据
- 在上下文注入、存储读写、工具调用时强制校验权威标签
- "权威只升不自救"原则将提示注入转化为可拦截的保护故障

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | TorchTitan 更新：新 SOTA 模型、大规模性能与 TitanRL 强化学习支持 | PyTorch | 分布式训练 |
| 2 | AMD MI350 GPU 上的 FlyDSL：用统一算子 API 加速 PyTorch | PyTorch | GPU加速 |
| 3 | 3D-MoE：面向 3D 堆叠近存处理的 MoE 解码软硬件协同设计 | arXiv cs.AR | MoE硬件 |
| 4 | OptEP：基于重建最优的专家剪枝，大幅保留 MoE 模型能力 | arXiv cs.LG | MoE剪枝 |
| 5 | 面向 Transformer 推理的存内处理(PID)碳足迹建模与替换周期优化 | arXiv cs.AR | 存内推理 |
| 6 | ASE：激活空间探索，在固定 rollout 预算下提升 RLVR 训练多样性 | arXiv cs.LG | RL训练 |
| 7 | ClawMMU：为有状态的 LLM Agent 提供保护一致的虚拟内存 | arXiv cs.OS | Agent安全 |

*自动生成 · 2026-10-22 · jeffinchen daily tech reading list*

