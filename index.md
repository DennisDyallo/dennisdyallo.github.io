---
layout: default
title: Home
---

[Home](/) | [Blog](/blog) | [About](/about) | [GitHub](https://github.com/dennisdyallo)

# Welcome to My Digital Garden 🌱

I'm a .NET developer working on security and cryptography. Here's where I share my journey, learnings, and projects.

## Latest Posts
{% for post in site.posts limit:5 %}
  * [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%B %d, %Y" }}
{% endfor %}