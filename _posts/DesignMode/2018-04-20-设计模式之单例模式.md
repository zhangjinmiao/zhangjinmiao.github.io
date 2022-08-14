---
layout: post
title: 设计模式之单例模式
categories: DesignMode
description: 单例模式
keywords: designMode, singleton
---

## 单例模式:

- 定义：确保一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。


- 类型：创建类模式


- 类图：

![](https://github.com/zhangjinmiao/zhangjinmiao.github.io/raw/master/assets/images/2018/designmode/singleton.jpg)

**类图知识点：**

1. 类图分为三部分，依次是类名、属性、方法
2. 以<<开头和以>>结尾的为注释信息
3. 修饰符+代表public，-代表private，#代表protected，什么都没有代表包可见。
4. 带下划线的属性或方法代表是静态的。



单例模式应该是23种设计模式中最简单的一种模式了。它有以下几个要素：

- 私有的构造方法


- 指向自己实例的私有静态引用


- 以自己实例为返回值的静态的公有的方法

单例模式根据实例化对象时机的不同分为两种：一种是饿汉式单例，一种是懒汉式单例。饿汉式单例在单例类被
加载时候，就实例化一个对象交给自己的引用；而懒汉式在调用取得实例方法的时候才会实例化对象。



## 单例模式的优点：

- 在内存中只有一个对象，节省内存空间。


- 避免频繁的创建销毁对象，可以提高性能。


- 避免对共享资源的多重占用。


- 可以全局访问。



## 单例模式注意事项

- 只能使用单例类提供的方法得到单例对象，不要使用反射，否则将会实例化一个新对象。


- 不要做断开单例类对象与类中静态引用的危险操作。-

-  多线程使用单例使用共享资源时，注意线程安全问题。

  

## 适用场景

- 需要频繁实例化然后销毁的对象。


- 创建对象时耗时过多或者耗资源过多，但又经常用到的对象。


- 有状态的工具类对象。


- 频繁访问数据库或文件的对象。


- 以及其他我没用过的所有要求只有一个对象的场景。



## 代码实现

### 饿汉模式

````java
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton() {
    }
    public static Singleton getInstance(){
        return instance;
    }
}
````



### 懒汉模式

````java
public class Singleton {
	private static Singleton singleton;
	private Singleton(){}
	public static synchronized Singleton getInstance(){
		if(singleton==null){
			singleton = new Singleton();
		}
		return singleton;
	}
}
````

懒汉模式也不是安全的。



###  静态内部类

也是一种懒汉模式。

````java
/**
 * @author jimzhang
 * <>静态内部类</>
 * @version V1.0.0
 * @date 2018-04-08 11:01
 */
public class Singleton {
    private static class SingletonHolder{
        private static final Singleton INSTANCE = new Singleton();
    }
    private Singleton() {
    }
    public static final Singleton getInstance(){
        return SingletonHolder.INSTANCE;
    }
}
````



### 双重校验锁

````java
/**
 * @author jimzhang
 * <>双重校验锁</>
 * @version V1.0.0
 * @date 2018-04-08 11:23
 */
public class Singleton {
    private volatile static Singleton instance;
    private Singleton(){}
    public static Singleton getInstance(){
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
````



### 枚举

````java
/**
 * @author: jimzhang
 * <>枚举</> 推荐
 * @date: 2018-04-08 11:11
 * @version: V1.0.0
 */
public enum Singleton {
    INSTANCE;
    public void whateverMethod() {
    }
}
````



### 使用 CAS 实现单例模式

````java
/**
 * @author jimzhang
 * <>使用 CAS 实现单例</>
 * 非阻塞：
 * 用CAS的好处在于不需要使用传统的锁机制来保证线程安全,CAS是一种基于忙等待的算法,依赖底层硬件的实现,相对于锁它没有线程切换和阻塞的额外消耗,可以支持较大的并行度。
 * CAS 的一个重要缺点在于如果忙等待一直执行不成功(一直在死循环中),会对CPU造成较大的执行开销。
 * @version V1.0.0
 * @date 2018-04-08 11:29
 */
public class Singleton {
    private static final AtomicReference<Singleton> INSTANCE = new AtomicReference<>();
    private Singleton(){}
    public static Singleton getInstance(){
        for (;;) {
            Singleton singleton = INSTANCE.get();
            if (null != singleton) {
                return singleton;
            }
            singleton = new Singleton();
            if (INSTANCE.compareAndSet(null,singleton)) {
                return singleton;
            }
        }
    }
}
````

AtomicReference：原子引用

赋值操作不是线程安全的。若想不用锁来实现，可以用 AtomicReference<V> 这个类，实现对象引用的原子更新。

**使用场景：** 一个线程使用 student 对象，另一个线程负责定时读表，更新这个对象。那么就可以用AtomicReference 这个类。







###  

###  





具体案例请见github。