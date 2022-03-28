---
layout: default
---

The [Guru Platform](https://www.getguru.fitness/) enables you to effortlessly add movement AI to your app.
This blog is a growing library of articles and discussions about what is possible with the platform today,
and how we're thinking about growing it in the future. Dive in!

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
