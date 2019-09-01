---
layout: post
title: Java8 如何将 Array 转换为 Stream
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, array convert stream
---

## 引言

在 java8 中，您可以使用 Arrays.Stream 或 Stream.of 将 Array 转换为 Stream。

## 1. 对象数组

对于对象数组，Arrays.stream 和 Stream.of 都返回相同的输出。

```java
public static void main(String[] args) {

    ObjectArrays();
  }

  private static void ObjectArrays() {
    String[] array = {"a", "b", "c", "d", "e"};
    //Arrays.stream
    Stream<String> stream = Arrays.stream(array);
    stream.forEach(x-> System.out.println(x));

    System.out.println("======");

    //Stream.of
    Stream<String> stream1 = Stream.of(array);
    stream1.forEach(x-> System.out.println(x));
  }
```
输出：

```java
a
b
c
d
e
======
a
b
c
d
e
```

查看 JDK 源码，对于对象数组，`Stream.of` 内部调用了 `Arrays.stream` 方法。

```java
// Arrays
public static <T> Stream<T> stream(T[] array) {
    return stream(array, 0, array.length);
}

// Stream
public static<T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}
```

## 2. 基本数组 

对于基本数组，Arrays.stream 和 Stream.of 将返回不同的输出。

```java
public static void main(String[] args) {

    PrimitiveArrays();
  }

private static void PrimitiveArrays() {
    int[] intArray = {1, 2, 3, 4, 5};

    // 1. Arrays.stream -> IntStream
    IntStream stream = Arrays.stream(intArray);
    stream.forEach(x->System.out.println(x));

    System.out.println("======");

    // 2. Stream.of -> Stream<int[]>
    Stream<int[]> temp = Stream.of(intArray);

    // 不能直接输出，需要先转换为 IntStream
    IntStream intStream = temp.flatMapToInt(x -> Arrays.stream(x));
    intStream.forEach(x-> System.out.println(x));

  }
```
输出：

```java
1
2
3
4
5
======
1
2
3
4
5
```

查看源码，

```java
// Arrays
public static IntStream stream(int[] array) {
    return stream(array, 0, array.length);
}

// Stream
public static<T> Stream<T> of(T t) {
    return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
```

### Which one
- 对于对象数组，两者都调用相同的 `Arrays.stream` 方法
- 对于基本数组，我更喜欢 `Arrays.stream`，因为它返回固定的大小 `IntStream`，更容易操作。

所以，推荐使用 `Arrays.stream`，不需要考虑是对象数组还是基本数组，直接返回对应的流对象，操作方便。


>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)