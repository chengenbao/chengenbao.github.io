---
layout: post
title: "技术速递 2026-02-27：torchforge RL框架、MoE训练优化、CUDA Kernel自动生成"
categories: ReadingList
tags: [ReadingList, Tech, LLM, PyTorch, MoE, CUDA, 推理优化]
---

> 每日精选：大模型 / PyTorch 框架 / 推理技术  
> 来源：pytorch.org/blog · huggingface.co/blog

---

## 1. Supercharging LLMs: Scalable RL with torchforge and Weaver

**来源：** [pytorch.org/blog](https://pytorch.org/blog/supercharging-llms-scalable-rl-with-torchforge-and-weaver/) | 2026-01-09

Meta PyTorch 团队开源 **torchforge**，PyTorch-native 的大规模 RL post-training 框架，支持单卡到 512+ GPU 集群无缝扩展。

**技术要点：**
- Forge + Weaver + Monarch 三件套：RL 原语 + verifier reward 信号 + 分布式容错协调
- 支持同步 PPO ↔ 全异步 off-policy 无缝切换
- TorchStore：RDMA 加速的 DTensor-native 权重同步，消除 training/generation lockstep
- 512 GPU 集群跑 GRPO，Qwen3-8B/32B 在 Math/GPQA/MMLU Pro 上有效提升

**[→ 阅读原文](https://pytorch.org/blog/supercharging-llms-scalable-rl-with-torchforge-and-weaver/)**

---

## 2. Efficient MoE Pre-training at Scale on 1K AMD GPUs with TorchTitan

**来源：** [pytorch.org/blog](https://pytorch.org/blog/efficient-moe-pre-training-at-scale-with-torchtitan/) | 2025-12-01

AMD + Meta 联手在 1024 块 MI325X GPU 上训练 DeepSeek-V3-671B，实现接近线性扩展。

**技术要点：**
- DeepSeek-V3-671B 获得 **2.77×** 速度提升（kernel 层优化）
- **96% scaling efficiency**：128 → 1024 GPU
- TorchTitan 单 TOML 文件配置 pipeline/tensor/data/expert 并行度
- AMD Primus-Turbo FP8 内核 drop-in，无需修改训练代码

**[→ 阅读原文](https://pytorch.org/blog/efficient-moe-pre-training-at-scale-with-torchtitan/)**

---

## 3. Mixture of Experts (MoEs) in Transformers

**来源：** [huggingface.co/blog](https://huggingface.co/blog/moe-transformers) | 2026-02-26

HuggingFace 深度解析 MoE 在 Transformers 库中的工程实现，从原理到代码集成全路径。

**技术要点：**
- 核心公式：模型容量 = 总参数，推理速度 = 活跃参数
- gpt-oss-20b：21B 总参数，每 token 激活 3.6B，M3 Ultra 上 ~115 tok/s
- Expert routing 挑战：load balancing、dead expert、token dropping
- 覆盖 DeepSeek-V3、Llama 4-Scout、Qwen3 等主流 MoE 架构

**[→ 阅读原文](https://huggingface.co/blog/moe-transformers)**

---

## 4. Custom CUDA Kernels for All: Claude and Codex Write Production Kernels

**来源：** [huggingface.co/blog](https://huggingface.co/blog/custom-cuda-kernels-agent-skills) | 2026-02-13

HuggingFace 构建 agent skill，让 Claude/Codex 自动为 diffusers/transformers 编写 production CUDA kernel。

**技术要点：**
- Agent skill 封装 H100/A100/T4 架构差异、shared memory vs register 选择等领域知识
- 生成 kernel 含完整 PyTorch binding，可被 `torch.compile` 识别
- 结合 Kernel Hub 实现 one-call kernel 加载

**[→ 阅读原文](https://huggingface.co/blog/custom-cuda-kernels-agent-skills)**
