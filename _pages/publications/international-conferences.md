---
title: "International Conferences"
layout: archive
permalink: /publications/international-conferences/
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories["International Conferences"] %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
