---
layout: post
title: Java8 forEach 使用
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, forEach
---

## 引言

在本文中，我们将向您展示如何使用新的 `java 8 foreach` 语句循环 `List` 和 `Map`。

## 1. forEach and Map

1. 普通方式遍历 Map

````java
	Map<String, Integer> items = new HashMap<>();
	items.put("A", 10);
	items.put("B", 20);
	items.put("C", 30);
	items.put("D", 40);
	items.put("E", 50);
	items.put("F", 60);

	for (Map.Entry<String, Integer> entry : items.entrySet()) {
		System.out.println("Item : " + entry.getKey() + " Count : " + entry.getValue());
	}
````
2. 
在 java8 中，可以使用 forEach + lambda 表达式循环 Map。

````java
	Map<String, Integer> items = new HashMap<>();
	items.put("A", 10);
	items.put("B", 20);
	items.put("C", 30);
	items.put("D", 40);
	items.put("E", 50);
	items.put("F", 60);
	
	items.forEach((k,v)->System.out.println("Item : " + k + " Count : " + v));
	
	items.forEach((k,v)->{
		System.out.println("Item : " + k + " Count : " + v);
		if("E".equals(k)){
			System.out.println("Hello E");
		}
	});
````


## 2. forEach and List

1. 普通方式遍历 List

````java
List<String> items = new ArrayList<>();
	items.add("A");
	items.add("B");
	items.add("C");
	items.add("D");
	items.add("E");

	for(String item : items){
		System.out.println(item);
	}
````

2.
在 java8 中，可以使用 forEach + lambda 表达式或方法引用循环 List。

````java
	List<String> items = new ArrayList<>();
	items.add("A");
	items.add("B");
	items.add("C");
	items.add("D");
	items.add("E");

	//lambda
	//Output : A,B,C,D,E
	items.forEach(item->System.out.println(item));
		
	//Output : C
	items.forEach(item->{
		if("C".equals(item)){
			System.out.println(item);
		}
	});
		
	//method reference
	//Output : A,B,C,D,E
	items.forEach(System.out::println);
	
	//Stream and filter
	//Output : B
	items.stream()
		.filter(s->s.contains("B"))
		.forEach(System.out::println);
````

>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)