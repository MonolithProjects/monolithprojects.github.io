# Introduction to the blog

The main reason why i write this technical blog is to better understand the topic or evetnually to make a note of how i resolved some problem. So the main reason is me :-)
But if it helps to you too, feel free to use the information in this blog anyhow you wish. Some of the blogs are written in English (sort of) and some in Slovak language (dosť často mixujem slovenčinu s anglickými výrazmi... because i am used to it).

List of the blogs:

<!-- {% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    {{ post.excerpt | strip_html | truncate: 156 }}
  </li>
{% endfor %} -->

{% for category in site.categories %}
  <h3>{{ category[0] }}</h3>
  <ul>
    {% for post in category[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {{ post.excerpt | strip_html | truncate: 156 }}
    {% endfor %}
  </ul>
{% endfor %}