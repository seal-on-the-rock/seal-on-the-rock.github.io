---
layout: default
title: Seal on the Rock
---

<!-- LOGO -->
<p align="center" style="margin: 40px 0;">
  <img src="/assets/images/logo.png" alt="Seal on the Rock logo" 
       style="max-width: 90%; height: auto; border-radius: 12px;">
</p>

<!-- TITLE -->
<h1 align="center">Seal on the Rock 🌊</h1>
<p align="center"><em>A personal journey through cloud computing, creation, and reflection.</em></p>

---

<!-- WELCOME -->
### 👋 Welcome

Welcome to my personal tech blog!  
Here I share insights, notes, and reflections on cloud computing, technology, and practical learning experiences.

---

<!-- ARTICLES LIST -->
### 📝 Latest Articles

<ul>
{% assign articles = site.static_files | where:"extname",".md" %}
{% for article in articles %}
  {% if article.path != "/index.md" %}
    <li><a href="{{ article.path | remove_first:'/' }}">{{ article.name | remove:'.md' | replace:'-',' ' | capitalize }}</a></li>
  {% endif %}
{% endfor %}
</ul>

---

<!-- SOCIAL LINKS -->
### 🔗 Connect

<p align="center">
  <a href="https://github.com/seal-on-the-rock">🐙 GitHub</a> |
  <a href="https://www.linkedin.com/in/wanghaibao">💼 LinkedIn</a> |
  <a href="https://twitter.com/yourprofile">🐦 Twitter</a>
</p>
