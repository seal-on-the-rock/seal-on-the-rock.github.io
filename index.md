---
layout: default
title: Seal on the Rock
---

<style>
  body {
    background: #0f172a;
    color: #e5e7eb;
  }

  a {
    text-decoration: none;
    color: inherit;
  }

  a:hover {
    opacity: 0.85;
  }

  .section {
    width: 80%;
    margin: 60px auto;
  }

  .card {
    background: #111827;
    border: 1px solid rgba(255,255,255,0.08);
    border-radius: 16px;
    padding: 24px;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px,1fr));
    gap: 20px;
  }

  .btn {
    padding: 12px 24px;
    border-radius: 999px;
    font-weight: 600;
    border: 1px solid rgba(255,255,255,0.2);
  }

  .btn-primary {
    background: #38bdf8;
    color: black;
    border: none;
  }

  .divider {
    height: 1px;
    background: rgba(255,255,255,0.1);
    margin: 40px 0;
  }
</style>

<!-- HERO -->
<section style="text-align:center; padding:60px 20px;">

  <div style="margin-bottom:30px;">
    <img src="/assets/images/logo.png" alt="logo"
         style="max-width:220px; border-radius:16px; box-shadow:0 10px 30px rgba(0,0,0,0.5);">
  </div>

  <h1 style="font-size:40px; margin-bottom:10px;">Seal on the Rock</h1>
  <p style="opacity:0.7;">Cloud Architecture · Systems · Experiments</p>

  <div style="margin-top:30px; display:flex; justify-content:center; gap:16px; flex-wrap:wrap;">
    <a href="#articles" class="btn btn-primary">Articles</a>
    <a href="#certifications" class="btn">Certifications</a>
  </div>

</section>

<div class="divider"></div>

<!-- ABOUT -->
<section class="section">
  <div class="card">
    <h2>About</h2>
    <p>
      System Engineer based in Japan. Focused on AWS, cloud architecture and infrastructure design.
    </p>

    <div style="margin-top:20px; display:flex; align-items:center; gap:12px;">
      <a href="https://www.tenda.co.jp/" target="_blank" style="display:flex; align-items:center; gap:10px;">
        <img src="/assets/images/tenda-logo.png" style="width:40px; height:40px; border-radius:8px;">
        <span style="font-weight:600;">株式会社テンダ</span>
      </a>
    </div>

  </div>
</section>

<div class="divider"></div>

<!-- CERTIFICATIONS -->
<section id="certifications" class="section" style="text-align:center;">
  <h2>Certifications</h2>

  <div style="margin:20px 0;">
    <a href="https://www.credly.com/users/wang-haibao/badges#credly" target="_blank" class="btn">
      Verify on Credly
    </a>
  </div>

  <div style="display:flex; flex-wrap:wrap; justify-content:center; gap:20px; margin-top:20px;">
    <img src="/assets/images/aws-certified-solutions-architect-associate.png" style="width:80px;">
    <img src="/assets/images/aws-certified-developer-associate.png" style="width:80px;">
    <img src="/assets/images/aws-certified-solutions-architect-professional.png" style="width:80px;">
    <img src="/assets/images/aws-certified-devops-engineer-professional.png" style="width:80px;">
    <img src="/assets/images/aws-certified-security-specialty.png" style="width:80px;">
    <img src="/assets/images/aws-certified-machine-learning-specialty.png" style="width:80px;">
    <img src="/assets/images/aws-certified-advanced-networking-specialty.png" style="width:80px;">
  </div>
</section>

<div class="divider"></div>

<!-- ARTICLES -->
<section id="articles" class="section">
  <h2>Articles</h2>

  <div class="grid">
    {% for post in site.posts %}
    <div class="card" style="transition:0.2s;">
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p style="font-size:14px; opacity:0.5;">{{ post.date | date: "%Y-%m-%d" }}</p>
    </div>
    {% endfor %}
  </div>
</section>

<div class="divider"></div>

<!-- FOOTER -->
<section style="text-align:center; margin:60px 0; opacity:0.5;">
  <a href="https://github.com/seal-on-the-rock">GitHub</a> ·
  <a href="https://www.linkedin.com/in/wanghaibao">LinkedIn</a> ·
  <a href="https://twitter.com/yourprofile">Twitter</a>
</section>
