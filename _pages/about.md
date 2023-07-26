---
permalink: /about/
title: "About"
layout: single
---

{% capture fig_img %}
![Logo Image]({{ "/assets/images/bio-photo.jpeg" | relative_url }})
{% endcapture %}

<figure style="width: 240px" class="align-center">
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Photo from Unsplash.</figcaption>
</figure>

A Developer and entrepreneur

Like to write about what I think and feel
