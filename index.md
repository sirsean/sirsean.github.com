---
layout: page
---
{% include JB/setup %}

{% for node in site.posts limit:5 %}
<h2><a href="{{BASE_PATH}}{{node.url}}">{{node.title}}</a> <small>{{node.date | date_to_string}}</small></h2>
{{node.content}}
<hr />
{% endfor %}

