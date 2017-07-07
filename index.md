---
---
# Blog Home

### This is the home page.

[Git]({{ site.github.url }}/git)

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
