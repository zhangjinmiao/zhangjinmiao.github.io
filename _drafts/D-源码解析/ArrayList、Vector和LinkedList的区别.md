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

  > - ArrayList和Vector由于是基于数组实现，所以在指定位置插入和删除时间复杂度为O(n)，还可能出现扩容问题，这比较消耗性能。
  > - LinkedList不会出现扩容问题，适合增删操作；查找元素需要遍历链表，时间复杂度为O(n)。

### 使用场景

  > - 快速插入、删除元素，使用LinkedList
  > - 快速随机访问元素，使用ArrayList
  > - 单线程，使用List，比如ArrayList
  > - 多线程，使用Vector
