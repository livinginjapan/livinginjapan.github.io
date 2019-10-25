---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default 
---


<ul >
{% for post in site.posts   %}
  <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {{post.excerpt}} 
  <a href="{{ post.url }}">Read more...</a><br><br>
{% endfor %}
</ul>
