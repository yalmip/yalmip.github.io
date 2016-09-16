---
layout: archive
author_profile: false
permalink: /references/
---

{% include base_path %}

<h3 class="archive__subtitle">References</h3>

{% assign items = site.references | sort: 'author' %}
{% for post in items %}
  {% include archive-single.html type="grid" %}
{% endfor %}

{% include paginator.html %}
