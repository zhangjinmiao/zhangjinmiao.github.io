---
layout: post
title: Java8 Streams Collectors 使用
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, streams Collectors,group by, count, sum
---

## 引言

本文展示如何过滤一个空值的流对象。

1. 检查包含空值的流。

````java
public static void main(String[] args) {

    Stream<String> language = Stream.of("java", "python", "node", null, "ruby", null, "php");

    List<String> result = language.collect(Collectors.toList());

    result.forEach(System.out::println);

    }
````

输出：

````java
java
python
node
null   // <--- NULL
ruby
null   // <--- NULL
php
````

2. 使用 Stream.filter (x-x! null) 

````java
public static void main(String[] args) {

    Stream<String> language = Stream.of("java", "python", "node", null, "ruby", null, "php");

    //List<String> result = language.collect(Collectors.toList());

    List<String> result = language.filter(x -> x!=null).collect(Collectors.toList());

    // 或使用 Objects: : nonNull 进行筛选
    List<String> result = language.filter(Objects::nonNull).collect(Collectors.toList());

        result.forEach(System.out::println);
    }
````

输出：

````java
java
python
node
ruby
php
````

>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)