---
layout: post
title: Redis 使用规约
categories: Redis
description: redis 系列学习使用篇
keywords: redis
---



## 1.  活跃度查询

### setbit 的使用

场景: 1亿个用户, 每个用户 登陆/做任意操作  ,记为 今天活跃,否则记为不活跃

 

每周评出: 有奖活跃用户: 连续7天活动

每月评,等等...



思路: 

| Userid | dt         | active |
| :----- | ---------- | ------ |
| 1      | 2017-07-27 | 1      |
| 1      | 2017-07-26 | 1      |

 

如果是放在表中, 1:表急剧增大,2:要用group ,sum运算,计算较慢

 

用: 位图法 bit-map

Log0721:  ‘011001...............0’

......

log0726 :   ‘011001...............0’

Log0727 :  ‘0110000.............1’

 

1: 记录用户登陆:

每天按日期生成一个位图, 用户登陆后,把user_id位上的bit值置为1

 

2: 把1周的位图  and 计算, 

位上为1的,即是连续登陆的用户

  

````shell
redis 127.0.0.1:6379> setbit mon 100000000 0

(integer) 0

redis 127.0.0.1:6379> setbit mon 3 1

(integer) 0

redis 127.0.0.1:6379> setbit mon 5 1

(integer) 0

redis 127.0.0.1:6379> setbit mon 7 1

(integer) 0

redis 127.0.0.1:6379> setbit thur 100000000 0

(integer) 0

redis 127.0.0.1:6379> setbit thur 3 1

(integer) 0

redis 127.0.0.1:6379> setbit thur 5 1

(integer) 0

redis 127.0.0.1:6379> setbit thur 8 1

(integer) 0

redis 127.0.0.1:6379> setbit wen 100000000 0

(integer) 0

redis 127.0.0.1:6379> setbit wen 3 1

(integer) 0

redis 127.0.0.1:6379> setbit wen 4 1

(integer) 0

redis 127.0.0.1:6379> setbit wen 6 1

(integer) 0

redis 127.0.0.1:6379> bitop and  res mon feb wen

(integer) 12500001

````



如上例,优点:

1: 节约空间, 1亿人每天的登陆情况,用1亿bit,约1200WByte,约10M 的字符就能表示

2: 计算方便



## 2. 消息订阅 

**使用办法:**

订阅端: Subscribe 频道名称

发布端: publish 频道名称 发布内容

 

**客户端例子:**

````shell
redis 127.0.0.1:6379> subscribe news
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news"
3) (integer) 1
1) "message"
2) "news"
3) "good good study"
1) "message"
2) "news"
3) "day day up"
````



**服务端例子:**

````shell
redis 127.0.0.1:6379> publish news 'good good study'
(integer) 1
redis 127.0.0.1:6379> publish news 'day day up'
(integer) 1
````



## 3. 标签云

使用 MySQL：

````sql
create table book (
bookid int,
title char(20)
)engine myisam charset utf8;

insert into book values 
(5 , 'PHP圣经'),
(6 , 'ruby实战'),
(7 , 'mysql运维')
(8, 'ruby服务端编程');


create table tags (
tid int,
bookid int,
content char(20)
)engine myisam charset utf8;

insert into tags values 
(10 , 5 , 'PHP'),
(11 , 5 , 'WEB'),
(12 , 6 , 'WEB'),
(13 , 6 , 'ruby'),
(14 , 7 , 'database'),
(15 , 8 , 'ruby'),
(16 , 8 , 'server');
````

 既有web标签,又有PHP,同时还标签的书,要用连接查询

````sql
select * from tags inner join tags as t on tags.bookid=t.bookid
where tags.content='PHP' and t.content='WEB';
````



换成key-value存储

````sh
set book:5:title 'PHP圣经'
set book:6:title 'ruby实战'
set book:7:title 'mysql运难'
set book:8:title ‘ruby server’

sadd tag:PHP 5
sadd tag:WEB 5 6
sadd tag:database 7
sadd tag:ruby 6 8
sadd tag:SERVER 8
````

查: 既有PHP,又有WEB的书

````sh
Sinter tag:PHP tag:WEB  #查集合的交集
````

查: 有PHP或有WEB标签的书

````sh
Sunin tag:PHP tag:WEB
````

查:含有ruby,不含WEB标签的书

````sh
Sdiff tag:ruby tag:WEB #求差集
````



## 4.Redis key 设计技巧 

1: 把表名转换为key前缀 如, tag:

2: 第2段放置用于区分区key的字段--对应mysql中的主键的列名,如userid

3: 第3段放置主键值,如2,3,4...., a , b ,c

4: 第4段,写要存储的列名

用户表 user  , 转换为key-value存储

| userid | username | password | email        |
| ------ | -------- | -------- | ------------ |
| 9      | Lisi     | 11111111 | lisi@163.com |



````sh
set  user:userid:9:username lisi
set  user:userid:9:password 111111
set  user:userid:9:email   lisi@163.com

keys user:userid:9*
````



 **注意：**

在关系型数据中，除主键外，还有可能其他列也步骤查询，如上表中， username 也是极频繁查询的，往往这种列也是加了索引的。

 

转换到 k-v 数据中，则也要相应的生成一条按照该列为主的key-value

> Set  user:username:lisi:uid  9  

 

这样，我们可以根据 username:lisi:uid ，查出 userid=9, 

再查user:9:password/email ...

 

完成了根据用户名来查询用户信息。







