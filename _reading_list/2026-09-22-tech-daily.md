---
layout: reading
title: "前沿技术速递：稀疏注意力、MoE 解码加速、NPU 编译器与模型压缩"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-09-22
---

# 📰 2026-09-22 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖长上下文稀疏注意力、MoE 解码加速、NPU 编译器、高带宽闪存仿真、LLM 压缩、分词器工程与大规模强化学习训练栈。

---

## 1. Elastic Threshold Attention：面向长上下文解码的学习型上下文稀疏注意力

**来源**：arXiv (cs.LG)  
**链接**：https://arxiv.org/abs/2609.20888  
**标签**：长上下文 · KV Cache · 稀疏注意力 · 推理加速

长上下文解码中庞大的 KV Cache 会造成严重的显存带宽瓶颈。传统稀疏注意力通过选择性加载缓解，但僵化的启发式规则会丢弃必要上下文导致质量下降。本文提出 Elastic Threshold Attention (ETA)，一种端到端可训练的架构，通过从 query 表征直接预测动态、上下文相关的阈值，在困难检索/推理步骤分配稠密级上下文、剪除常规 token，从而在不牺牲稠密模型质量的前提下获得硬件加速的解码速度。

**核心要点**：  
- 端到端可训练：从 query 表征预测动态上下文阈值，对困难步骤保留稠密上下文、对常规 token 剪枝
- 训练时用乘性抑制（multiplicative suppression）将低于阈值的 logits 压向零而非直接删除，避免表征坍缩
- 在平滑的均匀注意力下限上训练，提供分布式、抗坍缩的学习信号
- 兼顾硬件加速的解码速度与稠密模型质量，无需后处理稀疏化

---

## 2. CARDAN：面向 scratchpad 张量加速器的 MoE 解码多引擎数据流

**来源**：arXiv (cs.AR)  
**链接**：https://arxiv.org/abs/2609.21137  
**标签**：MoE · 推理加速 · 张量加速器 · 向量量化

在基于 scratchpad 的张量加速器（STA）上做 MoE 解码，瓶颈在于搬运专家权重时计算引擎闲置。由于路由后才知道激活哪些专家，这类流量既难以隐藏也难在不损失质量的前提下压缩。本文提出 CARDAN，将每个专家权重矩阵表示为向量量化分量加共享基低秩分量，并将该表示与多引擎解码数据流协同设计，分离专家共享与专家私有计算，从而在多个引擎上重叠 DMA 与计算。

**核心要点**：  
- 将专家权重分解为「向量量化分量 + 共享基低秩分量」，从表示层面降低专家切换流量
- 协同设计的数据流在多个引擎上重叠 DMA 搬运与计算，隐藏专家权重移动
- 在 AWS Trainium3 上跨 5 个 MoE 家族，困惑度匹配或优于 BF16 teacher
- 批量 1 解码比 AWS dense MoE megakernel 快 1.15–1.31x，批量 16 时达 1.7x

---

## 3. 用开源编译器工具编程 AMD XDNA NPU：以 FlashAttention 为案例

**来源**：arXiv (cs.AR)  
**链接**：https://arxiv.org/abs/2609.21264  
**标签**：NPU · 编译器 · FlashAttention · MLIR · 能效

空间型 NPU（如 AMD XDNA）将计算 tile 置于小容量本地存储旁，数据在 tile 间的搬移完全交给软件。将多阶段负载映射到这类设备，本质上是决定中间张量存放在哪里的问题。本文分享了用开源 IRON 与 MLIR-AIR 流程为 FlashAttention 做这类 mapping 的经验，并在 XDNA 1 与 XDNA 2 上对比了四种参考设计。

**核心要点**：  
- 对比四种设计：逐算子独立运行、算子间片上流式传输、三阶段注意力融合为单一 kernel
- 融合 kernel 将 QK^T 分数保留在 compute-tile 本地存储、在级联互连上规约部分结果，分数不回写共享 MemTile
- 在 XDNA 2 上完整端到端执行达 3.62 TFLOP/s，为 IRON 设计的 2 倍
- 能效为集成方案的 5.3–7.2 倍，展示了空间型 NPU 上编译器驱动优化的空间

---

## 4. HBFSim：在真实 GPU 执行下快速高精度仿真高带宽闪存

**来源**：arXiv (cs.AR)  
**链接**：https://arxiv.org/abs/2609.09800  
**标签**：高带宽闪存 · GPU · 推理内存 · 仿真

高带宽闪存（HBF）将大容量 NAND 放在 HBM 旁以缓解 LLM 推理的内存容量瓶颈，但其系统级行为在硬件就绪前无法评估。周期级 GPU 仿真器对生产级模型太慢，而 trace 回放也无法捕获不同 HBM-HBF 配置带来的分配、迁移与执行变化。本文核心洞见是：HBF 无需仿真 GPU 本身，只需建模其程序可见效应。

**核心要点**：  
- 关键洞察：HBF 只需建模「程序可见效应」，而非仿真 GPU 微架构，从而可在真实硬件运行中注入
- 注入机制必须不破坏原本能隐藏 I/O 延迟的 GPU 并发
- HBFSim 是开源 HBF 仿真器，在真实 GPU 上执行 LLM 负载，准确刻画 HBM-HBF 配置带来的分配/迁移/执行变化
- 填补了 HBF 系统级评估在「硬件可用前」的空白

---

## 5. 像物理学家一样剪枝 LLM：把删块建模成伊辛优化问题

**来源**：Hugging Face Blog  
**链接**：https://huggingface.co/blog/MultiverseComputingCAI/pruning-llms-like-a-physicist-block-removal-as-an  
**标签**：模型压缩 · 深度剪枝 · 伊辛模型 · 优化

删除整个 transformer block（深度剪枝）能带来可预测的推理加速与内存节省，且可干净地叠加量化等方案，但难点在于决定删哪些块。删除任一 block 的效果依赖于同时删除了哪些其他块，使这是一个组合问题而非排序问题——而这正是自旋系统物理学擅长描述的。本文把 block 选择重构为约束二值优化（CBO）问题，直接映射到伊辛玻璃（Ising glass）。

**核心要点**：  
- 将 block 选择建模为约束二值优化，映射到「固定上旋数的全连接伊辛玻璃」，其能量作为剪后模型 benchmark 分数的廉价代理
- 无需实际跑 benchmark 即可对海量候选配置排序，难点实例交给经典/量子启发式求解器
- 在 Llama-3.3-70B-Instruct 压缩 50% 时，MMLU 比最佳竞品删块方法高近 23 个百分点
- 适用于深度压缩区，并可推广到稠密 transformer 之外

---

## 6. tokenizers v1：编码、解码与可测量的扩展

**来源**：Hugging Face Blog  
**链接**：https://huggingface.co/blog/tokenizers-v1  
**标签**：Tokenizer · 分词 · 性能工程 · 预处理

分词器在历史上并非 ML 流程的瓶颈，但当模型变快、工作负载扩展到海量数据训练与高并发服务时，这一平衡正在改变。本文介绍 tokenizers v1 的设计：用比特流（bitstream）替代正则、引入词缓存（word cache）与合并循环（merge loop）方法，并对扩展性能做了可测量的基准测试。

**核心要点**：  
- 核心拆分：用「比特流而非正则」做词元化，降低预处理开销
- 引入词缓存（word cache）与合并循环（merge loop）方法，提升重复/批量场景吞吐
- 对编码/解码与扩展（scaling）做了可测量的基准，量化改进
- 面向 1.0.0 / RC 阶段实现，目标是把分词从「轻量环节」变成可加速的关键路径

---

## 7. Granite 4.2 LLMs：构建之路

**来源**：Hugging Face Blog (IBM)  
**链接**：https://huggingface.co/blog/ibm-granite/granite-4-2  
**标签**：模型训练 · 强化学习 · 推理模型 · 量化

Granite 4.2 是 IBM 首个稠密、仅解码器的推理 LLM 家族，提供 3B / 8B / 30B 三种规模，从 Granite-4.1 基座后训练而来。基座在约 15T token 上以五阶段策略从头预训练，将上下文窗口扩展到 512K，经 CoT/推理/agent 轨迹数据监督微调后，再用多阶段强化学习管线后训练。

**核心要点**：  
- 架构为稠密、仅解码器推理模型，三规模 3B/8B/30B，Apache 2.0 许可
- 基座约 15T token 五阶段预训练，上下文扩展到 512K；多阶段 RL 含 agentic RL（8B/30B 在真实沙箱用工具学习）
- 支持 thinking/non-thinking 切换、低开销 thinking 模式、原生工具调用
- 量化支持 FP8 / FP4 / GGUF，并给出面向 Agentic 编码（OpenCode/OpenHands）等场景的基础设施

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | 面向长上下文解码的学习型上下文稀疏注意力 | arXiv (cs.LG) | 推理加速 |
| 2 | 面向 scratchpad 张量加速器的 MoE 解码多引擎数据流 | arXiv (cs.AR) | MoE加速 |
| 3 | 以 FlashAttention 为案例 | arXiv (cs.AR) | NPU编译 |
| 4 | 在真实 GPU 执行下快速高精度仿真高带宽闪存 | arXiv (cs.AR) | 内存仿真 |
| 5 | 把删块建模成伊辛优化问题 | Hugging Face Blog | 模型压缩 |
| 6 | 编码、解码与可测量的扩展 | Hugging Face Blog | 分词工程 |
| 7 | 构建之路 | Hugging Face Blog (IBM) | 训练栈 |

*自动生成 · 2026-09-22 · jeffinchen daily tech reading list*
