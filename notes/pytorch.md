---
layout: note
title: PyTorch 学习资源汇总（面向系统开发者）
permalink: /notes/pytorch/
---
# PyTorch 学习资源汇总（面向系统开发者）

> 本文档整理了面向系统开发者的 PyTorch 学习资料，侧重底层实现和系统架构。

## 📚 目录

- [官方核心资源](#官方核心资源)
- [系统开发者学习路径](#系统开发者学习路径)
  - [第1阶段：基础使用（1-2周）](#第1阶段基础使用1-2周)
  - [第2阶段：深入原理（2-4周）](#第2阶段深入原理2-4周)
  - [第3阶段：系统层面（4-8周）](#第3阶段系统层面4-8周)
  - [第4阶段：高级主题](#第4阶段高级主题)
- [系统特性相关资料](#系统特性相关资料)
- [补充推荐资源](#补充推荐资源)

---

## 📚 官方核心资源

### PyTorch 官方文档
- **官方文档**: https://pytorch.org/docs/stable/index.html
- 特别关注：C++ API (LibTorch)、Autograd 机制、自定义算子

### PyTorch Internals
- **Edward Yang 的博客**: http://blog.ezyang.com/2019/05/pytorch-internals/
- Edward Yang（PyTorch 核心开发者）的博客，深入讲解内部实现

### PyTorch 源码
- **GitHub 仓库**: https://github.com/pytorch/pytorch
- 重点目录：
  - `aten/` - 底层张量库
  - `c10/` - 核心抽象层
  - `torch/csrc/` - Python C++ 绑定
  - `torch/csrc/autograd/` - 自动微分引擎

### 核心论文
- **"Automatic Differentiation in PyTorch"**
- **"PyTorch: An Imperative Style, High-Performance Deep Learning Library"**

### 技术博客
- **PyTorch 官方博客**: https://pytorch.org/blog/
- **Horace He 的性能优化博客**: https://horace.io/
  - GitHub: https://github.com/Horace-He

---

## 🛠️ 系统开发者学习路径

### 第1阶段：基础使用（1-2周）

#### ✅ Tensor 操作和 Autograd
- **Tensor 基础**: https://pytorch.org/tutorials/beginner/basics/tensorqs_tutorial.html
- **Autograd 机制**: https://pytorch.org/tutorials/beginner/basics/autogradqs_tutorial.html
- **Tensor 文档**: https://pytorch.org/docs/stable/tensors.html

#### ✅ 基本模型训练流程
- **完整训练流程**: https://pytorch.org/tutorials/beginner/basics/optimization_tutorial.html
- **60分钟入门**: https://pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html

#### ✅ 数据加载与预处理
- **Dataset & DataLoader**: https://pytorch.org/tutorials/beginner/basics/data_tutorial.html
- **自定义数据集**: https://pytorch.org/tutorials/beginner/data_loading_tutorial.html

---

### 第2阶段：深入原理（2-4周）

#### 🔍 Tensor 内存模型
- **Tensor Internals**: https://blog.ezyang.com/2019/05/pytorch-internals/
- **Storage 和 View**: https://pytorch.org/docs/stable/tensor_view.html
- **内存格式优化**: https://pytorch.org/tutorials/intermediate/memory_format_tutorial.html

#### 🔍 Autograd 实现机制
- **Autograd 引擎详解**: https://pytorch.org/blog/overview-of-pytorch-autograd-engine/
- **自定义 Autograd**: https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html
- **深入理解 Autograd**: https://pytorch.org/docs/stable/notes/autograd.html

#### 🔍 自定义算子（Python/C++）
- **C++ 扩展教程**: https://pytorch.org/tutorials/advanced/cpp_extension.html
- **自定义 Function**: https://pytorch.org/docs/stable/notes/extending.html
- **CUDA 扩展**: https://pytorch.org/tutorials/advanced/cpp_cuda_extension.html

#### 🔍 模型保存与加载格式
- **序列化语义**: https://pytorch.org/docs/stable/notes/serialization.html
- **保存和加载模型**: https://pytorch.org/tutorials/beginner/saving_loading_models.html

---

### 第3阶段：系统层面（4-8周）

#### ⚙️ TorchScript JIT 编译
- **TorchScript 介绍**: https://pytorch.org/docs/stable/jit.html
- **TorchScript 教程**: https://pytorch.org/tutorials/beginner/Intro_to_TorchScript_tutorial.html
- **性能分析**: https://pytorch.org/tutorials/beginner/profiler.html

#### ⚙️ 分布式训练架构
- **分布式概览**: https://pytorch.org/tutorials/beginner/dist_overview.html
- **DDP 教程**: https://pytorch.org/tutorials/intermediate/ddp_tutorial.html
- **FSDP (全分片数据并行)**: https://pytorch.org/tutorials/intermediate/FSDP_tutorial.html
- **RPC 框架**: https://pytorch.org/tutorials/intermediate/rpc_tutorial.html

#### ⚙️ CUDA 算子开发
- **CUDA 语义**: https://pytorch.org/docs/stable/notes/cuda.html
- **自定义 CUDA 算子**: https://pytorch.org/tutorials/advanced/cpp_cuda_extension.html
- **NVIDIA CUDA 编程指南**: https://docs.nvidia.com/cuda/cuda-c-programming-guide/

#### ⚙️ 性能优化技巧
- **Performance Tuning Guide**: https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html
- **PyTorch Profiler**: https://pytorch.org/tutorials/recipes/recipes_index.html#performance-and-profiling
- **Horace He 性能优化博客**: https://horace.io/

#### ⚙️ 源码阅读（核心模块）
- **PyTorch 源码**: https://github.com/pytorch/pytorch
- **源码导读 (Edward Yang)**: http://blog.ezyang.com/
- **ATen 库**: https://github.com/pytorch/pytorch/tree/main/aten

---

### 第4阶段：高级主题

#### 🚀 量化与剪枝
- **量化教程**: https://pytorch.org/tutorials/recipes/quantization.html
- **动态量化**: https://pytorch.org/tutorials/intermediate/dynamic_quantization_tutorial.html
- **剪枝教程**: https://pytorch.org/tutorials/intermediate/pruning_tutorial.html

#### 🚀 模型部署（ONNX、TorchServe）
- **ONNX 导出**: https://pytorch.org/tutorials/advanced/super_resolution_with_onnxruntime.html
- **TorchServe 官网**: https://pytorch.org/serve/
- **TorchServe GitHub**: https://github.com/pytorch/serve
- **Mobile 部署**: https://pytorch.org/mobile/home/

#### 🚀 编译优化（torch.compile）
- **torch.compile 介绍**: https://pytorch.org/tutorials/intermediate/torch_compile_tutorial.html
- **TorchDynamo**: https://pytorch.org/docs/stable/dynamo/index.html
- **TorchInductor**: https://dev-discuss.pytorch.org/t/torchinductor-a-pytorch-native-compiler-with-define-by-run-ir-and-symbolic-shapes/747

#### 🚀 自定义后端开发
- **后端扩展**: https://pytorch.org/tutorials/advanced/extend_dispatcher.html
- **Dispatcher 机制**: https://pytorch.org/tutorials/advanced/dispatcher.html

---

## 🔍 系统特性相关资料

### 内存管理
- **Caching Allocator**: https://pytorch.org/docs/stable/notes/cuda.html#memory-management
- **内存分析**: https://pytorch.org/blog/understanding-gpu-memory-1/
- 关键概念：引用计数、内存池

### 并行计算
- **CUDA Streams**: https://pytorch.org/docs/stable/notes/cuda.html#cuda-streams
- **多进程最佳实践**: https://pytorch.org/docs/stable/notes/multiprocessing.html
- 关键概念：多线程数据加载、异步执行

### 图优化
- **Graph Mode**: https://pytorch.org/docs/stable/jit.html
- **Operator Fusion**: https://pytorch.org/blog/optimizing-cuda-rnn-with-torchscript/
- 关键概念：算子融合、内存规划

### 类型系统
- **TorchScript 类型系统**: https://pytorch.org/docs/stable/jit_language_reference.html
- **静态类型推断**: https://pytorch.org/docs/stable/jit.html#type-annotations
- 关键概念：动态类型 vs 静态类型

### 算子调度
- **Dispatcher 设计文档**: http://blog.ezyang.com/2020/09/lets-talk-about-the-pytorch-dispatcher/
- **算子注册**: https://pytorch.org/tutorials/advanced/dispatcher.html
- 关键概念：多后端支持、动态分发

---

## 📚 补充推荐资源

### 🎓 CMU 深度学习系统课程（强烈推荐！）
- **课程主页**: https://dlsyscourse.org/
- **GitHub**: https://github.com/dlsyscourse/lecture-note
- 从零实现 PyTorch-like 框架，理解底层设计

### 📖 相关书籍/课程
- **《动手学深度学习》（李沐）**: https://zh.d2l.ai/
- **Stanford CS231n**: http://cs231n.stanford.edu/
- **Fast.ai**: https://www.fast.ai/

### 💬 开发者资源
- **PyTorch 贡献指南**: https://github.com/pytorch/pytorch/blob/main/CONTRIBUTING.md
- **PyTorch Dev Discussions**: https://dev-discuss.pytorch.org/
- **PyTorch Forums**: https://discuss.pytorch.org/

### 🔧 性能分析工具
- **PyTorch Profiler**
- **NVIDIA Nsight Systems**
- **TensorBoard**

---

## 📝 学习建议

### 实践项目

1. **自定义算子开发**
   - 使用 C++/CUDA 扩展 PyTorch
   - 实现高效的自定义层

2. **阅读经典模型实现**
   - torchvision/models 源码
   - 理解高效实现技巧

3. **分布式训练实验**
   - 从单机多卡到多机多卡
   - 理解通信开销和优化

### 调试技巧

- 使用小模型进行实验
- 打印 tensor shape 和设备分布
- 使用 `NCCL_DEBUG=INFO` 查看通信细节
- 利用 PyTorch Profiler 分析性能瓶颈

### 学习路径时间线

```
Week 1-2:   基础使用 → 熟悉 API 和基本流程
Week 3-6:   深入原理 → 理解 Tensor、Autograd 实现
Week 7-14:  系统层面 → 掌握分布式、JIT、性能优化
Week 15+:   高级主题 → 量化部署、编译优化、源码贡献
```

---

## 🎯 关键概念检查清单

学完每个阶段后，确保你理解这些概念：

### 第1阶段
- [ ] Tensor 的存储结构
- [ ] 自动微分的计算图
- [ ] DataLoader 的多进程机制

### 第2阶段
- [ ] Storage、View、Stride 的关系
- [ ] Autograd Function 的前向/反向传播
- [ ] C++ 扩展的编译和调用

### 第3阶段
- [ ] Tensor Parallelism vs Data Parallelism
- [ ] TorchScript 的 tracing 和 scripting
- [ ] CUDA kernel 的线程模型

### 第4阶段
- [ ] 量化的原理和精度损失
- [ ] ONNX 的算子映射
- [ ] torch.compile 的编译流程

---

## 📌 相关资源

### DP vs DDP

**DP (DataParallel)** - 单机多卡，已不推荐
- 单进程多线程
- 主 GPU 负载不均衡
- 受 Python GIL 限制

**DDP (DistributedDataParallel)** - 推荐使用
- 多进程并行
- 负载均衡
- 支持多机多卡
- 通信高效（AllReduce）

### Megatron-LM 学习资源

**官方资源**
- GitHub: https://github.com/NVIDIA/Megatron-LM
- 论文: "Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism"

**核心概念**
- Tensor Parallelism（张量并行）
- Pipeline Parallelism（流水线并行）
- Sequence Parallelism（序列并行）

**源码阅读路径**
```
megatron/
├── core/parallel_state.py        # 并行状态管理
├── core/tensor_parallel/         # 张量并行实现
├── core/pipeline_parallel/       # 流水线并行实现
├── model/transformer.py          # Transformer 实现
└── optimizer/optimizer.py        # 分布式优化器
```

**相关项目**
- Megatron-DeepSpeed: https://github.com/microsoft/Megatron-DeepSpeed
- DeepSpeed: https://github.com/microsoft/DeepSpeed
- ColossalAI: https://github.com/hpcaitech/ColossalAI

---

## 🌐 MLIR 相关资源

### 官方资源
- **官方文档**: https://mlir.llvm.org/
- **Toy Tutorial**: MLIR 经典入门教程
- **论文**: "MLIR: A Compiler Infrastructure for the End of Moore's Law"

### 学习路径
1. 理解编译器基础（前端、IR、后端）
2. 学习 LLVM IR 的基本概念
3. 阅读 MLIR 的设计理念和 Toy Tutorial
4. 动手实现简单的 Dialect
5. 研究实际项目中的应用（TensorFlow、PyTorch）

---

## 📝 更新日志

- 2024-01-XX: 初始版本，整理 PyTorch 系统开发学习资源
- 添加了 DP vs DDP 对比说明
- 添加了 Megatron-LM 学习资源
- 添加了 MLIR 相关资料

---

## 🤝 贡献

如果你发现了更好的学习资源，欢迎补充！

---

**Happy Learning! 🚀**