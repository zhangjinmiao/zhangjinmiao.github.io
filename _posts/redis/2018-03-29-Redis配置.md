---
layout: post
title: Redis 配置
categories: Redis
description: redis 系列学习配置篇
keywords: redis
---



## 1. 组件介绍

redis-benchmark、redis-check-dump、redis-sentinel、redis-check-aof、redis-cli、redis-server 构成了redis软件包，下面我们来了解一下它们分别是用来干什么的。  

|       组件        |                   用途                    |
| :-------------: | :-------------------------------------: |
|  redis-server   |             Redis服务器的启动程序。              |
|    redis-cli    | Redis命令行操作工具。当然，你也可以用telnet根据其纯文本协议来操作。 |
| redis-benchmark |   Redis性能测试工具，测试Redis在你的系统及你的配置下的读写性能   |
|   redis-stat    |    Redis状态检测工具，可以检测Redis当前状态参数及延迟状况。    |





## 2. 配置文件

redis 服务相关参数都需要在 redis.conf 文件中进行配置，所以我们有必要花点时间来简单了解一下 conf 文件中的基本参数。

|     参数      |           作用           |
| :---------: | :--------------------: |
|  daemonize  | 是否以后台daemon方式运行redis服务 |
| port redis  |      服务端口，默认6379       |
|   Timeout   |         请求超时时间         |
| requirepass |        连接数据库密码         |

redis.conf 中 daemonize 参数默认为no，为了让 redis 服务在后台运行，我们需要将 daemonize 参数设置为yes。



**文件参数详细说明：**

![](http://images.gitbook.cn/1583ff40-d29d-11e7-a17f-8b69bfba7264)



生产环境不建议一个Redis实例数据太多{（20-30）G数据内存对应（96-128）G实际内存）}，这种20%-23%的比例比较合适，因为磁盘读到内存的恢复时间也很慢，可以使用ssd磁盘来提高磁盘读取速度。

一个64G的内存服务器，建议不要超过23%的比例的数据，即64*0.23G=15G。

内存管理开销大（不要超过物理内存的3/5）。buffer io 可能会造成系统内存溢出（OOM-Out of Memory）。



## 3. 持久化配置

Redis 的持久化有 2 种方式：   1快照  2是日志

 

**Rdb 快照的配置选项**

````javascript
save 900 1      // 900内,有1条写入,则产生快照 

save 300 1000   // 如果300秒内有1000次写入,则产生快照

save 60 10000  // 如果60秒内有10000次写入,则产生快照

(这 3 个选项都屏蔽，则 rdb 禁用)


stop-writes-on-bgsave-error yes  // 后台备份进程出错时,主进程停不停止写入?

rdbcompression yes    // 导出的rdb文件是否压缩

Rdbchecksum   yes //  导入rbd恢复时数据时,要不要检验rdb的完整性

dbfilename dump.rdb  //导出来的rdb文件名

dir ./  // rdb 的放置路径
````



**Aof 的配置**

````javascript
appendonly no # 是否打开 aof日志功能

 
appendfsync always   # 每1个命令,都立即同步到aof. 安全,速度慢

appendfsync everysec # 折衷方案,每秒写1次

appendfsync no      # 写入工作交给操作系统,由操作系统判断缓冲区大小,统一写入到aof. 同步频率低,速度快,


no-appendfsync-on-rewrite  yes: # 正在导出rdb快照的过程中,要不要停止同步aof

auto-aof-rewrite-percentage 100 #aof文件大小比起上次重写时的大小,增长率100%时,重写

auto-aof-rewrite-min-size 64mb #aof文件,至少超过64M时,重写
````



解答几个问题：

1. 注: 在 dump rdb 过程中，aof 如果停止同步，会不会丢失?

   > 答: 不会，所有的操作缓存在内存的队列里，dump 完成后，统一操作。

2. 注: aof 重写是指什么?

   >答: aof 重写是指把内存中的数据逆化成命令，写入到.aof日志里。
   >
   >以解决 aof 日志过大的问题。

3. 问: 如果 rdb 文件和 aof 文件都存在，优先用谁来恢复数据?

   >答: aof 

4. 问: 2 种是否可以同时用?

   答: 可以，而且推荐这么做

5. 问: 恢复时 rdb 和 aof 哪个恢复的快

   答: rdb 快,，因为其是数据的内存映射,直接载入到内存，而 aof 是命令，需要逐条执行

   ​

## 4. 慢查询

Slowlog 显示慢查询

注:多慢才叫慢? 

> 答: 由slowlog-log-slower-than 10000 ,来指定,(单位是微秒)

 

服务器储存多少条慢查询的记录?

> 答: 由 slowlog-max-len 128 ,来做限制



查看redis服务器的信息

> Info [Replication/CPU/Memory..] 



## 5. Redis运维时需要注意的参数 

1: 内存

**Memory**

````sh
used_memory:859192 数据结构的空间

used_memory_rss:7634944 实占空间

mem_fragmentation_ratio:8.89 前2者的比例,1.N为佳,如果此值过大,说明redis的内存的碎片化严重,可以导出再导入一次.
````


2: 主从复制

**Replication**

````sh
role:slave

master_host:192.168.1.128

master_port:6379

master_link_status:up
````



3:持久化

**Persistence**

````sh
rdb_changes_since_last_save:0

rdb_last_save_time:1375224063
````



4: fork耗时

**Status**

````sh
latest_fork_usec:936  上次导出rdb快照,持久化花费微秒

注意: 如果某实例有10G内容,导出需要2分钟,

每分钟写入10000次,导致不断的rdb导出,磁盘始处于高IO状态.
````



5: 慢日志

````sh
config get/set slowlog-log-slower-than

CONFIG get/SET slowlog-max-len 

slowlog get N 获取慢日志
````













注: 



 





以解决 aof 日志过大的问题.

 

问: 如果rdb文件，和aof 文件都存在,优先用谁来恢复数据?

答: aof

 

问: 2 种是否可以同时用?

答: 可以，而且推荐这么做

 

问: 恢复时 rdb 和 aof 哪个恢复的快

答: rdb快，因为其是数据的内存映射,直接载入到内存，而 aof 是命令，需要逐条执行