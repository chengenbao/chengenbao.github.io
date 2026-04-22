---
layout: note
title: "大模型推理优化全景：KV Cache · 量化 · 投机解码 · 服务系统"
permalink: /notes/llm-inference/
---

# 大模型推理优化全景：KV Cache · 量化 · 投机解码 · 服务系统

> 覆盖：KV Cache 管理 / 量化压缩 / 投机解码 / Prefill-Decode 分离 / 推理服务框架  
> 对应框架：vLLM · SGLang · TensorRT-LLM · llama.cpp  
> 笔记整理：chengenbao · 2026-04

---

## 一、推理的本质瓶颈

LLM 推理与训练在计算特征上截然不同：

```
训练：
  大 batch → 高计算强度 → GPU 计算受限（Compute-bound）
  优化目标：最大化 MFU（模型浮点利用率）

推理（自回归解码）：
  每步只生成 1 token → 极低计算强度 → 内存带宽受限（Memory-bound）
  每步都要从 HBM 读取全部模型权重（70B 模型 = ~140GB）
  优化目标：最大化内存带宽利用率 + 降低延迟
```

**推理的两个阶段**：

| 阶段 | 特征 | 性能特征 |
|---|---|---|
| **Prefill（预填充）** | 处理输入 prompt，一次性计算所有 input token 的 KV | Compute-bound，与训练类似，GPU 利用率高 |
| **Decode（解码）** | 每步生成 1 个 token，循环执行 | Memory-bound，受 HBM 带宽限制，GPU 利用率低（通常 < 20%）|

---

## 二、KV Cache：推理加速的基石

### 2.1 为什么需要 KV Cache

自回归解码时，每生成一个新 token，都需要与**所有历史 token** 做 Attention：

```
没有 KV Cache：
  生成第 t 个 token 时，需要重新计算 token 1..t 的所有 Key/Value
  总计算量：O(t²)，随序列增长急剧膨胀

有 KV Cache：
  缓存历史 token 的 K/V，新 token 只需计算自己的 Q + 与缓存 K/V 做 Attention
  每步计算量：O(t)，总量 O(t²) 缩减为 O(t)（避免重复计算）
```

### 2.2 KV Cache 的内存开销

对于一个 70B 参数模型（如 Llama-3-70B）：
- 80 层，每层 K/V 各 8 个 head（GQA），每 head 128 dim，BF16
- **每 token 的 KV Cache = 80 × 2 × 8 × 128 × 2 bytes ≈ 327 KB**
- 100K tokens 的上下文 = 约 **32 GB**（接近一张 H100 的全部显存！）

**KV Cache 是当前推理显存的最大消耗者**，所有后续优化都围绕此展开。

### 2.3 GQA：减少 KV Head 数量

**Multi-Head Attention（MHA）**：Q/K/V 都有 H 个 head → KV Cache = H heads  
**Grouped-Query Attention（GQA）**：K/V 只有 H/G 个 head，G 个 Q 共享一组 KV → KV Cache 缩减 G 倍

```
MHA (H=64): KV Cache size = 64 heads 的 K + V
GQA (H=64, G=8): KV Cache size = 8 heads 的 K + V  →  缩减 8×
MQA (G=H): KV Cache size = 1 head 的 K + V  →  缩减 64×（精度损失较大）
```

Llama 3、Mistral、Qwen 2+ 均采用 GQA，是 2023 年后的工业标准。

---

## 三、PagedAttention 与 vLLM

### 3.1 传统 KV Cache 的问题：显存碎片化

传统实现预分配连续 KV Cache（按最大序列长度），导致：
- **内部碎片**：短请求也占用按最长长度分配的空间
- **外部碎片**：不同请求之间产生无法利用的碎片
- **GPU 利用率低**：实测只有 20-40% 的显存真正存放 KV，其余浪费

### 3.2 PagedAttention（vLLM 核心）

受操作系统虚拟内存分页管理启发：

```
将 KV Cache 切成固定大小的 Block（如每块 16 tokens）
用页表（Block Table）维护 逻辑 block → 物理 block 的映射
物理 block 按需分配，不再要求连续

优势：
  ✅ 显存利用率从 20-40% → >90%
  ✅ 支持 KV Cache 共享（prefix caching，多请求共享相同 prompt 的 KV）
  ✅ 批处理更多请求 → 吞吐量大幅提升
```

vLLM 开源（2023），已成为 LLM 推理服务的事实标准之一。

### 3.3 Continuous Batching（连续批处理）

传统静态 batching：等一批请求都完成才处理下一批，GPU 空闲等短请求。

```
连续批处理：
  每步 decode 后，检查是否有请求已完成（<EOS>）
  完成的请求立即移出 batch，新请求立即加入
  GPU 始终满载，无等待空洞
```

这是 vLLM / TGI / SGLang 的核心调度机制，配合 PagedAttention 大幅提升吞吐量。

---

## 四、KV Cache 高级优化

### 4.1 Prefix Caching（前缀缓存）

同一系统 prompt 被数千请求复用时，重复计算 KV 是巨大浪费：

```
System Prompt（1000 tokens）→ KV Cache 一次计算，缓存复用
每个新请求：直接从 cache 命中 system prompt 的 KV，只计算 user query 部分

命中率 > 90%（相同 system prompt 的生产系统中）
节省 prefill 时间和显存 I/O
```

SGLang 的 RadixAttention 用 Radix Tree 管理前缀 KV Cache，支持任意前缀共享。

### 4.2 KV Cache 量化

将 KV Cache 从 BF16（2 bytes/element）压缩到更低精度：

| 精度 | 压缩比 | 精度损失 | 代表方法 |
|---|---|---|---|
| FP16 → INT8 | 2× | 极小 | KVQuant, KIVI |
| FP16 → INT4 | 4× | 小 | KVQuant-4bit |
| FP16 → FP8 | 2× | 极小（Hopper 原生支持）| vLLM FP8 KV |

**关键技术**：KV 的分布与权重不同（激活值有异常值），需要 per-channel 或 per-token 动态量化。

### 4.3 KV Cache 稀疏化与压缩

不是所有 token 的 KV 都同等重要：

- **StreamingLLM**：只保留 attention sink（前几个 token）+ 滑动窗口，无限长度生成
- **H2O（Heavy Hitter Oracle）**：保留历史 attention score 高的 token KV，逐步 evict 低分 token
- **SnapKV**：在 prefill 阶段就预测哪些 KV 会被关注，提前压缩
- **MLA（DeepSeek）**：从架构层面解决，将 KV 压缩到低秩隐向量，Cache 减少 93%

---

## 五、模型量化

### 5.1 量化基础概念

```
浮点数（FP32/BF16）→ 低精度整数/浮点（INT8/INT4/FP8）

显存收益：
  FP32 → INT8：4× 压缩
  FP32 → INT4：8× 压缩
  BF16 → FP8：2× 压缩

推理加速：
  - 减少 HBM 读取量（Decode 阶段 memory-bound，直接提速）
  - 低精度矩阵乘法（INT8 GEMM 比 FP16 快 2-4×）
```

**量化分类**：

| 维度 | 类型 | 说明 |
|---|---|---|
| 量化对象 | W-only | 只量化权重（W4A16：4bit权重，16bit激活）|
| | W+A | 权重和激活都量化（W8A8, W4A8）|
| 量化时机 | PTQ（训练后量化）| 无需重新训练，速度快 |
| | QAT（训练感知量化）| 训练时模拟量化，精度高 |
| 粒度 | Per-tensor | 一个缩放因子，精度差 |
| | Per-channel | 每个输出通道一个因子，权重量化标准 |
| | Per-token | 每个 token 动态量化，激活量化必需 |

### 5.2 主流 PTQ 方法

**GPTQ（2022）**：
- 逐层量化，利用 Hessian 矩阵补偿量化误差
- 支持 INT4 权重量化，精度损失小
- 速度慢（需要校准数据集），但离线一次性
- 生产用途：AutoGPTQ，llama.cpp 的 GGUF 格式

**AWQ（Activation-aware Weight Quantization，2023）**：
- 观察：不同通道对精度影响差异巨大（1% 的 salient channels 贡献主要误差）
- 保护这 1% 的通道（scale up 使其减少量化误差），其余正常量化
- 更快的 calibration，比 GPTQ 精度更好
- 生产用途：AutoAWQ，vLLM 原生支持 AWQ INT4

**SmoothQuant（W8A8，2022）**：
- 激活值有异常值（outliers），直接量化误差大
- 将激活的难度"平滑"到权重：`Y = X W = (X/s)(sW)`，s 是 per-channel scale
- 实现真正的 W8A8，配合 INT8 GEMM 获得实际计算加速

**FP8 量化（2024+，Hopper/Blackwell）**：
- H100 原生支持 FP8 矩阵乘（GEMM），无需特殊软件
- 精度损失极小（FP8 保留浮点指数）
- 目前 **生产部署首选**（H100 以上硬件）
- vLLM/SGLang/TensorRT-LLM 均支持 FP8

**实践建议**（2026）：

| 场景 | 推荐方案 |
|---|---|
| H100/Blackwell GPU | FP8（原生支持，精度几乎无损）|
| 消费级 GPU（显存有限）| AWQ INT4（精度 > GPTQ，速度快）|
| 端侧部署（CPU/Apple M）| GGUF Q4_K_M（llama.cpp）|
| 极致精度要求 | BF16 不量化，或 GPTQ INT8 |

---

## 六、投机解码（Speculative Decoding）

### 6.1 核心思路

Decode 阶段每步只生成 1 token，GPU 利用率极低（< 20%）。投机解码利用这些"空闲算力"：

```
Step 1: 用小模型（Draft Model）快速生成 γ 个候选 token（γ 通常为 4-8）
Step 2: 用大模型（Target Model）一次性验证所有 γ 个 token（batch 并行）
Step 3: 
  - 如果 draft token i 被接受（大模型概率 ≥ draft 模型概率）：保留
  - 如果被拒绝：从大模型分布重新采样，后续 draft token 全部丢弃
  - 结果：等价于完全从大模型采样（无精度损失），但平均每轮产出多个 token

理论加速：α = 接受率，γ = draft 长度
  期望 token/round ≈ γ·α + 1
  实际加速比：1.5-4×（取决于任务和 draft 质量）
```

### 6.2 Draft Model 的选择

| 方案 | Draft 来源 | 特点 |
|---|---|---|
| **独立小模型** | 同家族小版本（如 Llama-3-8B draft Llama-3-70B）| 最通用，需额外显存 |
| **Self-Speculative** | 模型自身的早期层提前退出 | 无需额外模型，但灵活性受限 |
| **Medusa** | 在主模型上附加多个独立 head，并行预测多个未来 token | 结构简单，单模型推理 |
| **Eagle / Eagle-2** | 用 1 个 FFN 层作为 draft，共享主模型特征 | 高接受率（~70%+），速度快 |
| **n-gram draft** | 在输出历史中查找匹配前缀，复用已生成 token | 极简，适合重复性强的任务 |

### 6.3 关键指标

- **接受率（Acceptance Rate α）**：越高越好，取决于 draft 和 target 的分布相似度
- **草稿生成速度**：draft model 越快，整体加速越显著
- **延迟 vs 吞吐**：大 batch 时投机解码优势减弱（大 batch 时 GPU 利用率已经够高）

---

## 七、Prefill-Decode 分离（PD Disaggregation）

### 7.1 为什么要分离

混合部署时的矛盾：
- Prefill：Compute-bound，喜欢高并发大 batch
- Decode：Memory-bound，对延迟敏感

放在同一 GPU 上，新请求的 Prefill 会抢占正在 Decode 的请求的 GPU 时间 → **TTFT（Time To First Token）高，TBT（Time Between Tokens）抖动**。

### 7.2 分离架构

```
                     ┌─────────────────┐
Request ──────────── │  Prefill Cluster │ ──→ 计算 KV Cache
                     │  高 MFU，大 batch │
                     └─────────┬───────┘
                               │ 传输 KV Cache（关键开销！）
                     ┌─────────▼───────┐
                     │  Decode Cluster  │ ──→ 逐 token 生成
                     │  多副本，低延迟  │
                     └─────────────────┘
```

**优势**：
- Prefill 机器可用更大 batch，GPU 利用率更高
- Decode 机器数量可根据 QPS 独立扩缩容
- TTFT 和 TBT 分别优化

**核心挑战**：**KV Cache 传输开销**（Prefill 完成后要把 KV 搬到 Decode 机器）。解决方案：
- 同机器内：用共享内存 / NVLink 直接访问
- 跨机器：RDMA 传输 KV Cache，或通过网络直接映射（类似 NVSHMEM 思路）

代表系统：Mooncake（月之暗面）、Splitwise、DistServe

### 7.3 Chunked Prefill

折中方案（不完全分离）：将长 Prefill 切成小 chunk，与 Decode 交错执行：

```
轮次1：Prefill chunk (512 tokens)  + Decode batch
轮次2：Prefill chunk (512 tokens)  + Decode batch
...

优点：无需两套集群，TTFT / TBT 均有改善
缺点：不如完全分离的效果极致
```

---

## 八、推理服务框架全景

### 8.1 主流框架对比

| 框架 | 维护方 | 核心优势 | 适用场景 |
|---|---|---|---|
| **vLLM** | UC Berkeley（开源）| PagedAttention，生态丰富，API 兼容 OpenAI | 通用首选，研究友好 |
| **SGLang** | Stanford（开源）| RadixAttention（前缀缓存）, 结构化生成 | 高前缀重用，Agent 应用 |
| **TensorRT-LLM** | NVIDIA | H100 极致性能，FP8 原生，硬件深度优化 | 生产部署，NVIDIA GPU |
| **llama.cpp** | 社区 | CPU 推理，量化（GGUF），端侧部署 | 本地/边缘，无 GPU 场景 |
| **Ollama** | 社区 | 包装 llama.cpp，开箱即用 | 开发者本地测试 |
| **LMDeploy** | 上海AI Lab | TurboMind 引擎，国内生态 | 国内部署，与 OpenMMLab 集成 |

### 8.2 推理系统关键 Metrics

| 指标 | 含义 | 优化目标 |
|---|---|---|
| **TTFT** (Time To First Token) | 从请求到第一个 token 输出的延迟 | 越低越好（用户感知等待）|
| **TBT** (Time Between Tokens) | 相邻两个 token 之间的间隔 | 越稳定越好（流式体验）|
| **Throughput** (tokens/s) | 每秒生成的总 token 数 | 越高越好（成本效率）|
| **QPS** | 每秒处理请求数 | 越高越好 |
| **MFU** | 模型浮点利用率（实际/理论峰值）| 越高越好（GPU 效率）|
| **KV Cache 利用率** | 显存中有效 KV 占总分配比 | > 80% 为佳 |

---

## 九、长上下文推理优化

随着 context window 扩展到 128K-1M，推理面临新挑战：

### 9.1 注意力计算复杂度

标准 Attention：`O(n²)` 复杂度，128K context = 4× 16K 的计算量

**FlashAttention（v1/v2/v3）**：
- 分块（Tiling）计算，避免 O(n²) 的中间矩阵写入 HBM
- IO 复杂度：`O(n²/M)` 而非 `O(n²)`（M 是 SRAM 大小）
- H100 上 v3 实现接近硬件峰值（700+ TFLOPS FP8）

**稀疏注意力**：
- Sliding Window（Mistral 早期）：每个 token 只看最近 W 个 token
- **NSA（Native Sparse Attention，DeepSeek V3/R1）**：训练时就稀疏，非推理时 hack
- **MLA + 低秩 KV**：从架构上将 KV 投影到低维，从根本解决

### 9.2 位置编码外推

预训练 4K 上下文的模型直接推理 128K 会崩溃（RoPE 外推问题）：

| 方法 | 原理 | 效果 |
|---|---|---|
| **RoPE Scaling（线性缩放）**| 将位置索引按比例缩小 | 简单，有一定损失 |
| **YaRN** | NTK-aware 缩放 + 高频保留 | 更好的外推，无需微调 |
| **LongRoPE** | 分段非均匀缩放 | 支持 2M context |
| **长文本微调** | 用长文本继续训练 | 最稳定，但成本高 |

---

## 十、推理优化路线图总结

```
瓶颈识别：
  Prefill 慢？→ FlashAttention + Chunked Prefill + 更多 TP
  Decode 慢？→ 量化（FP8/INT4）+ 投机解码 + 更大 batch

显存不够？→ KV Cache 量化 + GQA + 减少 batch size

高并发低延迟？→ PagedAttention + Continuous Batching + PD 分离

长上下文？→ FlashAttention v3 + KV Cache 压缩/稀疏化

多请求共享前缀？→ Prefix Caching（SGLang RadixAttention）

优化优先级（ROI 从高到低）：
  1. 选对框架（vLLM/SGLang）+ 开启 PagedAttention：免费的 2-5× 吞吐提升
  2. FP8 量化（H100）或 AWQ INT4：2× 显存，接近 2× 速度
  3. KV Cache Prefix Caching：重复前缀场景 TTFT 大幅降低
  4. 投机解码：延迟敏感场景 1.5-3× 提速
  5. PD 分离：超高并发生产系统
```

---

## 参考资料

- [Efficient Memory Management for Large Language Model Serving with PagedAttention (vLLM, 2023)](https://arxiv.org/abs/2309.06180)
- [FlashAttention-3: Fast and Accurate Attention with Asynchrony and Low-precision (2024)](https://arxiv.org/abs/2407.08608)
- [AWQ: Activation-aware Weight Quantization for LLM Compression (2023)](https://arxiv.org/abs/2306.00978)
- [Fast Inference from Transformers via Speculative Decoding (2023)](https://arxiv.org/abs/2211.17192)
- [EAGLE-2: Faster Inference of Language Models with Dynamic Draft Trees (2024)](https://arxiv.org/abs/2406.16858)
- [DistServe: Disaggregating Prefill and Decoding for Goodput-Optimized LLM Serving (2024)](https://arxiv.org/abs/2401.09670)
- [SGLang: Efficient Execution of Structured Language Model Programs (2024)](https://arxiv.org/abs/2312.07104)
- [AI Infra: LLM 推理进展与调研分享（知乎，2026-03）](https://zhuanlan.zhihu.com/p/2018708746205967140)
- [LLM 推理性能优化路线图（lategege.com，2026-03）](https://www.lategege.com/p/llm-inference-performance-roadmap/)
