---
layout: post
title: List 源码解析之 LinkedList 源码分析
categories: [SourceCode]
description: LinkedList 源码分析
keywords: List, LinkedList, 源码
---
注：文中源码为 JDK 1.8 。

### LinkedList 简介
实现了 List 和 Deque 接口，既可以看作一个顺序容器，又可以看作一个队列（*Queue*），同时又可以看作一个栈（*Stack*）（处理栈和队列问题，首选 ArrayDeque，它的性能比 LinkedList 作栈和队列使用好很多）。
LinkedList 是一种双向链表，通过`first`和`last`引用分别指向链表的第一个和最后一个元素。
LinkedList 是非线程安全的，也就是说它不是同步的，适合单线程环境使用；需要多个线程并发访问，可以先采用`Collections.synchronizedList()`方法对其进行包装。
 LinkedList 实现了 Serializable 接口，因此它支持序列化，能够通过序列化传输，实现了 Cloneable 接口，能被克隆。


### 属性和构造函数

```java
transient int size = 0;    //数量    
transient Node<E> first;   //首节点    
transient Node<E> last;    //尾节点

private static class Node<E> {
  E item;
  Node<E> next;
  Node<E> prev;

  Node(Node<E> prev, E element, Node<E> next) {
    this.item = element;
    this.next = next;
    this.prev = prev;
  }
}


/**
 * Constructs an empty list.
 */
public LinkedList() {
}

/**
 * 构造一个包含指定collection 的元素的列表，这些元素按照
 * 该collection 的迭代器返回它们的顺序排列的
 */
public LinkedList(Collection<? extends E> c) {
  this();
  addAll(c);
}
```

### 存储

#### set(int index, E element)

```java
// 用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素  
public E set(int index, E element) {
  checkElementIndex(index);
  Node<E> x = node(index);
  E oldVal = x.item;
  x.item = element;
  return oldVal;
}
```

####  add(E e)

```java
// 添加指定元素到链表尾部，花费时间为常数时间
public boolean add(E e) {
  linkLast(e);
  return true;
}

/**
 * Links e as last element.
 链接e作为最后一个元素，默认向表尾节点加入新的元素
 */
void linkLast(E e) {
  final Node<E> l = last;
  final Node<E> newNode = new Node<>(l, e, null); // 当插入数据量大时，生成对象比较耗时
  last = newNode;
  if (l == null)
    first = newNode;
  else
    l.next = newNode;
  size++;
  modCount++;
}
```

###  add(int index, E element)

```java
// 在指定位置插入
public void add(int index, E element) {
  checkPositionIndex(index);

  if (index == size)
      linkLast(element);
  else
      linkBefore(element, node(index));
}

// 下标位置越界检查
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
      throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
  * Inserts element e before non-null Node succ.
  */
void linkBefore(E e, Node<E> succ) {
  // assert succ != null;
  final Node<E> pred = succ.prev;
  final Node<E> newNode = new Node<>(pred, e, succ);
  succ.prev = newNode;
  if (pred == null)
    first = newNode;
  else
    pred.next = newNode;
  size++;
  modCount++;
}
```

#### addAll(Collection<? extends E> c)

```java
// 按照指定collection 的迭代器所返回的元素顺序，将该collection 中的所有元素添加到此列表的尾部
public boolean addAll(Collection<? extends E> c) {
  return addAll(size, c);
}

// 具体看源码吧
```

### 读取

#### get(int index)

```java
// 返回指定下标的元素
public E get(int index) {
  checkElementIndex(index);
  return node(index).item;
}
```

### 删除

1.先找到要删除元素的引用，2.修改相关引用，完成删除操作

#### remove(int index)

使用下标计数

```java
// 删除指定位置的元素，并返回
public E remove(int index) {
  checkElementIndex(index); // 检查下标是否越界
  return unlink(node(index));
}

/**
  * Unlinks non-null node x.
  */
E unlink(Node<E> x) {
  // assert x != null;
  final E element = x.item;
  final Node<E> next = x.next;
  final Node<E> prev = x.prev;

  if (prev == null) { //删除的是第一个元素
      first = next;
  } else {
      prev.next = next;
      x.prev = null;
  }

  if (next == null) { //删除的是最后一个元素
      last = prev;
  } else {
      next.prev = prev;
      x.next = null;
  }

  x.item = null; //let GC work
  size--;
  modCount++;
  return element;
}
```

####  remove(Object o)

使用 equales 方法

```java
// 删除指定元素
public boolean remove(Object o) {
  if (o == null) {
    for (Node<E> x = first; x != null; x = x.next) {
      if (x.item == null) {
        unlink(x);
        return true;
      }
    }
  } else {
    for (Node<E> x = first; x != null; x = x.next) {
      if (o.equals(x.item)) {
        unlink(x);
        return true;
      }
    }
  }
  return false;
}
```

增删元素的时候，只需改变指针，不需要像数组那样对整体数据进行移动、复制等消耗性能的操作。

### 遍历

遍历的时候耗时，for 循环(无穷大) > ForEach > 迭代器，优先使用迭代器方式。

```java
for(Iterator<String> it = list.iterator();it.hasNext();){}
```
### 队列操作
#### E peek() 
```java
// 获取第一个元素
public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```
####  E peekFirst() 

```java
// 获取第一个元素，同E peek() 
public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }
```
#### E peekLast() 

  ```java
// 获取最后一个元素
public E peekLast() {
  final Node<E> l = last;
  return (l == null) ? null : l.item;
}
```

#### E pop()

 ```java
// 移除第一个元素并返回
public E pop() {
  return removeFirst();
}
```

#### E pollFirst() 

  ```java
// 移除第一个元素，同E pop()
public E pollFirst() {
  final Node<E> f = first;
  return (f == null) ? null : unlinkFirst(f);
}
 ```

#### E pollLast()

 ```java
// 移除最后一个元素
public E pollLast() {
  final Node<E> l = last;
  return (l == null) ? null : unlinkLast(l);
}
```

#### push(E e)

```java
// 队首添加一个元素
public void push(E e) {
  addFirst(e);
}
```

### 总结

1.  基于双向循环链表实现，不存在容量不足的问题，没有扩容的方法
2. 增删元素快，查找慢
3. 元素排列有序，可重复，可为 null
4. 实现了栈和队列的操作方法，因此也可以作为栈、队列和双端队列来使用。
5. 非同步，线程不安全



### 参考
>- [pzxwhc](https://github.com/pzxwhc/MineKnowContainer/issues/18)
>- [兰亭风雨](http://blog.csdn.net/ns_code/article/details/35787253)
>- [JCFInternals](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/3-LinkedList.md)
