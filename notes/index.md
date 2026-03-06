---
layout: default
title: Notes
permalink: /notes/
---
<div class="home">
  <h1 class="page-title">Notes</h1>
  <p class="page-desc">学习笔记 · 技术整理</p>
  <div class="notes-list">
    <a class="note-card" href="/notes/llm-parallel-training/">
      <div class="note-icon">⚙️</div>
      <div class="note-body">
        <h2>大模型训练并行技术全景：DP / TP / PP / SP / ZeRO</h2>
        <p>系统梳理五类并行策略：数据并行、张量并行、流水线并行、序列并行与 ZeRO 显存优化。含切分原理、通信分析、3D 并行架构及框架对比。</p>
        <span class="tag">分布式训练</span>
        <span class="tag">张量并行</span>
        <span class="tag">流水线并行</span>
        <span class="tag">ZeRO</span>
        <span class="tag">Megatron</span>
      </div>
    </a>
    <a class="note-card" href="/notes/mla-deepseek/">
      <div class="note-icon">⚡</div>
      <div class="note-body">
        <h2>Multi-Head Latent Attention (MLA): DeepSeek 高效推理的秘密</h2>
        <p>深入解析 DeepSeek V2/V3 的核心创新：通过低秩 KV 压缩将缓存减少 93%，同时超越标准 MHA 性能。含完整 PyTorch 实现。</p>
        <span class="tag">MLA</span>
        <span class="tag">DeepSeek</span>
        <span class="tag">Attention</span>
        <span class="tag">KV Cache</span>
      </div>
    </a>
    <a class="note-card" href="/notes/moe-survey/">
      <div class="note-icon">📚</div>
      <div class="note-body">
        <h2>Mixture of Experts (MoE) 完整技术综述</h2>
        <p>从 1991 年理论提出到 2024 年 DeepSeek-V3 工程突破，全面覆盖 MoE 架构、路由机制、负载均衡、训练推理优化，含代码示例与模型对比。</p>
        <span class="tag">MoE</span>
        <span class="tag">Survey</span>
        <span class="tag">DeepSeek</span>
        <span class="tag">Mixtral</span>
        <span class="tag">稀疏模型</span>
      </div>
    </a>
    <a class="note-card" href="/notes/latentmoe/">
      <div class="note-icon">🧠</div>
      <div class="note-body">
        <h2>LatentMoE：低维潜空间专家路由架构</h2>
        <p>从 Dense → 标准 MoE → LatentMoE 的演进脉络，讲清 All-to-All 通信瓶颈及低维潜空间如何同时压缩开销并指数扩展专家多样性。</p>
        <span class="tag">MoE</span>
        <span class="tag">LLM架构</span>
        <span class="tag">NVIDIA</span>
        <span class="tag">分布式推理</span>
      </div>
    </a>
    <a class="note-card" href="/notes/pytorch/">
      <div class="note-icon">⚙️</div>
      <div class="note-body">
        <h2>PyTorch 学习资源汇总</h2>
        <p>面向系统开发者的 PyTorch 学习路径，侧重底层实现和系统架构，包含 Megatron-LM、MLIR 相关资源。</p>
        <span class="tag">PyTorch</span>
        <span class="tag">Systems</span>
        <span class="tag">Megatron-LM</span>
        <span class="tag">MLIR</span>
      </div>
    </a>
    <a class="note-card" href="/notes/leetcode-top100/">
      <div class="note-icon">🧩</div>
      <div class="note-body">
        <h2>LeetCode 高频100题总结</h2>
        <p>按题型分类整理，每题含完整题目描述、核心思路、复杂度分析与 Python 代码模板。</p>
        <span class="tag">LeetCode</span>
        <span class="tag">算法</span>
        <span class="tag">Python</span>
        <span class="tag">面试</span>
      </div>
    </a>
    <a class="note-card" href="/notes/algo-tricks/">
      <div class="note-icon">🛠️</div>
      <div class="note-body">
        <h2>算法解题技巧总结</h2>
        <p>双指针、前缀和、哈希表、单调栈、二分、DFS/BFS、动态规划、贪心、分治、图论等核心解题技巧，含模板代码与经典例题。</p>
        <span class="tag">算法</span>
        <span class="tag">技巧</span>
        <span class="tag">模板</span>
        <span class="tag">Python</span>
      </div>
    </a>
  </div>
</div>