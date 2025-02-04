---
layout: default
title: Home
---

# Welcome to My Digital Garden 🌱

I'm a .NET developer working on security and cryptography. Here's where I share my journey, learnings, and projects.

## Latest Posts
{% raw %}
{% for post in site.posts limit:5 %}
  * [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%B %d, %Y" }}
{% endfor %}
{% endraw %}