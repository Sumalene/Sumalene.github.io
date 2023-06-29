---
layout: archive
permalink: app/
title: "APP"

author_profile: true
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.App %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}