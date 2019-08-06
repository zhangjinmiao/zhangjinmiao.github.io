---
layout: post
title: Java8 Streams Collectors 使用
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, streams Collectors,group by, count, sum
---
## 引言
在本文中，我们将向您展示如何使用 java8 流的 Collectors 对列表进行分组、计数、求和和排序。

## 1. 分组、计数和排序

1. 按列表分组并显示列表的总数。

````java
 List<String> items = Arrays.asList("apple", "apple", "banana",
        "apple", "orange", "banana", "papaya");

Map<String, Long> result = items.stream()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

    System.out.println(result);
````

输出：

````java
{
	papaya=1, orange=1, banana=2, apple=3
}
````

2. 添加排序

````java
List<String> items = Arrays.asList("apple", "apple", "banana",
        "apple", "orange", "banana", "papaya");

Map<String, Long> result = items.stream()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

    Map<String, Long> finalMap = new LinkedHashMap<>();

    result.entrySet().stream()
        .sorted(Entry.<String,Long>comparingByValue().reversed()).forEachOrdered(e->finalMap.put(e.getKey(),e.getValue()));

    System.out.println(finalMap);
````

输出：

````java
{
	apple=3, banana=2, papaya=1, orange=1
}

````

## 2.列出对象

按用户定义的对象列表进行“分组”的示例。

1. 按名称分组，并统计数量或求和。

````java
public class Item {

    private String name;
    private int qty;
    private BigDecimal price;

    //constructors, getter/setters 
}
````

````java
List<Item> items = Arrays.asList(
        new Item("apple", 10, new BigDecimal("9.99")),
        new Item("banana", 20, new BigDecimal("19.99")),
        new Item("orang", 10, new BigDecimal("29.99")),
        new Item("watermelon", 10, new BigDecimal("29.99")),
        new Item("papaya", 20, new BigDecimal("9.99")),
        new Item("apple", 10, new BigDecimal("9.99")),
        new Item("banana", 10, new BigDecimal("19.99")),
        new Item("apple", 20, new BigDecimal("9.99"))
    );

    Map<String, Long> couting = items.stream()
        .collect(Collectors.groupingBy(Item::getName, Collectors.counting()));

    System.out.println(couting);

    System.out.println("======");

    Map<String, Integer> sum = items.stream()
        .collect(Collectors.groupingBy(Item::getName, Collectors.summingInt(Item::getQty)));

    System.out.println(sum);
````

输出：

````java
{papaya=1, banana=2, apple=3, orang=1, watermelon=1}
======
{papaya=20, banana=30, apple=40, orang=10, watermelon=10}
````

2. 按价格分组，Collectors.groupingBy and Collectors.mapping 的使用：

````java
 List<Item> items = Arrays.asList(
        new Item("apple", 10, new BigDecimal("9.99")),
        new Item("banana", 20, new BigDecimal("19.99")),
        new Item("orang", 10, new BigDecimal("29.99")),
        new Item("watermelon", 10, new BigDecimal("29.99")),
        new Item("papaya", 20, new BigDecimal("9.99")),
        new Item("apple", 10, new BigDecimal("9.99")),
        new Item("banana", 10, new BigDecimal("19.99")),
        new Item("apple", 20, new BigDecimal("9.99"))
    );

System.out.println("=====>group by price:");
    // group by price
    Map<BigDecimal, List<Item>> groupByPriceMap = items.stream()
        .collect(Collectors.groupingBy(Item::getPrice));

    System.out.println(groupByPriceMap);

    Map<BigDecimal, Set<String>> collect = items.stream().collect(Collectors
        .groupingBy(Item::getPrice, Collectors.mapping(Item::getName, Collectors.toSet())));

    System.out.println("=====>group by + mapping to Set:");
    System.out.println(collect);

````

输出：

````java
=====>group by price:
{   19.99=[
        Item{name='banana', qty=20, price=19.99}, 
        Item{name='banana', qty=10, price=19.99}
        ], 
    29.99=[
        Item{name='orang', qty=10, price=29.99}, 
        Item{name='watermelon', qty=10, price=29.99}
        ], 
    9.99=[
        Item{name='apple', qty=10, price=9.99}, 
        Item{name='papaya', qty=20, price=9.99}, 
        Item{name='apple', qty=10, price=9.99}, 
        Item{name='apple', qty=20, price=9.99}
        ]
}
=====>group by + mapping to Set:
{   19.99=[banana], 
    29.99=[orang, watermelon], 
    9.99=[papaya, apple]
}
````


>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)