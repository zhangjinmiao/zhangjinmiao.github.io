---
layout: post
title: List 源码分析
categories: Java
description: ArrayList LinkedList Vector 
keywords: List, Java
---





## ArrayList源码分析 

ArrayList 是一个动态扩展的数组。

### 属性和构造函数



```java
private static final int DEFAULT_CAPACITY = 10; // 默认初始值
transient Object[] elementData; // 存放数据的数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}; // 存放的数组默认容量10
protected transient int modCount = 0; // List被修改的次数

// 构造函数
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *  is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) { // 创建对应容量的数组
      this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) { // 空数组
      this.elementData = EMPTY_ELEMENTDATA;
    } else {
      throw new IllegalArgumentException("Illegal Capacity: "+
                                         initialCapacity);
    }
}

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() { // 默认容量10的数组
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
/**
 * 构造一个包含指定collection 的元素的列表，这些元素按照
 * 该collection 的迭代器返回它们的顺序排列的
 */
public ArrayList(Collection<? extends E> c) {
  elementData = c.toArray();
  if ((size = elementData.length) != 0) {
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
      elementData = Arrays.copyOf(elementData, size, Object[].class);
  } else {
    // replace with empty array.
    this.elementData = EMPTY_ELEMENTDATA;
  }
}
```

### 存储

####  set(int index, E element)

```java
// 用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素    
public E set(int index, E element) {
  rangeCheck(index); // 越界检测

  E oldValue = elementData(index);
  elementData[index] = element; // 赋值到指定位置,复制的仅仅是引用
  return oldValue;
}

private void rangeCheck(int index) {
  if (index >= size)
    throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

####  add(E e)

```java
// 添加一个元素到此列表的尾部 时间复杂度O(1)，非常高效
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 容量 + 1 
    elementData[size++] = e;
    return true;
}

// 如有必要，增加此 ArrayList 实例的容量，以确保它至少能够容纳最小容量参数所指定的元素数
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
      minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

  	ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
      grow(minCapacity);
}

// 增加容量，确保可容纳元素
private void grow(int minCapacity) {
    // overflow-conscious code 注意溢出
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); // 扩容到原始容量的1.5倍
    if (newCapacity - minCapacity < 0)
      newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
      newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity); // 扩容并复制
  	// 由于Java GC自动管理了内存，这里也就不需要考虑源数组释放的问题。
  // 关于Java GC这里需要特别说明一下，有了垃圾收集器并不意味着一定不会有内存泄漏。对象能否被GC的依据是是否还有引用指向它，上面代码中如果不手动赋null值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。
}


private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
      throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
      Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}

```



![ArrayList_grow](https://github.com/CarpenterLee/JCFInternals/raw/master/PNGFigures/ArrayList_grow.png)



空间问题解决后，插入就变得容易了

![ArrayList_add](https://github.com/CarpenterLee/JCFInternals/raw/master/PNGFigures/ArrayList_add.png)



####  add(int index, E element)

```java
// 将指定的元素添加到此列表中的指定位置
// 如果当前位置有元素，则向右移动当前位于该位置的元素以及所有后续元素（将其索引加1）
public void add(int index, E element) {
    rangeCheckForAdd(index); // 数组越界检测
	// 如果数组长度不足，将进行扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
  	// 将 elementData 中从Index 位置开始、长度为size-index 的元素，
  	// 拷贝到从下标为index+1 位置开始的新的elementData 数组中。
  	// 即将当前位于该位置的元素以及所有后续元素右移一个位置。
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);  // 低效
    elementData[index] = element;
    size++;
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
      throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```



####  addAll(Collection<? extends E> c)

```java
// 按照指定collection 的迭代器所返回的元素顺序，将该collection 中的所有元素添加到此列表的尾部
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```



#### addAll(int index, Collection<? extends E> c)

```java
// 从指定的位置开始，将指定collection 中的所有元素插入到此列表中
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
      System.arraycopy(elementData, index, elementData, index + numNew,
                       numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```



### 读取

#### get(int index)

```java
// 返回此列表中指定位置上的元素
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}

E elementData(int index) {
  return (E) elementData[index];  //注意类型转换
}
```



### 删除

根据下标和指定对象删除，删除时，被移除元素以后的所有元素向左移动一个位置。

#### remove(int index)

```java
// 删除指定位置的元素，并返回删除元素
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
      System.arraycopy(elementData, index+1, elementData, index,
                       numMoved); // 向前移动
    elementData[--size] = null; // clear to let GC do its work，清除该位置的引用，让GC起作用

    return oldValue;
}
```



#### remove(Object o)

```java
// 移除此列表中首次出现的指定元素（如果存在）。这是应为ArrayList 中允许存放重复的元素
public boolean remove(Object o) {
  // 由于ArrayList 中允许存放null，因此下面通过两种情况来分别处理
    if (o == null) {
      for (int index = 0; index < size; index++)
        if (elementData[index] == null) {
          // 类似remove(int index)，移除列表中指定位置上的元素
          fastRemove(index);
          return true;
        }
    } else {
      for (int index = 0; index < size; index++)
        if (o.equals(elementData[index])) {
          fastRemove(index);
          return true;
        }
    }
    return false;
}

// 私有删除方法，不进行越界检测
private void fastRemove(int index) {
  modCount++;
  int numMoved = size - index - 1;
  if (numMoved > 0)
    System.arraycopy(elementData, index+1, elementData, index,
                     numMoved);
  elementData[--size] = null; // clear to let GC do its work
}
```



### 容量调整

如果添加元素前已经预测到了容量不足，可手动增加ArrayList实例的容量，以减少递增式再分配的数量。

#### ensureCapacity(int minCapacity)

```java
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
      // any size if not default element table
      ? 0
      // larger than default for default empty table. It's already
      // supposed to be at default size.
      : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
      ensureExplicitCapacity(minCapacity);
    }
}

// 具体查看ensureExplicitCapacity
```

在ensureExplicitCapacity中，可以看出，每次扩容时，会将老数组中的元素重新拷贝一份到新的数组中（***System.arraycopy()方法***），每次数组容量的增长大约是其原容量的1.5 倍。这种操作的代价是很高的，因此要尽量避免数组容量扩容，使用时尽可能的指定容量，或者根据需求，通过调用ensureCapacity 方法来手动增加ArrayList 实例的容量。

#### trimToSize()

该方法是将底层数组的容量调整为当前列表保存的实际元素的大小。



```java
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
      elementData = (size == 0)
        ? EMPTY_ELEMENTDATA
        : Arrays.copyOf(elementData, size);
    }
}
```



### 遍历

#### 通过迭代器遍历

> ```java
> Integer value = null;
> Iterator iter = list.iterator();
> while (iter.hasNext()) {
> value = (Integer)iter.next();
> }
> ```
>
> 

#### 随机访问，通过索引值去遍历（推荐）

> ```java
> Integer value = null;
> int size = list.size();
> for (int i=0; i<size; i++) {
> value = (Integer)list.get(i);        
> }
> ```
>
> 

#### ForEach循环遍历

> ```java
> Integer value = null;
> for (Integer integ:list) {
> value = integ;
> }
> ```



经测试，耗时：ForEach循环  > 随机 > 迭代器，优先使用迭代器方式 。

ForEach的本质也是迭代器模式，可以反编译查看，而且还多了一步赋值操作，增加了开销。



### 总结

size(), isEmpty(), get(), set()方法均能在常数时间内完成，add()方法的时间开销跟插入位置有关，addAll()方法的时间开销跟添加元素的个数成正比。其余方法大都是线性时间。

- 排列有序（索引从0开始），可插入空值，可重复

- 底层数组实现

- 读取快，增删慢（System.arraycopy低效）

- 非同步，线程不安全

- 当容量不够时，ArrayList是当前容量 * 1.5 + 1

- 创建时，初始化最小容量

  ​



## Vector 源码分析

### 属性和构造函数

```java
protected Object[] elementData; // 存放数据的数组
protected int elementCount; // 元素个数
protected int capacityIncrement;  // 容量增量

// 构造一个指定容量和增量的Vector
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
      throw new IllegalArgumentException("Illegal Capacity: "+
                                         initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}

// 构造一个指定容量，增量为0的Vector
public Vector(int initialCapacity) {
   this(initialCapacity, 0);
}

// 构造一个容量为10，增量为0的Vector
public Vector() {
  this(10);
}


```

### 存储

#### synchronized boolean add(E e)

多了**synchronized** 修饰。

```java

// 末尾插入元素，注意有synchronized修饰
public synchronized boolean add(E e) {
  modCount++;
  ensureCapacityHelper(elementCount + 1);
  elementData[elementCount++] = e;
  return true;
}

```



#### add(int index, E element)

```java
// 指定位置插入
public void add(int index, E element) {
  insertElementAt(element, index);
}

// 具体实现查看源码...
public synchronized void insertElementAt(E obj, int index) {
  modCount++;
  if (index > elementCount) {
    throw new ArrayIndexOutOfBoundsException(index
                                             + " > " + elementCount);
  }
  ensureCapacityHelper(elementCount + 1);
  System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
  elementData[index] = obj;
  elementCount++;
}
```



#### set(int index, E element)

```java
// 用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素
public synchronized E set(int index, E element) {
    if (index >= elementCount)
      throw new ArrayIndexOutOfBoundsException(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
  }
```



### 读取

#### get(int index)

同ArrayList

```java
// 返回此列表中指定位置上的元素
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```



### 删除

#### remove(int index)

```java
// 删除指定位置元素
public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
      throw new ArrayIndexOutOfBoundsException(index);
    E oldValue = elementData(index);

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
      System.arraycopy(elementData, index+1, elementData, index,
                       numMoved);
    elementData[--elementCount] = null; // Let gc do its work

    return oldValue;
}
```



#### remove(Object o)

```java
// 移除此列表中首次出现的指定元素（如果存在）。
public boolean remove(Object o) {
  return removeElement(o);
}

public synchronized boolean removeElement(Object obj) {
  modCount++;
  int i = indexOf(obj);
  if (i >= 0) {
    removeElementAt(i);
    return true;
  }
  return false;
}

public synchronized void removeElementAt(int index) {
  modCount++;
  if (index >= elementCount) {
    throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                             elementCount);
  }
  else if (index < 0) {
    throw new ArrayIndexOutOfBoundsException(index);
  }
  int j = elementCount - index - 1;
  if (j > 0) {
    System.arraycopy(elementData, index + 1, elementData, index, j);
  }
  elementCount--;
  elementData[elementCount] = null; /* to let gc do its work */
}
```



### 容量调整

和ArrayList类似，多了synchronized修饰。

#### ensureCapacity(int minCapacity)

```java
public synchronized void ensureCapacity(int minCapacity) {
    if (minCapacity > 0) {
      modCount++;
      ensureCapacityHelper(minCapacity);
    }
  }
```

#### trimToSize()

```java
public synchronized void trimToSize() {
  modCount++;
  int oldCapacity = elementData.length;
  if (elementCount < oldCapacity) {
    elementData = Arrays.copyOf(elementData, elementCount);
  }
}
```





## ArrayList和Vector的区别

1. ArrayList是线程不安全的，Vector是线程安全的。
2. Vector的性能比ArrayList差




## LinkedList 源码分析

实现了List和Deque接口，既可以看作一个顺序容器，又可以看作一个队列（*Queue*），同时又可以看作一个栈（*Stack*）。处理栈和队列问题，首选ArrayDeque，它的性能比LinkedList作栈和队列使用好很多。*LinkedList*通过`first`和`last`引用分别指向链表的第一个和最后一个元素，*LinkedList*通过`first`和`last`引用分别指向链表的第一个和最后一个元素。

为追求效率*LinkedList*没有实现同步（synchronized），如果需要多个线程并发访问，可以先采用`Collections.synchronizedList()`方法对其进行包装。

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

使用equales方法

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

遍历的时候耗时，for循环(无穷大) > ForEach > 迭代器，优先使用迭代器方式。

##  ArrayList、Vector和LinkedList的区别

- 实现方式

  >- ArrayList，Vector 是基于数组的实现。
  >
  >
  >- LinkedList 是基于链表的实现。

  ​


- 同步

  >- ArrayList,LinkedList 不是线程安全的。
  >- Vector 是线程安全的，实现方式是在方法中加 synchronized 进行限定。

  ​

- 性能消耗

  > - ArrayList和Vector由于是基于数组实现，所以在指定位置插入和删除时间复杂度为O(n)，还可能出现扩容问题，这比较消耗性能。
  > - LinkedList不会出现扩容问题，适合增删操作；查找元素需要遍历链表，时间复杂度为O(n)。


- 使用场景

  > - 快速插入、删除元素，使用LinkedList
  > - 快速随机访问元素，使用ArrayList
  > - 单线程，使用List，比如ArrayList
  > - 多线程，使用Vector



### 参考

>- [CarpenterLee——List](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/2-ArrayList.md)
>
>
>- [莫等闲](http://zhangshixi.iteye.com/blog/674856)