---
layout: default
title: Resume
permalink: /resume/
---

# Resume

## Experience
<div class="timeline">
{% for job in site.data.resume.experience %}
  <div class="timeline-item">
    <div class="timeline-marker"></div>
    <div class="timeline-content">
      <h3>{{ job.title }}</h3>
      <h4>{{ job.organization }}</h4>
      <p class="date">{{ job.date }}</p>
      <p>{{ job.description }}</p>
    </div>
  </div>
{% endfor %}
</div>

---

## Education
<div class="timeline">
{% for edu in site.data.resume.education %}
  <div class="timeline-item">
    <div class="timeline-marker"></div>
    <div class="timeline-content">
      <h3>{{ edu.degree }}</h3>
      <h4>{{ edu.school }}</h4>
      <p class="date">{{ edu.date }}</p>
      <p>{{ edu.description }}</p>
    </div>
  </div>
{% endfor %}
</div>
