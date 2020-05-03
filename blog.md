# Introduction to the blog

The main reason why i write this technical blog is to better understand the topic. So the main reason is me :-).  
But if it helps to you, feel free to use the information in this blog anyhow you wish. Some of the blogs are written in English (sort of) and some in Slovak language.

{% for post in site.posts %}
{{ post.title }}
{% endfor %}
{% for post in site.posts | sort post.date %}
{{ post.content }} {% endfor %}
