##1.Redis是什么
Redis是一个 key-value 的 nosql 数据库，先把数据存到内存中，然后根据一定的策略持久化到磁盘，即使断电也不会丢失，支持的数据类型比较多。

主要用来做缓存数据库的数据和 web 集群时当作中央缓存存放 session。

### 支持的数据类型
Redis的键值可以使用五种数据类型：字符串，散列表，列表，集合，有序集合。详细介绍见[Redis系列学习](https://www.jianshu.com/p/a819901f0661)



### 持久化策略
Snapshotting（快照）-rdb 和Append-Only file（追加）- aof

https://segmentfault.com/a/1190000009537768
http://blog.csdn.net/suibo0912hf/article/details/51684316
[redis.conf的配置解析](https://my.oschina.net/wfire/blog/301147)


##2. Redis 的使用场景

### 缓存
把经常需要查询的，很少修改的数据，放到读速度很快的空间（内存），已使下次访问减少时间，减轻压力。

###计数器
redis 中的计数器是 `原子性的内存操作`。
可解决库存溢出问题，进销存系统库存溢出。

###session缓存服务器
web 集群时作为 session 缓存服务器。

##3.  redis 和 memcache 的区别

|        |  mysql| redis | memcache |
|:---:|:------:|:-----:|:------------:|
| 类型 | 关系型 |  非关系型 | 非关系型 | 
| 存储位置 | 磁盘 | 磁盘和内存 | 内存 |
| 存储过期 | 不支持 | 支持 | 支持|
| 读写性能 | 低 | 非常高 | 非常高 |

1. redis 和 memcache 都是将数据存放在内存中，不过memcache 还可以 缓存其他东西，例如图片、视频等。
2. redis 不仅仅支持简单的k/v 类型的数据，同时还提供 list、set、hash 等数据结构的存储
3. 虚拟内存——redis 当物理内存用完时，可以将一些很久没用到的 value  交换到磁盘。

## 4.redis 对象保存方式

- **json 字符串**
需要手动把对象转换为 json 字符串，当作字符串处理，直接使用 set get 来设置或获取。
*优点：*设置和获取比较简单
*缺点：*没有提供专门的方法，需要把对象转换为 json

- **字节**
需要做序列化，就是把对象序列化为字节保存。

如果担心 JSON 转对象会消耗资源的情况，这个问题需要考虑几个地方：
1. 使用的 JSON 转换 lib 是否会存在性能问题
2.  数据的数据量级别，如果存储百万级的大数据对象，建议采用 `存储序列化对象方式`；如果是少量的数据级对象，或者数据对象字段不多，建议采用 JSON 转换成 String 的方式

总结：毕竟 redis 对存储字符类型这部分优化的非常好，具体采用的方式与方法，还是要看使用的场景。

##5. Redis的数据淘汰机制
在redis中，允许用户设置最大使用 `内存大小（server-maxmemory）`，在其限定的情况下是很有用的。譬如，在一台8G机子上部署了 4 个redis服务点，每一个服务的分配 1.5G 的内存大小，减少内存紧张的情况，由此获取更为稳健的服务。

**内存大小有限，需要保存有效的数据**
redis 内存数据集大小上升到一定大小的时候，就会执行数据淘汰机制，redis 提供了6 种数据淘汰策略：
**1. volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中，挑选最近最少使用的数据淘汰**
2. volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰
3. volatile-random：从已设置过期时间的数据集中，任意选择数据淘汰
**4. allkeys-lru：从数据集（server.db[i].dict）中，挑选最近最少使用的数据淘汰**
5. allkeys-random：从数据集中任意挑选数据淘汰
6. no-enviction：禁止驱逐数据

redis确定驱逐某个键值对后，会删除这个数据并将这个数据变更消息发布到本地（AOF持久化）和从机（主从连接）。

##6. Java 访问 Redis
1. 使用 jedis java 客户端来访问 redis 服务器，有点类似通过 jdbc 访问 mysql 一样
2. 如果使用 spring 集成时，可以使用 spring data 来访问 redis ，spring data 只是对jedis 的二次封装   。与 jdbcTemplate 和 jdbc 关系一样。

##7. Redis 集群
当一台数据无法满足需求时，可以使用redis 集群来处理，类似于 mysql 的读写分离。
 
