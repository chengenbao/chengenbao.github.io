---
layout: default
title: "Reading List"
permalink: /reading-list/
---

<div class="reading-list-container">
  <div class="header-section">
    <h1 class="page-title">📚 Reading List</h1>
    <p class="page-desc">每日精选 · 技术 & 财经</p>
    
    <!-- 分类筛选按钮 -->
    <div class="filter-bar">
      <button class="filter-btn active" data-filter="all">📋 全部文章 (4)</button>
      <button class="filter-btn" data-filter="tech">📰 技术速递 (2)</button>
      <button class="filter-btn" data-filter="finance">💰 财经精选 (2)</button>
    </div>
  </div>

  <div class="articles-list">
    <!-- 技术速递 -->
    <article class="article-card tech" data-type="tech">
      <div class="article-header">
        <span class="article-icon">📰</span>
        <h2 class="article-title">
          <a href="/reading-list/2026-02-28-tech/">技术速递 2026-02-28：TorchForge RL 后训练 / AMD 1K GPU MoE / HF Transformers v5 MoE / CUDA Kernel Agent</a>
        </h2>
        <span class="article-date">2026年02月28日</span>
      </div>
      <div class="article-meta">
        <span class="source">来源：pytorch.org/blog · huggingface.co/blog</span>
        <span class="tags">#大模型 #PyTorch框架 #推理技术</span>
      </div>
      <p class="article-summary">每日精选：大模型 / PyTorch 框架 / 推理技术的最新进展汇总</p>
    </article>

    <article class="article-card tech" data-type="tech">
      <div class="article-header">
        <span class="article-icon">📰</span>
        <h2 class="article-title">
          <a href="/reading-list/2026-02-27-tech/">技术速递 2026-02-27：torchforge RL框架、MoE训练优化、CUDA Kernel自动生成</a>
        </h2>
        <span class="article-date">2026年02月27日</span>
      </div>
      <div class="article-meta">
        <span class="source">来源：pytorch.org/blog · huggingface.co/blog</span>
        <span class="tags">#大模型 #PyTorch框架 #推理技术</span>
      </div>
      <p class="article-summary">每日精选：大模型 / PyTorch 框架 / 推理技术的最新进展汇总</p>
    </article>

    <!-- 财经精选 -->
    <article class="article-card finance" data-type="finance">
      <div class="article-header">
        <span class="article-icon">💰</span>
        <h2 class="article-title">
          <a href="/reading-list/2026-02-28-finance/">财经精选 2026-02-28：AI 贸易效应 / 国债远期利率 / 稳定币历史</a>
        </h2>
        <span class="article-date">2026年02月28日</span>
      </div>
      <div class="article-meta">
        <span class="source">来源：federalreserve.gov/econres（Fed Notes）</span>
        <span class="tags">#宏观经济 #金融科技 #资产配置</span>
      </div>
      <p class="article-summary">每日精选：宏观经济 / 金融科技 / 资产配置的深度分析</p>
    </article>

    <article class="article-card finance" data-type="finance">
      <div class="article-header">
        <span class="article-icon">💰</span>
        <h2 class="article-title">
          <a href="/reading-list/2026-02-27-finance/">财经精选 2026-02-27：AI贸易影响、美债长端利率、Stablecoin历史</a>
        </h2>
        <span class="article-date">2026年02月27日</span>
      </div>
      <div class="article-meta">
        <span class="source">来源：federalreserve.gov/econres（Fed Notes）</span>
        <span class="tags">#宏观经济 #金融科技 #资产配置</span>
      </div>
      <p class="article-summary">每日精选：宏观经济 / 金融科技 / 资产配置的深度分析</p>
    </article>
  </div>
</div>

<style>
.reading-list-container {
  max-width: 900px;
  margin: 0 auto;
  padding: 30px 20px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.header-section {
  text-align: center;
  margin-bottom: 40px;
}

.page-title {
  font-size: 2.5em;
  color: #2c3e50;
  margin: 0 0 10px 0;
  font-weight: 700;
}

.page-desc {
  font-size: 1.2em;
  color: #7f8c8d;
  margin: 0 0 30px 0;
}

.filter-bar {
  display: flex;
  justify-content: center;
  gap: 15px;
  flex-wrap: wrap;
}

.filter-btn {
  padding: 12px 24px;
  border: 2px solid #3498db;
  background: white;
  color: #3498db;
  border-radius: 25px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.filter-btn:hover {
  background: #3498db;
  color: white;
  transform: translateY(-2px);
}

.filter-btn.active {
  background: #3498db;
  color: white;
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3);
}

.articles-list {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

.article-card {
  background: white;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border-left: 5px solid #3498db;
  transition: all 0.3s ease;
}

.article-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

.article-card.tech {
  border-left-color: #e74c3c;
}

.article-card.finance {
  border-left-color: #f39c12;
}

.article-header {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  margin-bottom: 15px;
}

.article-icon {
  font-size: 24px;
  margin-top: 2px;
}

.article-title {
  flex: 1;
  margin: 0;
}

.article-title a {
  color: #2c3e50;
  text-decoration: none;
  font-size: 1.3em;
  font-weight: 600;
  line-height: 1.4;
}

.article-title a:hover {
  color: #3498db;
}

.article-date {
  color: #95a5a6;
  font-size: 14px;
  white-space: nowrap;
  font-weight: 500;
}

.article-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  flex-wrap: wrap;
  gap: 10px;
}

.source {
  color: #7f8c8d;
  font-size: 13px;
}

.tags {
  color: #3498db;
  font-size: 13px;
  font-weight: 500;
}

.article-summary {
  color: #555;
  line-height: 1.6;
  margin: 0;
  font-size: 15px;
}

@media (max-width: 768px) {
  .reading-list-container {
    padding: 20px 15px;
  }
  
  .page-title {
    font-size: 2em;
  }
  
  .filter-bar {
    gap: 10px;
  }
  
  .filter-btn {
    padding: 10px 18px;
    font-size: 13px;
  }
  
  .article-header {
    flex-direction: column;
    gap: 10px;
  }
  
  .article-date {
    align-self: flex-start;
  }
  
  .article-meta {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>

<script>
// 分类筛选功能
 document.querySelectorAll('.filter-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    // 更新按钮状态
    document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    
    const filter = btn.dataset.filter;
    const articles = document.querySelectorAll('.article-card');
    
    // 筛选文章
    articles.forEach(article => {
      if (filter === 'all' || article.dataset.type === filter) {
        article.style.display = 'block';
      } else {
        article.style.display = 'none';
      }
    });
  });
});
</script>
