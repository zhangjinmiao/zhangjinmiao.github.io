##1.  Java类加载器

JVM中类的装载是由类加载器（ClassLoader）和它的子类来实现的，Java中的类加载器是一个重要的Java运行时系统组件，它负责在运行时查找和装入类文件中的类。 

由于Java的跨平台性，经过编译的Java源程序并不是一个可执行程序，而是一个或多个类文件。当Java程序需要使用某个类时，JVM会确保这个类已经被`加载、连接（验证、准备和解析）和初始化`。

**类的加载**是指把类的.class文件中的数据读入到内存中，通常是创建一个字节数组读入.class文件，然后产生与所加载类对应的Class对象。加载完成后，Class对象还不完整，所以此时的类还不可用。

当类被加载后就进入**连接阶段**，这一阶段包括`验证、准备（为静态变量分配内存并设置默认的初始值）和解析（将符号引用替换为直接引用）`三个步骤。

最后JVM对类进行**初始化**，包括：
1)如果类存在直接的父类并且这个类还没有被初始化，那么就先初始化父类；
2)如果类中存在初始化语句，就依次执行这些初始化语句。 

类的加载是由**类加载器**完成的，类加载器包括：`根加载器（BootStrap）`、`扩展加载器（Extension）`、`系统加载器（System）`和`用户自定义类加载器（java.lang.ClassLoader的子类）`。

从Java 2（JDK 1.2）开始，类加载过程采取了**父亲委托机制（PDM）**。PDM更好的保证了Java平台的安全性，在该机制中，JVM自带的Bootstrap是根加载器，其他的加载器都有且仅有一个父类加载器。`类的加载首先请求父类加载器加载，父类加载器无能为力时才由其子类加载器自行加载。`JVM不会向Java程序提供对Bootstrap的引用。下面是关于几个类加载器的说明：
>
>Bootstrap：一般用本地代码实现，负责加载JVM基础核心类库（rt.jar）；
Extension：从java.ext.dirs系统属性所指定的目录中加载类库，它的父加载器是Bootstrap；
System：又叫应用类加载器，其父类是Extension。它是应用最广泛的类加载器。它从环境变量classpath或者系统属性java.class.path所指定的目录中记载类，是用户自定义加载器的默认父加载器。

>参考：http://mp.weixin.qq.com/s/gIuGxSlyjwuItslvu_W53Q


##2.  JVM内存

- **方法区、堆：**线程共享的，所有的运行在jvm上的程序都能访问这两个区域，堆，方法区和虚拟机的生命周期一样，随着虚拟机的启动而存在
- **虚拟机栈、本地方法栈、程序计数器：**依赖用户线程的启动和结束而建立和销毁

![JVM内存模型.png](http://upload-images.jianshu.io/upload_images/626005-3cde875e7bde7d20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


1. **方法区：**
这个区域用于存储被加载的`类的信息`，`常量`，`静态变量`以及即时编译器`编译后的代码`等数据。
虚拟机规范中定义了OOM(OutOfMemoryError)异常。

方法区也称为`永久区`，主要用于存放常量和类的定义信息。是一块独立于Java 堆的内存空间。虽然叫做永久区，但是在永久区中的对象，同样也是可以被 GC 回收的。只是对于 GC 的表现和 Java 堆空间略有不同。对永久区 GC 的回收，通常从两方面分析：
一是：GC 对永久区常量池的回收
二是：永久区对类元数据的回收

2. **堆：**
`所有线程都能访问`，随着虚拟机的启动而存在，这块区域很大，因为所有的线程都在这个区域 `保存实例化的对象`，只要是用到`new关键字`创建的对象都会进入到这个区域，包括`对象，数组`。（每一个类型中，每个接口实现类需要的内存不一样，一个方法内的多个分支需要的内存也不尽相同，我们只有在运行的时候才能知道要创建多少对象，需要分配多大的地址空间。GC关注的正是这样的部分内容，所以很多时候也将堆称为GC堆。）
堆还能进一步划分，比如
按照内存回收的角度来看，堆可以进一步划分为`新生代（Eden+Survivor）`和`老年代`。
按照内存分配的角度来看，堆可以进一步划分为多个`线程私有的分配缓存区`，即TLAB（Thread Local Allocation Buffer）。
这种进一步的划分是为了更高效地回收和分配内存。
java虚拟机规范中定义了OOM异常。


3. **Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。**
Java虚拟机栈描述的是Java方法（区别于native的本地方法）执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储`局部变量表`、`操作栈`、`动作链接`、`方法出口`等信息。每个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中`从入栈到出栈的过程`。
当进入一个方法时，栈帧的大小是编译器确定的，`运行时不会改变其大小`。当虚拟机栈不可扩展的时候，可能抛出`StackOverflowError ` 异常，反之，可能抛出 `OOM`异常。

4. **本地方法栈（Native Method Stacks）**与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法(也就是字节码)服务，而本地方法栈则为虚拟机所使用到的 `Native方法服务`。
同样在虚拟机规范中定义了StackOverflowError和OOM两种异常。

5. **程序计数器（Program Counter Register）**是一块较小的内存空间，线程隔离，即每个线程都有自己的程序计数器，并且互不影响。
分为两种情况：
- 当线程正在执行的是一个java方法，它的作用是作为字节码的行号指示器，指向下一条需要执行的指令。
- 当线程正在执行的是一个Native方法，那么它的值为空（Undefined）。

**java虚拟机规范中唯一没有定义OOM异常的内存区域。**

###对象的创建
大体上分为4个步骤：`类加载`、`分配内存`、`内存区域的初始化`、`虚拟机对对象进行必要的设置`。

- **类加载：**虚拟机遇到一条new指令的时候，首先检查是否有必要进行类加载，即检查指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用是否已经被加载、解析和初始化，若没有，则进行类加载。
- **分配内存：**若是虚拟机的垃圾回收器有压缩整理内存的功能，即指针将内存区域分为两个部分，一边已分配，另一边未分配，则采用指针碰撞（Bump the Pointer）进行内存分配。否则采用空闲列表（Free List）分配内存。
- **内存空间初始化为零值：**顾名思义，将对象分配到的内存空间进行初始化，但是不包括对象头。
- **对对象进行一些必要的设置：**例如这个对象是哪个类的实例，如何找到类的元数据信息，对象哈希码，GC分代年龄等信息。这些信息存放在对象头（Object Header）中。

至此，虚拟机认为这个对象已经创建完毕，但是在我们程序中，这个对象才刚刚可以使用。



### 堆与栈
>[JVM调优总结](http://pengjiaheng.iteye.com/blog/518623)
[聊一下你对javac的理解，它的基本结构，以及是如何将Java源码编译成Java字节码的](http://mp.weixin.qq.com/s/immDsrzqhwe9yT-ray1_mA)

**栈是运行时的单位，而堆是存储的单位。**
栈解决程序的运行问题，即程序如何执行，或者说如何处理数据；堆解决的是数据存储的问题，即数据怎么放、放在哪儿。



##3. StackOverflowError与OutOfMemoryError区别
在Java虚拟机规范中，定义了这么两种异常：StackOverflowError与OutOfMemoryError。

那么它们到底直接有啥区别呢？

在《The Java ® Virtual Machine Specification Java SE 8 Edition》中是这么说的：
>The following exceptional conditions are associated with native method stacks:
>
>If the computation in a thread requires a larger native method stack than is permitted, the Java Virtual Machine throws a StackOverflowError .
>
>如果在一个线程计算过程中不允许有更大的本地方法栈，那么JVM就抛出StackOverflowError
[**线程申请的栈深度大于虚拟机所允许的深度**]
>
>If native method stacks can be dynamically expanded and native method stack expansion is attempted but insufficient memory can be made available, or if insufficient memory can be made available to create the initial native method stack for a new thread, the Java Virtual Machine throws an OutOfMemoryError .
>
>如果本地方法栈可以动态地扩展，并且本地方法栈尝试过扩展了，但是没有足够的内容分配给它，再或者没有足够的内存为线程建立初始化本地方法栈，那么JVM抛出的就是OutOfMemoryError。
[**程序无法申请到足够的内存**]

**StackOverflowError的情况：**
```java
public class StackOverFlow {
    public int stackSize = 0;

    public void stackIncre() {
        stackSize++;
        stackIncre();
    }

    public static void main(String[] args) throws Throwable{
        StackOverFlow sof = new StackOverFlow();
        try {
            sof.stackIncre();
        } catch (Throwable e) {
            System.out.println(sof.stackSize);
            throw e;
        }
    }
}
```
无限制递归，直到堆栈溢出，运行输出如下：
解决办法：提高-XX:ThreadStackSize=512的值
```java
20704
Exception in thread "main" java.lang.StackOverflowError
	at com.jimzhang.core.jvm.stackoverflowerror.StackOverFlow.stackIncre(StackOverFlow.java:16)
	at com.jimzhang.core.jvm.stackoverflowerror.StackOverFlow.stackIncre(StackOverFlow.java:16)
	at com.jimzhang.core.jvm.stackoverflowerror.StackOverFlow.stackIncre(StackOverFlow.java:16)
	at com.jimzhang.core.jvm.stackoverflowerror.StackOverFlow.stackIncre(StackOverFlow.java:16)
........
```

**OutOfMemoryError 的情况：**
```java
public class HeapOverFlow {
    public static void main(String[] args) {
        ArrayList<HeapOverFlow> list = new ArrayList<HeapOverFlow>();

        while (true) {
            list.add(new HeapOverFlow());
        }
    }
}
```
无限制地往list里面塞对象，直到它扛不住：
```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:261)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:235)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:227)
	at java.util.ArrayList.add(ArrayList.java:458)
	at com.jimzhang.core.jvm.heapoverflow.HeapOverFlow.main(HeapOverFlow.java:18)

Process finished with exit code 1
```
我们都知道List是动态增长的么，它不够了就要扩展，不够了就要扩展，但是机器本地内存还是优先的，要是扩展不了了，就抛出OutOfMemoryError。

### 内存溢出的原因：
1. 内存中加载的数据量过于庞大，如一次从数据库取出过多数据；
2. 集合类中有对对象的引用，使用完后未清空，使得JVM不能回收；
3. 代码中存在死循环或循环产生过多重复的对象实体；
4. 静态变量和静态方法过多；递归；无法确定是否被引用的对象；
5. 使用的第三方软件中的BUG；
6. 启动参数内存值设定的过小；

### 内存溢出的解决方案：

第一步，修改JVM启动参数，直接增加内存。(-Xms，-Xmx参数一定不要忘记加。)

第二步，检查错误日志，查看“OutOfMemory”错误前是否有其它异常或错误。

第三步，对代码进行走查和分析，找出可能发生内存溢出的位置。

第四步，使用内存查看工具动态查看内存使用情况。（Jconsole）

>1. 优化程序代码，如果业务庞大，逻辑复杂，尽量减少全局变量的引用，让程序使用完变量的时候释放该引用能够让垃圾回收器回收，释放资源
>2. 物理解决，增大物理内存，然后通过：-Xms256m -Xmx256m -XX:MaxNewSize=256m -XX:MaxPermSize=256m的修改


>附录：
>**-Xss ：**单个线程堆栈大小值；JDK5.0 以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
**-XX:PermSize：** 应用服务器启动时，永久存储区的初始内存大（**JDK1.8失效了**）,默认值为 `64M`
**-XX:MaxPermSize：** 应用运行中，永久存储区的极限值。为了不消耗扩大JVM 永久存储区分配的开销，将此参数和-XX:PermSize这个两个值设为相等。（**JDK1.8失效了**）
**-Xms：** 启动应用时，JVM 堆空间的初始大小值，默认值是物理内存的`1/64 但小于1G`。
**-Xmx：** 应用运行中，JVM 堆空间的极限值。默认值是物理内存的 `1/4 但小于1G`。为了不消耗扩大JVM 堆控件分配的开销，将此参数和-Xms 这个两个值设为相等，考虑到需要开线程，讲此值设置为总内存的80%。
**-Xmn：** 此参数硬性规定堆空间的新生代空间大小，推荐设为堆空间大小的1/4。
**-XX:NewRatio：** 设置Yong 和 Old的比例，比如值为2，则Old Generation是 Yong Generation的2倍，即Yong Generation占据内存的1/3。
一般情况下，不允许-XX:Newratio值小于1，即Old要比Yong大
**-XX:Surviorratio：**设置Eden和一个Suivior的比例，比如值为5，即Eden是To(S2)的比例是5，（From和To是一样大的），此时Eden占据Yong Generation的5/7
**-XX:Newsize：**设置Yong Generation的初始值大小
**-XX:Maxnewsize：**设置Yong Generation的最大值大小
**-XX:+printGC：**打印GC的简要信息
**-XX:+PrintGCDetails：**打印GC详细信息
**-XX:+PrintGCTimeStamps：**打印CG发生的时间戳

![微信图片_20180104150411.jpg](http://upload-images.jianshu.io/upload_images/626005-03e2c2f3d45cbee2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 内存溢出的类型
JVM 管理两种类型的内存，堆和非堆。堆是给开发人员用的上面说的就是，是在 JVM 启动时创建；非堆是留给 JVM 自己用的，用来存放类的信息的。它和堆不同，运行期内 GC 不会释放空间。

**1. java.lang.OutOfMemoryError: PermGen space**
出现这种异常的原因：
- web app 用了大量的第三方 jar （*尤其使用到像Spring等框架的时候，由于需要使用到动态生成类，而这些类不能被GC自动释放*）或者应用有太多的 class 文件而恰好 MaxPermSize 设置较小（默认为64M），超出了也会导致这块内存的占用过多造成溢出
-  tomcat 热部署时侯不会清理前面加载的环境，只会将 context 更改为新部署的，非堆存的内容就会越来越多

**2. java.lang.OutOfMemoryError: Java heap space**
这种异常出现的较多：
默认空间 ( 即 -Xms) 是物理内存的 1/64 ，最大空间 (-Xmx) 是物理内存的 1/4 。如果内存剩余不到 40 ％， JVM 就会增大堆到 Xmx 设置的值，内存剩余超过 70 ％， JVM 就会减小堆到 Xms 设置的值。所以服务器的 Xmx 和 Xms 设置一般应该设置相同避免每次 GC 后都要调整虚拟机堆的大小。


**结论：**
1. **永久存储区溢出（java.lang.OutOfMemoryError:Java Permanent Space）”**
**原因：**永久存储区设置太小，不能满足系统需要的大小
**解决方法：**只需要调整-XX:PermSize 和-XX:MaxPermSize 这两个参数即可
2. **JVM 堆空间溢出（java.lang.OutOfMemoryError: Java heap space）”**
**原因：**JVM 堆空间不足
**解决方法：**只需要调整-Xms 和-Xmx 这两个参数即可

**建议”**：
>1. **尽早释放无用对象的引用。**好的办法是使用临时变量的时候，让引用变量在退出活动域后，自动设置为 null ，暗示垃圾收集器来收集该对象，防止发生内存泄露。
>2.  **避免使用 String ，应大量使用 StringBuffer** ，每一个 String 对象都得独立占用内存一块区域
>3. **尽量少用静态变量**，因为静态变量是全局的， GC 不会回收的
>4. **避免集中创建对象尤其是大对象:**JVM 会突然需要大量内存，这时必然会触发 GC 优化系统内存环境。所以：避免显示的声明数组空间，而且申请数量还极大
>5. **尽量运用对象池技术以提高系统性能**
>6. **不要在经常调用的方法中创建对象，尤其是忌讳在循环中创建对象。**可以考虑使用对象容器

##3. GC
>参考：http://www.cnblogs.com/sxdcgaq8080/p/7156227.html

GC是在什么时候，对什么东西，做了什么

### GC是在什么时候
当应用程序线程空闲；另一个是 java 内存堆不足时，会不断调用 GC ，若连续回收都解决不了内存堆不足的问题时，就会报 out of memory 错误。

**minor gc/full gc的触发条件**

- eden满了minor gc
- 升到老年代的对象大于老年代剩余空间full gc，或者小于时被HandlePromotionFailure参数强制full gc

**OOM的触发条件**

gc与非gc时间耗时超过了GCTimeRatio的限制引发OOM

**降低GC的调优的策略**

- 通过NewRatio控制新生代老年代比例
- 通过MaxTenuringThreshold控制进入老年前生存次数




### 对什么东西GC

- 超出作用域的对象/引用计数为空的对象

  给对象中添加一个引用计数器，每当有一个地方引用它时，计数器就加1；当引用失效时，计数器值就减1；任何时刻计数器都为0的对象就是不可能再被使用的。

  **Java语言没有选用引用计数法来管理内存，因为引用计数法不能很好的解决循环引用的问题。**

- 从gc root开始搜索，搜索不到的对象

  **使用根搜索算法来判定对象是否存活**

  GC Root Tracing 算法思路就是通过一系列的名为"GC  Roots"的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连，即从GC Roots到这个对象不可达，则证明此对象是不可用的。

- 从root搜索不到，而且经过第一次标记、清理后，仍然没有复活的对象



**哪些对象可以作为GC Roots**

- 虚拟机栈（栈帧中的本地变量表）中的引用的对象

- 方法区中的类静态属性引用的对象

- 方法区中的常量引用的对象    

- 本地方法栈中JNI（Native方法）的引用对象

  ​

####做了什么

- 新生代做的是复制清理
- from survivor、to survivor？：
- 老年代做的是标记清理
- 标记清理后碎片要不要整理？：
- 制清理和标记清理有有什么优劣势？：

串行、并行（整理/不整理碎片）、CMS等搜集器可作用的年代、特点、优劣势

控制/调整收集器选择的方式

### **JVM怎么判断一个对象已经消亡可以被回收？**
