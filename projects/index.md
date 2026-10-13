---
layout: page
title: Projects
nav_link: true
order: 3
---

# Projects

Open source software and hardware I have written, from developer tools to
the code behind my research. Every project is on
[GitHub](https://github.com/tkamucheka).

{% for group in site.data.projects %}
## {{ group.group }}

{{ group.intro }}

<div class="project-list">
  {% for project in group.projects %}
  <a class="project-card" href="{{ '/projects/' | append: project.slug | append: '/' | relative_url }}">
    <img class="project-card-banner" src="{{ '/public/images/projects/' | append: project.slug | append: '.svg' | relative_url }}" alt="">
    <span class="project-card-name">{{ project.name }}</span>
    <span class="project-card-summary">{{ project.summary }}</span>
    <span class="project-card-language">{{ project.language }}</span>
  </a>
  {% endfor %}
</div>
{% endfor %}
