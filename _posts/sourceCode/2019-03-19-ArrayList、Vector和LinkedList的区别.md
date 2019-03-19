---
layout: post
title: ArrayList、Vector、LinkedList 的区别
categories: [SourceCode]
description:  
keywords: List, Vector, 源码
---

注：文中源码为 JDK 1.8 。

### 实现方式

  >- ArrayList，Vector 是基于数组的实现。
  >
  >
  >- LinkedList 是基于链表的实现。
  ​

### 同步

  >- ArrayList,LinkedList 不是线程安全的。
  >- Vector 是线程安全的，实现方式是在方法中加 synchronized 进行限定。

### 性能消耗

  > - ArrayList 和 Vector 由于是基于数组实现，所以在指定位置插入和删除时间复杂度为 O(n)，还可能出现扩容问题，这比较消耗性能。
  > - LinkedList 不会出现扩容问题，适合增删操作；查找元素需要遍历链表，时间复杂度为 O(n)。

### 使用场景

  > - 快速插入、删除元素，使用 LinkedList
  > - 快速随机访问元素，使用 ArrayList
  > - 单线程，使用 List，比如 ArrayList
  > - 多线程，使用 Vector
