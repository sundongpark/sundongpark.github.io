---
title: "Publications"
layout: archive
permalink: /publications/
author_profile: true
sidebar_main: true
---

## International Conferences
{% for post in site.categories["International Conferences"] %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}

## International Journals
{% for post in site.categories["International Journals"] %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
