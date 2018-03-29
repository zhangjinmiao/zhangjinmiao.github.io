[TOC]

 																						   

# 第1章 简介

## 1.1 什么是redis

Redis—— Remote Dictionary Server，它是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API，我们可使用它构建高性能，可扩展的Web应用程序。

Redis是目前最流行的键值对存储数据库，从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。

如果你想了解Redis最新的资讯，可以访问 [[官方网站\]:]()<http://redis.io/>。

中文社区：<http://www.redis.cn/>

## 1.2 应用场景

在实际生产环境中，很多公司都曾经使用过这样的架构，使用MySQL进行海量数据存储的，通过Memcached将热点数据加载到cache，加速访问，但随着业务数据量的不断增加，和访问量的持续增长，我们遇到了很多问题：

- MySQL需要不断进行拆库拆表，Memcached也需不断跟着扩容，扩容和维护工作占据大量开发时间。
- Memcached与MySQL数据库数据一致性问题。
- Memcached数据命中率低或down机，大量访问直接穿透到DB，MySQL无法支撑。
- 跨机房cache同步问题。

以上问题都是非常的棘手，不过现在不用担心了，因为我们可以使用redis来完美解决。



1. **会话缓存（Session Cache）**
   Redis缓存会话有非常好的优势，因为Redis提供持久化，在需要长时间保持会话的应用场景中，如购物车场景这样的场景中能提供很好的长会话支持，能给用户提供很好的购物体验。

2. **全页缓存**
   在WordPress中，Pantheon提供了一个不错的插件`wp-redis`，这个插件能以最快的速度加载你曾经浏览过的页面。

3. **队列**
   Reids提供list和set操作，这使得Redis能作为一个很好的消息队列平台来使用。

   我们常通过Reids的队列功能做购买限制。比如到节假日或者推广期间，进行一些活动，对用户购买行为进行限制，限制今天只能购买几次商品或者一段时间内只能购买一次。也比较适合适用。

4. **排名**
   Redis在内存中对数字进行递增或递减的操作实现得非常好。所以我们在很多排名的场景中会应用Redis来进行，比如小说网站对小说进行排名，根据排名，将排名靠前的小说推荐给用户。

5. **发布/订阅**
   Redis提供发布和订阅功能，发布和订阅的场景很多，比如我们可以基于发布和订阅的脚本触发器，实现用Redis的发布和订阅功能建立起来的聊天系统。

   ​

## 1.3 **特点**

相对于其他的同类型数据库而言，Redis支持更多的数据类型，除了和string外，还支持lists（列表）、sets（集合）和zsets（有序集合）几种数据类型。  这些数据类型都支持push/pop、add/remove及取交集、并集和差集及更丰富的操作，而且这些操作都是原子性的。Redis具备以下**特点**：  

1. **异常快速**: Redis数据库完全在内存中，因此处理速度非常快，每秒能执行约11万集合，每秒约81000+条记录。 
2. **数据持久化**： redis支持数据持久化，可以将内存中的数据存储到磁盘上，方便在宕机等突发情况下快速恢复。 
3. **支持丰富的数据类型**: 相比许多其他的键值对存储数据库，Redis拥有一套较为丰富的数据类型。 
4. **数据一致性**： 所有Redis操作是**原子**的，这保证了如果两个客户端同时访问的Redis服务器将获得更新后的值。
5. **多功能实用工具**： Redis是一个多实用的工具，可以在多个用例如缓存，消息，队列使用(Redis原生支持发布/订阅)，任何短暂的数据，应用程序，如 Web应用程序会话，网页命中计数等。
6. **Redis支持数据的备份**，即master-slave模式的数据备份。

## 1.4 Redis与其他key-value存储有什么不同？

Redis有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis的数据类型都是基于**基本数据结构**的同时对程序员透明，无需进行额外的抽象。

Redis运行在内存中但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方面的另一个优点是， 相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。 同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。

# 第2章 详细介绍

## 2 .1  **组件介绍**

redis-benchmark、redis-check-dump、redis-sentinel、redis-check-aof、redis-cli、redis-server构成了redis软件包，下面我们来了解一下它们分别是用来干什么的。  

|       组件        |                   用途                    |
| :-------------: | :-------------------------------------: |
|  redis-server   |             Redis服务器的启动程序。              |
|    redis-cli    | Redis命令行操作工具。当然，你也可以用telnet根据其纯文本协议来操作。 |
| redis-benchmark |   Redis性能测试工具，测试Redis在你的系统及你的配置下的读写性能   |
|   redis-stat    |    Redis状态检测工具，可以检测Redis当前状态参数及延迟状况。    |

本课程着重讲解redis的入门知识，因此我们将只会使用到redis-server(启动服务)和redis-cli(连接服务器进行操作)，下面我们来学习如何启动redis服务。

## 2.2 启动redis

启动redis 服务非常的简单： 

`root@localhost:~# src/redis-server`

启动redis服务时可以指定很多配置参数，一般情况下，我们会将配置参数写到config文件中，在启动 redis 服务时，指定配置文件即可。  

redis软件包中提供了一个默认的配置文件redis.config，当我们需要指定配置文件启动时，需要按照如下方式启动： 

```
root@localhost:~# src/redis-server redis.config
```

redis服务启动以后，我们可以使用redis-cli工具来连接 redis 服务。  

```
root@localhost:~# src/redis-cli
127.0.0.1:6379>
```

## 2.3 **配置文件**

**基本参数** 

redis 服务相关参数都需要在redis.config文件中进行配置，所以我们有必要花点时间来简单了解一下 config 文件中的基本参数。

|     参数      |           作用           |
| :---------: | :--------------------: |
|  daemonize  | 是否以后台daemon方式运行redis服务 |
| port redis  |      服务端口，默认6379       |
|   Timeout   |         请求超时时间         |
| requirepass |        连接数据库密码         |

redis.config中daemonize参数默认为no，为了让redis服务在后台运行，我们需要将daemonize参数设置为yes。



# 第3章  命令

## 3.1 String字符串

### 添加键值 SET

string 是 redis 中最基础的数据类型， redis 字符串是二进制安全的，这意味着他们有一个已知的长度没有任何特殊字符终止，所以你可以存储任何东西，512兆为上限。  

SET指令是将字符串值 value 关联到 key 。

语法格式：  

```
SET key value EX seconds [PX milliseconds] [NX|XX]
```

示例：添加键page，值为‘hubwiz’。  

```
redis> SET page "hubwiz" OK
```

如果 key 已经持有其他值，SET就覆写旧值，无视类型。因此，对于某个原本带有生存时间（TTL）的键来说， 当SET命令成功在这个键上执行时， 这个键原有的 TTL 将被清除。

### 添加值和生成时间 SETEX

SETEX指令的作用是将值 value 关联到 key ，并将 key 的生存时间设为 seconds (以秒为单位)。如果 key 已经存在， SETEX命令将覆写旧值,语法格式：  

```
SETEX key seconds value
```

示例 - 设置page的值为‘hubwiz’，生存时间为60秒。  

```
redis> SETEX page 60 "hubwiz" OK
```

SETEX指令的作用类似如下两个命令：  

```
SET name "hubwiz"
EXPIRE key 60  # 设置生存时间 
```

不同之处是，SETEX是一个原子性(atomic)操作， 关联值 和 设置生存时间 两个动作会在同一时间内完成，该命令在 Redis 用作缓存时，非常实用。

### 获取字符串 GET

GET指令是返回 key 所关联的字符串值。如果 key 不存在那么返回特殊值 nil 。假如 key 储存的值不是字符串类型，返回一个错误，因为GET只能用于处理字符串值，语法格式：  

```
GET key 
```

示例 - 获取page和test的值。  

```
redis> GET page 
"hubwiz" 
redis> GET test
(nil) 
```

***返回值***  

当key不存在时，返回nil，否则返回key的值。 如果key的值不是字符串类型，那么将会返回一个错误。

### 追加字符串 APPEND

如果 key已经存在并且是一个字符串，APPEND命令将 value 追加到 key 原来的值的末尾，语法格式：  

```
APPEND key value
```

示例 - 向page追加字符‘.com’。

```
redis> APPEND page ".com" # 对已存在的字符串进行 APPEND
(integer) 10
redis> GET name
"hubwiz.com" 
```

如果 key 不存在，APPEND就简单地将给定 key 设为 value ，就像执行 SET key value 一样。

### 添加多个键值 MSET

MSET指令可以同时设置一个或多个 key-value 对，如果某个给定 key 已经存在，那么MSET会用新值覆盖原来的旧值,

语法格式：     

```
MSET key value [key value ...] 
```

示例 - 设置date、time和weather的值。  

```
redis> MSET date "2015.5.10" time "11:00 a.m." weather "sunny" 
OK 
```

MSET是一个**原子性(atomic)**操作，所有给定 key 都会在同一时间内被设置，某些给定 key 被更新而另一些给定 key 没有改变的情况，不可能发生。

### 获取多个键值 MGET 

执行MGET指令，将返回所有(一个或多个)给定 key 的值,

语法格式：  

```
MGET key [key ...]
```

示例 - 获取date、time、weather、year的值。  

```
redis> MGET date time weather year 
1) "2015.5.10" 
2) "11:00 a.m."
3) "sunny" 
4）(nil) 
```

如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回特殊值 nil 。因此，该命令永不失败。

### 覆写 SETRANGE

指令是用 value 参数覆写(overwrite)给定 key 所储存的字符串值，从偏移量offset开始。不存在的key当作空白字符串处理,

语法格式：  

```
SETRANGE key offset value
```

示例 - 覆写say的值。  

```
redis> SET say "hello world" OK
redis> SETRANGE say 6 "Redis" (integer) 11 
redis> GET say "hello Redis"
```

SETRANGE命令会确保字符串足够长以便将 value 设置在指定的偏移量上，如果给定 key 原来储存的字符串长度比偏移量小(比如字符串只有 5 个字符长，但你设置的 offset 是 10 )，那么原字符和偏移量之间的空白将用零字节(zerobytes, "\x00" )来填充。  注意你能使用的最大偏移量是 2^29-1(536870911) ，因为 Redis 字符串的大小被限制在 512 兆(megabytes)以内。如果你需要使用比这更大的空间，你可以使用多个 key 。

## 3.2 什么是hash

### 添加 HSET

在 redis 中，使用HSET命令来将哈希表 key 中的域 field 的值设为 value ，

语法如下：

```
HSET key field value
```

示例 - 添加键hubwiz，值为‘hubwiz.com’。

```
redis> HSET site huwiz "hubwiz.com" # 设置一个新域
(integer) 1
```

如果 key 不存在，一个新的哈希表被创建并进行HSET操作。

如果域 field 已经存在于哈希表中，旧值将被覆盖。

### 添加多个 HMSET

一次可以设置多个 field-value (域-值)对设置到哈希表 key 中,

语法如下：

```
HMSET key field value [field value ...]
```

示例 - 添加键www、lab。

```
redis> HMSET site www www.hubwiz.com lab lab123.hubwiz.com
OK
```

如果 key 不存在，将会创建一个空的哈希表并执行HMSET操作。

如果添加的域已存在哈希表中，那么它将被覆盖。

### 获取 HGET

HGET是用来获取指定 key 值的命令，

语法如下:

```
HGET key field
```

示例 - 获取域hubwiz的值。

```
redis> HGET site hubwiz
"www.hubwiz.com"
```

执行HGET命令，如果 key 存在，将返回哈希表 key 中给定域 field 的值，如果 key 不存在，则返回 (nil) 。

### 获取多个 HMGET

作为HMSET命令对应的获取命令，HMGET可以一次性获取哈希表 key 中，一个或多个给定域的值，基本语法：

```
HMGET key field [field ...]
```

示例 - 获取域www、lab、test的值。

```
redis> HMGET site www lab test # 返回值的顺序和传入参数的顺序一样
1) "www.hubwiz.com"
2) "lab.hubwiz.com"
3) (nil)  # 不存在的域返回nil值
```

如果给定的域不存在于哈希表，那么返回一个 nil 值。

因为不存在的 key 被当作一个空哈希表来处理，所以对一个不存在的 key 进行HMGET操作将返回一个只带有 nil 值的表。

### 获取全部值 HGETALL

基本语法如下：

```
HGETALL key
```

示例 - 获取people全部域的值。

```
redis> HSET people jack "Jack Sparrow"
(integer) 1
redis> GET name
redis> HSET people gump "Forrest Gump"
(integer) 1

redis> HGETALL people
1) "jack"          # 域
2) "Jack Sparrow"  # 值
3) "gump"		   # 域
4) "Forrest Gump"  # 值
```

在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

### 是否存在 HEXISTS

在应用环境中，我们经常会需要知道一个 key 中是否存在某个 field ，HEXISTS命令可以帮助我们达到这个目的，

基本语法：

```
HEXISTS key field
```

示例 - 验证键www是否存在。

```
redis> HEXISTS site www 
(integer) 0
```

查看哈希表 key 中，给定域 field 是否存在。

如果哈希表含有给定域，返回 1 。

如果哈希表不含有给定域，或 key 不存在，返回 0 。

### 获取域 HKEYS

我们经常会遇见这样的应用场景，比如在线用户列表、课堂列表等等，这时候我们可以使用HKEYS来获取哈希表 key 中的所有域,

基本语法：

```
HKEYS key
```

示例 - 查看键people中所有的域。

```
redis> HMSET people jack "Jack Sparrow" gump "Forrest Gump"
OK

redis> HKEYS people
1) "jack"
2) "gump"
```

当 key 存在时，将返回一个包含哈希表中所有域的表。 当 key 不存在时，返回一个空表。

### 返回域数量 HLEN

HLEN命令将返回哈希表 key 中域的数量,什么时候会用到它呢？比如：在线聊天室，显示在线用户数，

基本语法：

```
HLEN key
```

示例 - 查看db键中域的个数。

```
redis> HSET db redis redis.com
(integer) 1

redis> HSET db mysql mysql.com
(integer) 1

redis> HLEN db
(integer) 2

redis> HSET db mongodb mongodb.org
(integer) 1

redis> HLEN db
(integer) 3
```

当 key 存在时，将返回哈希表中域的数量。 当 key 不存在时，返回 0 。

### 删除域 HDEL

有添加就必定有删除的需求，当我们想要删除哈希表 key 中的一个或多个指定域时，可以使用HDEL命令，

基本语法：

```
HDEL key field [field ...]
```

示例 - 删除键people中的jack域。

```
redis> HDEL people jack
(integer) 1
```

如果是不存在的域，那么它将被忽略掉。

## 3.3 列表List

 **LPUSH** 命令插入一个新的元素到头部, 而**RPUSH**插入一个新元素到尾部.当一个这两个操作在一个空的Key上被执行的时候一个新的列表被创建。

一个列表最多可以包含 23的平方 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。 从时间复杂度的角度来看 Redis 列表的主要特征是在头和尾的元素插入和删除是固定时间，即便是数以百万计的插入。在列表的两端访问元素是非常快的但是如果你试着访问一个非常大的列表的中间的元素是很慢的，因为那是一个O(N)操作。

**列表操作：**

- 在一个社交网络中建立一个时间线模型，使用 **LPUSH **去添加新的元素到用户的时间线，使用 **LRANGE** 去接收一些最近插入的元素。
- 你可以将 **LPUSH **和 **LTRIM** 一起用去创建一个永远也不会超过指定元素数目的列表，但是记住是最后的N个元素。
- 列表能够被用来作为消息传递primitive。

### 插入 LPUSH

LPUSH的作用是将一个或多个值 value 插入到列表 key 的表头，基本语法：

```
LPUSH key value [value ...]
```

示例：将Tony添加到朋友列表。

```
redis> LPUSH friends Tony
```

如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表头,比如说，对空列表 mylist 执行命令 LPUSH mylist a b c ，列表的值将是 c b a 。

如果 key 不存在，一个空列表会被创建并执行LPUSH操作。

执行成功时，返回列表长度，当 key 存在但不是列表类型时，返回一个错误。

### 设值 LSET

LSET可以将列表 key 下标为index的元素的值设置为 value

 基本语法：

```
LSET key index value
```

示例

```
redis>LSET friends 0 Lucy
ok
```

需要注意的是，列表 key 必须是已存在的，而且index不能超出列表长度范围。

### 移除第一个元素 LPOP

LPOP命令执行时会移除列表第一个元素，并将其返回,

基本语法：

```
LPOP key
```

示例 - 取出friends中的第一个元素。

```
redis>LPOP friends
Lucy
```

请注意，LPOP命令会移除列表中的元素，如果仅仅是想要获取该元素，那么就不应该使用LPOP操作，因为redis中有专门获取元素的命令。

### 获取 LINDEX

如果要获取列表元素，LINDEX命令是比较常用的，使用LINDEX，我们可以获取到指定位置的 value ，

基本语法：

```
LINDEX key index
```

示例 - 获取friends的第一个元素。

```
redis>LINDEX friends 0
Tony
```

下标 (index)为正数时，0表示第一个元素，1表示第二个元素，以此类推。

下标 可以是负数，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

### 插入元素 Linsert

插入元素是一个必要功能，LINSERT可以将值 value 插入到列表 key 当中，位于值 pivot 之前或之后，

基本语法：

```
LINSERT key BEFORE|AFTER pivot value
```

示例 - 将Andy插入到Lucy之前。

```
redis> LINSERT friends BEFORE "Lucy" "Andy"
(integer) 3
```

当 pivot 不存在于列表 key 时，不执行任何操作。

当 key 不存在时， key 被视为空列表，不执行任何操作。

如果 key 不是列表类型，返回一个错误。

### 移除 LREM

在redis中，移除列表元素使用LREM命令，根据参数 count 的值，移除列表中与参数 value 相等的元素，基本语法：

```
LREM key count value
```

示例 - 移除friends中，所有的名叫‘Tom’的元素。

```
redis> LREM friends 0 Tom
(integer) 1
```

count 的值可以是以下几种：

count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。

count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。

count = 0 : 移除表中所有与 value 相等的值。

### 返回长度 LLEN

现在friends列表中记载着我所有朋友的名字，可是要怎样才能知道我现在拥有多少个朋友呢？

在redis中，LLEN命令可以获取到列表的长度，基本语法：

```
LLEN key
```

示例 - 查看myList列表长度。

```
redis> LLEN mylist
(integer) 0
```

返回列表 key 的长度。

如果 key 不存在，则 key 被解释为一个空列表，返回 0 。

如果 key 不是列表类型，返回一个错误。

### 选取 LTRIM

LTRIM可以对一个列表进行修剪，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除，基本语法：

```
LTRIM key start stop
```

示例 - 只保留列表 list 的前三个元素，其余元素全部删除。

```
LTRIM list 0 2
```

下标(index)参数start和stop都以 0 为底，也就是说，以 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

## 3.4 集合Set

Redis 集合（Set）是一个无序的字符串集合. 你可以快速的完成**添加**、**删除**、以及测试元素**是否存在**。

Redis 集合拥有令人满意的不允许包含相同成员的属性。多次添加相同的元素，最终在集合里只会有一个元素。 实际上说这些就是意味着在添加元素的时候无须检测元素是否存在。 一个Redis集合的非常有趣的事情是他支持一些服务端的命令从现有的集合出发去进行集合运算，因此你可以在非常短的时间内进行合并（unions）, 求交集（intersections）,求差集（differences of sets）。

**操作：**

- Redis 集合是很擅长表现关系的。你可以使用Redis集合创建一个tagging系统去表现每一个tag。接下来你能够使用SADD命令将有一个给定tag的所有对象的所有ID添加到一个用来展现这个特定tag的集合里。你想要同时有三个不同tag的所有对象的ID吗？使用SINTER就好了。
- 使用 SPOP 或者 SRANDMEMBER 命令你可以使用集合去随意的抽取元素。

### 添加 SADD

集合操作中，SADD命令可以将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略，

基本语法：

```
SADD key member [member ...]
```

示例 - 添加‘tom’到room集合中。

```
redis> SADD room "tom"
(integer) 1
```

假如 key 不存在，则创建一个只包含 member 元素作成员的集合。

当 key 不是集合类型时，返回一个错误。

### 移除 SPOP

如果我们需要随机取出集合中的某个元素，可以使用SPOP命令，基本语法：

```
SPOP key
```

示例:随机取出website集合中的元素。

```
redis> SPOP website
"hubwiz.com"
```

需要注意的是，执行SPOP命令返回的元素将被移除该集合。

### 获取全部 SMEMBERS

如果要获取集合中全部的元素，则需要使用SMEMBERS命令，

基本语法如下：

```
SMEMBERS key
```

示例 - 获取website集合中全部的元素。

```
redis> SMEMBERS website  #获取website集合中全部的元素
1) "www.hubwiz.com"
2) "lab123.hubwiz.com"
```

SMEMBERS命令只会返回集合中的全部成员，并不会移除它们，如果集合不存在，则视为空集合。

### 获取数量 SCARD

如果想要查看集合中元素的数量，可以使用SCARD命令，

基本语法：

```
SCARD key
```

示例 - 查看website集合中元素的数量。

```
redis>SCARD website
(integer) 3
```

执行SCARD命令，当集合存在时，返回集合中元素的数量，若集合不存在，则返回0。

### 差集 SDIFF

假如现在有两个集合，我们想要获取到它们之间不同的元素，通常情况下，我们需要通过循环集合来比较，然后取得不同的元素。在redis里面取得集合的差集非常简单，通过SDIFF命令即可轻松实现，

基本语法：

```
SDIFF key [key ...]
```

示例 - 取得mySet1和mySet2的差集。

```
redis> SMEMBERS mySet1
1) "bet man"
2) "start war"
3) "2012"

redis> SMEMBERS mySet2
1) "hi, lady"
2) "Fast Five"
3) "2012"

redis> SDIFF mySet1 mySet2
1) "bet man"
2) "start war"
```

如果 key 都存在，则返回一个集合的全部成员，该集合是所有给定集合之间的差集。

不存在的 key 被视为空集。

### 交集 SINTER

在 redis 中获取集合的交集也是非常简单的，执行SINTER命令将返回集合的交集，

基本语法：

```
SINTER key [key ...]
```

示例 - 获取集合group1和group2的交集。

```
redis> SMEMBERS group1
1) "LI LEI"
2) "TOM"
3) "JACK"

redis> SMEMBERS group2
1) "HAN MEIMEI"
2) "JACK"

redis> SINTER group1 group2
1) "JACK"
```

当集合都存在时，将返回一个集合的全部成员，该集合是所有给定集合的交集。

不存在的集合被视为空集。因此，当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。

### 并集 SUNION

既然有差集和交集运算，当然少不了并集，在 redis 中，执行SUNION命令将返回给定集合的并集，基本语法：

```
SUNION key [key ...]
```

示例：获取集合songs和my_songs的并集。

```
redis> SMEMBERS songs
1) "Billie Jean"

redis> SMEMBERS my_songs
1) "Believe Me"

redis> SUNION songs my_songs
1) "Billie Jean"
2) "Believe Me"
```

如果给定的集合都存在，则返回一个集合的全部成员，该集合是所有给定集合的并集。

同样，不存在的集合被视为空集。

### 是否包含 SISMEMBER

如果要判断集合是否包含某个元素也不需要循环对比了，因为 redis 提供SISMEMBER命令可以实现这个功能，基本语法：

```
SISMEMBER key member
```

示例 - 判断 member 元素是否集合 key 的成员。

```
redis> SMEMBERS website
1) "hubwiz.com"
2) "google.com"
3) "baidu.com"

redis> SISMEMBER website "hubwiz.com"
(integer) 1
```

如果集合包含给定的元素，则返回1，反之则返回0。

### 移动 SMOVE

执行SMOVE可以移动元素，基本语法：

```
SMOVE source destination member
```

将 member 元素从 source 集合移动到 destination 集合。SMOVE是**原子性**操作，因此可以保证数据的一致性。

示例 - 将songs集合中的歌曲‘Believe Me’移动到‘my_songs’集合。

```
redis> SMEMBERS songs
1) "Billie Jean"
2) "Believe Me"

redis> SMEMBERS my_songs
(empty list or set)

redis> SMOVE songs my_songs "Believe Me"
(integer) 1

redis> SMEMBERS songs
1) "Billie Jean"

redis> SMEMBERS my_songs
1) "Believe Me"
```

如果 source 集合不存在或不包含指定的 member 元素，则SMOVE命令不执行任何操作，仅返回 0 。否则， member 元素从 source 集合中被移除，并添加到 destination 集合中去。

当 destination 集合已经包含 member 元素时，SMOVE命令只是简单地将 source 集合中的 member 元素删除。

当 source 或 destination 不是集合类型时，返回一个错误。



### 移除 SREM

---

执行命令SREM可以将元素从集合中移除，基本语法：

```
SREM key member [member ...]
```

示例 - 从languages集合中移除ruby。

```
redis> SMEMBERS languages    # 测试数据
1) "c"
2) "lisp"
3) "python"
4) "ruby"

redis> SREM languages ruby    # 移除单个元素
(integer) 1
```

移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略。

当 key 不是集合类型，返回一个错误。

## 3.5 有序集合ZSet

有序集合的每个成员都关联了一个评分，这个评分被用来按照从**最低分到最高分**的方式排序集合中的成员。集合的成员是唯一的，但评分可以重复。

使用有序集合你可以以非常快的速度 添加 、 删除 和 更新 元素。因为元素是有序的, 所以可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

**作用：**

- 在一个大型的在线游戏中展示一个排行榜，在那里一旦一个新的分数被提交，你可以使用**ZADD**命令去更新它.你也可用使用**ZRANGE**命令来得到顶级的用户,你还可以使用ZRANK命令根据用户名返回该用户在排行榜中的位次。同时使用**ZRANK** 和**ZRANGE**你可以显示和给定用户分数相同的所有用户。所有这些操作都非常的快速。
- 有序集合常常被用来索引存储在 Redis 中的数据。举个例子，如果你有许多的哈希（Hashes）来代表用户，你可以使用一个有序集合，这个集合中的元素的年龄字段被用来当做评分，而ID作为值。因此，使用**ZRANGEBYSCORE**命令，那是微不足道的并且能够很快的接收到给定年龄段的所有用户。
- 有序集合或许是最高级的Redis数据类型，因此花点时间查看完整的有序集合命令列表 去发现你能用 Redis 做些什么。****

### 添加 ZADD

---

在redis中，使用ZADD命令将一个或多个 member 元素及其 score 值加入到有序集 key 当中，基本语法：

```
ZADD key score member [score member ...]
```

示例 - 添加google.com到rank集合，评分10。

\# 添加单个元素

```
redis> ZADD rank 10 google.com
(integer) 1
```

如果某个 member 已经是有序集的成员，那么更新这个 member 的 score 值，并通过重新插入这个 member 元素，来保证该 member 在正确的位置上。

score 值可以是整数值或双精度浮点数。

如果 key 不存在，则创建一个空的有序集并执行ZADD操作。

当 key 存在但不是有序集类型时，返回一个错误。



### 移除 ZREM

------

*ZREM*命令可以移除指定成员，基本语法如下：

```
ZREM key member [member ...]
```

**示例 **- 移除rank集合中google.com元素。

```
redis> ZREM rank google.com
```

执行成功，google.com元素将从rank集合中移除，如果google.com不存在，将被忽略。

如果 **rank **集合存在但不是有序集类型时，返回一个错误。



### 获取评分 ZSCORE

------

redis中使用*ZSCORE*命令来获取成员评分，基本语法：

> ```
> ZSCORE key member
> ```

**示例 **- 获取集合salary中元素peter的评分。

```
redis> ZSCORE salary peter              # 注意返回值是字符串
```

如果 **peter** 是集合 **salary** 的成员，则返回成员 **peter **的评分值。

如果 **peter** 元素不是有序集 **salary** 的成员，或 salary 不存在，则返回 **nil**。



### 获取成员 ZRANGE

------

如果想要获取集合成员，可以使用*ZRANGE*命令，基本语法：

```
ZRANGE key start stop [WITHSCORES]
```

**示例 **- 获取salary集合的全部成员。

```
redis > ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"
```

返回的成员的位置按 score 值递增(从小到大)来排序。



### 查看数量 ZCARD

------

如果需要查看集合成员的数量，那么我们需要使用到*ZCARD*命令，基本语法如下：

```
ZCARD key
```

**示例 **- 查看salary集合的成员数量。

```
redis > ZCARD salary
(integer) 2
```

执行成功，将返回有序集** salary **的成员总数。



### 查看指定评分的数量 ZCOUNT

------

除了*ZCARD*命令以外，*ZCOUNT*命令也可以查看成员的数量，和前者不同的是，*ZCOUNT*命令可以设定评分的最小和最大值：

```
ZCOUNT key min max
```

**示例 **- 查看薪水在2000-5000之间的人数。

```
redis> ZRANGE salary 0 -1 WITHSCORES    # 测试数据
1) "jack"
2) "2000"
3) "peter"
4) "3500"
5) "tom"
6) "5000" 

redis> ZCOUNT salary 2000 5000 
(integer) 3
```

执行成功，将返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的成员的数量。



### 获取元素排名 ZRANK

------

*ZRANK*命令可以获取到给定元素在集合中的排名，排名依据 **评分（score）**值递增(从小到大)顺序排列，语法格式：

> ZRANK key member

**示例 **- 显示 tom 的薪水排名。

```
redis> ZRANGE salary 0 -1 WITHSCORES        # 显示所有成员及其 score 值
1) "peter"
2) "3500"
3) "tom"
4) "4000"
5) "jack"
6) "5000" 

redis> ZRANK salary tom   
(integer) 1
```

排名以 0 为底，也就是说， score 值最小的成员排名为 0 。

使用 ZREVRANK 命令可以获得成员按 score 值递减(从大到小)排列的排名。



### 评分增量 ZINCRBY

------

*ZINCRBY*命令可以为给定的成员评分值加上增量，语法格式：

> ZINCRBY key increment member

**示例 **- 为salary集合中的tom加薪500。

```
redis> ZSCORE salary 
tom"2000" 

redis> ZINCRBY salary 500 tom   # tom 加薪啦！
"2500"
```

可以通过传递一个负数值 increment ，让 score 减去相应的值，比如 ZINCRBY key -5 member ，就是让 member 的 score 值减去 5 。

当 key 不存在，或 member 不是 key 的成员时， ZINCRBY key increment member 等同于 ZADD key increment member 。

当 key 不是有序集类型时，返回一个错误。

score 值可以是整数值或双精度浮点数。





## 3.6 键管理 Keys

### Keys

**Redis **的*keys*命令用于管理键。使用 *Redis* 的*keys*命令,语法格式：

> KEYS pattern

查找所有符合给定模式 pattern 的 key 。

**示例 **- 查找包含o的键。

```
redis> MSET one 1 two 2 three 3 four 4  # 一次设置 4 个 key
OK 

redis> KEYS *o*
1) "four"
2) "two"
3) "one"
```

KEYS * 匹配数据库中所有 key 。

KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。

KEYS h*llo 匹配 hllo 和 heeeeello 等。

KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo 。

特殊符号用 \ 隔开。



### EXISTS

------

*EXISTS*命令的作用是判断指定key是否存在，语法格式如下：

> EXISTS key

**示例 **- 判断db是否存在。

```
redis> SET db "redis"
OK 

redis> EXISTS db
(integer) 1 

redis> DEL db
(integer) 1 

redis> EXISTS db
(integer) 0
```

若 key 存在，返回 1 ，否则返回 0 。



## MOVE

------

MOVE命令的作用是将当前数据库的 key 移动到给定的数据库 db 当中,语法如下：

> MOVE key db

如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。

因此，也可以利用这一特性，将 MOVE 当作锁(locking)原语(primitive)。

**示例 **- 将数据库0中的song，移动到数据库1。

```
# key 存在于当前数据库 
redis> SELECT 0     # redis默认使用数据库 0，为了清晰起见，这里再显式指定一次。
OK 

redis> SET song "secret base - Zone"
OK 

redis> MOVE song 1    # 将 song 移动到数据库1
(integer) 1 

redis> EXISTS song    # song 已经被移走
(integer) 0 

redis> SELECT 1   # 使用数据库 1
OK 

redis:1> EXISTS song     # 证实 song 被移到了数据库 1 (注意命令提示符变成了"redis:1"，表明正在使用数据库 1)
(integer) 1
```



## RENAME

------

*RENAME*命令可以将原有的** key **修改为新的key名称，语法如下：

> RENAME key newkey

**示例 **- 重名message。

```
redis> SET message "hello world"OK redis> RENAME message greeting
OK 

redis> EXISTS message        # message 不复存在
(integer) 0 

redis> EXISTS greeting       # greeting 取而代之
(integer) 1
```

当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。

当 newkey 已经存在时， RENAME 命令将覆盖旧值。



## SORT

------

排序是很常见的需求，在 **redis** 中可以使用*SORT*命令来实现排序，语法格式如下：

> SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]

**示例**

```
# 开销金额列表 
redis> LPUSH cost 30 1.5 10 8
(integer) 4 

# 排序 
redis> SORT cost
1) "1.5"
2) "8"
3) "10"
4) "30"
```

使用*SORT*命令，可以返回或保存给定列表、集合、有序集合 key 中经过排序的元素。

排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。





## DUMP

**redis **支持序列化，使用*DUMP*命令来序列化给定key的值，语法如下：

> DUMP key

**示例 **- 序列化say。

```
redis> SET say "hello world!"
OK 

redis> DUMP say
"\x00\x15hello world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde" 

redis> DUMP not-exists-key
(nil)
```

执行DUMP命令序列化成功后，将返回被序列化的值，若key不存在，则返回** nil **。





## EXPIRE

------

为key设置生存时间需要使用*EXPIRE*命令，语法格式：

> EXPIRE key seconds

**示例 **- 设置rank的过期时间为30秒。

```
redis> EXPIRE rank 30  # 设置过期时间为 30 秒
(integer) 1
```

为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。

在 Redis 中，带有生存时间的 key 被称为『易失的』(volatile)。

生存时间可以通过使用 DEL 命令来删除整个 key 来移除，或者被 SET 和 GETSET 命令覆写(overwrite)，这意味着，如果一个命令只是修改(alter)一个带生存时间的 key 的值而不是用一个新的 key 值来代替(replace)它的话，那么生存时间不会被改变。



## TTL

------

*TTL*命令的作用是获取给定** key **剩余生存时间(TTL, time to live)，语法格式如下：

> TTL key

**示例 **- 查看key剩余生存时间。

```
# 有剩余生存时间的 key 
redis> EXPIRE key 10086
(integer) 1 

redis> TTL key
(integer) 10084
```

当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间。

# 第4章 安装

## 4.1 Windows下安装

<https://github.com/ServiceStack/redis-windows>

### 启动

```
redis-server.exe redis.windows.conf
```

### 将Redis作为windows服务启动

#### 安装为window服务：

```
redis-server --service-install redis.windows.conf
```

#### 启动：

```
redis-server --service-start
```

#### 停止：

```
redis-server --service-stop
```

**安装多个实例**：

```
redis-server --service-install –service-name redisService1 –port 10001
redis-server --service-start –service-name redisService1
redis-server --service-install –service-name redisService2 –port 10002
redis-server --service-start –service-name redisService2
redis-server --service-install –service-name redisService3 –port 10003
redis-server --service-start –service-name redisService3
```

#### 卸载：

```
redis-server --service-uninstall
```

## 4.2 Linux下安装 





## 4.3 Unbuntu安装

### 1. 下载文件

```
root@localhost:~# wget http://download.redis.io/releases/redis-3.0.0.tar.gz
```

### 2.  解压

```
root@localhost:~# tar zxvf redis-3.0.tar.gz
```

### 3. 编译

```
root@localhost:~#cd redis-3.0.0 && make
```

#### 4. 查看版本

```
root@localhost:~# src/redis-server -v
Redis server v=3.0.0 sha=00000000:0 malloc=jemalloc-3.6.0 bits=32 build=a8a321b3ed54eaaa
```



# 第5章 可视化工具

[redis-desktop-manager](https://redisdesktop.com/download)

 

 

# 第6章 Redis实战应用

## 6.1 数据缓存

**Tip: **

1. Spring Redis默认使用JDK进行序列化和反序列化，因此被缓存对象需要实现java.io.Serializable接口，否则缓存出错。
2. 当被缓存对象发生改变时，可以选择更新缓存或者失效缓存，但一般而言，后者优于前者，因为执行速度更快。

**Watchout! **

在同一个Class内部调用带有缓存注解的方法，缓存并不会生效。

 

## 6.2 Session共享

利用了redis的**堆外存**特性。

要保证分布式应用的可伸缩性，带状态的Session对象是绕不过去的一道坎。一种方式是将Session持久化到数据库中，缺点是读写成本太高。另一种方式是去Session化，比如Play直接将Session存到客户端的Cookie中，缺点是存储信息的大小受限。

***将Session缓存到Redis中，既保证了可伸缩性，同时又避免了前面两者的限制。***

 

## 6.3 分布式锁

 

## 6.4 全局计数器 

 

 

# 附录

## Redis命令参考

<http://redisdoc.com/index.html>

 

# 參考

[Redis 备份、容灾及高可用实战](https://mp.weixin.qq.com/s/41A4gOPBcypR5hN0jSADoA)







