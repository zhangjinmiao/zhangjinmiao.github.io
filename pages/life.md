---
layout: page
title: 我的生活故事
titlebar: life
subtitle: <span class="mega-octicon octicon-organization"></span>&nbsp;&nbsp; 说出你的故事..
description: 说出你的故事
keywords: 生活,故事
comments: true
menu: life
css: ['pages/blog-page.css']
permalink: /life
---

<div class="row">

    <div class="col-md-12">
    
        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='life' %}
                <li class="posts-list-item">
                    <div class="posts-content">
                        <span class="posts-list-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
                        <a class="posts-list-name bubble-float-left" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                        <span class='circle'></span>
                    </div>
                </li>
                {% endif %}
            {% endfor %}
        </ul> 
    
        <!-- Pagination -->
        {% include pagination.html %}
    
        <!-- Comments -->
       <div class="comment">
         {% include comments.html %}
       </div>
    </div>

</div>
<script>
    $(document).ready(function(){

        // Enable bootstrap tooltip
        $("body").tooltip({ selector: '[data-toggle=tooltip]' });
    
    });
</script>