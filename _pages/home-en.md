---
permalink: /en/
title: ""
layout: archive
author_profile: true
---

<p class="page__lead">Research notes and writing at the intersection of data science and life science — NGS analysis, clinical data, and genomics.</p>

<h3 class="archive__subtitle">Latest posts</h3>

{% assign posts = site.posts | where: "lang", "en" %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
