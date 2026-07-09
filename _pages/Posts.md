---
title: "Posts"
layout: archive
permalink: /posts/
author_profile: true
sidebar_main: true
---

{% assign posts = site.posts %}
{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
