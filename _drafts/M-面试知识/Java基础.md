##1. Switch支持的类型

支持byte,short,char,int四种整型类型、枚举类型和java.lang.String类型。

byte,short,char,int 的包装类也可以使用，但其实是将引用类型自动拆箱为整型类型了。


##2. String 和 StringBuilder 的区别？StringBuilder 和 StringBuffer 的区别？

**String**  是内容不可变的字符串：底层使用了一个不可变的字符数组(final char[])，因此每次对其操作改变其变量值，其实是生成一个新的对象，然后将变量引用指向新对象；因此速度慢。
```java
/** The value is used for character storage. */
    private final char value[];
```
String  str = new String("aaa);

**StringBuffer** 则不同，对其操作即直接操作对象指向的引用，无需产生新对象，速度很快；它是`线程安全的`，在维护多线程的同步等也会消耗一点性能。

**StringBuilder** 是jdk5之后新增的，其用法与StringBuffer完全一致，但它是`线程不安全的`，在单线程中最佳，因为其不需要维护线程的安全，因此是`最快的`。
StringBuilder ，StringBuffer 是内容可变的字符串，底层使用的是可变的数组（没有使用final 来修饰）

**String是存放在常量池，在编译期已经被确定了。new String()不是字符串常量，它有自己的地址空间，存放在堆空间。而其它两个都存放在堆空间。**

```
 /**
     * The value is used for character storage.
     */
    char[] value;
```
最经典的就是拼接字符串：
1. String 进行拼接，String  = "a" + "b"; 创建了许多中间对象，**引号创建的字符串在字符串池中。**
2. StringBuilder 或 StringBuffer
```
StringBuilder sb = new StringBuilder；
sb.append("a").append("b");
```

所以，要使用StringBuilder 或 StringBuffer 来拼接字符串

StringBuilder 是线程不安全的，效率较高；
StringBuffer 是线程安全的，效率较低


**String.intern()方法的含义：如果常量池中已经存在当前 String ，则返回池中的对象；如果常量池中不存在当前 String 对象，则先将 String 加入常量池，并返回池中的对象引用。**

>String：http://mp.weixin.qq.com/s/r1TK-wkHykzkRWUWcGr9yw
##3. java 中的集合
Java 中的集合分为value，key-value（collection Map）两种

存储值的有 list 和 set
- list 是有序的，可以重复
- set 是无序的，不可重复，根据 equals 和hashCode 判断，也就是如果一个对象要存储在 set 中，必须重写 equals 和 hashCode 方法。

存储 key-value 的为 map

##4. ArrayList 和 LinkedList 的区别
List 常用的有 ArrayList 和 LinkedList 。

ArrayList 底层使用的是`数组`， LinkedList 底层使用的是` 链表`。
- **数组**具有索引，`查询`特定元素`比较快`，而`插入和删除比较慢`（数组在内存中是一块连续的内存，如果插入或删除时均要移动内存）
- **链表**不要求内存是连续的，在当前元素中存放下一个或上一个元素的地址，查询时从头部开始，一个一个的找，所以`查询效率低`，插入时不需要移动内存，只需改变引用指向即可，所以`插入或删除效率高`。

**ArrayList 使用在查询比较多，但是插入和删除比较少的情况，而 LinkedList 使用在查询比较少而插入和删除比较多的情况。**

##5. HashMap 和 HashTable 的区别?HashTable 和 ConcurrentHashMap 的区别？

**相同点：**
HashMap 和 HashTable  都可以用来存储key-value 的数据。
**区别：**
1. HashMap 是可以把 null 作为 key 或 value的，而 HashTable 是不可以的。
2.  HashMap 是线程不安全的，效率高，而 HashTable 是线程安全的，效率低

**既想线程安全又想效率高？**

通过把整个 Map 分为 N 个 Segment（类似 HashTable），可以提供相同的线程安全，但是效率提升 N 倍，默认提升16倍。


##6. 实现一个拷贝文件的工具使用字节流还是字符流？

拷贝的文件不确定是只包含字符流，有可能有字节流（图片、声音、图像等），为了通用性要使用字节流。

##7. 线程的实现方式
**实现方式**
1. 通过实现 Runnable 接口
2. 通过继承 Thread 类

继承扩展性不强，因为 Java 支支持单继承，如果一个类继承 Thread 就不能继承其他类了。
**启动方式：**


**线程池**



**线程并发库**


##8.  抽象类和接口
###1.  抽象类

抽象类是用来捕捉子类的通用特性的 。它不能被实例化，只能被用作子类的超类。比如：GenericServlet

```java
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
    // abstract method
    abstract void service(ServletRequest req, ServletResponse res);
 
    void init() {
        // Its implementation
    }
    // other method related to Servlet
}
```

HttpServlet类继承GenericServlet

```java
public class HttpServlet extends GenericServlet {
    void service(ServletRequest req, ServletResponse res) {
        // implementation
    }
 
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        // Implementation
    }
 
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        // Implementation
    }
 
    // some other methods related to HttpServlet
}
```
###2. 接口

接口是抽象方法的集合。

###抽象类和接口的对比

| **参数**      | **抽象类**                                  | **接口**                                   |
| ----------- | ---------------------------------------- | ---------------------------------------- |
| 默认的方法实现     | 它可以有默认的方法实现                              | 接口完全是抽象的。它根本不存在方法的实现                     |
| 实现          | 子类使用**extends**关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。 | 子类使用关键字**implements**来实现接口。它需要提供接口中所有声明的方法的实现 |
| 构造器         | 抽象类可以有构造器                                | 接口不能有构造器                                 |
| 与正常Java类的区别 | 除了你不能实例化抽象类之外，它和普通Java类没有任何区别            | 接口是完全不同的类型                               |
| 访问修饰符       | 抽象方法可以有**public**、**protected**和**default**这些修饰符 | 接口方法默认修饰符是**public**。你不可以使用其它修饰符。        |
| main方法      | 抽象方法可以有main方法并且我们可以运行它                   | 接口没有main方法，因此我们不能运行它。                    |
| 多继承         | 抽象方法可以继承一个类和实现多个接口                       | 接口只可以继承一个或多个其它接口                         |
| 速度          | 它比接口速度要快                                 | 接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。            |
| 添加新方法       | 如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。 | 如果你往接口中添加方法，那么你必须改变实现该接口的类。              |

###3. 什么时候使用抽象类和接口

- 如果你拥有一些方法并且想让它们中的一些有默认实现，那么使用抽象类吧。
- 如果你想实现多重继承，那么你必须使用接口。由于**Java不支持多继承**，子类不能够继承多个类，但可以实现多个接口。因此你就可以使用接口来解决它。
- 如果基本功能在不断改变，那么就需要使用抽象类。如果不断改变基本功能并且使用接口，那么就需要改变所有实现了该接口的类。

## 9. 聊一下static／final／volatile／transient修饰符的用法
>参考：http://mp.weixin.qq.com/s/jNXMTCsnYr0-Z3wxBMPA3A
### static
static是静态修饰关键字，可以修饰变量和程序块以及类方法。

**修饰的变量**在内存中只有一个拷贝，JVM只为其分配一次，在类的加载过程中完成内存的分配，既不位于堆也不位于栈，位于数据区，既可以通过类名直接访问，也可以通过对象来访问。

**修饰的代码块**，JVM优先加载，一般用于系统初始化。

**修饰的方法**，独立于任何实例，不能被abstract修饰，方法内不能用this／super，也不能直接访问实例变量／实例方法。

### final
final可以修饰变量、方法和类。

**修饰的变量** 只能被赋值一次，赋值后值不再改变（这里也是常见的考点？如果是引用类型只是地址不会变），位于常量池；通常情况下，final变量有三个地方可以赋值：直接赋值，构造函数，或初始化块中。

**修饰的方法**不能被子类覆盖，但可以继承。

**修饰的类** 不能被继承，没有子类，final类中所有方法都是final的。

### volatile

volatile修饰的变量保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是**立即可见的**。因为当对普通变量进行读写的时候，每个线程先从内存拷贝变量到CPU缓存中。如果计算机有多个CPU，每个线程可能在不同的CPU上被处理，这意味着每个线程可以拷贝到不同的CPU cache中。而 **volatile修饰的变量，JVM保证了每次读变量都从内存中读**，跳过CPU cache这一步。

volatile修饰的变量无法保证对变量的任何操作都是原子性的。这里面试官可能会引申出更多的多线程相关的知识，如***怎么保证原子性等？***

volatile修饰的变量禁止进行指令重排序，所以能在一定程度上保证有序性。只能保证该变量所在的语句还是原来的位置，并不能保证该语句之前或之后的语句是否被打乱。

####使用场景

**状态标记量**，代码如下：
```java
volatile boolean flag = false;
while(!flag){
    doSomething();
}
public void setFlag() {
    flag = true;
}
```

**双重检查**
```java
class Singleton{
    private volatile static Singleton instance = null;
     
    private Singleton() {
         
    }
     
    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

### transient
transient**只能修饰变量**，而不能修饰方法和类。

transient**修饰的变量不再能被序列化**，一个静态变量不管是否被transient修饰，均不能被序列化。一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。也可以认为在将持久化的对象反序列化后，被transient修饰的变量将按照普通类成员变量一样被初始化。
