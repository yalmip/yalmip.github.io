---
layout: archive
permalink: /tags/
title: "Tag Index"
excerpt: "An archive of posts sorted by tag frequency."
share: false
---

{{ page.excerpt | markdownify }}

 {% capture page_tags %}{% for tag in page.tags %}{{ tag | downcase }}#{{ tag }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
  {% assign tag_hashes = (page_tags | split: ',' | sort:0) %}

  <p class="page__taxonomy">
    <strong><i class="fa fa-fw fa-tags" aria-hidden="true"></i> {{ site.data.ui-text[site.locale].tags_label | default: "Tags:" }} </strong>
    <span itemprop="keywords">
    {% for hash in tag_hashes %}
      {% assign keyValue = hash | split: '#' %}
      {% capture tag_word %}{{ keyValue[1] | strip_newlines }}{% endcapture %}
      <a href="{{ base_path }}{{ tag_word | slugify | prepend: path_type | prepend: site.tag_archive.path }}" class="page__taxonomy-item" rel="tag">{{ tag_word }}</a>{% unless forloop.last %}<span class="sep">, </span>{% endunless %}
    {% endfor %}
    </span>
  </p>
  
<ul class="tag__list">


  {% assign sorted_tags = site.tags | sort_tags_by_name %}
  {% for tag in sorted_tags %}

<li> {{tag.name}}</li>

  <!--  <li><a href="{{ site.url }}/tag/{{ tag[0] | replace:' ','-' | downcase }}/" class="tag__item"><span class="tag__name">{{ tag[0] }}</span> <span class="tag__count">{{ tag[1] }}</span></a></li> -->
  
  {% endfor %}
</ul>
