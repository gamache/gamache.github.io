---
layout: default
is_index_page: true
---
<h1 class="site-title">{{ site.title }}</h1>

<div class="index">
  {% for post in site.posts %}
  <p class="entry">
    <span class="date">{{ post.date | date: "%Y-%m" }}</span>
    <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
  </p>
  {% endfor %}
</div>
