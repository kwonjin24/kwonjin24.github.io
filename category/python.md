---
layout: default
title: "Python"
---

<h1>Category: {{ page.title }}</h1>

<ul>
  {% for post in site.categories.Python %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span>({{ post.date | date: "%Y년 %m월 %d일" }})</span>
    </li>
  {% endfor %}
</ul>
