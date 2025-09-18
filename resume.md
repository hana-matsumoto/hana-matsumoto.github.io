---
layout: default
title: Resume
permalink: /resume/
---

# Resume

## Education
{% for item in site.data.resume.education %}
- **{{ item.degree }}**, {{ item.school }} ({{ item.years }})  
  {{ item.details }}
{% endfor %}

---

## Experience
<div class="timeline">
{% for job in site.data.resume.experience %}
  <div class="timeline-item">
    <div class="timeline-date">{{ job.years }}</div>
    <div class="timeline-content">
      <h3>{{ job.title }}</h3>
      <p><strong>{{ job.company }}</strong> — {{ job.location }}</p>
      <p>{{ job.description }}</p>
    </div>
  </div>
{% endfor %}
</div>
