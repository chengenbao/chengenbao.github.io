---
layout: reading
title: "大模型量化对齐、注意力优化、扩散LLM加速、存内计算"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-05-19
---

# 📰 2026-05-19 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖大模型量化对齐、注意力机制硬件优化、扩散LLM推测解码加速、超参数跨架构迁移、过程奖励模型可靠性、片上SRAM计算引擎及vLLM推理工程实践。

---

## 1. 量化压缩破坏模型对齐：跨模型与数据集的偏见涌现

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2605.15208  
**标签**：量化 · 安全对齐 · LLM压缩 · 偏见涌现 · 后训练量化

Post-training quantization（PTQ）是部署大模型的主流压缩手段，但本文系统性地揭示了一个被忽视的副作用：量化会破坏安全对齐，导致偏见在压缩后的模型中涌现。研究跨多个主流模型系列与多种量化精度，发现低比特量化会放大模型在性别、种族等维度的偏见，且这一现象在不同架构和数据集上具有一致性。

**核心要点**：
- 后训练量化不仅损失能力，还会系统性地破坏安全对齐属性
- 偏见涌现（Bias Emergence）在 4bit/8bit 量化下均有显著观测
- 现有量化流程缺乏对对齐属性的保护机制，需引入额外校验步骤
- 对 Llama、Mistral 等多系列模型进行了实验验证，结论具有普适性

---

## 2. GQLA：面向硬件自适应的大模型分组查询隐空间注意力

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2605.15250  
**标签**：注意力机制 · KV缓存压缩 · MLA · 硬件自适应 · 推理优化

Multi-head Latent Attention（MLA，DeepSeek-V2/V3使用）通过联合压缩 KV 来降低推理内存开销，但其矩阵吸收技术在不同硬件上存在效率不一致的问题。GQLA 提出了分组查询隐空间注意力，在 MLA 基础上进一步引入分组查询设计，在不同硬件（GPU/NPU）上实现更优的吞吐与内存权衡，同时保持与 GQA 接近的模型质量。

**核心要点**：
- 在 MLA 的 KV 压缩基础上引入分组查询，兼顾内存与计算效率
- 解决 MLA 矩阵吸收在特定硬件上的加速失效问题
- 在推理延迟、内存占用、模型质量之间构建更优的 Pareto 前沿
- 适用于需要在边缘/NPU上部署大模型的场景

---

## 3. PSD：通过并行推测解码推进扩散 LLM 的 Pareto 前沿

**来源**：arXiv cs.CL  
**链接**：https://arxiv.org/abs/2605.15609  
**标签**：扩散LLM · 推测解码 · 推理加速 · 非自回归生成 · dLLM

扩散大语言模型（dLLMs）通过迭代去噪masked token序列来生成文本，天然支持并行解码，但解码步骤仍是瓶颈。PSD（Parallel Speculative Decoding）将推测解码引入 dLLM 框架，利用小模型草稿+大模型验证的机制，大幅减少所需迭代步数，在保持生成质量的前提下将推理速度提升显著。

**核心要点**：
- 首次将推测解码（Speculative Decoding）应用于扩散语言模型
- 草稿模型并行生成多个候选步，验证模型批量确认，减少迭代轮次
- 在质量-速度 Pareto 前沿上超越现有 dLLM 推理方法
- 为非自回归生成模型的工业部署提供了新的加速路径

---

## 4. GQA-μP：分组查询注意力的最大化参数化更新规则

**来源**：arXiv cs.LG  
**链接**：https://arxiv.org/abs/2605.15290  
**标签**：超参数迁移 · GQA · μP · 训练稳定性 · 模型缩放

μP（maximal update parameterization）使得超参数可以从小模型迁移到大模型，大幅降低调参成本。但原始 μP 未考虑 GQA（Grouped Query Attention）这一现代 LLM 的标配结构。本文推导了 GQA 下的 μP 更新规则（GQA-μP），填补了这一理论空白，使得超参数迁移在使用 GQA 的模型架构上也能正确工作。

**核心要点**：
- 推导出 GQA 架构下的正确 μP 初始化与学习率缩放规则
- 原始 μP 在 GQA 模型上存在超参数迁移偏差，本文给出理论修正
- 实验验证在不同 GQA 分组数下超参数均可从小模型正确迁移
- 对现代以 GQA 为标配的 LLM 训练具有直接工程价值

---

## 5. 基于学习可靠性的过程奖励模型

**来源**：arXiv cs.CL  
**链接**：https://arxiv.org/abs/2605.15529  
**标签**：过程奖励模型 · PRM · RLHF · 推理链 · 奖励校准

Process Reward Models（PRMs）为推理链的每一步提供奖励信号，但现有 PRM 通常输出点估计，无法衡量自身的不确定性。本文提出为 PRM 引入可靠性学习（Learned Reliability），使模型能够在每步奖励预测时同时输出置信度，从而在不确定的推理步骤上降低其权重，提升 RLHF 训练和搜索解码的效果。

**核心要点**：
- 现有 PRM 缺乏不确定性建模，导致低质量步骤奖励被过度信任
- 引入可靠性估计头，为每步奖励附加置信权重
- 在数学推理和代码生成基准上验证了可靠性加权的优越性
- 对 Best-of-N 搜索和 RLHF 训练流程均有实质性改进

---

## 6. 基于 10T SRAM 的 XNOR 片上存算引擎：AI 硬件面积效率提升

**来源**：arXiv cs.AR  
**链接**：https://arxiv.org/abs/2605.16161  
**标签**：存内计算 · SRAM · XNOR · AI硬件加速 · 面积效率

本文提出利用 10T SRAM 单元实现 XNOR 运算的片上存算架构（Digital Custom Compute Engine），专为二值/低精度神经网络推理设计。相比传统 SRAM+外部 ALU 方案，该架构将计算单元内嵌于存储阵列，大幅减少数据搬运开销，在面积效率（GOPS/mm²）上取得显著提升，适合边缘 AI 推理场景。

**核心要点**：
- 10T SRAM 单元原生支持 XNOR 运算，消除数据搬运瓶颈
- 相比标准 6T SRAM+外挂计算，面积效率显著改善
- 针对二值网络（BNN）和低精度量化模型优化，能效比突出
- 为边缘侧 AI 推理芯片设计提供了面积-性能权衡的新参考点

---

## 7. vLLM 与 PyTorch 协同优化：改善 aarch64 开发者体验

**来源**：PyTorch Blog  
**链接**：https://pytorch.org/blog/vllm-and-pytorch-work-together-to-improve-the-developer-experience-on-aarch64/  
**标签**：vLLM · PyTorch · aarch64 · 推理部署 · CUDA轮子

PyTorch 2.11 实现了在 aarch64 Linux 平台上直接通过 PyPI 安装支持 CUDA 的 PyTorch 轮子，消除了过去需要自定义编译的复杂流程。vLLM 与 PyTorch 团队联合推动了这一改进，使 ARM 架构服务器上的 LLM 推理部署更加便捷，对基于 AWS Graviton、NVIDIA Grace 等 aarch64 平台的推理集群尤为重要。

**核心要点**：
- aarch64 CUDA PyTorch 轮子现可直接 pip install，无需源码编译
- vLLM 在 aarch64 上的安装和性能得到系统性改善
- 对 ARM+NVIDIA Grace 等异构推理服务器部署链路有实质性简化
- 是 PyTorch 生态拥抱 ARM 服务器市场的重要里程碑

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | 量化压缩破坏模型对齐：偏见涌现的系统性研究 | arXiv cs.LG | 量化·安全对齐 |
| 2 | GQLA：硬件自适应分组查询隐空间注意力 | arXiv cs.LG | 注意力优化·推理 |
| 3 | PSD：并行推测解码加速扩散LLM | arXiv cs.CL | 推理加速·dLLM |
| 4 | GQA-μP：分组查询注意力的超参数迁移规则 | arXiv cs.LG | 训练·超参数迁移 |
| 5 | 基于可靠性学习的过程奖励模型 | arXiv cs.CL | RLHF·PRM |
| 6 | 10T SRAM XNOR存算引擎：AI硬件面积效率 | arXiv cs.AR | 硬件加速·存内计算 |
| 7 | vLLM+PyTorch协同优化aarch64推理体验 | PyTorch Blog | 推理部署·工程 |

---

*自动生成 · 2026-05-19 · jeffinchen daily tech reading list*

