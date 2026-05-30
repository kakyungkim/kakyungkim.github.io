---
permalink: /kr/
title: ""
layout: archive
author_profile: true
---

<p class="page__lead">데이터 과학과 생명과학이 만나는 지점에서의 연구 노트와 글 — NGS 분석, 임상 데이터, 유전체학.</p>

<h3 class="archive__subtitle">최근 글</h3>

{% assign posts = site.posts | where: "lang", "kr" %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
