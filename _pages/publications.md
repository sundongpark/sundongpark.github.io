---
title: "Publications"
layout: archive
permalink: /publications/
author_profile: true
sidebar_main: true
---

## International Journals

{% assign posts = site.categories["International Journals"] %}
{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
