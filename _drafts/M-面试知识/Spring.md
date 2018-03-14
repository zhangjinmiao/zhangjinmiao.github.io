#1. Spring 事务的原理
Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。对于纯JDBC操作数据库，想要用到事务，可以按照以下步骤进行：
>1. 获取连接 Connection con = DriverManager.getConnection()
>2. 开启事务con.setAutoCommit(true/false);
>3. 执行CRUD
>4. 提交事务/回滚事务 con.commit() / con.rollback();
>5. 关闭连接 conn.close();

使用Spring的事务管理功能后，我们可以不再写步骤 2 和 4 的代码，而是由Spirng 自动完成。那么Spring是如何在我们书写的 CRUD 之前和之后开启事务和关闭事务的呢？解决这个问题，也就可以从整体上理解Spring的事务管理实现原理了。
下面简单地介绍下，注解方式为例子

配置文件开启注解驱动，在相关的类和方法上通过注解@Transactional标识。

spring 在启动的时候会去解析生成相关的bean，这时候会查看拥有相关注解的类和方法，并且为这些类和方法生成代理，并根据@Transaction的相关参数进行相关配置注入，这样就在代理中为我们把相关的事务处理掉了（开启正常提交事务，异常回滚事务）。

真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

---
Spring 的事务，可以说是 Spring AOP 的一种实现。
AOP面向切面编程，即在不修改源代码的情况下，对原有功能进行扩展，通过代理类来对具体类进行操作。 

spring是一个容器，通过spring这个容器来对对象进行管理，根据配置文件来实现spring对对象的管理。

spring的事务声明有两种方式，编程式和声明式。spring主要是通过“`声明式事务`”的方式对事务进行管理，即在配置文件中进行声明，`通过AOP将事务切面切入程序`，最大的好处是大大减少了代码量。

>参考：
[事务管理实现原理](https://my.oschina.net/huangyong/blog/159852)

#2. Spring 事务的传播行为和隔离级别
## 传播行为 (Propagation)
1) **REQUIRED** 需要，这个是默认的属性 
Support a current transaction, create a new one if none exists. 
如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。 
被设置成这个级别时，会为每一个被调用的方法创建一个逻辑事务域。如果前面的方法已经创建了事务，那么后面的方法支持当前的事务，如果当前没有事务会重新建立事务。 

2) **MANDATORY**  必要的
Support a current transaction, throw an exception if none exists.支持当前事务，如果当前没有事务，就抛出异常。 

3) **NEVER** 绝不
Execute non-transactionally, throw an exception if a transaction exists. 
以非事务方式执行，如果当前存在事务，则抛出异常。 

4) **NOT_SUPPORTED** 
Execute non-transactionally, suspend the current transaction if one exists. 
以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 

5) **REQUIRES_NEW** 
Create a new transaction, suspend the current transaction if one exists. 
新建事务，如果当前存在事务，把当前事务挂起。 

6) **SUPPORTS** 支持
Support a current transaction, execute non-transactionally if none exists. 
支持当前事务，如果当前没有事务，就以非事务方式执行。 

7) **NESTED** 嵌套的
Execute within a nested transaction if a current transaction exists, behave like PROPAGATION_REQUIRED else. 
支持当前事务，新增Savepoint点，与当前事务同步提交或回滚。 
嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。 

###PROPAGATION_NESTED 与PROPAGATION_REQUIRES_NEW的区别 
它们非常 类似,都像一个嵌套事务，如果不存在一个活动的事务，都会开启一个新的事务。使用PROPAGATION_REQUIRES_NEW时，内层事务与外层事务就像两个独立的事务一样，一旦内层事务进行了提交后，外层事务不能对其进行回滚。两个事务互不影响。两个事务不是一个真正的嵌套事务。同时它需要JTA 事务管理器的支持。 
使用PROPAGATION_NESTED时，外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚，它是一个真正的嵌套事务。 

## 隔离级别 (lsolation)

###事务并发引起的三种情况

1. **Dirty Reads 脏读**
一个事务正在对数据进行更新操作，但是更新还未提交，另一个事务这时也来操作这组数据，并且读取了前一个事务还未提交的数据，而前一个事务如果操作失败进行了回滚，后一个事务读取的就是错误数据，这样就造成了脏读。

2.  **Non-Repeatable Reads 不可重复读**
一个事务多次读取同一数据，在该事务还未结束时，另一个事务也对该数据进行了操作，而且在第一个事务两次读取之间，第二个事务对数据进行了更新，那么第一个事务前后两次读取到的数据是不同的，这样就造成了不可重复读。

3.  **Phantom Reads 幻像读** 
第一个数据正在查询符合某一条件的数据，这时，另一个事务又插入或删除了一条符合条件的数据，第一个事务在第二次查询符合同一条件的数据时，发现多了或少了一条前一次查询时没有的数据，仿佛幻觉一样，这就是幻像读。

### 不可重复度和幻像读的区别 
不可重复读是指同一查询在同一事务中多次进行，由于其他提交事务所做的修改，每次返回不同的结果集，此时发生不可重复读。

幻像读是指同一查询在同一事务中多次进行，由于其他提交事务所做的插入或删除操作，每次返回不同的结果集，此时发生幻像读。

表面上看，区别就在于不可重复读能看见其他事务提交的修改，而幻读能看见其他事务提交的插入和删除。 

>作者：小朋友
链接：https://www.zhihu.com/question/47007926/answer/253406510
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
>
>**脏读：**（同时操作都没提交的读取）脏读又称无效数据读出。一个事务读取另外一个事务还没有提交的数据叫脏读。
例如：事务T1修改了一行数据，但是还没有提交，这时候事务T2读取了被事务T1修改后的数据，之后事务T1因为某种原因Rollback了，那么事务T2读取的数据就是脏的。
解决办法：把数据库的事务隔离级别调整到READ_COMMITTED
>
>**不可重复读：**（同时操作，事务一分别读取事务二操作时和提交后的数据，读取的记录内容不一致）不可重复读是指在同一个事务内，两个相同的查询返回了不同的结果。
 例如：事务T1读取某一数据，事务T2读取并修改了该数据，T1为了对读取值进行检验而再次读取该数据，便得到了不同的结果。 
>解决办法：把数据库的事务隔离级别调整到REPEATABLE_READ
>
>**幻读：**（和可重复读类似，但是事务二的数据操作仅仅是插入和删除，不是修改数据，读取的记录数量前后不一致）
例如：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入（注意时插入或者删除，不是修改））了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样。这就叫幻读。
解决办法：把数据库的事务隔离级别调整到SERIALIZABLE_READ

###数据库隔离级别
|隔离级别	|隔离级别的值	| 导致的问题
|:---:| :---:| :---:|
|Read-Uncommitted	 |0	|导致脏读
|Read-Committed	| 1	|避免脏读，允许不可重复读和幻读
|Repeatable-Read	| 2	|避免脏读，不可重复读，允许幻读
|Serializable	| 3	|串行化读，事务只能一个一个执行，避免了脏读、不可重复读、幻读。执行效率慢，使用时慎重

**总结：**
隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。
大多数的数据库默认隔离级别为 Read Commited，比如 SqlServer、Oracle
少数数据库默认隔离级别为：Repeatable Read 比如： MySQL InnoDB

### Spring中的隔离级别（lsolation的属性值）
1. **default**

默认的事务隔离级别，使用数据库默认的事务隔离级别，另外四个与 JDBC 的隔离级别相对应。

2. **read_uncommitted**

读未提交，一个事务可以操作另外一个未提交的事务，不能避免脏读，不可重复读，幻读，隔离级别最低，并发性能最高

3. **read_committed**

读已提交，一个事务不可以操作另外一个未提交的事务， 能防止脏读，不能避免不可重复读，幻读。

4. **repeatable_read**

能够避免脏读，不可重复读，不能避免幻读

5. **serializable**

隔离级别最高，消耗资源最低，代价最高，能够防止脏读， 不可重复读，幻读。

### 隔离级别解决事务并行引起的问题 
||Dirty reads | non-repeatable reads | phantom reads |
|:---:| :---:| :---:| :---:|
|Read Uncommitted |会 |会| 会
|READ COMMITTED |不会 |会| 会 
|REPEATABLE READ |不会 |不会| 会 
|Serializable |不会 |不会 |不会 


#3. SpringMVC 的执行流程
1. 用户向服务器发送请求，请求被 Spring 前端控制器 `DispatcherServlet`捕获（**捕获**）
2. DispatcherServlet 对请求 URL 进行解析，得到请求资源标识符（URI）,然后根据该 URI 调用 HandlerMapping，获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以 HandlerExecutionChain 对象的形式返回（**查找 Handler**）  

3. DispatcherServlet 根据获得的 Handler，选择一个合适的 HandlerAdapter，提取 Request 中的模型数据，填充 Handler 入参，开始执行 Handler （Controller），Handler 执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象（**执行Handler**）
4. DispatcherServlet 根据返回的ModelAndView，选择一个合适的 ViewResolver（必须是已经注册到 Spring 容器中的 ViewResolver）（**选择 ViewResolver**）
5. 通过 ViewResolver 结合 Model 和 View ，来渲染视图，DispatcherServlet 将渲染结果返回客户端。（**渲染返回**）

>核心控制器捕获请求——>查找 Handler——>执行 Handler——>选择 ViewResolver——>通过 ViewResolver 渲染视图并返回


#4. SpringMVC 和 Struts2 的区别
1. **核心控制器（前端控制器，预处理控制器）：**核心控制器主要用途是`处理所有请求`，然后对那些特殊的请求（控制器）统一进行处理（字符编码、文件上传、参数接受、异常处理等），springMVC 核心控制器是 `Servlet`，而 Struts2 是  `Filter`。

2. **控制器实例：**SpringMVC 比 Struts2 快一些（理论上），SpringMVC 是`基于方法设计`的，而 Struts2 是`基于对象`，每次发一次请求都会实例一个action，每个action 都会被注入属性，而 SpringMVC 更像 Servlet 一样，`只有一个实例，每次请求执行对应的方法即可`（注意：由于是单例实例，所以应当避免全局变量的修改，这样会产生线程安全问题）

3. **管理方式：**SpringMVC 是Spring 中的一个模块，所以 Spring 对 SpringMVC 的控制器管理更加简单方便，而且提供了全注解方式进行管理，各种功能的注解比较全面，使用简单；而 Struts2 需要采用 XML 很多的配置参数来管理（虽然也可以使用注解）。

4. **参数传递：**Struts2 自身提供多种参数接受，其实都是通过`ValueStack` 进行传递和赋值（重量级），而 SpringMVC 是通过`方法的参数`进行接收（轻量级）。

5. **学习难度：** Struts2 加了很多新的技术点，比如拦截器、值栈及 OGNL 表达式，学习成本较高；而 SpringMVC 比较简单，容易上手。

6. **Interceptor 的实现机制：** struts2 有`自己的 interceptor 机制`，spring mvc 用的是`独立的 AOP 方式`，这样导致 struts2 的配置文件量还是比 spring mvc 大，虽然 struts2 的配置能继承。所以从使用上来讲，spring mvc 使用更加简洁，开发效率 spring mvc 比 struts2 更高。

spring mvc 是`方法级别`的拦截，一个方法对应一个 request 上下文， 而方法同时又跟一个 url 对应，所以从架构本身上 spring mvc 就容易实现 restful url。
struts2 是`类级别`的拦截，一个类对应一个 request 上下文，实现 restful url 要费劲，因为 struts2 action 的一个方法可以对应一个 url ，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。

spring mvc `方法之间基本上是独立的`，`独享request 和 response 数据`，请求数据通过参数获取，处理结果通过 ModelMap 交给框架，方法之间不共享变量；而 struts2 搞的就比较乱，虽然方法之间也是独立的，但其所有action 变量是共享的，虽不会影响程序运行，但给我们编码，读程序时带来麻烦。

7.  spring mvc 处理 ajax 请求，直接通过返回数据，方法中使用注解 @ResponseBody，spring mvc 自动帮我们将对象转为 JSON 数据。
而 struts2 是通过插件的方式进行处理。 

#5. 简单介绍下 Spring 或 Spring 的两大核心

spring 是J2EE 应用框架，是轻量级的 IOC 和 AOP 的容器框架，主要针对 JavaBean 的生命周期进行管理的轻量级容器，可以单独使用，也可以和 struts2 框架，mybatis 框架等组合使用。

1. IOC（Inversion of Controller）或 DI （Denpendency Injection）：控制反转

原来：service 需要调用 dao，service就需要创建 dao
现在：spring 发现 service 依赖于 dao就 主动给你注入
**核心原理：** 配置文件 + 反射（或工厂）+ 容器 （map）

2.AOP：面向切面编程
**核心原理：**使用动态代理的设计模式在执行方法前后或出现异常后做相关逻辑

我们主要使用AOP来做：
- 事务处理：执行方法前，开启事务；执行完成后关闭事务；出现异常后回滚事务
- 权限判断：在执行方法前，判断是否具有权限
- 日志：在执行前进行日志处理
- ...

#6. Spring的BeanFactory与ApplicationContext区别
ApplicationContext和BeanFacotry相比,提供了更多的扩展功能，但其主要区别在于后者是延迟加载,如果Bean的某一个 属性 没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常;而ApplicationContext则在初始化自身是 检验，这样有利于检查所依赖属性是否注入;所以通常情况下我们选择使用ApplicationContext。
