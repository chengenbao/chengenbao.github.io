---
layout: note
title: "DeepSpeed-MoE：端到端 MoE 训练与推理加速方案（ICML 2022）"
permalink: /notes/deepspeed-moe/
---

# DeepSpeed-MoE：端到端 MoE 训练与推理加速方案

> 论文：[DeepSpeed-MoE: Advancing Mixture-of-Experts Inference and Training to Power Next-Generation AI Scale](https://arxiv.org/abs/2201.05596)  
> 作者：Samyam Rajbhandari, Conglong Li, Zhewei Yao, Minjia Zhang 等（微软）  
> 发表：ICML 2022  
> 笔记整理：chengenbao · 2026-03

---

## 一、论文解决的核心问题

MoE（Mixture of Experts）架构通过稀疏激活大幅降低训练成本——与同等质量的 Dense 模型相比可节省 **5 倍**训练计算。但 MoE 在实际部署中面临三大挑战：

1. **应用范围局限**：此前 MoE 研究主要集中在 encoder-decoder 模型（如 Switch Transformer、GShard），在自回归 NLG 任务（如 GPT 系列）上的探索不足。
2. **参数效率低，内存需求大**：MoE 模型参数规模远超同质量 Dense 模型（如 Switch-Base 比 T5-large 多 10 倍参数），导致巨大的显存压力。
3. **推理性能差**：模型体量大到单 GPU 放不下，而多 GPU 推理方案（专家并行）的通信开销和内存带宽瓶颈尚未有效解决。

DeepSpeed-MoE 对此提出了**模型架构创新 + 模型压缩 + 推理系统优化**的端到端解决方案。

---

## 二、MoE 用于自回归 NLG：训练成本降低 5 倍

论文首次系统地将 MoE 应用到基于 Transformer 的自回归语言模型（GPT 风格），并给出了清晰的结论：

- **"350M + MoE-128"**：以 350M 参数的 Dense 模型为基座，配置 128 个专家，使用 Top-2 路由。
- **"1.3B + MoE-128"**：以 1.3B Dense 模型为基座，128 个专家，总参数约 52B。
- 在零样本评估中，**1.3B + MoE-128** 用 13B Dense 模型的训练成本达到了 **6.7B Dense 模型的质量**，实现约 5 倍训练成本节省。
- 在 LAMBADA、PIQA、BoolQ、RACEh、TriviaQA、WebQs 等下游任务上均验证了 MoE 架构的优势。

---

## 三、PR-MoE：金字塔残差 MoE 架构

PR-MoE（Pyramid-Residual MoE）是论文提出的核心架构创新，目标是**提高参数效率、减少模型规模**。

### 3.1 金字塔结构（Pyramid）

基于关键观察：**模型深层添加专家的效果优于浅层**。因此 PR-MoE 采用金字塔式专家分配——越靠后的层使用越多的专家。

例如在 128 GPU 的训练设置中，不同层的专家数量可以是 {32, 64, 128}，对应的数据并行度分别为 {4, 2, 1}。

### 3.2 残差连接（Residual）

传统 MoE 使用 Top-k 路由选择 k 个专家。PR-MoE 采用一种折中方案：

- **固定一个 expert**（类似残差路径，始终被激活）
- **动态选择另一个 expert**（通过门控机制路由）

这种设计在减少通信量的同时保持了足够的模型容量，相比标准 MoE **模型规模降低约 3 倍**。

### 3.3 混合并行训练策略

由于不同层的专家数量不同，PR-MoE 采用**专家并行（EP）与数据并行（DP）混合**的策略：

- 非专家模块：全量数据并行
- 专家模块：根据每层专家数量动态调整 EP/DP 比例

---

## 四、MoS：知识蒸馏进一步压缩

**Mixture of Students（MoS）** 在 PR-MoE 基础上引入知识蒸馏，用大型 MoE 模型（教师）训练更小的 MoE 模型（学生），进一步将模型规模降低至 **3.7 倍**。

结合 PR-MoE + MoS，DeepSpeed-MoE 在保持模型质量的同时大幅缩减了推理时需要加载的参数量。

---

## 五、推理系统优化

这是论文最具系统工程价值的部分。对于 1.3B + MoE-128 模型（总参数约 52B），DeepSpeed-MoE 推理系统实现了**比现有方案 7.3 倍的延迟和成本改善**。

### 5.1 灵活的混合并行策略

推理时同时使用三种并行：

| 并行方式 | 作用对象 | 目的 |
|---------|---------|------|
| 张量并行（TP） | 非专家层 + 专家内部 | 降低单 GPU 显存压力 |
| 专家并行（EP） | 专家层 | 将不同专家分布到不同 GPU |
| 数据并行（DP） | 全局 | 提高吞吐 |

例如：16 GPU 上，非专家参数用 4 路 DP × 4 路 TP，专家参数用 8 路 EP × 2 路 TP。

### 5.2 通信优化

专家并行的核心通信操作是 **All-to-All**（每个 GPU 将 token 发送给目标专家所在的 GPU）。DeepSpeed-MoE 做了三层优化：

**（1）替换通信库**：使用微软 SCCL 替代 NCCL，减少通信开销。

**（2）分层 All-to-All（Hierarchical All-to-All）**：
- 先在节点内做局部 All-to-All（走 NVLink，带宽高）
- 再在节点间做全局 All-to-All（走 InfiniBand）
- 通信复杂度从 O(p) 降至 O(G + p/G)，其中 p 为总 GPU 数，G 为单节点 GPU 数

**（3）Tensor-Expert 协同并行**：
- 非专家层使用 TP 后，同一 TP 组内的 GPU 输入完全一致
- 后续 MoE 层利用这一特性，TP 组内 GPU 之间无需重复 All-to-All
- 先在不同 TP 组的对应 GPU 间做 All-to-All，再组内 All-Gather
- 通信开销从 O(p) 降至 O(L) + O(p/L)

### 5.3 Kernel 优化

针对 MoE 的 Gate 模块和稀疏计算：

- **稠密表示替代稀疏张量**：用 Mapping Table 记录 token-expert 对应关系，避免处理稀疏 one-hot 向量
- **算子融合**：将 Gate 内的 mask 生成、Top-k、scatter 等操作融合为单个 CUDA kernel
- **效果**：Gate kernel 延迟降低 **6 倍**

---

## 六、核心实验结果

### 训练效率

| 模型 | 训练成本（相对） | 模型质量（零样本） |
|------|----------------|-------------------|
| 6.7B Dense | 1× | 基准 |
| 1.3B + MoE-128 | ~0.2×（5倍节省） | ≈ 6.7B Dense |
| 350M + MoE-128 | 极低 | 与 1.3B Dense 可比 |

### 推理效率

| 对比维度 | DeepSpeed-MoE vs 现有 MoE 推理 | vs 同质量 Dense 模型 |
|---------|------|------|
| 延迟改善 | **7.3×** | **4.5×** |
| 成本降低 | — | **9×** |

### 模型压缩

| 技术 | 模型规模降低 |
|------|------------|
| PR-MoE | ~3× |
| PR-MoE + MoS（蒸馏） | **3.7×** |

---

## 七、关键设计思想总结

1. **MoE 不止于训练**：论文的核心贡献在于证明 MoE 不仅能降低训练成本，通过系统级优化还能在推理端获得显著收益，打破了"MoE 只适合训练"的认知。

2. **非均匀专家分配**：PR-MoE 的金字塔结构揭示了一个重要直觉——深层需要更多专家来处理更高层次的语义特征，浅层则不需要。

3. **压缩与蒸馏协同**：架构压缩（PR-MoE）+ 知识蒸馏（MoS）的组合策略，比单独使用任一方法效果更好。

4. **通信是 MoE 推理的关键瓶颈**：分层 All-to-All 和 Tensor-Expert 协同并行两项优化，将通信开销从 O(p) 降至亚线性，是推理加速的主要来源。

5. **端到端思维**：不是孤立地优化某一环节，而是从模型架构、压缩策略到推理系统全链路协同设计。

---

## 八、与后续工作的关联

DeepSpeed-MoE 是 MoE 推理优化的奠基性工作，其思想深刻影响了后续发展：

- **DeepSeek-V2/V3**：采用了更极致的 MoE 架构（256 个 routed experts + 1 shared expert），在路由机制和负载均衡上做了大量创新。
- **Mixtral 8×7B**：Meta 的开源 MoE 模型，验证了 MoE 在开源社区的可行性。
- **DeepEP**：DeepSeek 开源的专家并行通信库，可视为 DeepSpeed-MoE 通信优化思路的进一步深化和专业化。
- **Switch Transformer / ST-MoE**：Google 在路由策略和训练稳定性方面的持续探索。

---

## 参考

- 论文原文：[arXiv:2201.05596](https://arxiv.org/abs/2201.05596)
- DeepSpeed 官方博客：[deepspeed.ai/2022/01/14/deepspeed-moe](https://www.deepspeed.ai/2022/01/14/deepspeed-moe.html)
- DeepSpeed GitHub：[microsoft/DeepSpeed](https://github.com/microsoft/DeepSpeed)
