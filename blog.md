# Introduction to the blog

The main reason why i write this technical blog is to better understand the topic or evetnually to make a note of how i resolved some problem. So the main reason is me :-)
But if it helps to you too, feel free to use the information in this blog anyhow you wish. Some of the blogs are written in English (sort of) and some in Slovak language (dosť často mixujem slovenčinu s anglickými výrazmi... because i am used to it).

Zoznam blogov:

{% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% endfor %}

---
{% for post in site.posts | sort post.date %}
{{ post.content }} {% endfor %}
