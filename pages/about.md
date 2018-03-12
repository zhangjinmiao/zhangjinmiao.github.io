---
layout: page
title: About
description: 若水三千只取一瓢饮
keywords: Zhang jinmiao, jimzhang, 山川尽美, 火炎焱
comments: true
menu: 关于
permalink: /about/
---

我是Jimzhang，Java后端工程师，目前从事支付行业。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
  {% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
