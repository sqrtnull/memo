---
layout: home
author: sqrtnull
---

自分の考察などのメモ。**嘘がいっぱい書いてある可能性大。**

<a>
    <object height="32em" data="{{'assets/sqrtnull_logo.svg'|relative_url}}"></object>
</a>

{% assign post = site.memo | group_by: 'category' %} {% for cat in post %}

<h2> {{ cat.name }} </h2>
  {% assign items = cat.items | sort: 'title' %} {% for item in items %}
 - [**{{item.title}}**]({{ item.url | relative_url }}) {% endfor %}
{% endfor %}