---
layout: default
title: Quant
permalink: /quant/
---

<h1>Quant</h1>

{% for category in site.categories %}
{% if category[0] == "quant" %}
<ul class="arc-list">
    {% for post in category.last %}
        {{ post.date | date:"%d/%m/%Y"}}
	<h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    {% endfor %}
</ul>
{% endif %}
{% endfor %}
