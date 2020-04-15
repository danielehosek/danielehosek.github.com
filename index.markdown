---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
---

<h2>Latest Posts</h2>

<div id="posts">

{% for post in site.posts limit:10 %}
   <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
   {% if post.content.size > 2000 %}
      <div>
      {{ post.excerpt }} <!-- bad! content gives you rendered html and you will truncate in the middle of a node -->
      </div>
      <h4><a href="{{ post.url }}">read more</a></h4>
   {% else %}
      {{ post.content }}
   {% endif %}
   <hr>
{% endfor %}
</div>
    
