---
permalink: /kr/topics/
title: "주제"
author_profile: true
---

{% assign topics = "AI & Tooling,Computational Pathology,Data Science,Life Science" | split: "," %}
{% assign mine = site.posts | where: "lang", "kr" %}

{% for topic in topics %}
  {% assign inTopic = mine | where_exp: "p", "p.categories contains topic" %}
  {% if inTopic.size > 0 %}
## {{ topic }}
<ul class="topic-list">
{% for post in inTopic %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> <small>· {{ post.date | date: "%Y-%m-%d" }}</small></li>
{% endfor %}
</ul>
  {% endif %}
{% endfor %}
