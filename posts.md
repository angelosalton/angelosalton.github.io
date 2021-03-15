---
layout: page
title: Posts
hero_height: is-small
---

{% for post in site.posts %}
<div class="box">
  <a href="{{ post.url }}">
    <h1 class="title">{{ post.title }}</h1>
  </a>
  <p>{{ post.date | date_to_long_string }}</p>
  <p>{{ post.excerpt }}</p>
</div>
{% endfor %}