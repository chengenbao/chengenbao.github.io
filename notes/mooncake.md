---
layout: note
title: "Mooncake：以 KVCache 为中心的 LLM 分离式推理架构"
permalink: /notes/mooncake/
---

# Mooncake：以 KVCache 为中心的 LLM 分离式推理架构

> 来源：月之暗面（Moonshot AI）+ 清华大学  
> 论文一：[arXiv 2407.00079](https://arxiv.org/abs/2407.00079)（2024-06，FAS 2025 Best Paper）  
> 论文二：[arXiv 2604.15039](https://arxiv.org/abs/2604.15039)（2026-04，PrfaaS）  
> 开源：[github.com/kvcache-ai/Mooncake](https://github.com/kvcache-ai/Mooncake)  
> 笔记整理：chengenbao · 2026-04

---

## 一、核心洞见：KVCache 调度才是核心

传统 LLM 推理优化聚焦于"GPU 计算调度"，Mooncake 提出了一个根本性的视角转变：

> **LLM 服务的核心瓶颈不是计算，而是 KVCache 的存储、传输与复用调度。**

这个洞见驱动了整个架构设计：把集群中所有异构存储资源（GPU HBM、CPU DRAM、NVMe SSD）统一管理为一个分层 KVCache 池，围绕 KV 的流动来调度计算。

---

## 二、问题定义：双约束优化

Kimi 作为 MaaS（Model as a Service）服务商，面对的是一个带约束的优化问题：

```
最大化：有效吞吐量（直接影响营收）

约束 1：TTFT（Time To First Token）≤ SLO_TTFT
约束 2：TBT（Time Between Tokens）   ≤ SLO_TBT
约束 3：过载场景下需尽早拒绝请求，避免浪费已完成的 Prefill 计算
```

**已有系统（如 vLLM）的局限**：
- 假设资源充足，所有请求都会被处理
- 没有解决 **SLO 约束下的 KVCache 感知调度**
- 没有处理**过载场景**（GPU 供给紧张时，MaaS 普遍面临高峰期过载）

---

## 三、论文一：Mooncake 架构（2024）

### 3.1 整体架构

```
                ┌──────────────────────────────────────┐
                │       Conductor（全局 KVCache 调度器）  │
                │  • 为每个请求选择最优 (Prefill, Decode) 对  │
                │  • 预测 KVCache 热度，执行复制/换出         │
                │  • 过载预测 + 早期拒绝策略                │
                └────────┬─────────────────┬─────────────┘
                         │                 │
         ┌───────────────▼──────┐  ┌───────▼──────────────┐
         │   Prefill Cluster    │  │   Decode Cluster      │
         │  Compute-intensive   │  │  Memory bandwidth-    │
         │  大 batch，追求高 MFU  │  │  intensive，追求低 TBT │
         └───────────────┬──────┘  └───────▲──────────────┘
                         │  KVCache 流式传输  │
                         └──────────────────┘
         ┌────────────────────────────────────────────────┐
         │        分层 KVCache 存储池（全集群共享）            │
         │  GPU HBM（热）→ CPU DRAM（温）→ NVMe SSD（冷）   │
         └────────────────────────────────────────────────┘
```

### 3.2 请求调度流程

每个请求到达后，Conductor 执行三步：

```
Step 1：KVCache 复用
  从全局 KVCache 池找到可复用的 KV blocks（匹配 prefix）
  将这些 KV 迁移/预热到目标 Prefill 节点
  → 减少 Prefill 重复计算

Step 2：流式 Prefill + KV 传输
  Prefill 按 chunk/layer 执行
  每完成一部分，立即将产出的 KVCache 流式传输到 Decode 节点
  → 不等待 Prefill 全部完成，减少 Decode 等待时间（降低 TTFT）

Step 3：Decode 节点接收并推理
  Decode 节点将收到的 KV 加入 continuous batching
  开始 token-by-token 生成
```

### 3.3 分层 KVCache 存储

充分利用集群中被低估的非 GPU 资源：

| 存储层 | 介质 | 特点 | 存放内容 |
|---|---|---|---|
| L1 | GPU HBM | 最快，最贵 | 活跃请求的 KV |
| L2 | CPU DRAM | 快，低成本 | 近期请求 KV，可快速恢复 |
| L3 | NVMe SSD | 慢，超低成本 | 历史请求 KV，长期保留 |

**调度策略**：
- **热块复制**：高频访问的 KV blocks 在多个节点保留副本，防止拉取时网络拥塞
- **冷块换出**：低频 KV 逐级下沉到 DRAM → SSD，释放 HBM 空间
- **预取**：Conductor 预测未来访问，提前将 KV 从低层提升到高层

### 3.4 过载场景的早期拒绝（Early Rejection）

**核心问题**：如果让请求完成 Prefill 后才发现 Decode 节点无空位，则浪费了 Prefill 计算资源。

**解决方案**：
1. **输出长度预测**：预测每个请求的生成 token 数，估算 Decode 资源占用
2. **短期负载预测**：预测未来若干秒内 Decode 节点的可用 slot
3. **早期拒绝**：如果预测 Prefill 完成时 Decode 无空位 → 立即返回 429，不启动 Prefill

**工程陷阱**：朴素早期拒绝会引发**负载震荡**（oscillation）：
```
过载 → 大量拒绝 → 负载骤降 → 大量接受 → 重新过载 → ...
```
解决方法：预测窗口平滑 + 滞后控制（类似 TCP 拥塞控制的思路）。

### 3.5 实验结果

| 场景 | 指标 | 提升 |
|---|---|---|
| 长上下文模拟场景（最优）| 吞吐量 | **+525%** vs baseline |
| Kimi 真实生产负载 | 处理请求数 | **+75%** |

---

## 四、Transfer Engine：零拷贝传输基础设施

Mooncake 开源的核心组件，是整个 KVCache 流动的传输层。

### 4.1 两大核心抽象

**Segment（段）**：可被远端读写的地址空间
- RAM Segment：DRAM 或 GPU VRAM，非持久化
- NVMeof Segment：NVMe SSD 上的持久化存储

**BatchTransfer（批量传输）**：异步传输操作，相当于更灵活的 AllScatter/AllGather
- 支持非连续地址空间的批量 Read/Write
- 全异步，不阻塞调用线程

### 4.2 支持的传输后端

| 后端 | 传输路径 | 特点 |
|---|---|---|
| **RdmaTransport** | 跨节点 DRAM↔DRAM / VRAM（GPUDirect RDMA）| 主力，高带宽低延迟 |
| **NvlinkTransport** | 节点内 GPU↔GPU | 零拷贝，最快 |
| **NVMeoFTransport** | NVMe↔DRAM/VRAM（cuFile）| GPU 直接读 SSD |
| **TcpTransport** | 跨节点 DRAM↔DRAM | 兜底方案 |
| **EfaTransport** | AWS EFA | 云场景 |
| **HipTransport** | ROCm AMD GPU | 国产/AMD 适配 |

**关键设计**：多网卡 Pooling + 自动重试 + 自适应路由（Transfer Engine 根据内存类型自动选择最优后端）。

### 4.3 生态集成（截至 2026-04）

| 框架 | 集成内容 |
|---|---|
| **vLLM v1** | KV Connector，PD 分离标准后端 |
| **TensorRT-LLM** | KVCache Transfer |
| **SGLang** | PD 分离 + EPD（多模态 Encode 分离）|
| **LMDeploy** | PD 分离后端 |
| **PyTorch Ecosystem** | 正式加入（2026-02）|
| **TorchSpec** | 推理-训练解耦中的 hidden states 传输 |
| FlexKV（腾讯+NVIDIA）| 分布式 KVCache 复用 |

Transfer Engine 已成为业界 **PD 分离的事实标准传输层**。

---

## 五、论文二：PrfaaS — 跨数据中心 Prefill（2026-04）

**《Prefill-as-a-Service: KVCache of Next-Generation Models Could Go Cross-Datacenter》**  
arXiv:2604.15039，发布于 2026-04-17

### 5.1 问题：传统 PD 分离的网络边界

传统 PD 分离被**局限在单一高带宽网络域**（同机房 InfiniBand）内：

```
Dense Attention 模型（Llama 3 等）：
  每次 Prefill 产生巨量 KV traffic
  → 跨数据中心的以太网带宽根本传不动
  → Prefill 和 Decode 必须共在一个 IB 高带宽域

问题：
  - 无法利用其他数据中心的闲置算力
  - 异构加速器（便宜卡）无法参与
  - Prefill/Decode 无法独立扩缩容
```

**转机**：Hybrid Attention 架构（MLA、Mamba 混合等）**大幅压缩 KV 体积**，使跨数据中心传输从"不可能"变为"可行"。

### 5.2 PrfaaS 架构

```
 其他数据中心（廉价/异构加速器）     本地主数据中心
┌─────────────────────────┐       ┌──────────────────────────┐
│   Prefill Cluster        │       │   本地 PD Cluster         │
│   compute-dense          │  KV   │   ┌──────────┐           │
│   专门处理长上下文 Prefill │──────▶│   │  Decode  │           │
│   可用廉价/异构卡         │  传输  │   └──────────┘           │
│                          │  以太网│   本地也有部分 Prefill     │
└─────────────────────────┘       └──────────────────────────┘
```

**核心思路：选择性卸载（Selective Offloading）**

不是所有请求都发到外部 Prefill，而是根据请求特征智能决策：

```
短请求（输入 < 阈值）   → 本地 Prefill（延迟更低，带宽开销小不值得外包）
长上下文请求            → 卸载到外部 Prefill 集群（算力密集，外部更高效）
```

### 5.3 四大系统设计要点

| 要点 | 内容 |
|---|---|
| **模型侧 KV 效率** | 依赖 Hybrid Attention（MLA/Mamba）减少 KV 体积，这是前提条件 |
| **选择性卸载** | 根据请求长度+当前带宽状态，智能决策本地还是外包 |
| **带宽感知调度** | 实时监测跨数据中心带宽，动态调整卸载比例，避免拥塞 |
| **Cache-aware 请求路由** | 相同前缀的请求路由到同一 Prefill 节点，最大化 KV 复用 |

### 5.4 实验结果

案例：内部 1T 参数 Hybrid MoE 模型

| 基线 | vs PrfaaS 吞吐提升 |
|---|---|
| 同构 PD（纯主数据中心）| **+54%** |
| 朴素异构（全部卸载，无智能调度）| **+32%** |

跨数据中心带宽消耗：仅适度（commodity Ethernet，非 IB）。

---

## 六、与相关工作横向对比

| 系统 | PD 分离 | 多层 KV 存储 | 全局 KV 复用 | 过载早期拒绝 | 跨数据中心 |
|---|---|---|---|---|---|
| **Mooncake** | ✅ 核心 | ✅ HBM/DRAM/SSD | ✅ 全局调度 | ✅ | ✅（PrfaaS）|
| vLLM | ✅ v0.6+ | ❌ | 局部前缀 | ❌ | ❌ |
| SGLang | ✅ 可选 | ❌ | RadixAttention 前缀 | ❌ | ❌ |
| DistServe | ✅ | ❌ | ❌ | ❌ | ❌ |
| TensorRT-LLM | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## 七、核心结论与工程启示

**理论层面**：
1. KVCache 调度是 LLM 服务的核心问题，不是 GPU 计算本身
2. Hybrid Attention（KV 体积压缩）不仅是模型优化，更是**系统架构的解锁器**——使跨数据中心部署成为可能

**工程层面**：
1. **Transfer Engine** 是最有价值的可复用组件，零拷贝 RDMA + 多后端自适应，可直接集成到 vLLM/SGLang
2. **选择性卸载 + 带宽感知调度 + Cache-aware routing** 是 PD 分离系统设计的三个核心技巧
3. **过载早期拒绝** 在生产系统中必须处理，但要注意震荡风险，需要平滑控制
4. **分层 KVCache**（DRAM/SSD）在长上下文场景下收益显著，是低成本扩容 KV 容量的关键路径

---

## 参考资料

- [Mooncake: A KVCache-centric Disaggregated Architecture for LLM Serving (arXiv 2407.00079)](https://arxiv.org/abs/2407.00079)
- [Prefill-as-a-Service: KVCache of Next-Generation Models Could Go Cross-Datacenter (arXiv 2604.15039)](https://arxiv.org/abs/2604.15039)
- [Mooncake GitHub (kvcache-ai/Mooncake)](https://github.com/kvcache-ai/Mooncake)
- [Mooncake Transfer Engine 文档](https://kvcache-ai.github.io/Mooncake/design/transfer-engine/index.html)
- [vLLM Mooncake Connector 集成文档](https://docs.vllm.ai/en/latest/features/mooncake_connector_usage/)
- [Mooncake 加入 PyTorch 生态 (2026-02)](https://pytorch.org/blog/mooncake-joins-pytorch-ecosystem/)
