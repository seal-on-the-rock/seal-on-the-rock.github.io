---
layout: default
title: Seal on the Rock
---

<style>
  body {
    background: #0b1220;
    color: #e5e7eb;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  }

  a { text-decoration: none; color: inherit; }

  .container {
    width: 90%;
    max-width: 1100px;
    margin: 40px auto;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
    gap: 20px;
  }

  .card {
    background: #111827;
    border: 1px solid rgba(255,255,255,0.06);
    border-radius: 20px;
    padding: 24px;
    transition: all 0.25s ease;
  }

  .card:hover {
    transform: translateY(-6px);
    border-color: rgba(56,189,248,0.6);
    box-shadow: 0 10px 30px rgba(0,0,0,0.4);
  }

  .hero {
    grid-column: span 2;
    text-align: center;
    padding: 40px 20px;
  }

  .logo {
    width: 160px;
    border-radius: 16px;
    margin-bottom: 20px;
  }

  .title {
    font-size: 32px;
    font-weight: 700;
  }

  .subtitle {
    opacity: 0.6;
    margin-top: 6px;
  }

  .btn {
    display: inline-block;
    margin-top: 16px;
    padding: 10px 18px;
    border-radius: 999px;
    border: 1px solid rgba(255,255,255,0.2);
    font-size: 14px;
  }

  .btn-primary {
    background: #38bdf8;
    color: black;
    border: none;
  }

  .cert-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(60px,1fr));
    gap: 12px;
    margin-top: 16px;
  }

  .cert-grid img {
    width: 100%;
    border-radius: 8px;
  }

  .article-card {
    border-top: 1px solid rgba(255,255,255,0.08);
    padding-top: 12px;
    margin-top: 12px;
  }
</style>

<div class="container">

  <!-- HERO CARD -->
  <div class="card hero">
    <img src="/assets/images/logo.png" class="logo">
    <div class="title">Seal on the Rock</div>
    <div class="subtitle">Cloud Architecture · Systems · Experiments</div>

    <a href="#articles" class="btn btn-primary">Articles</a>
    <a href="#certifications" class="btn">Certifications</a>
  </div>

  <!-- ABOUT CARD -->
  <div class="card">
    <h3>About</h3>
    <p style="opacity:0.7; font-size:14px;">
      System Engineer in Japan focused on AWS, cloud architecture and infrastructure design.
    </p>

    <div style="margin-top:16px; display:flex; align-items:center; gap:10px;">
      <a href="https://www.tenda.co.jp/" target="_blank" style="display:flex; align-items:center; gap:10px;">
        <img src="/assets/images/tenda-logo.png" style="width:36px; height:36px; border-radius:6px;">
        <span style="font-size:14px;">株式会社テンダ</span>
      </a>
    </div>
  </div>

  <!-- CERT CARD -->
  <div id="certifications" class="card">
    <h3>Certifications</h3>

    <a href="https://www.credly.com/users/wang-haibao/badges#credly" target="_blank" class="btn" style="margin-bottom:12px; display:inline-block;">
      Verify on Credly
    </a>

    <div class="cert-grid">
      <img src="/assets/images/aws-certified-solutions-architect-associate.png">
      <img src="/assets/images/aws-certified-developer-associate.png">
      <img src="/assets/images/aws-certified-solutions-architect-professional.png">
      <img src="/assets/images/aws-certified-devops-engineer-professional.png">
      <img src="/assets/images/aws-certified-security-specialty.png">
      <img src="/assets/images/aws-certified-machine-learning-specialty.png">
      <img src="/assets/images/aws-certified-advanced-networking-specialty.png">
    </div>
  </div>

  <!-- ARTICLES CARD -->
  <div id="articles" class="card" style="grid-column: span 2;">
    <h3>Articles</h3>

    {% for post in site.posts %}
    <div class="article-card">
      <a href="{{ post.url }}">{{ post.title }}</a>
      <div style="font-size:12px; opacity:0.5;">{{ post.date | date: "%Y-%m-%d" }}</div>
    </div>
    {% endfor %}

  </div>

  <!-- LINKS CARD -->
  <div class="card">
    <h3>Links</h3>
    <p style="font-size:14px; opacity:0.7; line-height:1.8;">
      <a href="https://github.com/seal-on-the-rock">GitHub</a><br>
      <a href="https://www.linkedin.com/in/wanghaibao">LinkedIn</a><br>
      <a href="https://twitter.com/yourprofile">Twitter</a>
    </p>
  </div>

</div>
