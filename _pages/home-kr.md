---
permalink: /kr/
title: ""
layout: archive
author_profile: true
---

<p class="page__lead">무엇을 만들었고 어디까지 검증했는지, 그 과정에서 배운 것을 적습니다. 바이오인포매틱스, 모델 검증, AI 에이전트.</p>

{% assign featured = site.posts | where: "lang", "kr" | where: "featured", true %}
{% if featured.size > 0 %}
<h3 class="archive__subtitle">⭐ 추천 글</h3>
{% for post in featured %}
  {% include archive-single.html %}
{% endfor %}
{% endif %}

<h3 class="archive__subtitle">최근 글</h3>

{% assign posts = site.posts | where: "lang", "kr" %}
{% for post in posts %}
  {% unless post.featured %}{% include archive-single.html %}{% endunless %}
{% endfor %}
