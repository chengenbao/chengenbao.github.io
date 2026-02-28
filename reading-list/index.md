---
layout: default
title: "Reading List"
permalink: /reading-list/
---

<div class="reading-list-container">
  <!-- 左侧文章列表（保持原有样式） -->
  <div class="articles-section">
    <h1 class="page-title">Reading List</h1>
    <p class="page-desc">每日精选 · 技术 & 财经</p>

    <div class="notes-list">
      <div class="note-card" data-type="tech">
        <div class="note-icon">📰</div>
        <div class="note-body">
          <h2><a href="/reading-list/2026-02-28-tech/">技术速递 2026-02-28：TorchForge RL 后训练 / AMD 1K GPU MoE / HF Transformers v5 MoE / CUDA Kernel Agent</a></h2>
          <p class="note-meta">2026年02月28日</p>
          <p>每日精选：大模型 / PyTorch 框架 / 推理技术<br>来源：pytorch.org/blog · huggingface.co/blog</p>
        </div>
      </div>

      <div class="note-card" data-type="finance">
        <div class="note-icon">💰</div>
        <div class="note-body">
          <h2><a href="/reading-list/2026-02-28-finance/">财经精选 2026-02-28：AI 贸易效应 / 国债远期利率 / 稳定币历史</a></h2>
          <p class="note-meta">2026年02月28日</p>
          <p>每日精选：宏观经济 / 金融科技 / 资产配置<br>来源：federalreserve.gov/econres（Fed Notes）</p>
        </div>
      </div>

      <div class="note-card" data-type="tech">
        <div class="note-icon">📰</div>
        <div class="note-body">
          <h2><a href="/reading-list/2026-02-27-tech/">技术速递 2026-02-27：torchforge RL框架、MoE训练优化、CUDA Kernel自动生成</a></h2>
          <p class="note-meta">2026年02月27日</p>
          <p>每日精选：大模型 / PyTorch 框架 / 推理技术<br>来源：pytorch.org/blog · huggingface.co/blog</p>
        </div>
      </div>

      <div class="note-card" data-type="finance">
        <div class="note-icon">💰</div>
        <div class="note-body">
          <h2><a href="/reading-list/2026-02-27-finance/">财经精选 2026-02-27：AI贸易影响、美债长端利率、Stablecoin历史</a></h2>
          <p class="note-meta">2026年02月27日</p>
          <p>每日精选：宏观经济 / 金融科技 / 资产配置<br>来源：federalreserve.gov/econres（Fed Notes）</p>
        </div>
      </div>
    </div>
  </div>
  
  <!-- 右侧分类筛选面板 -->
  <div class="sidebar-section">
    <div class="filter-panel">
      <h3>📂 分类筛选</h3>
      <div class="category-filters">
        <button class="category-btn active" data-type="all">📋 全部文章</button>
        <button class="category-btn" data-type="tech">📰 技术速递</button>
        <button class="category-btn" data-type="finance">💰 财经精选</button>
      </div>
      
      <div class="filter-stats">
        <p>共 <span id="total-count">4</span> 篇文章</p>
        <p>📰 技术速递: <span id="tech-count">2</span> 篇</p>
        <p>💰 财经精选: <span id="finance-count">2</span> 篇</p>
      </div>
    </div>
  </div>
</div>

<style>
.reading-list-container {
  display: flex;
  max-width: 1200px;
  margin: 0 auto;
  gap: 30px;
  padding: 20px;
}

.articles-section {
  flex: 1;
  min-width: 0;
}

.sidebar-section {
  width: 220px;
  position: sticky;
  top: 20px;
  height: fit-content;
}

.filter-panel {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e1e4e8;
}

.filter-panel h3 {
  margin: 0 0 15px 0;
  color: #333;
  font-size: 16px;
}

.category-filters {
  margin-bottom: 20px;
}

.category-btn {
  display: block;
  width: 100%;
  padding: 12px 15px;
  margin-bottom: 8px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 6px;
  cursor: pointer;
  text-align: left;
  font-size: 14px;
  transition: all 0.2s ease;
}

.category-btn:hover {
  background: #f0f8ff;
  border-color: #007cba;
}

.category-btn.active {
  background: #007cba;
  color: white;
  border-color: #007cba;
}

.filter-stats {
  border-top: 1px solid #eee;
  padding-top: 15px;
  font-size: 13px;
  color: #666;
}

.filter-stats p {
  margin: 5px 0;
}

/* 保持原有样式 */
.page-title {
  color: #333;
  margin-bottom: 10px;
}

.page-desc {
  color: #666;
  margin-bottom: 30px;
  font-size: 16px;
}

.note-card {
  display: flex;
  background: white;
  border: 1px solid #e1e4e8;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: box-shadow 0.3s ease;
}

.note-card:hover {
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}

.note-icon {
  font-size: 24px;
  margin-right: 15px;
  margin-top: 5px;
}

.note-body h2 {
  margin: 0 0 10px 0;
}

.note-body h2 a {
  color: #007cba;
  text-decoration: none;
  font-size: 18px;
}

.note-body h2 a:hover {
  text-decoration: underline;
}

.note-meta {
  color: #666;
  font-size: 14px;
  margin-bottom: 8px;
}

.note-body p {
  margin: 5px 0;
  font-size: 14px;
  color: #555;
}

@media (max-width: 768px) {
  .reading-list-container {
    flex-direction: column;
  }
  
  .sidebar-section {
    width: 100%;
    position: static;
  }
}
</style>

<script>
// 分类筛选功能
 document.querySelectorAll('.category-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    // 更新按钮状态
    document.querySelectorAll('.category-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    
    const type = btn.dataset.type;
    const cards = document.querySelectorAll('.note-card');
    
    // 筛选卡片
    cards.forEach(card => {
      if (type === 'all' || card.dataset.type === type) {
        card.style.display = 'flex';
      } else {
        card.style.display = 'none';
      }
    });
    
    // 更新统计
    updateStats();
  });
});

// 更新统计信息
function updateStats() {
  const totalCards = document.querySelectorAll('.note-card');
  const techCards = document.querySelectorAll('.note-card[data-type="tech"]');
  const financeCards = document.querySelectorAll('.note-card[data-type="finance"]');
  const visibleCards = document.querySelectorAll('.note-card[style*="display: flex"]');
  
  document.getElementById('total-count').textContent = visibleCards.length;
  document.getElementById('tech-count').textContent = techCards.length;
  document.getElementById('finance-count').textContent = financeCards.length;
}

// 初始化
updateStats();
</script>
