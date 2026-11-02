---
layout: reading
title: "大模型微调 · TPU/Mac/自研芯片后端 · 多模态检索 · 时间序列集成"
category: tech
tags: [Tech, 多源, 前沿]
date: 2026-11-02
---

# 📰 2026-11-02 · 每日技术速递

> 今日精选 7 篇深度技术文章，覆盖 大模型 RL 微调 · TPU/Apple Silicon/自研芯片后端 · 多模态检索编码 · 时间序列集成 · 浏览器端 GPU 算子。

---

## 1. 用 GRPO 把 350M 小模型的结构化输出合规率调高

**来源**：Hugging Face  
**链接**：https://huggingface.co/blog/grpo-with-trl-ifstruct  
**标签**：GRPO · RLHF · 结构化输出 · TRL · 小模型微调

本文给出一份公开、低成本的配方：用 TRL 库以 Group Relative Policy Optimization（GRPO）微调 LiquidAI 的 LFM2.5-350M，并在 IFStruct 结构化输出基准上评估。整个训练约需 500 条样本、100 步，足够在免费 Colab/Kaggle GPU 上跑完。结果显示，即使轻量微调也能把合规率从 22.6% 提升到 29.7%，让小模型在 schema 遵从这一关键工程能力上逼近远更大的模型。

**核心要点**：
- 使用 TRL 的 GRPO 做强化式微调，目标是让模型稳定输出合规、可解析的 JSON/YAML 等格式，而非泛化推理。
- 评估在 IFStruct 基准上做，覆盖 JSON/YAML、wrapper key 与 bare list 等多种格式与实体类型，并给出分格式细分。
- 全流程可在免费 GPU 上完成（微调）并在 MacBook 本地用 llama.cpp 起 OpenAI 兼容服务做评测，复现成本极低。

---

## 2. NeoMME：统一的原生多模态、多语言编码器

**来源**：Hugging Face  
**链接**：https://huggingface.co/blog/Hcompany/neomme  
**标签**：多模态编码 · 视觉文档检索 · 双向 Transformer · 离散扩散 · 量化

NeoMME 是一族 260M/800M 的多语言多模态编码器，不走「预训练视觉塔 + 因果语言模型」的老路：单一双向 Transformer 同时处理文本 token 与原始图像 patch，并用掩码离散扩散目标从零训练。在视觉文档检索（ColPali 式 page-image 方案）上，它一次前向即产出稠密与 late-interaction 两种向量，并借助分层 token 池化与非对称量化把 late-interaction 索引从每页约 1.5MB 压到 6KB（小 255 倍）而保留 95% 以上 nDCG@10。

**核心要点**：
- 单一 Transformer 编码文本与图像，无独立视觉塔/文本编码器/解码器，参数与算力开销更低，模态间共享计算路径。
- 图像按 32×32 patch 进入、保持长宽比的动态分辨率，16K 上下文、滑窗+全局交替注意力，并配 131k 多语言 BPE 词表。
- 260M 版在 L40S 上约 51 页/秒（约 ColModernVBERT 的 2 倍吞吐），已接入 Transformers，Apache 2.0 开源。

---

## 3. Hugging Face ❤️ TorchTPU：让 TPU 成为 PyTorch 一等公民

**来源**：Hugging Face  
**链接**：https://huggingface.co/blog/hf-torchtpu  
**标签**：TPU · PyTorch 后端 · torch.compile · 分布式训练 · 原生支持

TorchTPU 为 PyTorch 新增原生 TPU 后端，从此 TPU 只是又一个 device，不再需要把代码移植到 torch_xla 或改学 JAX。它内置为 TPU 调优的 Pallas kernel、分布式训练支持，以及 torch.compile 集成。Transformers、Diffusers、Kernels 均可原生跑在 TPU 上——从 Hub 选模型、发到 "tpu" 即可。

**核心要点**：
- 相比旧路径 torch_xla（惰性张量、图重编译、难调试），TorchTPU 让你写普通 PyTorch 代码即可，一键 `pip install torch-tpu` 装好。
- Transformers v5 以模型定义为唯一真源，加载时传 `device_map="tpu"` 即可；Diffusers 中 `pipe.to("tpu")` 即可用 eager 模式，部署再补一次 compile。
- 支持张量并行（`TensorParallelConfig`）把过大模型按芯片分片加载，实测 Cloud TPU v6e 上可跑 Qwen-Image-2.1、MiniMax-H3 等生成模型。

---

## 4. PyTorch 2.15 的 MPS 后端：Mac 上跑模型更快更省

**来源**：Hugging Face  
**链接**：https://huggingface.co/blog/mps-pytorch-2-15  
**标签**：Apple Silicon · MPS · FP8 · MoE 加速 · Metal kernel

PyTorch 2.15 大幅强化 MPS（Apple Silicon）后端：FP8 登陆 Mac、MoE 解码显著提速、大量算子改用专用 Metal kernel 替代 MPSGraph。对本地跑 LLM/Diffusers/ComfyUI 的用户，这是一次实打实的性能与内存升级。

**核心要点**：
- FP8（float8_e4m3fn）在 MPS 上可用，`F.scaled_mm` 直接做缩放矩阵乘，权重仅需 BF16 一半存储。
- MoE 解码比 2.14 快 5–6.5×：M3 Ultra 上 Qwen3.6-35B-A3B 从 9.4 提到 61.3 tokens/s，靠原生 grouped matmul、更少 GPU-CPU 同步与更快 top-k 路由。
- 视频生成内存大降：Wan2.2 的 tiled VAE 解码峰值从 270GiB 降到约 8GiB；并修复了 Qwen-Image-2.1 在 MPS 上 VAE 静默损坏的问题（PSNR 从 10.06 升到 29.95 dB）。

---

## 5. 在 AWS Trainium 上做系统级性能剖析（TorchNeuron 系列一）

**来源**：Hugging Face  
**链接**：https://huggingface.co/blog/amazon/profiling-torchneuron-part-1  
**标签**：AWS Trainium · TorchNeuron · torch.profiler · NKI · 性能剖析

本系列从零讲清「如何系统分析推理性能、定位时间花在哪」。以 AWS Trainium 与其原生 PyTorch 后端 TorchNeuron 为例，覆盖如何用 torch.profiler 抓取干净 trace、读 key_averages() 表找出吞掉 Neuron 设备时间的算子，以及如何用 Neuron Explorer 把一个算子从框架分发追到硬件执行。

**核心要点**：
- Neuron 后端走 PyTorch 的 PrivateUse1 扩展点接入，无需新增类似 cuda 的 dispatch key，TorchNeuron 即可用标准 PyTorch API（含 eager 与 torch.compile）。
- 通过 NeuronConfig 选择采集数据流（device/runtime）、指定 NeuronCore，并把设备时间关联回 key_averages() 表做 CPU 与设备时间对比。
- 以 RMSNorm 为例对比三种实现（朴素算子拼装 / 内置 `torch.rms_norm` 融合 kernel / `torch.compile`），脚本与复现步骤全部公开。

---

## 6. 用 IBM Granite 时间序列集成模型做「免训练」集成预测

**来源**：Hugging Face  
**链接**：https://huggingface.co/blog/ibm-research/granite-timeseries-ensemble-r1  
**标签**：时间序列 · 集成预测 · 概率预测 · Granite · 免训练

预训练时序模型能免训练直接预测，但「选哪个模型」本身就是难题。Granite Timeseries Ensemble 把四个互补的 Granite 架构合进一个可扩展接口，返回中位数与分位预测，无需重训、也无需拟合集成权重，相当于给时序预测一个「合理默认解」。

**核心要点**：
- 默认配方组合四种架构：PatchTST（Patched Transformer）、PatchTST-FM-r2（Patched Conformer）、FlowState（状态空间模型）、TTM-r3（Tiny Time Mixers MLP mixer）。
- 两种合并方式：推荐的 Probabilistic Ensemble（均匀概率池化，无拟合步骤）与自适应 IQR Ensemble，后者按区间调整权重。
- 输出点位预测 + 0.1~0.9 分位预测，可用分位表达预测区间（如 0.1/0.9 覆盖名义 80% 区间），模型与配方均宽松授权。

---

## 7. @huggingface/kernels：浏览器端 200+ WebGPU 算子库

**来源**：Hugging Face  
**链接**：https://huggingface.co/blog/webgpu-kernels  
**标签**：WebGPU · WGSL · 浏览器推理 · 算子库 · 众包基准

Hugging Face WebAI 团队发布浏览器端高性能推理的「地基层」：@huggingface/kernels 用于在 Hub 上加载并运行优化过的 WebGPU kernel，首发 207 个 kernel，覆盖矩阵乘、归一化、卷积、注意力、量化、数据布局变换等。每个 kernel 都是带接口、shader 模板、正确性用例与基准用例的完整版本化包。

**核心要点**：
- 207 个算子以独立仓库形式发布（webgpu-kernels 组织下），Apache-2.0；JS loader 负责从 Hub 下载、准备并运行。
- 每个 kernel 附带 manifest（算子契约）、正确性测试、基准/调优用例与参数化 WGSL 模板，把「shader」变成可审计、可复用的软件产物。
- 配套 Fleet 浏览器内基准工具，众包真实 GPU 上的正确性与性能证据，帮助发现异常慢/错的 case 并改进 kernel 变体。

---

## 📊 今日速览

| # | 标题摘要 | 来源 | 方向 |
|---|---------|------|------|
| 1 | 用 GRPO 提升 350M 小模型结构化输出合规率 | Hugging Face | 大模型微调/RL |
| 2 | NeoMME：统一的多模态多语言编码器 | Hugging Face | 多模态检索编码 |
| 3 | TorchTPU：TPU 原生 PyTorch 后端 | Hugging Face | 加速器后端/TPU |
| 4 | PyTorch 2.15 MPS 后端性能与内存升级 | Hugging Face | Apple Silicon/Metal |
| 5 | Trainium 系统级性能剖析（TorchNeuron） | Hugging Face | 自研芯片/性能剖析 |
| 6 | Granite 时间序列集成预测模型 | Hugging Face | 时间序列/集成 |
| 7 | 浏览器端 200+ WebGPU 算子库 | Hugging Face | 浏览器推理/算子 |

---

*自动生成 · 2026-11-02 · jeffinchen daily tech reading list*

