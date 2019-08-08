---
layout: post
title: Java8 Stream 已经被操作或关闭
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, stream closed,open
---

## 引言
在 java8 中，Stream 不能被重用，一旦它被使用或使用，流将被关闭。


## 1. 流关闭
查看下面的示例，它将抛出一个 IllegalStateException，表示“ stream is closed”。

````java
public static void main(String[] args) {
    String[] array = {"a", "b", "c", "d", "e"};
    Stream<String> stream = Arrays.stream(array);

    // loop a stream
    stream.forEach(x-> System.out.println(x));

    // 拒绝过滤，抛出 throws IllegalStateException
    long count = stream.filter(x -> "b".equals(x)).count();
    System.out.println(count);
  }
````

输出：

````java
a
b
c
d
e
Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
	at java.util.stream.AbstractPipeline.<init>(AbstractPipeline.java:203)
	at java.util.stream.ReferencePipeline.<init>(ReferencePipeline.java:94)
	at java.util.stream.ReferencePipeline$StatelessOp.<init>(ReferencePipeline.java:618)
	at java.util.stream.ReferencePipeline$2.<init>(ReferencePipeline.java:163)
	at java.util.stream.ReferencePipeline.filter(ReferencePipeline.java:162)
	at com.jimzhang.stream.ClosedTest.main(ClosedTest.java:22)
````

## 2. 重用流
不管出于什么原因，你真的想重用一个数据流，试试下面的 Supplier 解决方案:

````java
public static void main(String[] args) {

    String[] array = {"a", "b", "c", "d", "e"};
    Supplier<Stream<String>> streamSupplier = () -> Stream.of(array);

    //get new stream
    streamSupplier.get().forEach(x-> System.out.println(x));

    //get another new stream

    long count = streamSupplier.get().filter(x -> "b".equals(x)).count();
    System.out.println(count);
  }
````

输出：
````java
a
b
c
d
e
1
````
每个 get ()将返回一个新的流。



>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)

