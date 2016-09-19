---
layout: archive
author_profile: false
permalink: /faq/
sidebar: false
---

{% include base_path %}
{% assign items = site.posts | sort: 'title' %}
{% for post in items %}
{% if post.category == "faq" %}
  {% include archive-single.html type="grid" %}      
{% endif %}
{% endfor %}
{% include paginator.html %}
