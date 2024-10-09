---
layout: archive
title: "Human Brain, Movement Disorders & Biomarkers"
permalink: /brain/
author_profile: true

---
To understand my research, it's important to first know the basics of brain circuits—complex networks that control movement and key functions. In healthy brains, these circuits ensure smooth motor coordination. But in Parkinson’s disease (PD) and dystonia, disruptions in these systems lead to movement problems. I've outlined some key insights into how brain circuits work normally and how they malfunction in PD and dystonia.  

{% include base_path %}

{% assign sorted_posts = site.brain | sort: 'order' %}
{% for post in sorted_posts %}
  {% include archive-single.html %}
{% endfor %}
