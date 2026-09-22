---
permalink: /en/
title: ""
layout: archive
author_profile: true
---

<p class="page__lead">What I build, how far I check it, and what the checking turns up. Bioinformatics, model validation, and AI agents.</p>

{% assign featured = site.posts | where: "lang", "en" | where: "featured", true %}
{% if featured.size > 0 %}
<h3 class="archive__subtitle">⭐ Featured</h3>
{% for post in featured %}
  {% include archive-single.html %}
{% endfor %}
{% endif %}

<h3 class="archive__subtitle">Latest posts</h3>

{% assign posts = site.posts | where: "lang", "en" %}
{% for post in posts %}
  {% unless post.featured %}{% include archive-single.html %}{% endunless %}
{% endfor %}
