---
layout: page
title: Tutorials
permalink: /tutorials/
order: 1
---

 <div class="home">
  
  <p>I find it immensely helpful to learn stuff by writing about it. I research, learn enough and write a tutorial about it. It's here so can I reference it anytime.</p>

  <ul class="post-list">

      {% for post in site.posts %}
        {% for tag in post.tags %}
          {% if tag contains 'tutorial' %}
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