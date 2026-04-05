---
layout: none
title: Seal on the Rock
---

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Seal on the Rock</title>

<style>
body {
  margin:0;
  background:#0b1220;
  color:#e5e7eb;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.container {
  max-width:1200px;
  margin:40px auto;
  padding:0 20px;
  display:grid;
  grid-template-columns: repeat(12, 1fr);
  gap:20px;
}

.card {
  background:#111827;
  border:1px solid rgba(255,255,255,0.06);
  border-radius:20px;
  padding:24px;
  transition:all .25s ease;
}

.card:hover {
  transform:translateY(-6px);
  box-shadow:0 20px 40px rgba(0,0,0,0.4);
  border-color:rgba(56,189,248,0.6);
}

.hero {
  grid-column: span 7;
  display:flex;
  flex-direction:column;
  justify-content:center;
}

.hero img {
  width:140px;
  border-radius:16px;
  margin-bottom:20px;
}

.hero h1 {
  margin:0;
  font-size:36px;
}

.hero p {
  opacity:0.6;
  margin-top:10px;
}

.buttons {
  margin-top:20px;
  display:flex;
  gap:12px;
}

.btn {
  padding:10px 18px;
  border-radius:999px;
  border:1px solid rgba(255,255,255,0.2);
}

.btn.primary {
  background:#38bdf8;
  color:black;
  border:none;
}

.about { grid-column: span 5; }
.cert { grid-column: span 4; }
.links { grid-column: span 3; }
.articles { grid-column: span 8; }

.cert-grid {
  display:grid;
  grid-template-columns: repeat(3,1fr);
  gap:10px;
  margin-top:12px;
}

.cert-grid img {
  width:100%;
  border-radius:8px;
}

.article {
  border-top:1px solid rgba(255,255,255,0.08);
  margin-top:12px;
  padding-top:12px;
}

.footer {
  grid-column: span 12;
  text-align:center;
  opacity:0.5;
}

@media (max-width:900px){
  .hero, .about, .cert, .links, .articles {
    grid-column: span 12;
  }
}
</style>
</head>

<body>

<div class="container">

<!-- HERO -->
<div class="card hero">
  <img src="/assets/images/logo.png">
  <h1>Seal on the Rock</h1>
  <p>Cloud Architecture · Systems · Experiments</p>

  <div class="buttons">
    <a href="#articles" class="btn primary">Articles</a>
    <a href="#cert" class="btn">Certifications</a>
  </div>
</div>

<!-- ABOUT -->
<div class="card about">
  <h3>About</h3>
  <p style="opacity:0.7; font-size:14px;">
    System Engineer in Japan focused on AWS, cloud architecture and infrastructure design.
  </p>

  <div style="margin-top:16px; display:flex; align-items:center; gap:10px;">
    <a href="https://www.tenda.co.jp/" target="_blank" style="display:flex; align-items:center; gap:10px;">
      <img src="/assets/images/tenda-logo.png" style="width:36px; border-radius:6px;">
      <span style="font-size:14px;">株式会社テンダ</span>
    </a>
  </div>
</div>

<!-- CERT -->
<div id="cert" class="card cert">
  <h3>Certifications</h3>
  <a href="https://www.credly.com/users/wang-haibao/badges#credly" target="_blank" class="btn">Verify</a>

  <div class="cert-grid">
    <img src="/assets/images/aws-certified-solutions-architect-associate.png">
    <img src="/assets/images/aws-certified-developer-associate.png">
    <img src="/assets/images/aws-certified-solutions-architect-professional.png">
    <img src="/assets/images/aws-certified-devops-engineer-professional.png">
    <img src="/assets/images/aws-certified-security-specialty.png">
    <img src="/assets/images/aws-certified-machine-learning-specialty.png">
  </div>
</div>

<!-- LINKS -->
<div class="card links">
  <h3>Links</h3>
  <div style="display:flex; flex-direction:column; gap:10px; font-size:14px; opacity:0.7;">
    <a href="https://github.com/seal-on-the-rock">GitHub</a>
    <a href="https://www.linkedin.com/in/wanghaibao">LinkedIn</a>
    <a href="https://twitter.com/yourprofile">Twitter</a>
  </div>
</div>

<!-- ARTICLES -->
<div id="articles" class="card articles">
  <h3>Articles</h3>

  {% for post in site.posts %}
  <div class="article">
    <a href="{{ post.url }}">{{ post.title }}</a>
    <div style="font-size:12px; opacity:0.5;">{{ post.date | date: "%Y-%m-%d" }}</div>
  </div>
  {% endfor %}

</div>

<!-- FOOTER -->
<div class="footer">
  © Seal on the Rock
</div>

</div>

</body>
</html>
