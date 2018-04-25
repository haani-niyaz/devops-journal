---
layout: page
title: Articles
permalink: /articles/
order: 2
---

<div class="home">

  <ul class="post-list">
      {% for post in site.posts %}
        {% for tag in post.tags %}
          {% if tag contains 'article' %}
        <li>
          <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

          <h2>
            <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
          </h2>

        </li>
         {% endif %}
        {% endfor %}
      {% endfor %}
  </ul>

  <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p>
  
</div>