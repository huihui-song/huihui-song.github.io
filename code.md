---
layout: default
title: Code
permalink: /code/
---

{% for category in site.categories %}
{% if category[0] == "code" %}
<ul class="arc-list">
    {% for post in category.last %}
        {{ post.date | date:"%d/%m/%Y"}}
	<h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    {% endfor %}
</ul>
{% endif %}
{% endfor %}

