---
layout: reading
title: "技术速递 2026-03-09：推理计算效率 × FlashAttention-4 × LLM内省机制"
category: tech
tags: [Tech, arXiv, 推理优化, 注意力内核, 推测解码, LLM评测]
date: 2026-03-09
---

# 📰 2026-03-09 · 每日技术速递

> 今日聚焦：**推理计算效率** × **注意力硬件优化** × **LLM 内省机制**  
> 来源：arXiv cs.LG / cs.CL（2026-03-05 提交批次）

---

## 1. Reasoning Theater：从激活探测解耦 CoT 与模型真实置信度

**arXiv:2603.05488** | cs.CL · cs.LG

大模型 Chain-of-Thought 中存在"表演性推理"——模型已对答案高度自信，却仍继续生成冗余推理 token。研究者对 DeepSeek-R1 671B 和 GPT-OSS 120B 进行激活探测（activation probing）、强制提前终止和 CoT 监控三方对比：

- 在简单 MMLU 回忆题中，正确答案可从激活值**远早于**CoT 结尾解码
- 在困难 GPQA-Diamond 多跳推理题中，拐点（回溯/aha 时刻）与探针检测到的置信跃变高度重合
- 基于探针的提前退出策略：MMLU token 减少 **80%**，GPQA-Diamond 减少 **30%**，准确率基本持平

🔗 [arXiv:2603.05488](https://arxiv.org/abs/2603.05488)

---

## 2. FlashAttention-4：为 Blackwell GPU 重新设计注意力内核

**arXiv:2603.05451** | cs.CL

Blackwell 架构张量核心算力翻倍，但共享内存带宽与指数单元扩展较慢，形成新瓶颈。FlashAttention-4 方案：
- 全异步 MMA 操作与更大 tile 尺寸
- 软件模拟指数运算（conditional softmax rescaling）
- CuTe-DSL 实现，编译速度比 C++ 模板快 20-30×

**性能：** B200 BF16 达 **1613 TFLOPs/s（71%）**，比 cuDNN 1.3×，比 Triton 2.7×。

🔗 [arXiv:2603.05451](https://arxiv.org/abs/2603.05451)

---

## 3. OPSDC：基于在策略自蒸馏的推理压缩

**arXiv:2603.05433** | cs.LG

无需 ground-truth 答案、无 token 预算，仅通过"be concise"指令的 per-token 反向 KL 自蒸馏：

- MATH-500：token 减少 **57-59%**，准确率提升 **9-16 pp**
- AIME 2024（14B）：+10 分，压缩 41%

🔗 [arXiv:2603.05433](https://arxiv.org/abs/2603.05433)

---

## 4. 推测解码词表裁剪

**arXiv:2603.05210** | cs.CL · cs.LG

将草稿词表选择建模为约束优化（TPE 探索 Pareto 前沿），词表压缩 97%，领域任务延迟降低 **16%**，吞吐提升 **20%**。

🔗 [arXiv:2603.05210](https://arxiv.org/abs/2603.05210)

---

## 5. LLM 裁判可靠性压力测试

**arXiv:2603.05399** | cs.AI

Judge Reliability Harness 开源库：4 judge × 4 benchmark 测试发现，没有任何裁判在所有场景均可靠——格式变化、改写、标签翻转均导致一致性下降。

🔗 [arXiv:2603.05399](https://arxiv.org/abs/2603.05399)

---

## 6. 无检索事实核查：INTRA

**arXiv:2603.05471** | cs.CL · cs.AI

INTRA 方法利用 LLM 内部表示交互特征进行事实核查，跨 9 数据集达 SOTA，强泛化至长尾知识、多语言、长文本场景。

🔗 [arXiv:2603.05471](https://arxiv.org/abs/2603.05471)

---

*覆盖方向：推理优化·注意力内核·LLM评测·推测解码·可解释性*
