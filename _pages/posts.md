---
title: "Posts"
layout: archive
permalink: /posts/
author_profile: true
sidebar_main: true
---

## Research Notes
{% for post in site.categories["Research Notes"] %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}

## Software Engineering
{% for post in site.categories["Software Engineering"] %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
