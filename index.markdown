---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
---


<div id="posts">

{% for post in site.posts limit:10 %}
   <header class="post-header">
      <h2 class="post-title p-name">{{ post.title }}</h2>
      <p class="post-meta">
         <time class="dt-published">{{post.date | date: "%B %-d, %Y" }}
         </time>
      </p>
   </header>
   {% if post.content.size > 2000 %}
      <div>
      {{ post.excerpt }} <!-- bad! content gives you rendered html and you will truncate in the middle of a node -->
      </div>
      <h4><a href="{{ post.url }}">>> read more</a></h4>
   {% else %}
      {{ post.content }}
   {% endif %}
{% endfor %}
</div>
    
