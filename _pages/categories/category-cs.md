---
layout: archive
permalink: cs/
title: "COMPUTER SCIENCE"

author_profile: true
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.CS %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}