---
layout: post
title: Java8 Streams map 使用
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, streams map
---
## 引言
在 Java 8 中，stream (). Map ()允许您将一个对象转换为其他对象。查看下面例子：

## 1. 将 List 中的字符串转为大写

````java
public static void main(String[] args) {
    List<String> alpha = Arrays.asList("a", "b", "c", "d");

    //Before Java8
    List<String> alphaUpper = new ArrayList<>();
    for (String s : alpha) {
      alphaUpper.add(s.toUpperCase());
    }

    System.out.println(alpha); //[a, b, c, d]
    System.out.println(alphaUpper); //[A, B, C, D]

    // Java 8
    List<String> collect = alpha.stream().map(String::toUpperCase).collect(Collectors.toList());
    System.out.println(collect); //[A, B, C, D]

    // Extra, streams apply to any data type.
    List<Integer> num = Arrays.asList(1,2,3,4,5);
    List<Integer> collect1 = num.stream().map(n -> n * 2).collect(Collectors.toList());
    System.out.println(collect1); //[2, 4, 6, 8, 10]
  }
````

## 2. 将 List 中的对象转为字符串


````java
public class Developer {


  private String name;

  private BigDecimal salary;

  private Integer age;

  //...

}
````


````java
public static void main(String[] args) {
    List<Developer> persons = Arrays.asList(
        new Developer("zhangsan", 20),
        new Developer("lisi",21),
        new Developer("wangwu",22));

    //Before Java 8
    List<String> result = new ArrayList<>();
    for (Developer developer : persons) {
      result.add(developer.getName());
    }

    System.out.println(result); // [zhangsan, lisi, wangwu]

    //Java 8
    List<String> collect = persons.stream().map(x -> x.getName()).collect(Collectors.toList());
    System.out.println(collect); // [zhangsan, lisi, wangwu]
  }
````


## 3. 将 List 中的对象转为另一个对象

````java
public class Person {

  private String name;
  private int age;
  private String extra;

  //...

}
````

1. Java 8 之前：

````java
List<Developer> developers = Arrays.asList(
        new Developer("zhangsan", 20),
        new Developer("lisi",21),
        new Developer("wangwu",22));

List<Person> result = new ArrayList<>();

    for (Developer developer : developers) {
      Person person = new Person();
      person.setName(developer.getName());
      person.setAge(developer.getAge());

      if ("lisi".equals(developer.getName())) {
        person.setExtra("i am lisi");
      }

      result.add(person);
    }

    System.out.println(JSONUtil.toJsonStr(result));
  }
````

2. java 8

````java
List<Developer> developers = Arrays.asList(
        new Developer("zhangsan", 20),
        new Developer("lisi",21),
        new Developer("wangwu",22));

List<Person> result = developers.stream().map(temp -> {
      Person person = new Person();
      person.setName(temp.getName());
      person.setAge(temp.getAge());

      if ("lisi".equals(temp.getName())) {
        person.setExtra("i am lisi");
      }
      return person;
    }).collect(Collectors.toList());

    System.out.println(JSONUtil.toJsonStr(result));        
````


>源码见：[java-8-demo](https://github.com/zhangjinmiao/java-8-demo)

系列文章详见：[Java 8 教程](http://zhangjinmiao.github.io/java8/2019/07/27/Java-8-Tutorials.html)
