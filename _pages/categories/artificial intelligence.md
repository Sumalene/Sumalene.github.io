---
layout: archive
permalink: AI/
title: "数据建模"

author_profile: true
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.AI %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}
