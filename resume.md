---
layout: page
title: Resume
permalink: /resume/
---

# Resume

## Education
{% for item in site.data.resume.education %}
- **{{ item.degree }}**, {{ item.school }} ({{ item.date }})  
  {{ item.description }}
{% endfor %}

---

## Experience
<div class="timeline">
{% for job in site.data.resume.experience %}
  <div class="timeline-item">
    <div class="timeline-date">{{ job.date }}</div>
    <div class="timeline-content">
      <h3>{{ job.title }}</h3>
      <p><strong>{{ job.organization }}</strong></p>
      <p>{{ job.description }}</p>
    </div>
  </div>
{% endfor %}
</div>
