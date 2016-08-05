---
layout: page
title: Tags
permalink: /tags/
---

<div class="home">
 
  {% for tag in site.tags %}
 <h5 class="page-heading">{{tag | first }}</h5>
    <ul>
    {% for posts in tag %}
      {% for post in posts %}
      {% if post.url %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endif %}
      {% endfor %}
    {% endfor %}
    </ul>
  {% endfor %}

  <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p>
  
</div>