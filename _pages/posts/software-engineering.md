---
title: "Software Engineering"
layout: archive
permalink: /posts/software-engineering/
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories["Software Engineering"] %}
{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
