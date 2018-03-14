### ArrayList简介 

ArrayList是基于数组实现的， 是一个动态扩展的数组，容量可自动增长。
ArrayList是非线程安全的，只能在单线程环境下使用，多线程环境考虑使用Collections.synchronizedList(List<T> list)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的CopyOnWriteArrayList类。
ArrayList实现了Serializable接口，因此它支持序列化，能够通过序列化传输，实现了RandomAccess接口，支持快速随机访问，实际上就是通过下标序号进行快速访问，实现了Cloneable接口，能被克隆。

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

![ArrayList_grow.png](http://upload-images.jianshu.io/upload_images/626005-e889785a58c7bb8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


空间问题解决后，插入就变得容易了


![ArrayList_add.png](http://upload-images.jianshu.io/upload_images/626005-e54fe7ea07474382.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



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
    rangeCheck(index)![ArrayList_add.png](http://upload-images.jianshu.io/upload_images/626005-4973ec5f1213fd0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
;

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
>     value = (Integer)iter.next();
> }
> ```
>
> 

#### 随机访问，通过索引值去遍历（推荐）

> ```java
> Integer value = null;
> int size = list.size();
> for (int i=0; i<size; i++) {
>     value = (Integer)list.get(i);        
> }
> ```
>
> 

#### ForEach循环遍历

> ```java
> Integer value = null;
> for (Integer integ:list) {
>     value = integ;
> }
> ```



经测试，耗时：ForEach循环  > 随机 > 迭代器，优先使用迭代器方式 。

ForEach的本质也是迭代器模式，可以反编译查看，而且还多了一步赋值操作，增加了开销。



### 总结

size(), isEmpty(), get(), set()方法均能在常数时间内完成，add()方法的时间开销跟插入位置有关，addAll()方法的时间开销跟添加元素的个数成正比。其余方法大都是线性时间。

- 排列有序（索引从0开始），可插入空值，可重复

- 底层数组实现

- 读取快，增删慢（需要移动元素，插入删除效率低）

- 非同步，线程不安全

- 初始容量默认为10，当容量不够时，ArrayList是当前容量 * 1.5 + 1
扩容时，需要调用System.arraycopy，copy本来就是一个耗时的操作，所以尽量初始化容量。
即使理论上效率还可以
（System.arraycopy()方法是一个native的，最终调用了C语言的memmove()函数，比一般的复制方法的实现效率要高很多。）

- 创建时，初始化最小容量


  ​
### 参考

>- [CarpenterLee](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/2-ArrayList.md)
>- [莫等闲](http://zhangshixi.iteye.com/blog/674856)
>- [兰亭风雨](http://blog.csdn.net/mmc_maodun)

======华丽丽的分隔线======
作者：jimzhang
出处：http://www.jianshu.com/p/2f282e5ce305
版权所有，欢迎保留原文链接进行转载：)

