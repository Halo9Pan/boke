---
layout: page
title: Slides
comments: yes
permalink: /slides/
---

<div class="post" itemscope itemtype="http://schema.org/BlogPosting" >
  {% for slide in site.slides limit:site.paginate %}
  <h2>
  <a href="{{ site.baseurl }}{{ slide.url }}" title="{{ slide.title }}" rel="bookmark">{{ slide.title }}</a>
  </h2>
  <p class="post-meta">
    {{ slide.description }}
  </p>
  {% endfor %}
</div>
