---
layout: archive
permalink: /tag/
title: "Tag Index"
excerpt: "An archive of posts sorted by tag frequency."
share: false
---

{{ page.excerpt | markdownify }}

<ul class="tag__list">
  {% assign sorted_tags = site.tags | sort_tags_by_name %}
  {% for tag in sorted_tags %}
    <li><a href="{{ site.url }}/tag/{{ tag[0] | replace:' ','-' | downcase }}/" class="tag__item"></a></li>
  {% endfor %}
</ul>
