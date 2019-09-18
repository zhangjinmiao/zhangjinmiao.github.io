---
layout: post
title: Java 8 flatMap 示例
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, flatMap
---
## 引言
在 java8 中，Stream 可以保存不同的数据类型，例如:

```java
Stream<String[]>	
Stream<Set<String>>	
Stream<List<String>>	
Stream<List<Object>>
```
但是，流操作(filter、 sum、 distinct...)和收集器不支持它，因此，我们需要 flatMap ()来执行以下转换:

```java
Stream<String[]>		-> flatMap ->	Stream<String>
Stream<Set<String>>	-> flatMap ->	Stream<String>
Stream<List<String>>	-> flatMap ->	Stream<String>
Stream<List<Object>>	-> flatMap ->	Stream<Object>
```

Flatmap ()是如何工作的:
```shell
{ {1,2}, {3,4}, {5,6} } -> flatMap -> {1,2,3,4,5,6}

```

## 1.Stream + String [] + flatMap
下面的示例将打印一个空结果，因为 filter ()不知道如何过滤 String []流。

```java
private static void one() {
 

    //Stream<String[]>
    Stream<String[]> temp = Arrays.stream(data);

    //filter a stream of string[], and return a string[]?
    Stream<String[]> stream = temp.filter(x -> "a".equals(x.toString()));
    stream.forEach(System.out::println);
  }
```

在上面的例子中，我们应该使用 flatMap ()将流字符串[]转换为流字符串。

```java
private static void one() {

    //Stream<String[]>
    Stream<String[]> temp = Arrays.stream(data);

    //filter a stream of string[], and return a string[]?
//    Stream<String[]> stream = temp.filter(x -> "a".equals(x.toString()));
//    stream.forEach(System.out::println);


    // Stream<String>, GOOD!
    Stream<String> stringStream = temp.flatMap(x -> Arrays.stream(x));
    Stream<String> stream1 = stringStream.filter(x -> "a".equals(x.toString()));
    stream1.forEach(System.out::println);
  }
```

输出：
```java
a
```

## 2.Stream + Set + flatMap

```java
public class Student {

    private String name;
    private Set<String> book;

    public void addBook(String book) {
        if (this.book == null) {
            this.book = new HashSet<>();
        }
        this.book.add(book);
    }
    //getters and setters

}

```

```java

private static void two() {
    Student obj1 = new Student();
    obj1.setName("mkyong");
    obj1.addBook("Java 8 in Action");
    obj1.addBook("Spring Boot in Action");
    obj1.addBook("Effective Java (2nd Edition)");

    Student obj2 = new Student();
    obj2.setName("zilap");
    obj2.addBook("Learning Python, 5th Edition");
    obj2.addBook("Effective Java (2nd Edition)");

    List<Student> list = new ArrayList<>();
    list.add(obj1);
    list.add(obj2);

    List<String> collect = list.stream().map(x -> x.getBook()) //Stream<Set<String>>
        .flatMap(x -> x.stream()) //Stream<String>
        .distinct()
        .collect(Collectors.toList());
    collect.forEach(x-> System.out.println(x));
  }
```


输出：
```java
Spring Boot in Action
Effective Java (2nd Edition)
Java 8 in Action
Learning Python, 5th Edition
```
尝试注释 flatMap (x-x.stream ()) ，Collectors.toList ()将提示一个编译器错误，因为它不知道如何收集 Set 对象流。

## 3.Stream + Primitive + flatMapToInt
对于基本类型，可以使用 flatMapToInt。

```java
private static void three() {
    int[] intArray = {1, 2, 3, 4, 5, 6};

    //1. Stream<int[]>
    Stream<int[]> streamArray = Stream.of(intArray);

    //2. Stream<int[]> -> flatMap -> IntStream
    IntStream intStream = streamArray.flatMapToInt(x -> Arrays.stream(x));

    intStream.forEach(x -> System.out.println(x));
  }
```

输出：
```java
1
2
3
4
5
6
```

>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)