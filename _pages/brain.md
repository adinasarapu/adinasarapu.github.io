---
layout: archive
title: "Human Brain, Movement Disorders & Biomarkers"
permalink: /brain/
author_profile: true

---
To better understand my research, it’s essential to grasp the fundamentals of brain circuits. These intricate networks are responsible for controlling movement and other critical functions. In healthy individuals, these circuits work seamlessly to maintain motor coordination and balance. However, in conditions like Parkinson’s disease (PD) or dystonia, disruptions in these circuits lead to the characteristic movement impairments. To provide context for my work, I’ve compiled some key insights on brain circuits in both healthy states and in the altered conditions seen in PD and dystonia.  

{% include base_path %}

{% assign sorted_posts = site.brain | sort: 'order' %}
{% for post in sorted_posts %}
  {% include archive-single.html %}
{% endfor %}
