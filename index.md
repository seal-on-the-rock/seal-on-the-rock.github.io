---
layout: default
title: Seal on the Rock
---

<style>
  a {
    text-decoration: none;
    color: inherit;
  }
  a:hover {
    opacity: 0.8;
  }
</style>

<!-- ======= HERO SECTION ======= -->
<section style="text-align:center; padding:60px 20px; background: linear-gradient(135deg, #0f2027, #203a43, #2c5364); color:white; border-radius:20px;">
  
  <!-- BIG LOGO (SOUL 🔥) -->
  <div style="margin-bottom:30px;">
    <img src="/assets/images/logo.png" alt="Seal on the Rock logo" 
         style="max-width:220px; height:auto; border-radius:16px; box-shadow:0 10px 30px rgba(0,0,0,0.4);">
  </div>

  <h1 style="font-size:42px; margin-bottom:10px;">Seal on the Rock</h1>
  <p style="font-size:18px; opacity:0.9;">Cloud • Architecture • Experiments • Growth</p>
  
  <div style="margin-top:30px; display:flex; justify-content:center; gap:16px; flex-wrap:wrap;">
    <a href="#articles" style="padding:12px 24px; background:#00c6ff; color:black; border-radius:999px; font-weight:bold;">Read Blog</a>
    <a href="https://www.credly.com/users/wang-haibao/badges#credly" target="_blank" style="padding:12px 24px; border:1px solid white; border-radius:999px;">View Certifications</a>
  </div>
</section>

<!-- ======= ABOUT CARD ======= -->
<section style="display:flex; justify-content:center; margin-top:-40px;">
  <div style="background:white; padding:30px; border-radius:20px; width:80%; box-shadow:0 10px 30px rgba(0,0,0,0.1);">
    <h2>👋 About Me</h2>
    <p>
      こんにちは！I'm <b>Seal</b>, a System Engineer based in Japan 🇯🇵<br>
      Passionate about <b>AWS / Cloud Architecture / DevOps</b>
    </p>

    <!-- COMPANY LINK WITH LOGO -->
    <div style="margin-top:20px; display:flex; align-items:center; gap:12px;">
      <a href="https://www.tenda.co.jp/" target="_blank" style="display:flex; align-items:center; gap:10px;">
        <img src="/assets/images/tenda-logo.png" alt="Tenda" style="width:40px; height:40px; border-radius:8px;">
        <span style="font-weight:bold; color:#333;">株式会社テンダ</span>
      </a>
    </div>

    <p style="margin-top:15px;">
      Currently leveling up toward <b>Cloud Architect & Platform Engineer</b> 🚀
    </p>
  </div>
</section>

<!-- ======= CERTIFICATIONS ======= -->
<section style="text-align:center; margin:60px 0;">
  <h2>🏆 Certifications</h2>
  <p>
    👉 <a href="https://www.credly.com/users/wang-haibao/badges#credly" target="_blank">Verify all badges on Credly</a>
  </p>
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

<!-- ======= ARTICLES ======= -->
<section id="articles" style="width:80%; margin:auto;">
  <h2>📝 Latest Articles</h2>
  <div style="display:grid; grid-template-columns:repeat(auto-fit, minmax(280px,1fr)); gap:20px;">
    {% for post in site.posts %}
    <div style="padding:20px; border-radius:16px; background:white; box-shadow:0 5px 15px rgba(0,0,0,0.08); transition:0.2s;">
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p style="font-size:14px; color:gray;">{{ post.date | date: "%Y-%m-%d" }}</p>
    </div>
    {% endfor %}
  </div>
</section>

<!-- ======= FOOTER ======= -->
<section style="text-align:center; margin:80px 0; color:gray;">
  <a href="https://github.com/seal-on-the-rock">GitHub</a> ·
  <a href="https://www.linkedin.com/in/wanghaibao">LinkedIn</a> ·
  <a href="https://twitter.com/yourprofile">Twitter</a>
</section>
