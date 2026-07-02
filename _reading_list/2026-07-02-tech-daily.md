---
layout: reading
title: "大模型 KV Cache / GRPO 训练 / 隐私 AI / 脉冲神经网络 · 2026-07-02"
category: tech
tags: [Tech, arXiv, PyTorch]
date: 2026-07-02
---

## 1. SeKV：面向长上下文 LLM 的分辨率自适应 KV 缓存与层级语义记忆

**来源**：cs.CL / arXiv  
**链接**：https://arxiv.org/abs/2606.31145  
**标签**：KV Cache · 长上下文 · LLM · 强化学习 · 推理加速

大语言模型在处理长上下文时，KV Cache 的内存占用随序列长度线性增长，成为主要瓶颈。SeKV 提出了一种分辨率自适应的 KV Cache 方案，结合层级语义记忆机制，在不同重要性的 token 上使用不同精度的缓存策略，从而在维持模型性能的同时大幅降低内存开销。该方法在多个长上下文基准上取得了领先的效果。

**核心要点**：
- 分辨率自适应策略：对语义重要性高的 token 保留高精度 KV，对其余 token 使用压缩存储
- 层级语义记忆：引入多级内存结构，模拟人类的短期/长期记忆，提升超长序列的检索效率
- 实验结果：在 LongBench 等评测上，内存节省 40%+ 同时保持接近原始模型的准确率

---

## 2. 轨道数据中心中的 AI 调度：热串扰感知的可持续调度策略

**来源**：cs.AR / arXiv  
**链接**：https://arxiv.org/abs/2606.26150  
**标签**：LLM · 训练优化 · 热管理 · 轨道数据中心 · CI/CD

地面 AI 训练面临日益严峻的能源与水资源危机，轨道数据中心（ODC）因零运营碳排放被视为替代方案。本文研究了卫星集群中 AI 推理/训练任务的调度问题，重点分析轨道环境中芯片间热串扰（Thermal Crosstalk）对 GPU 性能的影响，提出了一种可持续的热感知调度算法，能显著降低热致降频和性能损失。

**核心要点**：
- 首次系统分析轨道数据中心中 GPU 集群的热串扰问题，建立热传导模型
- 提出热串扰感知的任务调度算法，通过合理分配 AI 负载降低芯片温度峰值
- 仿真结果：相比传统调度策略，可将热致性能损失降低 30% 以上

---

## 3. Predictable GRPO：大模型强化学习训练动态的闭合形式建模

**来源**：cs.LG / arXiv  
**链接**：https://arxiv.org/abs/2606.30789  
**标签**：LLM · GRPO · 训练优化 · 优化 · CI/CD

GRPO（Group Relative Policy Optimization）已成为提升 LLM 推理能力的标准工具，但其训练动态至今难以预测，调参成本极高。本文提出了 Predictable GRPO，通过推导 GRPO 训练过程的闭合形式解析模型，揭示了奖励方差、梯度范数与训练稳定性之间的数学关系，使训练行为可预测、可控制，为后续 RL post-training 研究提供了理论基础。

**核心要点**：
- 推导 GRPO 训练动态的闭合形式解，揭示 reward 方差对梯度更新的影响机制
- 基于解析模型提出自适应超参数调整策略，无需大量试错即可稳定训练
- 在多个数学推理任务上验证，训练曲线预测误差 <5%，调参效率提升显著

---

## 4. EnclaveX：基于 CPU/GPU TEE 的端到端隐私 AI 推理框架

**来源**：cs.OS / arXiv  
**链接**：https://arxiv.org/abs/2606.31408  
**标签**：LLM · 训练优化 · GPU · TEE · 隐私计算

LLM 的大规模部署依赖集中式云基础设施，用户数据与模型权重面临被窥探的风险。EnclaveX 提出了一个端到端的隐私计算框架，通过统一调度 CPU TEE（Trusted Execution Environment）与 GPU TEE，在不可信的云环境中实现 LLM 推理的全程机密性保护，同时将性能开销控制在合理范围内。

**核心要点**：
- 设计 CPU+GPU 混合 TEE 架构，解决传统 TEE 方案不支持 GPU 加速的局限
- 提出安全 KV Cache 传递协议，防止推理中间状态在 TEE 边界泄漏
- 实测开销：相比非加密推理，吞吐率下降 <15%，实现工业可用的隐私 LLM 推理

---

## 5. SpikON：面向在线脉冲神经网络学习的双并行高效加速器

**来源**：cs.AR / arXiv  
**链接**：https://arxiv.org/abs/2606.30926  
**标签**：训练优化 · 硬件加速器 · 脉冲神经网络 · SNN · 高效计算

脉冲神经网络（SNN）因其事件驱动、低功耗特性被视为 AI 芯片的未来方向，但现有加速器难以高效支持在线无监督学习（STDP）。SpikON 提出双并行计算架构，同时加速脉冲传播与突触权重更新，在保持在线学习能力的同时大幅提升吞吐量，能效比相比 GPU 方案提升 10 倍以上。

**核心要点**：
- 双并行设计：前向脉冲传播与 STDP 权重更新流水线并行，消除传统串行瓶颈
- 硬件感知的稀疏计算：利用脉冲稀疏性跳过零激活计算，动态节省功耗
- FPGA 实测：相比 GPU 基线，能效比提升 10.3×，支持实时在线学习

---

## 6. Miles：RadixArk 面向大规模 LLM RL 后训练的 PyTorch 原生框架

**来源**：PyTorch Blog  
**链接**：https://pytorch.org/blog/miles-a-pytorch-native-stack-for-large-scale-llm-rl-post-training/  
**标签**：PyTorch · RL后训练 · SGLang · Megatron-LM · 分布式训练

Miles 是 RadixArk 开源的大规模 LLM 强化学习后训练框架，以 SGLang 负责 rollout 生成、NVIDIA Megatron-LM 负责训练、Ray 进行任务编排，并集成 PyTorch 原生扩展能力。框架专为千卡规模的 RLHF/GRPO 训练设计，解决了 rollout 与 training 之间的高效调度与通信问题。

**核心要点**：
- SGLang + Megatron-LM 解耦设计：rollout 与 training 分布在不同节点，最大化 GPU 利用率
- 支持 GRPO/PPO 等多种 RL 算法，以 PyTorch 原生方式扩展至千卡集群
- Ray 编排层统一管理数据流与容错，支持断点续训和动态扩缩容

---

## 7. Cross-Repository CI Relay：PyTorch 跨仓库 CI 的可扩展解决方案

**来源**：PyTorch Blog  
**链接**：https://pytorch.org/blog/introducing-cross-repository-ci-relay-scalable-ci-for-pytorchs-out-of-tree-backends/  
**标签**：PyTorch · CI/CD · 跨仓库 · 编译器后端 · 工程效率

PyTorch 引入了跨仓库 CI Relay（CRCR）机制，当 pytorch/pytorch 主库有 PR 或提交时，自动触发并追踪下游仓库（如各编译器后端）的 CI 流程。该机制解决了 PyTorch 生态中 out-of-tree 后端（如 XPU、MUSA 等）无法及时感知上游变更并验证兼容性的问题，大幅提升了 PyTorch 多后端协作的工程效率。

**核心要点**：
- 自动触发机制：pytorch/pytorch 的每次 PR/commit 自动在下游仓库发起 CI 验证
- 统一状态追踪：在 pytorch/pytorch PR 页面展示所有下游 CI 状态，便于快速定位兼容性问题
- 无需修改下游仓库代码，通过 GitHub Actions relay 实现跨组织的 CI 联动

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | SeKV：面向长上下文 LLM 的分辨率... | cs.CL / arXiv | KV Cache |
| 2 | 轨道数据中心中的 AI 调度：热串扰感知... | cs.AR / arXiv | LLM |
| 3 | Predictable GRPO：大模型... | cs.LG / arXiv | LLM |
| 4 | EnclaveX：基于 CPU/GPU ... | cs.OS / arXiv | LLM |
| 5 | SpikON：面向在线脉冲神经网络学习的... | cs.AR / arXiv | 训练优化 |
| 6 | Miles：RadixArk 面向大规模... | PyTorch Blog | PyTorch |
| 7 | Cross-Repository CI ... | PyTorch Blog | PyTorch |

---

*自动生成 · 2026-07-02 · jeffinchen daily tech reading list*
