---
layout: archive
title: "Human Brain"
permalink: /brain/
author_profile: true
redirect_from:
  - /brain
---

{% include base_path %}

{% for post in site.brain reversed %}
  {% include archive-single.html %}
{% endfor %} 
