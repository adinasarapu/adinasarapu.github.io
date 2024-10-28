---
layout: archive
title: "Human Brain, Movement Disorders & Biomarkers"
permalink: /brain/
author_profile: true

---  
Neurological disorders caused by dysfunctions in the basal ganglia, leading to impaired movement and coordination, often with neurodegenerative features.  

For instance, **Parkinson's Disease** and **Huntington's Disease** showcase how basal ganglia dysfunction can lead to severe motor and cognitive challenges. Meanwhile, conditions like **Tourette Syndrome** and **Dystonia** highlight the diverse range of movement disorders that can occur due to these brain circuit issues. This summary helps illustrate the complex connections between these disorders and their effects on our movement and overall brain function.

| **Disorder**                          | **What It Is**                                                                                      | **Movement Disorder?** | **Neurodegenerative Disorder?** |
|---------------------------------------|------------------------------------------------------------------------------------------------------|------------------------|----------------------------------|
| **Parkinson's Disease**               | A condition caused by the loss of dopamine-producing neurons in the brain, leading to tremors, stiffness, and slowed movement. | Yes                    | Yes                              |
| **Huntington's Disease**              | A genetic disorder that causes the breakdown of nerve cells in the brain, resulting in uncontrolled movements and cognitive decline. | Yes                    | Yes                              |
| **Chorea**                            | Characterized by irregular, involuntary movements that can happen in various conditions, including Huntington's Disease. | Yes                    | Yes (if related to HD)          |
| **Progressive Supranuclear Palsy (PSP)** | A neurodegenerative disorder that leads to issues with balance, rigidity, and problems with eye movements. | Yes                    | Yes                              |
| **Dystonia**                          | A movement disorder that causes sustained muscle contractions and abnormal postures, often stemming from basal ganglia issues. | Yes                    | No                               |
| **Tourette Syndrome**                 | A condition marked by repetitive tics, both motor and vocal, linked to dysfunction in the basal ganglia. | Yes                    | No                               |
| **Wilson's Disease**                  | A genetic disorder that causes copper accumulation in the body, leading to various movement issues and cognitive problems. | Yes                    | No                               |
| **Essential Tremor**                  | A common movement disorder that results in rhythmic shaking, primarily in the hands, potentially linked to basal ganglia circuitry. | Yes                    | No                               |

{% include base_path %}
{% assign sorted_posts = site.brain | sort: 'order' %}
{% for post in sorted_posts %}
  {% include archive-single.html %}
{% endfor %}
