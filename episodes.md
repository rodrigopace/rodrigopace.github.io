---
layout: page
title: All Episodes
---

  {% for post in site.episodes reversed %}
  <article>
    <h3>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h3>
  </article>
{% endfor %}
