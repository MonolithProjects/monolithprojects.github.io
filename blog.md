# Blog

{% for post in site.posts %}
{{ post.title }}
{% endfor %}
{% for post in site.posts | sort post.date %}
{{ post.content }} {% endfor %}
