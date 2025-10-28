# Posts

<ul>
  {% assign visible_posts = site.posts | where: "published", true %}
  {% for post in visible_posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.date | date: "%Y-%m-%d" }} - {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
