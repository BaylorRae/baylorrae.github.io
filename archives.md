---
layout: page
title: the archives
permalink: /archives
---

{% for post in site.posts %}
{% if forloop.first %}
  <h2>{{ post.date | date: '%Y %B' }}</h2>
{% else %}
{% capture year %}{{ post.date | date: '%Y %B' }}{% endcapture %}
{% capture nyear %}{{ post.next.date | date: '%Y %B' }}{% endcapture %}
{% if year != nyear %}
  <h2>{{ post.date | date: '%Y %B' }}</h2>
{% endif %}
{% endif %}

  <article>
    <h3>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </h3>
  </article>
{% endfor %}
