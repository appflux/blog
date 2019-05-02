---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

{% for post in site.posts %}
  <div>
    <h2>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h2>
  </div>
  <div>
    {{ post.excerpt }}
    <br>
    <i style="margin-left: 80%; font-size: 13px;">
      <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
    </i>
  </div>
{% endfor %}
