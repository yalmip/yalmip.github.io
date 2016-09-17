---
layout: archive
permalink: /tags/
title: "Tag Index"
excerpt: "An archive of posts sorted by tag frequency."
share: false
---

{{ page.excerpt | markdownify }}

<ul class="tag__list">
  {% assign sorted_tags = site.tags | sort_tags_by_name %}
  {% for tag in sorted_tags %}
  HEJHEJ
    <li><a href="{{ site.url }}/tag/{{ tag[0] | replace:' ','-' | downcase }}/" class="tag__item"><span class="tag__name">{{ tag[0] }}</span> <span class="tag__count">{{ tag[1] }}</span></a></li>
  {% endfor %}
</ul>
