---
layout: post
title: Java 8 Lambda 表达式比较器使用
categories: Java8
description: Java 8 教程系列解读
keywords: Java 8, 系列, lambda, sort
---

在这个例子中，我们将向您展示如何使用 `java8 lambda` 表达式编写一个 `Comparator` 来对 `List` 进行排序。

1. 经典的比较器示例：
````java
	Comparator<Developer> byName = new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o1.getName().compareTo(o2.getName());
		}
	};
````

2. 使用 lambda：
````java
	Comparator<Developer> byName = 
		(Developer o1, Developer o2)->o1.getName().compareTo(o2.getName());

````

## 1.没有 Lambda 的排序

先新建一个 Developer 类，然后比较 Developer 对象的年龄，通常我们使用 Collections.sort 并传递匿名 Comparator 类，如下所示:

````java
package com.jimzhang.lambda;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

/**
 * 〈一句话功能简述〉<br> 〈排序〉
 *
 * @author zhangjinmiao
 * @create 2019/8/4 10:19
 */
public class TestSorting {

  public static void main(String[] args) {
    List<Developer> listDevs = getDevelopers();

    System.out.println("Before Sort");
    for (Developer developer : listDevs) {
      System.out.println(developer.toString());
    }

    //sort by age
    Collections.sort(listDevs, new Comparator<Developer>() {
      @Override
      public int compare(Developer o1, Developer o2) {
        return o1.getAge() - o2.getAge();
      }
    });

    System.out.println("After Sort");
    for (Developer developer : listDevs) {
      System.out.println(developer);
    }

  }

  public static List<Developer> getDevelopers() {
    List<Developer> developers = new ArrayList<>();
    developers.add(new Developer("lisi", new BigDecimal("8000"),23));
    developers.add(new Developer("wangwu", new BigDecimal("9000"),24));
    developers.add(new Developer("maliu", new BigDecimal("10000"),25));
    developers.add(new Developer("zhangsan", new BigDecimal("7000"),22));
    return developers;
  }
}
````

当排序要求改变时，您只需传入另一个新的匿名 Comparator 类:
````java
//sort by age
	Collections.sort(listDevs, new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o1.getAge() - o2.getAge();
		}
	});
	
	//sort by name	
	Collections.sort(listDevs, new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o1.getName().compareTo(o2.getName());
		}
	});
				
	//sort by salary
	Collections.sort(listDevs, new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o1.getSalary().compareTo(o2.getSalary());
		}
	});		
````
这是可行的，但是，您是否觉得仅仅因为想要更改一行代码就创建一个类有点奇怪？


## 2.使用 Lambda 排序
在 Java 8 中，List 接口直接支持排序方法，不再需要使用 Collections.sort。

````java
//List.sort() since Java 8
	listDevs.sort(new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o2.getAge() - o1.getAge();
		}
	});	
````

## 3.更多例子
1. 按年龄排序
````java
    //sort by age
	Collections.sort(listDevs, new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o1.getAge() - o2.getAge();
		}
	});
	
	//lambda
	listDevs.sort((Developer o1, Developer o2)->o1.getAge()-o2.getAge());
	
	//lambda, valid, parameter type is optional
	listDevs.sort((o1, o2)->o1.getAge()-o2.getAge());
````
2. 按名字排序
````java
	//sort by name
	Collections.sort(listDevs, new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o1.getName().compareTo(o2.getName());
		}
	});
		
	//lambda
	listDevs.sort((Developer o1, Developer o2)->o1.getName().compareTo(o2.getName()));		
	
	//lambda
	listDevs.sort((o1, o2)->o1.getName().compareTo(o2.getName()));		

````
3. 按薪水排序
````java
	//sort by salary
	Collections.sort(listDevs, new Comparator<Developer>() {
		@Override
		public int compare(Developer o1, Developer o2) {
			return o1.getSalary().compareTo(o2.getSalary());
		}
	});				

	//lambda
	listDevs.sort((Developer o1, Developer o2)->o1.getSalary().compareTo(o2.getSalary()));
	
	//lambda
	listDevs.sort((o1, o2)->o1.getSalary().compareTo(o2.getSalary()));

````
4. 反向排序
1. 薪水正序排序
````java
Comparator<Developer> salaryComparator = (o1, o2)->o1.getSalary().compareTo(o2.getSalary());
	listDevs.sort(salaryComparator);
````
输出：
````java
Developer{name='zhangsan', salary=7000, age=22}
Developer{name='lisi', salary=8000, age=23}
Developer{name='wangwu', salary=9000, age=24}
Developer{name='maliu', salary=10000, age=25}
````
2.反向排序
````java
Comparator<Developer> salaryComparator = (o1, o2)->o1.getSalary().compareTo(o2.getSalary());
	listDevs.sort(salaryComparator.reversed());
````
输出：
````java
Developer{name='maliu', salary=10000, age=25}
Developer{name='wangwu', salary=9000, age=24}
Developer{name='lisi', salary=8000, age=23}
Developer{name='zhangsan', salary=7000, age=22}
````


