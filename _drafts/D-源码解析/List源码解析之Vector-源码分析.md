### Vector简介
和ArrayList一样，Vector也是基于数组实现的，是动态数组，容量可自动增长。
与ArrayList不同的是，它有好多方法都加入了`synchronized`修饰，所以是线程安全的，可用于多线程环境。
Vector没有实现Serializable接口，不支持序列化，实现了Cloneable接口，能被克隆，实现了RandomAccess接口，支持快速随机访问。


### 属性和构造函数
```java
    protected Object[] elementData; // 存放数据的数组
    protected int elementCount; // 元素个数
    protected int capacityIncrement;  // 容量增量
    
    // 构造一个指定容量和增量的Vector
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
          throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
    
    // 构造一个指定容量，增量为 0 的Vector
    public Vector(int initialCapacity) {
       this(initialCapacity, 0);
    }
    
    // 构造一个容量为10，增量为 0 的Vector
    public Vector() {
      this(10);
    }
```

### 存储

#### synchronized boolean add(E e)

多了synchronized 修饰。
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
```java
ensureCapacity(int minCapacity)

    public synchronized void ensureCapacity(int minCapacity) {
        if (minCapacity > 0) {
          modCount++;
          ensureCapacityHelper(minCapacity);
        }
      }

trimToSize()

    public synchronized void trimToSize() {
      modCount++;
      int oldCapacity = elementData.length;
      if (elementCount < oldCapacity) {
        elementData = Arrays.copyOf(elementData, elementCount);
      }
    }
```

### 总结
- 默认初始大小10，扩容也是使用copy原数组的方式
- 很多方法都加入了`synchronized`同步语句，来保证线程安全
- 允许元素为null，也可以插入重复值
- 性能消耗也主要来源于 扩容
- 与ArrayList实现大同小异，基本不用


### 参考

>- [pzxwhc](https://github.com/pzxwhc/MineKnowContainer/issues/18)
>- [兰亭风雨](http://blog.csdn.net/ns_code/article/details/35793865)

======华丽丽的分隔线======
作者：jimzhang
出处：http://www.jianshu.com/p/03d79f4bf42e
版权所有，欢迎保留原文链接进行转载：)
