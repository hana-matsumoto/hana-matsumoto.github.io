---
layout: single
title: Resume
permalink: /resume/
---

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
      <div class="timeline-header">
        <h3>{{ job.title }}</h3> <span class="job-date">{{ job.date }}</span>
      </div>
      <p><strong>{{ job.organization }}</strong></p>
      <ul>
        {% for bullet in job.description %}
          <li>{{ bullet }}</li>
        {% endfor %}
      </ul>
    </div>
  {% endfor %}
</div>
