---
layout: default
title: Artificial-Intelligence
permalink: /ai/
---

<h1>Artificial Intelligence</h1>

{% for category in site.categories %}
{% if category[0] == "ai" %}
<ul class="arc-list">
    {% for post in category.last %}
        {{ post.date | date:"%d/%m/%Y"}}
	<h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    {% endfor %}
</ul>
{% endif %}
{% endfor %}
