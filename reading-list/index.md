---
layout: default
title: Reading List
permalink: /reading-list/
---
<div class="home">
  <h1 class="page-title">Reading List</h1>
  <p class="page-desc">每日精选 · 技术 & 财经</p>

  <div class="notes-list">
    {% assign items = site.reading_list | sort: 'date' | reverse %}
    {% for item in items %}
    <a class="note-card" href="{{ item.url }}">
      <div class="note-icon">{% if item.category == 'finance' %}💰{% else %}📰{% endif %}</div>
      <div class="note-body">
        <h2>{{ item.title }}</h2>
        <div class="post-meta">{{ item.date | date: "%Y年%m月%d日" }}</div>
        {% if item.excerpt %}<p>{{ item.excerpt | strip_html | truncatewords: 20 }}</p>{% endif %}
      </div>
    </a>
    {% endfor %}
  </div>
</div>
