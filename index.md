---
layout: none
---
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Seal on the Rock</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">

<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
  font-family: 'Inter', sans-serif;
  background: linear-gradient(180deg, #f5f7fa, #e4ecf7);
  color: #1a1a1a;
}

.container {
  max-width: 1100px;
  margin: auto;
  padding: 40px 20px;
}

/* LAYOUT: HERO + ABOUT SIDE BY SIDE */
.top-grid {
  display: grid;
  grid-template-columns: 1.2fr 1fr;
  gap: 24px;
  margin-bottom: 40px;
}

@media (max-width: 768px) {
  .top-grid {
    grid-template-columns: 1fr;
  }
}

/* COMMON CARD */
.section-card {
  background: white;
  border-radius: 24px;
  padding: 40px;
  margin-bottom: 40px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.06);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.section-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 20px 50px rgba(0,0,0,0.12);
}

/* HERO */
.hero {
  text-align: center;
}

.hero img {
  width: 260px;
  border-radius: 20px;
  transition: transform 0.4s ease, box-shadow 0.4s ease;
}

.hero img:hover {
  transform: scale(1.05);
  box-shadow: 0 20px 40px rgba(0,0,0,0.15);
}

.hero p {
  margin-top: 16px;
  color: #555;
}

/* CERTIFICATIONS */
.cert-section {
  text-align: center;
}

.cert-title {
  font-weight: 600;
  margin-bottom: 20px;
}

.cert-link {
  display: inline-block;
  margin-bottom: 30px;
  padding: 8px 16px;
  background: #111;
  color: white;
  border-radius: 999px;
  font-size: 14px;
  text-decoration: none;
  transition: transform 0.2s, box-shadow 0.2s;
}

.cert-link:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.1);
}

.cert-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
  gap: 16px;
}

.cert-grid img {
  width: 100%;
  border-radius: 12px;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.cert-grid img:hover {
  transform: scale(1.08);
  box-shadow: 0 10px 20px rgba(0,0,0,0.1);
}

/* ABOUT */
.about {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
}

.about-text h3 {
  margin-bottom: 8px;
}

.company-logo {
  width: 70px;
  transition: transform 0.3s;
}

.company-logo:hover {
  transform: scale(1.1);
}

/* ARTICLE CARDS */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 24px;
}

.card {
  background: white;
  padding: 20px;
  border-radius: 20px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.05);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 40px rgba(0,0,0,0.1);
}

.card a {
  text-decoration: none;
  color: inherit;
}

.card-title {
  font-weight: 600;
  margin-bottom: 10px;
}

.card-date {
  font-size: 12px;
  color: #777;
  margin-bottom: 10px;
}

.tag-container {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.tag {
  font-size: 11px;
  padding: 4px 8px;
  background: #eef2f7;
  border-radius: 999px;
  color: #555;
  transition: background 0.2s;
}

.tag:hover {
  background: #dbe7ff;
}

/* SOCIAL */
.social {
  text-align: center;
  margin-top: 40px;
}

.social a {
  margin: 0 10px;
  color: #555;
  text-decoration: none;
  transition: color 0.3s;
}

.social a:hover {
  color: #000;
}

</style>
</head>

<body>

<div class="container">

  <!-- TOP SECTION (HERO + ABOUT) -->
  <div class="top-grid">

    <!-- HERO CARD -->
    <div class="section-card hero">
      <img src="/assets/images/logo_1.png" alt="logo">
      <p>Cloud / Architecture / DevOps</p>
    </div>

    <!-- ABOUT CARD -->
    <div class="section-card about">
      <div class="about-text">
        <h3>About Me</h3>
        <p>System Engineer passionate about AWS and architecture design.</p>
      </div>
      <a href="https://www.tenda.co.jp/" target="_blank">
        <img class="company-logo" src="/assets/images/company-logo.png" alt="company">
      </a>
    </div>

  </div>

  <!-- CERT CARD -->
  <div class="section-card cert-section">
    <div class="cert-title">AWS Certifications</div>
    <a class="cert-link" href="https://www.credly.com/users/wang-haibao/badges#credly" target="_blank">
      View All Badges
    </a>

    <div class="cert-grid">
      <img src="/assets/images/aws-certified-cloud-practitioner.png">
      <img src="/assets/images/aws-certified-ai-practitioner.png">
      <img src="/assets/images/aws-certified-solutions-architect-associate.png">
      <img src="/assets/images/aws-certified-developer-associate.png">
      <img src="/assets/images/aws-certified-solutions-architect-professional.png">
      <img src="/assets/images/aws-certified-devops-engineer-professional.png">
      <img src="/assets/images/aws-certified-generative-ai-developer-professional.png">
      <img src="/assets/images/aws-certified-generative-ai-developer-professional-.png">
      <img src="/assets/images/aws-certified-security-specialty.png">
      <img src="/assets/images/aws-certified-machine-learning-specialty.png">
      <img src="/assets/images/aws-certified-advanced-networking-specialty.png">
    </div>
  </div>

  <!-- ARTICLES -->
  <div class="grid">
    {% for post in site.posts %}
    <div class="card">
      <a href="{{ post.url }}">
        <div class="card-title">{{ post.title }}</div>
        <div class="card-date">{{ post.date | date: "%Y-%m-%d" }}</div>
      </a>

      {% if post.tags %}
      <div class="tag-container">
        {% for tag in post.tags %}
          <div class="tag">{{ tag }}</div>
        {% endfor %}
      </div>
      {% endif %}

    </div>
    {% endfor %}
  </div>

  <!-- SOCIAL -->
  <div class="social">
    <a href="https://github.com/seal-on-the-rock">GitHub</a>
    <a href="https://www.linkedin.com/in/wanghaibao">LinkedIn</a>
    <a href="https://twitter.com/yourprofile">Twitter</a>
  </div>

</div>

</body>
</html>
