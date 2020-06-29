---
title: "Posts by Year"
permalink: /year-archive/
layout: posts
author_profile: true
---

<ul>
	{% for post in site.posts %}
		<li>
			<a href="{{ post.url  }}">{{ post.title  }}</a>
		</li>
	{% endfor %}
</ul>
