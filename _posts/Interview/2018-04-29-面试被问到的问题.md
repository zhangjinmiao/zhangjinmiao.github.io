---
layout: post
title: 面试被问到的问题
categories: Interview
description: 将面试过程中遇到的问题记录下来
keywords: Interview, 面试
---



## 基本概念

1. java 接口的修饰符：

   - 接口的方法默认是 public abstract
   - 接口的属性默认是 public static final 常量，且必须赋初值

   **分析：**

   1. 接口用于描述系统对外提供的所有服务,因此接口中的成员常量和方法都必须是公开(public)类型的,确保外部使用者能访问它们；

   2. 接口仅仅描述系统能做什么,但不指明如何去做,所以接口中的方法都是抽象(abstract)方法

   3. 接口不涉及和任何具体实例相关的细节,因此接口没有构造方法,不能被实例化,没有实例变量，只有静态（static）变量

   4. 接口的中的变量是所有实现类共有的，既然共有，肯定是不变的东西，因为变化的东西也不能够算共有。所以变量是不可变(final)类型，也就是常量了

      ​

2. ````java
   System.out.println("5" + 2)
   ````

   输出：52

   任何和字符串进行+运算的结果都相当于字符串的连接。

   ​

3. IO 流

   Java的IO操作中有面向 `字节(Byte)` 和面向 `字符(Character)` 两种方式。

   - 面向字节的操作为以8位为单位对二进制的数据进行操作，对数据不进行转换，这些类都是`InputStream` 和`OutputStream` 的子类。


- 面向字符的操作为以字符为单位对数据进行操作，在读的时候将二进制数据转为字符，在写的时候将字符转为二进制数据，这些类都是 `Reader` 和 `Writer` 的子类。

  ​

   **总结：**以InputStream（输入）/OutputStream（输出）为后缀的是字节流；
   以Reader（输入）/Writer（输出）为后缀的是字符流。

   ![](https://github.com/zhangjinmiao/zhangjinmiao.github.io/raw/master/assets/images/2018/interview/io.jpg)

   ​

1. ArrayList 和 LinkedList 区别

   1. ArrayList是实现了基于`动态数组`的数据结构，LinkedList基于`链表`的数据结构。
   2. 对于随机访问 get 和 set，ArrayList 优于LinkedList，因为LinkedList 要移动指针。
   3. 对于新增和删除操作 add 和 remove，LinedList 比较占优势，因为 ArrayList 要移动数据。

   ​

2. volatile 和 synchronized 的区别

   - volatile 轻量级，只能修饰变量。synchronized 重量级，还可修饰方法


- volatile 只能保证数据的`可见性`，不能用来同步，因为多个线程并发访问volatile修饰的变量不会阻塞。
- 仅仅使用 volatile 并`不能保证线程安全性`。而 synchronized 则`可实现线程的安全性`。

   ​

1. 异常

   Java标准库内建了一些通用的异常，这些类以`Throwable` 为顶层父类。

   `Throwable` 又派生出` Error` 类和` Exception` 类。

   - 错误：`Error` 类以及他的子类的实例，代表了JVM本身的错误。错误不能被程序员通过代码处理，Error 很少出现。因此，程序员应该关注Exception为父类的分支下的各种异常类。
   - 异常：`Exception` 以及他的子类，代表程序运行时发送的各种不期望发生的事件。可以被Java异常处理机制使用，是异常处理的核心。

   ![](https://github.com/zhangjinmiao/zhangjinmiao.github.io/raw/master/assets/images/2018/interview/throwable.jpg)

   ​

   总体上根据Javac对异常的处理要求，将异常类分为2类。

   - **非检查异常**（unckecked exception）：Error 和 RuntimeException 以及他们的子类。javac 在编译时，不会提示和发现这样的异常，不要求在程序处理这些异常。所以如果愿意，我们可以编写代码处理（使用try…catch…finally）这样的异常，也可以不处理。

     > 对于这些异常，我们应该修正代码，而不是去通过异常处理器处理 。这样的异常发生的原因多半是代码写的有问题。如除0错误ArithmeticException，错误的强制类型转换错误ClassCastException，数组索引越界ArrayIndexOutOfBoundsException，使用了空对象NullPointerException等等。

   - **检查异常**（checked exception）：除了Error 和 RuntimeException的其它异常。javac强制要求程序员为这样的异常做预备处理工作（使用try…catch…finally或者throws）。在方法中要么用try-catch语句捕获它并处理，要么用throws子句声明抛出它，否则编译不会通过。

     > 这样的异常一般是由程序的运行环境导致的。因为程序可能被运行在各种未知的环境下，而程序员无法干预用户如何使用他编写的程序，于是程序员就应该为这样的异常时刻准备着。如SQLException , IOException,ClassNotFoundException 等。

     ![](https://github.com/zhangjinmiao/zhangjinmiao.github.io/raw/master/assets/images/2018/interview/e1.jpg)

     ![](https://github.com/zhangjinmiao/zhangjinmiao.github.io/raw/master/assets/images/2018/interview/e2.jpg)

   参考：http://www.importnew.com/26613.html

   ​

2. AIO 和 NIO 有什么区别

   ​

3. 继承、重载、重写

   ````java
   public class A {
       public void whoAmI(){
           System.out.println("I am A");
       }
   }

   public class B extends A {
       // 重写父类方法
       public void whoAmI() {
           System.out.println("I am B");
       }
   }

   public class TestMain {
       public static void main(String[] args) {
           A a = new B();
           test(a);

       }
       public static void test(A a){
           System.out.println("test A");
           a.whoAmI();
       }
       public static void test(B b){
           System.out.println("test B");
           b.whoAmI();
       }
    /*
     * test() 方法共有2个属于重载。
     */
   }
   ````

   输出:

   test A
   I am B

   ​

   **改造1：**

   将 test(A a)方法注释掉，test(a) 编译报错：a 对象不能应用于 test(B b)方法上

   **改造2：**

   将 main 方法改为如下：

   ````java
   public static void main(String[] args) {
           B a = new B();
           test(a);
   }
   ````

   输出：

   test B
   I am B

   ​

   **改造3：**

   在 2 的基础上，将test(B b) 注释掉

   输出：

   test A
   I am B

   由此可知：方法重载时，子类对象找不到精确匹配时，会找匹配父类的方法

    

   **关于继承、重载和重写的概念：**

   >**继承：**继承的作用在于代码的复用。由于继承意味着父类的所有方法亦可在子类中使用，所以发给父类的消息亦可发给衍生类。
   >
   >**方法重载(Overloading):**
   >
   >　　　 Java允许在一个类中，多个方法拥有相同的名字，但在名字相同的同时，必须有不同的参数，这就是重载。
   >
   >　　　　编译时编译器会根据实际情况挑选出正确的方法，如果编译器找不到匹配的参数或者找出多个可能的匹配就会产生编译时错误，这个过程被称为重载的解析　　　　
   >
   >　　　　**重载规则:**
   >
   >　　　　　　（一）再使用方法重载的时候，必须通过方法中不同的参数列表来实现方法的重载。如：方法的参数个数不同或者方法的参数类型不同。
   >
   >　　　　　　（二）不能通过访问权限，返回值类型和抛出的异常来实现重载
   >
   >　　　　　　（三）方法的异常类型和抛出异常的数目不会影响方法的重载，也就是说重载的方法中允许抛出不同的异常
   >
   >　　　　　　（四）可以有不同的返回值类型，只要方法的参数列表不同即可
   >
   >　　　　　　（五）可以有不同的访问修饰符
   >
   >**方法重写(Overiding)：**
   >
   >　　　　Java程序中类的继承特性可以产生一个子类，子类继承父类就拥有了父类的非私有的属性（方法和变量），在子类中可以增加自己的属性（方法和变量），同时也可以对父类中的方法进行扩展，以增强自己的功能，这样就称之为重写，也称为复写或者覆盖。
   >
   >　　　　**重写规则:**　　　　
   >
   >　　　　　　在进行方法重写的时候需要遵循以下规则才能实现方法重写：
   >
   >　　　　　　（一）子类方法的参数列表必须和父类中被重写的方法的参数列表相同（参数个数和参数类型），否则只能实现方法的重载。
   >
   >　　　　　　（二）子类方法的返回值类型必须和父类中被重写的方法返回值类型相同，否则只能实现方法重载。
   >
   >　　　　　　（三）在Java规定，子类方法的访问权限不能比父类中被重写的方法的访问权限更小，必须大于或等于父类的访问权限。
   >
   >　　　　　　（四）在重写的过程中，如果父类中被重写的方法抛出异常，则子类中的方法也要抛出异常。但是抛出的异常也有一定的约束--->子类不能抛出比父类更多的异常，只能抛出比父类更小的异常，或者不抛出异常。

   这里需要注意：构造方法不能被继承，因此不能被重写，在子类中只能通过super关键字调用父类的构造方法；但是可以被重载。

   ​

4. Java中equals()和hashCode()的区别和联系

   Java的基类Object提供了一些方法，其中equals()方法用于判断两个对象是否相等，hashCode()方法用于计算对象的哈希码。equals()和hashCode()都不是final方法，都可以被重写(overwrite)。

   ​

   **equls()**

   Object类中equals()方法实现如下：

   ````java
   public boolean equals(Object obj) {
       return (this == obj);
   }
   ````

   ​

   重写equals()方法应该遵守的约定：

   （1）自反性：x.equals(x)必须返回true。

   （2）对称性：x.equals(y)与y.equals(x)的返回值必须相等。

   （3）传递性：x.equals(y)为true，y.equals(z)也为true，那么x.equals(z)必须为true。

   （4）一致性：如果对象x和y在equals()中使用的信息都没有改变，那么x.equals(y)值始终不变。

   （5）非null：x不是null，y为null，则x.equals(y)必须为false。

   ​

   **hashCode()：**

   Object类中hashCode()方法的声明如下：

   ````java
   public native int hashCode();
   ````

   可以看出，hashCode()是一个native方法，而且返回值类型是整形；实际上，该native方法将对象在内存中的地址作为哈希码返回，可以保证不同对象的返回值不同。

   ​

   JDK中对hashCode()方法的作用，以及实现时的注意事项做了说明：

   （1）hashCode()在哈希表中起作用，如java.util.HashMap。

   （2）如果对象在equals()中使用的信息都没有改变，那么hashCode()值始终不变。

   （3）如果两个对象使用equals()方法判断为相等，则hashCode()方法也应该相等。

   （4）如果两个对象使用equals()方法判断为不相等，则不要求hashCode()也必须不相等；但是开发人员应该认识到，不相等的对象产生不相同的hashCode可以提高哈希表的性能。

   ​

   **hashCode()的作用**

   > 当我们向哈希表(如HashSet、HashMap等)中添加对象object时，首先调用hashCode()方法计算object的哈希码，通过哈希码可以直接定位object在哈希表中的位置(一般是哈希码对哈希表大小取余)。如果该位置没有对象，可以直接将object插入该位置；如果该位置有对象(可能有多个，通过链表实现)，则调用equals()方法比较这些对象与object是否相等，如果相等，则不需要保存object；如果不相等，则将该对象加入到链表中。

   ​

   **String中equals()和hashCode()的实现**

   ````java
   public boolean equals(Object anObject) {
           if (this == anObject) {
               return true;
           }
           if (anObject instanceof String) {
               String anotherString = (String)anObject;
               int n = value.length;
               if (n == anotherString.value.length) {
                   char v1[] = value;
                   char v2[] = anotherString.value;
                   int i = 0;
                   while (n-- != 0) {
                       if (v1[i] != v2[i])
                           return false;
                       i++;
                   }
                   return true;
               }
           }
           return false;
     }
    
   public int hashCode() {
           int h = hash;
           if (h == 0 && value.length > 0) {
               char val[] = value;

               for (int i = 0; i < value.length; i++) {
                   h = 31 * h + val[i];
               }
               hash = h;
           }
           return h;
    }
   ````

   通过代码可以看出以下几点：

   1、String的数据是final的，即一个String对象一旦创建，便不能修改；形如String s = “hello”; s = “world”;的语句，当s = “world”执行时，并不是字符串对象的值变为了”world”，而是新建了一个String对象，s引用指向了新对象。

   2、String类将hashCode()的结果缓存为hash值，提高性能。

   3、String对象equals()相等的条件是二者同为String对象，长度相同，且字符串值完全相同；不要求二者是同一个对象。

   4、String的hashCode()计算公式为：s[0]*31^(n-1) + s[1]*31^(n-2) + … + s[n-1]

   关于hashCode()计算过程中，为什么使用了数字31，主要有以下原因：

   1、使用质数计算哈希码，由于质数的特性，它与其他数字相乘之后，计算结果唯一的概率更大，哈希冲突的概率更小。

   2、使用的质数越大，哈希冲突的概率越小，但是计算的速度也越慢；31是哈希冲突和性能的折中，实际上是实验观测的结果。

   3、JVM会自动对31进行优化：31 * i == (i << 5) – i

   ​

   **总结：**

   - 如果两个对象相同，那么它们的hashCode值一定要相同；
   - 如果两个对象的hashCode相同，它们并不一定相同  

   ​

5. ThreadLocal 原理和使用场景

   参考：

   >- http://cmsblogs.com/?p=2442
   >- http://blog.xiaohansong.com/2016/08/06/ThreadLocal-memory-leak/#
   >- https://www.jianshu.com/p/ee8c9dccc953

   ​

   **原理：**

   threadLocal是一个`保存线程本地化变量`的容器，当在多线程环境下使用 ThreadLocal 维护变量时，其会为每个线程分配一个独立的`变量副本`，这样一来每个线程都只能对其变量副本进行读写而不会影响到其他线程的变量副本，从而保证了线程安全。

    

   **ThreadLocal相当于提供了一种线程隔离，将变量与线程相绑定。**

   ​

    ThredLocal的实现是这样的：

   > 首先在每个线程对象内部保存了一个map，这个map的**key是ThreadLocal实例，value是ThreadLocal中要保存的值**，每当使用ThreadLocal对变量副本进行set的时候，首先会从当前线程对象内部拿到相应的map，然后将ThreadLocal实例自身作为key，要保存的值作为value，put进map中，这样一来就实现了每个线程保存了独立的变量副本，它们之间互不影响。

   ​

   ![](https://github.com/zhangjinmiao/zhangjinmiao.github.io/raw/master/assets/images/2018/interview/threadlocal.jpg)

   `ThreadLocal`的实现是这样的：每个`Thread` 维护一个 `ThreadLocalMap` 映射表，这个映射表的 `key` 是 `ThreadLocal` 实例本身，`value` 是真正需要存储的 `Object`。

   也就是说 `ThreadLocal` 本身并不存储值，它只是作为一个 `key` 来让线程从 `ThreadLocalMap` 获取 `value`。值得注意的是图中的虚线，表示 `ThreadLocalMap` 是使用 `ThreadLocal` 的弱引用作为 `Key` 的，弱引用的对象在 GC 时会被回收。

   **常用方法**

   ````java
   public T get() {
     Thread t = Thread.currentThread();
     ThreadLocalMap map = getMap(t);
     if (map != null) {
       ThreadLocalMap.Entry e = map.getEntry(this);
       if (e != null) {
         @SuppressWarnings("unchecked")
         T result = (T)e.value;
         return result;
       }
     }
     return setInitialValue();
   }

   public void set(T value) {
     Thread t = Thread.currentThread();
     ThreadLocalMap map = getMap(t);
     if (map != null)
       map.set(this, value);
     else
       createMap(t, value);
   }

   protected T initialValue() {
     return null;
   }

   private T setInitialValue() {
     T value = initialValue();
     Thread t = Thread.currentThread();
     ThreadLocalMap map = getMap(t);
     if (map != null)
       map.set(this, value);
     else
       createMap(t, value);
     return value;
   }
   ````

   ​

   **内存泄漏**

   `ThreadLocalMap`使用`ThreadLocal`的弱引用作为`key`，如果一个`ThreadLocal`没有外部强引用来引用它，那么系统 GC 的时候，这个`ThreadLocal`势必会被回收，这样一来，`ThreadLocalMap`中就会出现`key`为`null`的`Entry`，就没有办法访问这些`key`为`null`的`Entry`的`value`，如果当前线程再迟迟不结束的话，这些`key`为`null`的`Entry`的`value`就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value`永远无法回收，造成内存泄漏。

   ​

   其实，`ThreadLocalMap`的设计中已经考虑到这种情况，也加上了一些防护措施：在`ThreadLocal`的`get()`,`set()`,`remove()`的时候都会清除线程`ThreadLocalMap`里所有`key`为`null`的`value`。

   但是这些被动的预防措施并不能保证不会内存泄漏：

   - 使用`static`的`ThreadLocal`，延长了`ThreadLocal`的生命周期，可能导致的内存泄漏（参考[ThreadLocal 内存泄露的实例分析](http://blog.xiaohansong.com/2016/08/09/ThreadLocal-leak-analyze/)）。
   - 分配使用了`ThreadLocal`又不再调用`get()`,`set()`,`remove()`方法，那么就会导致内存泄漏。

   **根源**

   ThreadLocal`内存泄漏的根源是：由于`ThreadLocalMap`的生命周期跟`Thread`一样长，如果没有手动删除对应`key`就会导致内存泄漏，而不是因为弱引用。

   ​

   **使用：**

   每次使用完`ThreadLocal`，都调用它的`remove()`方法，清除数据。

   ​

   **应用场景：**

   当我们只想在本身的线程内使用的变量，可以用 ThreadLocal 来实现，并且这些变量是和线程的生命周期密切相关的，线程结束，变量也就销毁了。

   1. 比如线程中处理一个非常复杂的业务，可能方法有很多，那么，使用 ThreadLocal 可以代替一些参数的显式传递；
   2. 上下文管理器、数据库连接等可以用到 ThreadLocal;

   **总结：**

   >- ThreadLocal 不是用于解决共享变量的问题的，也不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制。这点至关重要。
   >- 每个Thread内部都有一个ThreadLocal.ThreadLocalMap类型的成员变量，该成员变量用来存储实际的ThreadLocal变量副本。
   >- ThreadLocal并不是为线程保存对象的副本，它仅仅只起到一个索引的作用。它的主要目的是为每一个线程隔离一个类的实例，这个实例的作用范围仅限于线程内部。

   ​

6. ​

   ​



## 写出运行结果

1. 运行结果：

   pong

   ping

   ````java
   public static void main(String[] args) {
     Thread t = new Thread(){
       @Override
       public void run() {
         pong();
       }
     };
     t.run();
     System.out.println("ping");
   }

   static void pong(){
     System.out.println("pong");
   }
   ````

   这道题是考了Thread 的 start() 和 run() 方法的区别：

   > 1.  start():
   >
   > - 使该线程开始执行；Java 虚拟机调用该线程的 `run` 方法。
   >
   >
   > - 结果是两个线程并发地运行；当前线程（从调用返回给 `start` 方法）和另一个线程（执行其 `run` 方法，本例中为 main 方法）。
   >
   >
   > - 多次启动一个线程是非法的。特别是当线程已经结束执行后，不能再重新启动。
   >
   > 用start方法来启动线程，真正实现了多线程运行，这时无需等待run方法体代码执行完毕而直接继续执行下面的代码。通过调用Thread类的 start()方法来启动一个线程，这时此线程处于就绪（可运行）状态，并没有运行，一旦得到cpu时间片，就开始执行run()方法，这里方法 run()称为线程体，它包含了要执行的这个线程的内容，Run方法运行结束，此线程随即终止。
   >
   > 通过调用`run()` 方法才是正确的使用线程的方式。
   >
   > 1. run()
   >
   > - 如果该线程是使用独立的 `Runnable` 运行对象构造的，则调用该 `Runnable` 对象的 `run` 方法；否则，该方法不执行任何操作并返回.
   > - `Thread` 的子类应该重写该方法。
   >
   > run()方法只是类的一个普通方法而已，如果直接调用Run方法，程序中依然只有主线程这一个线程，其程序执行路径还是只有一条，还是要顺序执行，还是要等待run方法体执行完毕后才可继续执行下面的代码，这样就没有达到写线程的目的。

   **总结：**

   start() 方法的作用是启动一个新线程，新线程会执行相应的run()方法，start()不能被重复调用。

   而run()方法则只是普通的方法调用，在调用线程中顺序运行而已。

   ​

   **JDK1.8.0_152**中关于`start()` 和` run()` 的源码

   ````java
   public synchronized void start() {
           /**
            * This method is not invoked for the main method thread or "system"
            * group threads created/set up by the VM. Any new functionality added
            * to this method in the future may have to also be added to the VM.
            *
            * A zero status value corresponds to state "NEW".
            */
     		 // 如果线程不是"就绪状态"，则抛出异常！  
           if (threadStatus != 0)
               throw new IllegalThreadStateException();

           /* Notify the group that this thread is about to be started
            * so that it can be added to the group's list of threads
            * and the group's unstarted count can be decremented. */
      		// 将线程添加到ThreadGroup中  
           group.add(this);

           boolean started = false;
           try {
              // 通过start0()启动线程,新线程会调用run()方法  
               start0();
              // 设置started标记=true  
               started = true;
           } finally {
               try {
                   if (!started) {
                       group.threadStartFailed(this);
                   }
               } catch (Throwable ignore) {
                   /* do nothing. If start0 threw a Throwable then
                     it will be passed up the call stack */
               }
           }
       }
   private native void start0();
   ````

   ````java
   /**
        * If this thread was constructed using a separate
        * <code>Runnable</code> run object, then that
        * <code>Runnable</code> object's <code>run</code> method is called;
        * otherwise, this method does nothing and returns.
        * <p>
        * Subclasses of <code>Thread</code> should override this method.
        *
        * @see     #start()
        * @see     #stop()
        * @see     #Thread(ThreadGroup, Runnable, String)
        */
       @Override
       public void run() {
           if (target != null) {
               target.run();
           }
       }
   ````

   ​

2. 输出：

   >static A
   >static B
   >I'm A class
   >HelloA
   >I'm B class
   >HelloB

   ````java
   public class HelloA {
       public HelloA(){
           System.out.println("HelloA");
       }
       {
           System.out.println("I'm A class");
       }
       static {
           System.out.println("static A");
       }
   }

   public class HelloB extends HelloA {
       public HelloB(){
           System.out.println("HelloB");
       }
       {
           System.out.println("I'm B class");
       }
       static {
           System.out.println("static B");
       }

       public static void main(String[] args) {
           new HelloB();
       }
   }
   ````

   **分析：**

   - 首先去看父类里面有没有 `静态代码块`，如果有，它先去执行父类里面静态代码块里面的内容，当父类的静态代码块里面的内容执行完毕之后， 接着去执行子类(自己这个类)里面的 `静态代码块`，
   - 当子类的静态代码块执行完毕之后，它接着又去看父类有没有 `非静态代码块`，如果有就执行父类的非静态代码块，父类的非静态代码块执行完毕，接着执行父类的 `构造方法`，
   - 父类的构造方法执行完毕之后，它接着去看子类有没有非静态代码块，如果有就执行子类的非静态代 码块，子类的非静态代码块执行完毕再去执行子类的构造方法

   总结：

   `父类静态代码块`——> `子类静态代码块`——>`父类非静态代码块，构造方法`——>`子类非静态代码块，构造方法`

   这个就是一个对象的初始化顺序。

   ![](https://github.com/zhangjinmiao/zhangjinmiao.github.io/raw/master/assets/images/2018/interview/jvm.png)

   - 静态代码优先于非静态的代码,是因为被 `static` 修饰的成员都是类成员,会随着 JVM 加载类的时候加载而执行,而没有被 `static` 修饰的成员也被称为实例成员,需要创建对象才会随之加载到堆内存。所以静态的会优先非静态的。 


- 执行构造器(构造方法)的时候,在执行方法体之前存在隐式三步: 1,super语句,2,初始化非静态变量; 3,构造代码块。

1. ​



## 并发相关

1. 什么是 CopyOnWrite 容器

   ​

2. ​





## 数据库

1. 事务隔离级别有哪些，区别

   ​

2. ​



## Spring

1. spring 事务的传播行为，区别

   ​

2. ​



## JVM

1. 对象在 JVM 有哪些状态

   ​

2. 常用垃圾收集器

   ​

3. 遇到 OOM 如何处理

   ​

4. 如何减少上下文切换



## 编程

1. 二叉树迭代：先根，中根，后根

   ​

2. 简单实现一个 HashMap

   思路：

   > 1. HashMap有3个要素：hash函数+数组+单链表
   >
   > 2. 对于hash函数而言，需要考虑:
   >
   >    - 要快，对于给定的Key，要能够快速计算出在数组中的index。那么什么运算够快呢？显然是位运算！
   >    - 要均匀分布，要较少碰撞。说白了，我们希望通过hash函数，让数据均匀分布在数组中，不希望大量数据发生碰撞，导致链表过长。那么怎么办到呢？也是利用位运算，通过对数据的二进制的位进行移动，让hash函数得到的数据散列开来，从而减低了碰撞的概率。
   >
   > 3. 发生了碰撞怎么办
   >
   >    JDK的HashMap是通过单链表解决的。那么除了这个方法，还有其他思路么？
   >
   >    - 如果发生冲突，那么记下这个冲突的位置为index，然后在加上固定步长，即index+step，找到这个位置，看一下是否仍然冲突，如果继续冲突，那么按照这个思路，继续加上固定步长。其实这就是所谓的`线性探测`来解决Hash冲突的方法

   ​

3. 简单实现一个阻塞队列



## 架构设计

1. 设计一个发号器，毫秒级不能重复并且可支持多业务

   方案一：
   如果没有并发，订单号只在一个线程内产生，那么由于程序是顺序执行的，不同订单的生成时间戳正常不同，因此用时间戳+随机数（或自增数）就可以区分各个订单。
   如果存在并发，且订单号是由一个进程中的多个线程产生的，那么只要把线程ID添加到序列号中就可以保证订单号唯一。
   如果存在并发，且订单号是由同一台主机中的多个进程产生的，那么只要把进程ID添加到序列号中就可以保证订单号唯一。
   如果存在并发，且订单号是由不同台主机产生的，那么MAC地址、IP地址或CPU序列号等能够区分主机的号码添加到序列号中就可以保证订单号唯一。

   ​

   方案二：
   时间戳+用户ID+几个随机数+乐观锁。

   ​

   方案三：
   用redis的原子递增，做好高可用集群。

   ​

   方案四（非纯数字）：
   java自带uuid。

   **缺点**：性能比较差，并且 UUID 比较长，占用空间大，间接导致数据库性能下降，更重要的是，UUID 并不具有有序性，这导致 B+ 树索引在写的时候会有过多的随机写操作（连续的ID会产生部分顺序写）

   ​

   ​

   方案五：**snowflake算法**

   snowflake是twitter开源的分布式ID生成算法，其**核心思想为，**一个long型的ID：

   - 41bit作为毫秒数
   - 10bit作为机器编号
   - 12bit作为毫秒内序列号


   算法单机每秒内理论上最多可以生成1000*(2^12)，也就是400W的ID，完全能满足业务的需求。

   参考：[vesta-id-generator](https://gitee.com/robertleepeak/vesta-id-generator)

   ​

   **总之，思路如下：**

   ***思路一：基于数据库生成***

   标识的生成方法有很多，有集中式的，分布式的；有后端的，前端的，当然还有人工的。 并没有一种通用的生成方法来适应各种应用场景。

   人工生成的确是一种方式，比如电子邮箱，微信ID，各种论坛的账号。在人想出标识的那一刻，是无法判断是否是唯一的，对这种生成方式的结果，显然在录入时都需要进行唯一性校验。所以，下面描述的几种生成方式，是在生成的那一刻就在一个命名空间内唯一，而不再需要进行唯一性校验。

   而基于数据库生成，一般包含以下几种：

- MySQL(5.6) AUTO_INCREMENT 特性
- Postgres(REL 9.6 Stable) SEQUENCE 特性
- Oracle 数据库的 SEQUENCE 特性，有知道这一特性如何实现的，可以在 知乎 做一下解答。
- Flickr Ticket Servers ，同时支持Sharding (文章发表于2010年2月8日，算法上线于2006年1月13日)。

   一般地，这种类型的生成方案，都可以设置其实初始值，以及增量步长。

   ​

   ***思路二：基于分布式集群协调器生成***

   在不使用数据库的情况下，通过一个后台服务对外提供高可用的、固定步长标识生成，则需要分布式的集群协调器进行。

   一般的，主流协调器有两类：

- 以强一致性为目标的：ZooKeeper为代表
- 以最终一致性为目标的：Consul为代表

   ZooKeeper的强一致性，是由Paxos协议保证的；Consul的最终一致性，是由Gossip协议保证的。

   在步长累计型生成算法中，最核心的就是保持一个累计值在整个集群中的「强一致性」。同时，这也

   会为唯一性标识的生成带来新的形成瓶颈。

   ​

   ***思路三：划分命名空间并行生成***

   似乎对于分布式的ID生成，以Twitter Snowflake为代表的， Flake 系列算法，经常可以被搜索引擎找到，但似乎MongoDB的ObjectId算法，更早地采用了这种思路。MongoDB 1.0 是在2009年8月27日 发布 的，并且0.9.10(2009年8月24日发布)和1.0两个版本没有差异。

   在StackOverflow上，最早的一个关于ObjectId的问题（http://stackoverflow.com/questions/2138687/whats-mongodb-hashs-size/2146071），时间是2010年1月27日。不知道Twitter的同学，是不是受此启发呢？

   https://docs.mongodb.com/manual/reference/method/ObjectId/

   ​

1. ​

2. ​

3. ​