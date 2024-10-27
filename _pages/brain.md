---
layout: archive
title: "Human Brain, Movement Disorders & Biomarkers"
permalink: /brain/
author_profile: true

---
To understand movement disorders, it’s essential to start with the basics of brain circuits-complex networks responsible for controlling movement and other key functions. In a healthy brain, these circuits ensure smooth motor coordination. However, in Parkinson’s disease (PD) and dystonia, disruptions in these systems lead to movement issues.  
{% include base_path %}
{% assign sorted_posts = site.brain | sort: 'order' %}
{% for post in sorted_posts %}
  {% include archive-single.html %}
{% endfor %}
