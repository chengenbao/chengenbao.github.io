---
layout: note
title: "大模型训练技术全景：预训练 · 微调 · 对齐 · 推理扩展"
permalink: /notes/llm-training/
---

# 大模型训练技术全景：预训练 · 微调 · 对齐 · 推理扩展

> 覆盖：Pre-training / SFT / RLHF / DPO / GRPO / Test-Time Scaling  
> 时间线：GPT-3（2020）→ InstructGPT（2022）→ Llama 3（2024）→ DeepSeek-R1（2025）  
> 笔记整理：chengenbao · 2026-04

---

## 一、大模型训练全流程

```
原始文本语料（数 TB～PB）
        ↓ 数据清洗 / 去重 / 配比
预训练语料（数万亿 tokens）
        ↓ Pre-training（自回归 Next Token Prediction）
Base Model（有能力，但"不听话"）
        ↓ SFT（有监督微调）
Instruction-following Model
        ↓ RLHF / DPO / GRPO（对齐）
Chat / Reasoning Model（能力 + 对齐）
        ↓ 可选：Test-Time Compute Scaling
Reasoning Model（o1 / R1 范式）
```

---

## 二、预训练（Pre-training）

### 2.1 数据工程：决定模型上限

预训练数据是模型能力的天花板，数据质量 > 数据数量。

**数据来源与配比（以 Llama 3 为例）**：

| 来源 | 占比 | 说明 |
|---|---|---|
| Common Crawl（过滤后） | ~67% | 质量差异大，需多轮过滤 |
| GitHub 代码 | ~8% | 提升逻辑/推理能力 |
| 书籍 / 论文 | ~10% | 高质量长文本 |
| 多语言数据 | ~5% | 提升多语言能力 |
| 合成数据 | 增长中 | GPT-4 生成，Phi 系列大量使用 |

**关键处理步骤**：
1. **URL 过滤**：去除 NSFW、低质网站
2. **重复数据去除**（Deduplication）：MinHash / SimHash，防止记忆单个样本
3. **语言识别**：保留目标语言
4. **质量过滤**：perplexity 过滤、规则过滤（标点比例、行长度等）
5. **数据配比调优**：不同来源数据的混合比例对最终性能影响显著（DoReMi 等方法自动搜索配比）

**Scaling Law 指导数据量**：Chinchilla（2022）发现最优训练：**模型参数数 × 20 ≈ 最优 token 数**。但实践中普遍 over-train（Llama 3 用 15T tokens 训 8B 参数），因为推理时小模型更经济。

### 2.2 Tokenizer

- 主流：**BPE（Byte Pair Encoding）**，如 GPT 系列、Llama
- 中文处理关键：词表大小和分词粒度直接影响中文压缩率和效率
- 词表大小：32K（早期）→ 64K（Llama 3）→ 128K（GPT-4o）
- **Vocabulary extension**：在预训练后扩展词表以增强特定语言效率

### 2.3 模型架构演进

Transformer Decoder-only 是当前主流，但细节持续演进：

| 组件 | 早期设计 | 当前主流 | 改进效果 |
|---|---|---|---|
| 位置编码 | 绝对位置编码 | **RoPE**（旋转位置编码） | 支持长上下文外推 |
| 注意力 | MHA（多头注意力） | **GQA**（分组查询注意力） | KV Cache 减少 8× |
| FFN 激活 | ReLU | **SwiGLU / GeGLU** | 更好的训练稳定性 |
| 归一化位置 | Post-LN | **Pre-LN / RMSNorm** | 训练更稳定 |
| 归一化方法 | LayerNorm | **RMSNorm** | 计算更简单高效 |
| 上下文长度 | 2K | **128K ～ 1M+** | RoPE + 长文本微调 |
| 专家机制 | Dense FFN | **MoE（稀疏激活）** | 参数增加但计算量不变 |

**RoPE 详解**：通过旋转矩阵编码相对位置，天然支持外推。通过 YaRN、RoPE Scaling 等方法可将预训练长度扩展至数倍。

### 2.4 分布式训练并行策略

训练百亿参数以上模型必须多卡/多机，核心是如何切分：

```
3D 并行 = 数据并行（DP）+ 张量并行（TP）+ 流水线并行（PP）

数据并行（DP）：
  每卡持有完整模型，数据切分 → 梯度 all-reduce
  ZeRO-1/2/3：进一步切分优化器状态/梯度/参数，显存从 O(M) 降至 O(M/N)

张量并行（TP）：
  矩阵运算按列/行切分到多卡
  通信：All-Reduce（节点内 NVLink，带宽密集）

流水线并行（PP）：
  模型层按深度切分，不同 micro-batch 在不同阶段重叠执行
  通信：点对点传 activation（节点间带宽要求低）

序列并行（SP）：
  超长序列的 attention 按 sequence 维度切分
  DeepSpeed Ulysses / Ring Attention
```

**主流框架**：Megatron-LM（TP+PP）、DeepSpeed（ZeRO）、PyTorch FSDP2

### 2.5 训练稳定性技巧

- **梯度裁剪**（Gradient Clipping）：防止梯度爆炸，阈值通常 1.0
- **学习率调度**：Warmup（线性）+ Cosine Decay，峰值 LR 随模型规模增大而降低
- **优化器**：AdamW 主流；Muon（基于 Nesterov+正交化）在 Llama 3.1 训练中显示潜力
- **Loss spike 处理**：跳过坏 batch、回滚检查点、降低 LR
- **混合精度（BF16）**：FP16 数值范围窄易溢出，BF16 更稳定，配合 FP32 master weights

---

## 三、监督微调（SFT）

将 base model 调教成能遵循指令的 Chat Model。

### 3.1 数据格式

```
System: 你是一个有帮助的助手。
User: 帮我解释量子纠缠。
Assistant: 量子纠缠是...
```

训练时只对 **Assistant 回答部分** 计算 loss（User/System 部分 mask 掉）。

### 3.2 数据质量 > 数量

LIMA（2023）论文：**仅用 1000 条高质量 SFT 数据可达到 GPT-4 级对话能力**，强调精选 > 堆量。

Alpaca → ShareGPT → Open Hermes → 合成数据（GPT-4 生成 + 自动过滤）是演进路径。

### 3.3 高效微调（PEFT）

全量 SFT 开销大，PEFT 方法只微调少量参数：

| 方法 | 参数量 | 原理 | 适用场景 |
|---|---|---|---|
| **LoRA** | ~0.1% | 低秩矩阵分解：W' = W + AB | 通用微调首选 |
| **QLoRA** | ~0.1% | LoRA + 4bit 量化底座 | 单卡消费级 GPU |
| **DoRA** | ~0.1% | 权重分解为幅度+方向，分别学习 | 性能优于 LoRA |
| **LoRA+** | ~0.1% | A/B 使用不同 LR | 收敛更快 |
| Prefix Tuning | 极少 | 在输入前加可训练 prefix token | 较少使用 |
| Adapter | 少量 | 在层间插入小网络 | 渐被 LoRA 取代 |

---

## 四、对齐训练（Alignment）

让模型输出更符合人类偏好，核心是：**Helpful · Harmless · Honest（3H）**。

### 4.1 RLHF（人类反馈强化学习）

**InstructGPT（2022）** 确立了三步范式：

```
Step 1: SFT
  收集示范数据 → 监督微调 base model

Step 2: Reward Model 训练
  收集偏好对（response A vs B，人工标注哪个更好）
  训练 RM：r(x, y) → scalar

Step 3: PPO 强化学习
  用 RM 作为奖励信号，对 SFT model 做 PPO
  同时用 KL divergence 约束模型不偏离 SFT 太远：
  reward = r(x, y) - β · KL(π_θ || π_SFT)
```

**PPO 的问题**：训练复杂（需要 4 个模型同时加载：actor/critic/ref/reward），显存和工程开销巨大。

### 4.2 DPO（直接偏好优化，2023）

**核心洞见**：PPO 的 RM + RL 两步可以合并为一个闭式优化目标。

```
DPO Loss：
L_DPO = -E[ log σ( β·log(π_θ(y_w|x)/π_ref(y_w|x)) 
                  - β·log(π_θ(y_l|x)/π_ref(y_l|x)) ) ]

其中：y_w = 偏好回答，y_l = 非偏好回答
```

**优势**：无需 RM，无需 RL 采样，直接 offline 训练，简单稳定。  
**局限**：offline 数据质量上限，模型分布偏移后难以持续改进（需要定期刷新数据）。

**DPO 变体**：
- **SimPO**：去掉 reference model，用平均 log-prob 作归一化
- **IPO**：修复 DPO 过拟合问题
- **RPO**：结合 SFT loss 防止能力退化

### 4.3 GRPO（Group Relative Policy Optimization，2024）

DeepSeek-Math / DeepSeek-R1 提出，专为**可验证奖励（数学/代码）**设计：

```
对同一问题采样 G 个回答 {y_1, ..., y_G}
用规则打分：r_i = 1（答对）或 0（答错）
组内归一化：A_i = (r_i - mean(r)) / std(r)

GRPO Loss = -E[ A_i · log π_θ(y_i | x) ] + β·KL
```

**与 PPO 的区别**：
- 不需要单独的 Critic 网络（用组内均值代替 baseline）
- 奖励来自规则（精确匹配），而非神经网络 RM（避免 reward hacking）
- 天然适合数学推理：答案对错可自动判断

**性能**：DeepSeek-R1-Zero 纯用 GRPO（无 SFT 热身）就能在 AIME 达到 71%，证明 RL 可以自发涌现推理能力。

---

## 五、Test-Time Compute Scaling（推理时计算扩展）

2024 年最重要的范式转变：不只靠大模型，还靠**推理时多想一会儿**。

### 5.1 两种 Scaling 的对比

| | Train-Time Scaling | Test-Time Scaling |
|---|---|---|
| 方式 | 增大模型/数据/算力 | 推理时增加计算（token）|
| 代表 | GPT-4 → GPT-4o | OpenAI o1 → DeepSeek-R1 |
| 上限 | 受数据质量制约 | 受问题难度和 token budget 制约 |
| 适用 | 通用能力提升 | 复杂推理任务 |

### 5.2 Long Chain-of-Thought（Long CoT）

核心思路：让模型在给出最终答案前，先进行**扩展思考（extended thinking）**：

```
<think>
让我仔细分析这个问题...
首先考虑情况1：...如果不成立，考虑情况2...
验证：带入检验，答案为 42。
实际上，等等，我刚才的思路有问题，重新想...
</think>
答案是 42。
```

关键特征：
- 允许**自我纠错（self-correction）**：中途发现错误可以回溯
- 允许**分支探索**：尝试多条路径
- token 预算越大，准确率越高（在一定范围内）

### 5.3 DeepSeek-R1 训练流程

```
Stage 1: Cold Start
  小量 long-CoT SFT 数据热身，让模型学会格式

Stage 2: GRPO RL（主要阶段）
  奖励 = 答案正确性（规则验证）+ 格式奖励（<think> 标签）
  模型自发学会：回溯、反思、验证

Stage 3: Rejection Sampling + SFT
  用 RL 模型生成大量数据，过滤高质量 CoT → 再次 SFT 蒸馏

Stage 4: 最终 GRPO
  综合能力的最后对齐
```

**关键发现（R1-Zero）**：即使不做 Stage 1 的 SFT 热身，纯粹的 GRPO 也能让 base model 自发学会：
- 分配更多 token 给难题（token scaling behavior）
- 主动回溯和自我纠错（"Wait, let me reconsider..."）
- 这些能力**不是 SFT 数据教会的**，而是 RL 过程中涌现的

### 5.4 Test-Time Scaling 的维度

| 方法 | 原理 | 代表 |
|---|---|---|
| **更长 CoT** | 增加 thinking token 数量 | o1, R1 |
| **Best-of-N** | 采样 N 个回答，取最高奖励 | 简单但有效 |
| **Beam Search** | 维护 top-K 候选路径 | Tree of Thoughts |
| **MCTS** | 用蒙特卡洛树搜索探索解空间 | AlphaCode 2 |
| **Process RM** | 过程奖励模型评估中间步骤 | Math-Shepherd |

---

## 六、近期重要进展时间线

| 时间 | 事件 | 核心贡献 |
|---|---|---|
| 2020-05 | GPT-3 | 1750亿参数，few-shot prompting 涌现 |
| 2022-01 | InstructGPT | RLHF 三步范式建立 |
| 2022-03 | Chinchilla | Scaling Law 修正：数据量与参数量应匹配 |
| 2023-02 | Llama 1 | 开源大模型生态起点 |
| 2023-05 | DPO | 替代 PPO，极大简化对齐训练 |
| 2023-10 | Mistral 7B | GQA + Sliding Window Attention |
| 2024-04 | Llama 3 | 15T token 训练，GQA 全面采用 |
| 2024-09 | OpenAI o1 | Test-Time Scaling 范式诞生 |
| 2025-01 | DeepSeek-R1 | 开源推理模型，GRPO 技术公开 |
| 2025-03 | Llama 4 | MoE 架构，多模态，iRoPE |
| 2025-04 | Qwen3 | 混合思考模式（thinking/non-thinking 切换） |

---

## 七、工程实践要点

### 训练基础设施 Checklist

```
计算：
  ✅ 选择并行策略（模型规模决定 TP/PP/DP 配置）
  ✅ 激活检查点（Gradient Checkpointing）权衡显存 vs 计算
  ✅ FlashAttention 开启（注意力计算 IO 优化）
  ✅ BF16 混合精度

数据：
  ✅ 数据去重（训练集内 + 与评测集去重）
  ✅ Token budget 规划（总 tokens = epochs × dataset size）
  ✅ Packing（短样本拼接填满 sequence length，提升 GPU 利用率）

稳定性：
  ✅ 定期保存 checkpoint（loss spike 回滚）
  ✅ 监控：loss、梯度 norm、学习率、GPU 利用率
  ✅ 异常 batch 检测（超长序列、异常 loss）
```

### RLHF / DPO 实践注意事项

- **Reference model**：保存 SFT 模型为 reference，RL 过程中冻结
- **KL 系数 β**：太大 → 模型不学；太小 → 崩溃（偏离 SFT 太远）。通常 0.01-0.1
- **奖励 hacking**：模型找到虚假高奖励策略（如过长回答），需要 length normalization
- **数据新鲜度**：DPO 训练一段时间后，offline 数据分布与当前策略偏差增大，需要重新采样

---

## 参考资料

- [Scaling Laws for Neural Language Models (Kaplan et al., 2020)](https://arxiv.org/abs/2001.08361)
- [Training language models to follow instructions (InstructGPT, 2022)](https://arxiv.org/abs/2203.02155)
- [Direct Preference Optimization (DPO, 2023)](https://arxiv.org/abs/2305.18290)
- [DeepSeekMath: Towards Ultimate Mathematical Reasoning (GRPO, 2024)](https://arxiv.org/abs/2402.03300)
- [DeepSeek-R1: Incentivizing Reasoning Capability (2025)](https://arxiv.org/abs/2501.12948)
- [The Llama 3 Herd of Models (Meta, 2024)](https://arxiv.org/abs/2407.21783)
- [LLM Post-Training 全景指南：从 RLHF 到 GRPO 再到 Agentic RL（知乎）](https://zhuanlan.zhihu.com/p/2012909606771400823)
