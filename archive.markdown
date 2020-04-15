---
layout: page
permalink: /archive/
title: Archive
---


<div id="archives">

{% for category in site.categories %}
  <div class="archive-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>
    <p></p>

    <h3 class="category-head">#{{ category_name }}</h3>
    <a name="{{ category_name | slugize }}"></a>
    {% for post in site.categories[category_name] %}
    <article class="archive-item">
      {{post.date | date: "%B %-d, %Y" }} <a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a>
    </article>
    {% endfor %}
  </div>
{% endfor %}
</div>