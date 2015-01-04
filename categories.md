---
layout: page
title: 目录
---

<div>
{% for cat in site.categories %}
<a href="#{{ cat[0] }}" title="{{ cat[0] }}" rel="{{ cat[1].size }}">{{ cat[0] }} ({{ cat[1].size }})</a>
{% endfor %}
</div>

{% for cat in site.categories %}
  <h3 id="{{ cat[0] }}">{{ cat[0] }}</h3>
  <ul>
	{% for post in cat[1] %}
	  <li>
	  	<time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
	  	<a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
	  </li>
	{% endfor %}
   </ul>
{% endfor %}

