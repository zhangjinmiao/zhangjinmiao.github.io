---
layout: post
title: Redis 命令
categories: Redis
description: redis 系列学习命令篇
keywords: redis, key, string, 
---



## 1. Key 





![Redis的keys命令](http://images.gitbook.cn/ab494f20-d2a3-11e7-a17f-8b69bfba7264)



## 2. String



![Redis的字符串命令](http://images.gitbook.cn/13723540-d2a8-11e7-a17f-8b69bfba7264)



## 3. Hash

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象，能够存储 key 对多个属性的数据。

![redis-hash命令](http://images.gitbook.cn/933d4870-d2b5-11e7-a17f-8b69bfba7264)



## 4. List

Redis Lists 基于 Linked List 实现。这意味着即使在一个 list 中有数百万个元素，在头部或尾部添加一个元素的操作，其时间复杂度也是常数级别的。用 LPUSH 命令在十个元素的list头部添加新元素，和在千万元素 list 头部添加新元素的速度相同。

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。



![redis-list命令](http://images.gitbook.cn/9ca48d00-d2b6-11e7-a17f-8b69bfba7264)



## 5. Set

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

![redis-set命令](http://images.gitbook.cn/26740590-d2b9-11e7-a17f-8b69bfba7264)



## 6. Sorted set

Redis  有序集合和集合一样也是 string 类型元素的集合，且不允许重复的成员。不同的是每个元素都会关联一个**double** 类型的分数，redis 正是通过分数来为集合中的成员进行从小到大的排序。

![redis有序集合命令](http://images.gitbook.cn/5f2af410-d2ba-11e7-a17f-8b69bfba7264)







## 7. 其他

````sh
redis 127.0.0.1:6380> time # 显示服务器时间 , 时间戳(秒), 微秒数
1) "1375270361"
2) "504511"
````



````sh
redis 127.0.0.1:6380> dbsize  # 当前数据库的key的数量
(integer) 2
````



Flushall  清空所有库所有键 

Flushdb  清空当前库所有键

Showdown [save/nosave]



**注: 如果不小心运行了flushall, 立即 shutdown nosave ,关闭服务器**

**然后 手工编辑 aof 文件, 去掉文件中的 “flushall ”相关行, 然后开启服务器,就可以导入回原来数据.**

**如果,flushall之后,系统恰好bgrewriteaof了,那么aof就清空了,数据丢失.**





