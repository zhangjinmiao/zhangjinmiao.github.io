---
layout: post
title: Java8 Streams filter 使用
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, forEach, streams filter
---
## 引言

在本教程中，我们将向您展示几个 java8 示例，以演示 Streams filter ()、 collect ()、 findAny ()和 orElse ()的使用。

### 什么是流
Stream（流）是一个来自数据源的元素队列并支持聚合操作

- **元素** 是特定类型的对象，形成一个队列。 Java 中的 Stream 并不会存储元素，而是按需计算。
- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器 generator 等。
- **聚合操作** 类似 SQL 语句一样的操作， 比如 filter, map, reduce, find, match, sorted 等。

和以前的 Collection 操作不同， Stream 操作还有两个基础的特征：

- **Pipelining:** 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- **内部迭代：** 以前对集合遍历都是通过 Iterator 或者 For-Each 的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream 提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

### 生成流
在 Java 8 中, 集合接口有两个方法来生成流：

- stream() − 为集合创建串行流。

- parallelStream() − 为集合创建并行流。

## 1. Streams filter() and collect()
1.java 8 之前过滤 list 使用如下方式：

````java
List<String> lines = Arrays.asList("spring", "node", "php");

    List<String> result = getFilterOutput(lines, "php");
    for (String temp : result) {
      System.out.println(temp);    //output : spring, node
    }
  }

  private static List<String> getFilterOutput(List<String> lines, String filter) {
    List<String> result = new ArrayList<>();
    for (String line : lines) {
      if (!"php".equals(line)) { // we dont like php
        result.add(line);
      }
    }
    return result;
  }
````

2.java 8 之后，使用 stream.filter() 过滤 list，使用 collect() 将流转为 list.

````java 
List<String> lines = Arrays.asList("spring", "node", "php");

    List<String> result = lines.stream()          // convert list to stream
        .filter(line -> !"php".equals(line))      // we dont like php
        .collect(Collectors.toList());            // collect the output and convert streams to a List

    result.forEach(System.out::println);          //output : spring, node
````


## 2. Streams filter(), findAny() and orElse()
java8 之前，获取 name 通过如下方式：

````java
public static void main(String[] args) {
    List<Developer> persons = Arrays.asList(
        new Developer("zhangsan", 20),
        new Developer("lisi",21),
        new Developer("wangwu",22));

    Developer developer = getByNameBefore(persons,"lisi");
    System.out.println(developer == null ? "不存在" : developer.toString());
}

private static Developer getByNameBefore(List<Developer> persons, String lisi) {
    Developer result = null;
    for (Developer developer : persons) {
      if (developer.getName().equals(lisi)) {
        result = developer;
      }
    }

    return result;
  }
````

java8 之后：

````java
public static void main(String[] args) {
    List<Developer> persons = Arrays.asList(
        new Developer("zhangsan", 20),
        new Developer("lisi",21),
        new Developer("wangwu",22));

    Developer developer = persons.stream().filter(x -> "lisi".equals(x.getName())).findAny()
        .orElse(null);
    System.out.println(developer);

    // 不存在的
    Developer developer1 = persons.stream().filter(x -> "asan".equals(x.getName())).findAny()
        .orElse(null);
    System.out.println(developer1);
}
````
多条件查询使用：

````java
public static void main(String[] args) {
    List<Developer> persons = Arrays.asList(
        new Developer("zhangsan", 20),
        new Developer("lisi",21),
        new Developer("wangwu",22));

    Developer result1 = persons.stream()
        .filter(p -> "lisi".equals(p.getName()) && p.getAge() == 21).findAny().orElse(null);

    System.out.println(result1);

    Developer result2 = persons.stream().filter(p -> {
      if ("lisi".equals(p.getName()) && p.getAge() == 21) {
        return true;
      }
      return false;
    }).findAny().orElse(null);

    System.out.println(result2);

  }
````
## 3. Streams filter() and map()

````java
public static void main(String[] args) {
    List<Developer> persons = Arrays.asList(
        new Developer("zhangsan", 20),
        new Developer("lisi",21),
        new Developer("wangwu",22));

    String name = persons.stream().filter(x -> "lisi".equals(x.getName()))
        .map(Developer::getName)     //convert stream to String
        .findAny().orElse("");

    System.out.println("name : " + name);

    List<String> collect = persons.stream().filter(x -> "lisi".equals(x.getName()))
        .map(Developer::getName).collect(Collectors.toList());

    collect.forEach(System.out::println);
  }
````

