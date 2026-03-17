---
layout: page
title: 归档
permalink: /archive/
---

{% assign postsByYear = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year in postsByYear %}
<details {% if forloop.first %}open{% endif %}>
  <summary><strong>{{ year.name }} 年（{{ year.items | size }} 篇）</strong></summary>
  <ul>
  {% for post in year.items %}
    <li><a href="{{ post.url }}">{{ post.title }}</a> — {{ post.date | date: "%m-%d" }}</li>
  {% endfor %}
  </ul>
</details>
{% endfor %}