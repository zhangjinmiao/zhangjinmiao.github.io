---
layout: post
title: Java 8 将 list 转为 map
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, list, map
---
## 引言

创建一个对象类
```java
public class Hosting {

    private int Id;
    private String name;
    private long websites;

    public Hosting(int id, String name, long websites) {
        Id = id;
        this.name = name;
        this.websites = websites;
    }

    //getters, setters and toString()
}

```

## 1.List to Map – Collectors.toMap()
创建 Hosting 对象的列表，并使用 Collectors.toMap 将其转换为 Map。

```java
public static void testOne(){
    List<Hosting> list = new ArrayList<>();
    list.add(new Hosting(1, "liquidweb.com", 80000));
    list.add(new Hosting(2, "linode.com", 90000));
    list.add(new Hosting(3, "digitalocean.com", 120000));
    list.add(new Hosting(4, "aws.amazon.com", 200000));
    list.add(new Hosting(5, "mkyong.com", 1));

    // key = id, value - websites
    Map<Integer, String> result1 = list.stream()
        .collect(Collectors.toMap(Hosting::getId, Hosting::getName));

    System.out.println("result1：" + result1);

    // key = name, value - websites
    Map<String, Long> result2 = list.stream()
        .collect(Collectors.toMap(Hosting::getName, Hosting::getWebsites));

    System.out.println("result2：" + result2);

    // key = id, value = name 另一种写法
    Map<Integer, String> result3 = list.stream()
        .collect(Collectors.toMap(x -> x.getId(), x -> x.getName()));

    System.out.println("result3：" + result3);
  }
```

## 2.List to Map – Duplicated Key
重复的 key 抛出异常。

```java
private static void testTwo() {
    List<Hosting> list = new ArrayList<>();
    list.add(new Hosting(1, "liquidweb.com", 80000));
    list.add(new Hosting(2, "linode.com", 90000));
    list.add(new Hosting(3, "digitalocean.com", 120000));
    list.add(new Hosting(4, "aws.amazon.com", 200000));
    list.add(new Hosting(5, "mkyong.com", 1));

    list.add(new Hosting(6, "linode.com", 100000)); // new line

    Map<String, Long> result1 = list.stream()
        .collect(Collectors.toMap(Hosting::getName, Hosting::getWebsites));

    System.out.println("result1：" + result1);
  }
```

输出——下面的错误消息有点误导人，它应该显示“ linode”而不是键的值。

```java
Exception in thread "main" java.lang.IllegalStateException: Duplicate key 90000
	at java.util.stream.Collectors.lambda$throwingMerger$0(Collectors.java:133)
	at java.util.HashMap.merge(HashMap.java:1245)
	//...
```

要解决上面重复的关键问题，传入第三个 mergeFunction 参数，如下所示:

```java
private static void testTwo() {

    List<Hosting> list = new ArrayList<>();
    list.add(new Hosting(1, "liquidweb.com", 80000));
    list.add(new Hosting(2, "linode.com", 90000));
    list.add(new Hosting(3, "digitalocean.com", 120000));
    list.add(new Hosting(4, "aws.amazon.com", 200000));
    list.add(new Hosting(5, "mkyong.com", 1));

    list.add(new Hosting(6, "linode.com", 100000)); // new line

//    Map<String, Long> result1 = list.stream()
//        .collect(Collectors.toMap(Hosting::getName, Hosting::getWebsites));

    Map<String, Long> result1 = list.stream().collect(
        Collectors.toMap(Hosting::getName, Hosting::getWebsites, (oldValue, newValue) -> oldValue));

    System.out.println("result1：" + result1);
  }
```

输出：
```java
result1：{liquidweb.com=80000, mkyong.com=1, digitalocean.com=120000, aws.amazon.com=200000, linode.com=90000}
```

使用新值：
```java
Map<String, Long> result1 = list.stream().collect(
                Collectors.toMap(Hosting::getName, Hosting::getWebsites,
                        (oldValue, newValue) -> newValue
                )
        );

```

输出：
```java
result1：{liquidweb.com=80000, mkyong.com=1, digitalocean.com=120000, aws.amazon.com=200000, linode.com=100000}
```

## 3.List to Map – Sort & Collect

先排序再收集。
```java
private static void testThree() {
    List<Hosting> list = new ArrayList<>();
    list.add(new Hosting(1, "liquidweb.com", 80000));
    list.add(new Hosting(2, "linode.com", 90000));
    list.add(new Hosting(3, "digitalocean.com", 120000));
    list.add(new Hosting(4, "aws.amazon.com", 200000));
    list.add(new Hosting(5, "mkyong.com", 1));
    list.add(new Hosting(6, "linode.com", 100000));

    // use oldValue
    Map<String, Long> result1 = list.stream().sorted(Comparator.comparingLong(Hosting::getWebsites).reversed())
        .collect(Collectors.toMap(Hosting::getName,Hosting::getWebsites,(oldValue, newValue) -> oldValue,
            LinkedHashMap::new));

    System.out.println("result1：" + result1);
  }
```

输出：
```java
result1：{aws.amazon.com=200000, digitalocean.com=120000, linode.com=100000, liquidweb.com=80000, mkyong.com=1}
```

在上面的例子中，流是在收集之前排序的，所以“ linode. com 100000”变成了“ oldValue”。

>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)