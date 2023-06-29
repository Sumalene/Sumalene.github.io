---
layout: archive
permalink: project/
title: "PROJECT"

author_profile: true
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.Project %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}