---
layout: archive
author_profile: false
permalink: /references/
---

{% include base_path %}

{% assign items = site.references | sort: 'author' %}
{% for post in items %}
  {% include archive-single.html type="grid" %}
{% endfor %}

{% include paginator.html %}
