---
layout: page
title: Quantitation Blog
tagline: Math, CS, Data
---
{% include JB/setup %}


Hi, I'm [David](http://stat.yale.edu/~wdb22). This blog is about the "three pillars of quantitation": mathematics, computer science, and data analysis. They make for a powerful combination. I strive to continually develop my expertise in all three.

The posts listed below include tutorials, solved math problems, practical advice, and other reflections.


## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

