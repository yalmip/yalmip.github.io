---
layout: archive
permalink: /tags/
title: "Tag Index"
excerpt: "An archive of posts sorted by tag frequency."
share: false
---

{{ page.excerpt | markdownify }}

<ul class="tag__list">

{% capture tags %}
  {% for tag in site.tags %}
    {{ tag[1].size | plus: 1000 }}#{{ tag[0] }}#{{ tag[1].size }}
  {% endfor %}
{% endcapture %}
{% assign sortedtags = tags | split:' ' | sort %}
{% for tag in sortedtags reversed %}
    {% assign tagitems = tag | split: '#' %}
    <li><a href="/tags/#{{ tagitems[1] }}">{{ tagitems[1] }} ({{ tagitems[2] }})</a></li>
{% endfor %}


  {% assign sorted_tags = site.tags | sort_tags_by_name %}
  {% for tag in sorted_tags %}

<li> {{tag}}</li>

  <!--  <li><a href="{{ site.url }}/tag/{{ tag[0] | replace:' ','-' | downcase }}/" class="tag__item"><span class="tag__name">{{ tag[0] }}</span> <span class="tag__count">{{ tag[1] }}</span></a></li> -->
  
  {% endfor %}
</ul>
