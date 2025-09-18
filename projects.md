---
layout: page
title: Projects
permalink: /projects/
---
<div class="gallery">
  {% for project in site.projects %}
    <div class="card">
      <a href="{{ project.url | relative_url }}">
        <img src="{{ project.image }}" alt="{{ project.title }}">
        <h3>{{ project.title }}</h3>
        <p>{{ project.description }}</p>
      </a>
    </div>
  {% endfor %}
</div>
