---
layout: post
title: Map源码解析之HashMap源码分析
categories: 源码分析, 集合
description: HashMap 源码分析
keywords: Map, HashMap, 源码
---

## 实现原理

HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

HashMap是数组+链表+红黑树（**JDK1.8增加了红黑树部分**）实现的。

**HashMap的工作原理：**

HashMap基于hashing原理，当我们往 HashMap 中 put 元素时，先根据 key 的 hash 值得到这个 Entry 元素在数组中的位置（即下标），然后把这个 Entry 元素放到对应的位置中，如果这个 Entry 元素所在的位子上已经存放有其他元素就在同一个位子上的 Entry 元素以链表的形式存放，新加入的放在链头( ***JDK 1.8 以前碰撞节点会在链表头部插入，而 JDK 1.8 开始碰撞节点会在链表尾部插入，对于扩容操作后的节点转移 JDK 1.8 以前转移前后链表顺序会倒置，而 JDK 1.8 中依然保持原序***。)，从 HashMap 中 get  Entry 元素时先计算 key 的 hashcode，找到数组中对应位置的某一 Entry 元素，然后通过 key 的 equals 方法在对应位置的链表中找到需要的 Entry 元素，所以 HashMap 的数据结构是数组和链表的结合，此外 HashMap 中 key 和 value 都允许为 null，key 为 null 的键值对永远都放在以 table[0] 为头结点的链表中。。 HashMap在每个链表节点中储存键值对对象。

当两个不同的键对象的hashcode相同时会发生什么？ 它们会储存在同一个bucket位置的链表中。键对象的equals()方法用来找到键值对。

作者：潜龙勿用

链接：https://www.zhihu.com/question/20733617/answer/259163516

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

![hashmap结构](E:\github\zhangjinmiao.github.io\assets\images\blog\hashmap结构.jpg)


Node[] table的初始化长度length(默认值是16)，Load factor为负载因子(默认值是0.75)，threshold是HashMap所能容纳的最大数据量的Node(键值对)个数。

threshold就是在此Load factor和length(数组长度)对应下允许的最大元素数目，超过这个数目就重新resize(扩容)，扩容后的HashMap容量是之前容量的**两倍**。

在HashMap中，**哈希桶数组table的长度length大小必须为2的n次方**(一定是合数)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数[2].

HashMap采用这种非常规设计，主要是**为了在取模和扩容时做优化**，同时为了减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。

当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能。

## 属性和构造函数

>- **初始容量**：表示哈希表在其容量自动增加之前可以达到多满的一种尺度
>- **加载因子**：当哈希表中的条目超过了容量和加载因子的乘积的时候，就会进行重哈希操作。

```java
//======常量==========
// 默认初始化容量16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认负载因子 加载因子过高，容易产生哈希冲突，加载因子过小，容易浪费空间，0.75是一种折中。
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// ======属性=======
transient Node<K,V>[] table; // 哈希桶数组
//
transient Set<Map.Entry<K,V>> entrySet;
// 大小
transient int size;
// 用来记录HashMap内部结构发生变化的次数
transient int modCount;
// 所能容纳的key-value对极限  (int)(capacity * load factor)
int threshold;
//哈希表的装载因子 
final float loadFactor;


```

Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash; //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next; //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

### 确定哈希桶数组索引位置——hash(Object key)

本质上就三部：**取key的hashCode值、高位运算、取模运算**。

```java
方法一：
static final int hash(Object key) {   //jdk1.8 & jdk1.7
    int h;
    // h = key.hashCode() 为第一步 取hashCode值
    // h ^ (h >>> 16)  为第二步 高位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

方法二：
static int indexFor(int h, int length) {  
//jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
    return h & (length-1);  //第三步 取模运算
}

```

只要它的hashCode()返回值相同，那么程序调用方法一所计算得到的Hash码值总是相同的。



###  put(K key, V value)

>JDK1.7
>
>当我们往HashMap 中put 元素的时候，先根据key 的hashCode 重新计算hash 值，根据hash 值得到这个元素在数组中的位置（即下标），如果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放，**新加入的放在链头**，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。
>
>

当我们往HashMap 中put 元素的时候，先根据key 的
hashCode 重新计算hash 值，根据hash 值得到这个元素在数组中的位置（即下标），如
果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放，新
加入的放在链头，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到

```java
// 添加一个元素
public V put(K key, V value) {
  // 对key的hashCode()做hash
  return putVal(hash(key), key, value, false, true);
}
// 内部实现
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
   		// 步骤①：tab为空则创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
  		// 步骤②：计算index，并对null做处理
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
          // 步骤③：节点key存在，直接覆盖value
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
          // 步骤④：判断该链为红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
           // 步骤⑤：该链为链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                      // 链表长度大于8转换为红黑树进行处理
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                  // key已经存在直接覆盖value
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
   // 步骤⑥：超过最大容量 就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

 ![hashmap之put](E:\github\zhangjinmiao.github.io\assets\images\blog\hashmap之put.jpg)


### 读取 get(Object key)

```java
// 根据key获取value
public V get(Object key) {
  Node<K,V> e;
  return (e = getNode(hash(key), key)) == null ? null : e.value;
}
// 内部方法
/**
 * Implements Map.get and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @return the node, or null if none
 */
final Node<K,V> getNode(int hash, Object key) {
  Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
  // table有元素
  if ((tab = table) != null && (n = tab.length) > 0 &&
      (first = tab[(n - 1) & hash]) != null) {
    // 从第1个node开始
    if (first.hash == hash && // always check first node
        ((k = first.key) == key || (key != null && key.equals(k))))
      return first;
    // first的下一个node
    if ((e = first.next) != null) {
      // 若是链表
      if (first instanceof TreeNode)
        return ((TreeNode<K,V>)first).getTreeNode(hash, key);
      //  否则循环查找
      do {
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
          return e;
      } while ((e = e.next) != null);
    }
    // 就一个node，first，取的还不是它，直接退出
  }
  // table没元素直接返回null
  return null;
}
```

### 移除元素remove(Object key)

```java
// 根据key移除元素，并返回移除的元素
public V remove(Object key) {
  Node<K,V> e;
  return (e = removeNode(hash(key), key, null, false, true)) == null ?
    null : e.value;
}
```

### 扩容机制resize

当元素个数 > 数组大小 （默认16）* 负载因子（默认0.75f）时开始扩容，即默认情况下元素个数大于12个时，数组大小扩为2 * 16 = 32，即扩大一倍，然后重新计算每个元素在数组中的位置，，而这是一个非常消耗性能的操作，所以如果我们已经预知HashMap 中元素的个数，那么预设元素的个数能够有效的提高HashMap 的性能。

**使用一个新的数组代替已有的容量小的数组**

### 线程安全性

高并发情况下会出现**死循环**，即会出现 **环形链表**。

### 总结

1. 扩容是一个特别耗性能的操作，所以当程序员在使用HashMap的时候，估算map的大小，初始化的时候给一个大致的数值，避免map进行频繁的扩容。
2. 负载因子是可以修改的，也可以大于1，但是建议不要轻易修改，除非情况非常特殊。
3. HashMap是线程不安全的，不要在并发的环境中同时操作HashMap，建议使用ConcurrentHashMap。
4. JDK1.8引入红黑树大程度优化了HashMap的性能
5. 使用可变对象做Key会造成value找不到的情况，所以使用`String`，`Integer`等不可变类型用作Key。



参考：

- [美团点评技术团队 Java 8系列之重新认识HashMap ](https://zhuanlan.zhihu.com/p/21673805)

- [疫苗：Java HashMap的死循环](https://coolshell.cn/articles/9606.html)

  ​
