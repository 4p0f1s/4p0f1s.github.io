---
layout: default
title: HackThisSite
permalink: Blog/HTS/
---

<div class="posts">
  {% for post in site.categories.HTS %}
    <article class="post">

      <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>

      <div class="entry">
        {{ post.excerpt }}
      </div>

      <a href="{{ post.url }}" class="read-more">Read More</a>
    </article>
  {% endfor %}
