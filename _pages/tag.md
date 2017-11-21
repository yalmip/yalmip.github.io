---
layout: archive
permalink: /tags/
title: "Tag Index"
excerpt: "An archive of posts sorted by tag."
share: false
---

{{ page.excerpt | markdownify }}

<ul class="tag__list">

{% capture site_tags %}{% for tag in site.tags %}{{ tag | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
{% assign tag_words = site_tags | split:',' | sort %}
<div>
  {% for item in (0..site.tags.size) %}{% unless forloop.last %}
    {% capture this_word %}{{ tag_words[item] }}{% endcapture %}
    <h2 id="{{ this_word | cgi_escape | downcase | replace: '+', '-' }}">{{ this_word | downcase}}</h2>
    {% for post in site.tags[this_word] %}{% if post.title != null %}
      <div>
        <span style="float: left;">
        {% if post.category != null %}
          <a href="{{ post.url }}">{{ post.title }}  ({{post.category}})</a>
        {% endif %}
         {% if post.category == null %}
          <a href="{{ post.url }}">{{ post.title }}  (article)</a>
        {% endif %}
        </span>   
      </div>
      <div style="clear: both;"></div>
    {% endif %}{% endfor %}
  {% endunless %}{% endfor %}
</div>
</ul>
