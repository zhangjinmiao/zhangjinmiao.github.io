---
layout: page
title: About
description: 若水三千只取一瓢饮
keywords: Zhang jinmiao, jimzhang, 山川尽美, 火炎焱
comments: true
menu: 关于
permalink: /about/
---

## 关于博客

一直以来，都想整个博客把自己工作和学习中遇到的问题以知识的形式记录下来，经朋友介绍使用了该博客模板，简介大方，基于强大的 `github` 部署，无需备案（其实有独立域名更好，有时间再整）。博客内容包括自己原创和转载网上优秀文章（当然我会注明出处，但若因此造成侵权行为，烦请原作者 **告知**，我会一并删除。

## 关于我自己

我是Jimzhang，Java后端工程师，目前从事支付行业。

## 联系方式

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
  {% endfor %}

## 技能标签

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
