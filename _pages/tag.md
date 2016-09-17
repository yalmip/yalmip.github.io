---
layout: archive
permalink: /tags/
title: "Tag Index"
excerpt: "An archive of posts sorted by tag frequency."
share: false
---

{{ page.excerpt | markdownify }}

<ul class="tag__list">

{% capture site_tags %}{% for tag in site.tags %}{{ tag | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
{% assign tag_words = site_tags | split:',' | sort %}
<ul class="tags">
  {% for item in (0..site.tags.size) %}{% unless forloop.last %}
    {% capture this_word %}{{ tag_words[item] }}{% endcapture %}
   <li>
      <a href="#{{ this_word | cgi_escape | downcase}}" class="tag">{{ this_word }}
        <span>({{ site.tags[this_word].size }})</span>
      </a>
  </li>  
  {% endunless %}{% endfor %}
</ul>
<div>
  {% for item in (0..site.tags.size) %}{% unless forloop.last %}
    {% capture this_word %}{{ tag_words[item] }}{% endcapture %}
    <h2 id="{{ this_word | cgi_escape | downcase}}">{{ this_word | downcase}}</h2>
    {% for post in site.tags[this_word] %}{% if post.title != null %}
      <div>
        <span style="float: left;">
          <a href="{{ post.url }}">{{ post.title }}</a>
        </span>
        <span style="float: right;">
          {{ post.date | date_to_string }}
        </span>
      </div>
      <div style="clear: both;"></div>
    {% endif %}{% endfor %}
  {% endunless %}{% endfor %}
</div>
</ul>
