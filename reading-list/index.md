---
layout: default
title: "Reading List"
permalink: /reading-list/
---

<div class="reading-list">
  <div class="header">
    <h1>Reading List</h1>
    <p>每日精选 · 技术 & 财经</p>
    <div class="filters">
      <button class="filter-btn active" data-filter="all">全部 (4)</button>
      <button class="filter-btn" data-filter="tech">技术速递 (2)</button>
      <button class="filter-btn" data-filter="finance">财经精选 (2)</button>
    </div>
  </div>

  <div class="articles">
    <article class="article tech">
      <div class="article-header">
        <span class="icon">📰</span>
        <div class="titles">
          <h2 class="main-title"><a href="/reading-list/2026-02-28-tech/">技术速递（2026-02-28）</a></h2>
          <p class="sub-title">TorchForge RL 后训练 / AMD 1K GPU MoE / HF Transformers v5 MoE / CUDA Kernel Agent</p>
        </div>
        <time>2026-02-28</time>
      </div>
      <div class="meta">
        <span class="source">pytorch.org/blog · huggingface.co/blog</span>
        <span class="tags">#大模型 #PyTorch #推理技术</span>
      </div>
      <p class="summary">TorchForge + Weaver LLM大规模RL后训练，AMD MI325X + TorchTitan 1024 GPU MoE预训练，HF Transformers v5 MoE加载优化，AI Agent自动生成CUDA Kernel</p>
    </article>

    <article class="article tech">
      <div class="article-header">
        <span class="icon">📰</span>
        <div class="titles">
          <h2 class="main-title"><a href="/reading-list/2026-02-27-tech/">技术速递（2026-02-27）</a></h2>
          <p class="sub-title">torchforge RL框架、MoE训练优化、CUDA Kernel自动生成</p>
        </div>
        <time>2026-02-27</time>
      </div>
      <div class="meta">
        <span class="source">pytorch.org/blog · huggingface.co/blog</span>
        <span class="tags">#大模型 #PyTorch #推理技术</span>
      </div>
      <p class="summary">PyTorch生态技术进展：RL框架优化、MoE训练性能提升、CUDA Kernel自动生成工具</p>
    </article>

    <article class="article finance">
      <div class="article-header">
        <span class="icon">💰</span>
        <div class="titles">
          <h2 class="main-title"><a href="/reading-list/2026-02-28-finance/">财经精选（2026-02-28）</a></h2>
          <p class="sub-title">AI 基建热潮推动全球贸易增长，远端国债利率创50年新高 + 稳定币监管框架对比</p>
        </div>
        <time>2026-02-28</time>
      </div>
      <div class="meta">
        <span class="source"><a href="https://www.federalreserve.gov/econres/notes/" target="_blank">Federal Reserve Economic Data</a></span>
        <span class="tags">#宏观经济 #AI贸易 #国债市场 #金融科技 #稳定币</span>
      </div>
      <p class="summary"><strong>核心发现一：</strong>美联储研究显示，2025年上半年AI基础设施建设热潮推动全球AI相关贸易额增长65%，达到2720亿美元，占全球贸易增量的50%。同时，长期国债远期利率出现50年来最大涨幅，反映市场对供给冲击风险和债务可持续性的重新定价。<br><br><strong>核心发现二：</strong>基于《GENIUS Act》框架，对比分析19世纪美国私人银行券与现代稳定币监管挑战。研究发现历史银行券因缺乏统一标准导致价值波动，当前稳定币在透明度、储备管理方面面临类似挑战，建议建立分级监管体系并加强国际合作。</p>
    </article>

    <article class="article finance">
      <div class="article-header">
        <span class="icon">💰</span>
        <div class="titles">
          <h2 class="main-title"><a href="/reading-list/2026-02-27-finance/">财经精选（2026-02-27）</a></h2>
          <p class="sub-title">AI贸易影响、美债长端利率、Stablecoin历史</p>
        </div>
        <time>2026-02-27</time>
      </div>
      <div class="meta">
        <span class="source">federalreserve.gov/econres</span>
        <span class="tags">#宏观经济 #金融科技 #资产配置</span>
      </div>
      <p class="summary">AI贸易对全球经济影响分析，美债长端利率走势，稳定币监管框架对比研究</p>
    </article>
  </div>
</div>

<style>
.reading-list {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.header h1 {
  font-size: 1.8em;
  color: #333;
  margin: 0 0 5px 0;
  font-weight: 600;
}

.header p {
  color: #666;
  margin: 0 0 20px 0;
  font-size: 14px;
}

.filters {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.filter-btn {
  padding: 6px 16px;
  border: 1px solid #ddd;
  background: white;
  color: #666;
  border-radius: 15px;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.2s;
}

.filter-btn:hover {
  border-color: #999;
}

.filter-btn.active {
  background: #333;
  color: white;
  border-color: #333;
}

.articles {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

.article {
  padding: 20px;
  border: 1px solid #eee;
  border-radius: 8px;
  background: white;
}

.article.tech {
  border-left: 1px solid #eee;
}

.article.finance {
  border-left: 1px solid #eee;
}

.article-header {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  margin-bottom: 10px;
}

.icon {
  font-size: 18px;
  margin-top: 4px;
}

.titles {
  flex: 1;
}

.main-title {
  margin: 0 0 5px 0;
  font-size: 18px;
  font-weight: 600;
  line-height: 1.3;
}

.main-title a {
  color: #333;
  text-decoration: none;
}

.main-title a:hover {
  color: #007cba;
}

.sub-title {
  margin: 0;
  font-size: 14px;
  color: #888;
  line-height: 1.4;
}

time {
  color: #999;
  font-size: 12px;
  white-space: nowrap;
  margin-top: 2px;
}

.meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
  font-size: 12px;
}

.source {
  color: #666;
}

.source a {
  color: #666;
  text-decoration: none;
}

.source a:hover {
  color: #007cba;
  text-decoration: underline;
}

.tags {
  color: #007cba;
}

.summary {
  margin: 0;
  color: #555;
  font-size: 14px;
  line-height: 1.5;
}
</style>

<script>
 document.querySelectorAll('.filter-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    
    const filter = btn.dataset.filter;
    const articles = document.querySelectorAll('.article');
    
    articles.forEach(article => {
      if (filter === 'all' || article.classList.contains(filter)) {
        article.style.display = 'block';
      } else {
        article.style.display = 'none';
      }
    });
  });
});
</script>
