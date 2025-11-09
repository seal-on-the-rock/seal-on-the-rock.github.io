---
layout: default
title: Seal on the Rock
---

<!-- ======= FAVICON ======= -->
<link rel="icon" href="/favicon.ico" type="image/x-icon">
<link rel="shortcut icon" href="/favicon.ico" type="image/x-icon">
<!-- ======================= -->

<!-- ======= iOS / Android ======= -->
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.ico">
<link rel="manifest" href="/manifest.webmanifest">
<meta name="theme-color" content="#ffffff">
<!-- ========================================== -->

<!-- LOGO -->
<p align="center" style="margin: 40px 0;">
  <img src="/assets/images/logo.png" alt="Seal on the Rock logo" 
       style="max-width: 90%; height: auto; border-radius: 12px;">
</p>

<!-- TITLE -->
<h1 align="center">Seal on the Rock 🌊</h1>
<p align="center"><em>A personal journey through cloud computing, creation, and reflection.</em></p>

---


### 👋 About Me
こんにちは！I'm **Seal**, a system engineer (SE) at  
[株式会社テンダ](https://www.tenda.co.jp/) 

I’m deeply passionate about **AWS** ☁️  
Self-learning cloud architecture and infrastructure design.  
Recently earned **AWS Certified Solutions Architect – Professional (SAP-C02)**   
👉 [View my certification](https://www.credly.com/badges/88bcf2eb-cf09-433e-9a8f-e89660985f8c/public_url)

<p align="center" style="margin: 40px 0;">
  <img src="/assets/images/aws-certified-solutions-architect-professional.png" alt="AWS Certified Solutions Architect - Professional certificate" 
       style="max-width: 20%; height: auto; border-radius: 12px;">
</p>

---

<!-- ARTICLES LIST -->
### 📝 Latest Articles

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}
  </li>
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
