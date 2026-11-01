---
layout: page
title: Home
---

# Biruk's blog

I write about reinforcement learning, machine learning, and the experiments I'm exploring along the way.

[Read all posts]({{ '/posts/' | relative_url }})

## Latest posts

<ul class="post-list">
  {% for post in site.posts limit:5 %}
    <li>
      <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
      <h2><a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a></h2>
      {% if post.excerpt %}
        <p>{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
      {% endif %}
    </li>
  {% endfor %}
</ul>

You can also find the [code and results from my experiments on GitHub](https://github.com/biruk50/RL).
