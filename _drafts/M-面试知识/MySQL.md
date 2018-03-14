
##1. 数据库优化方面的事情

- 定位：查找、定位慢查询
- 优化手段
  1. 创建索引：创建合适的索引，可以在索引中查询，查询到后直接找对应的记录；
  2. 分表：当一张表的数据比较多或者一张表的某些字段值比较多并且很少使用时，采用水平分表和垂直分表来优化；
  3. 读写分离：当一台服务器不能满足需求时，采用读写分离的方式进行集群；
  4. 缓存：使用redis来进行缓存
  5. 其他的优化技巧

##2. 查找慢查询并定位慢查询

项目部署之前，在启动MySQL数据库时开启慢查询，并把执行慢的语句写到日志中，在运行一段时间后，通过查看日志找到慢查询语句。

使用explain来详细分析语句的执行过程。



##3.  数据库优化之数据库表设计遵循范式

数据库表设计时需要遵循的方式

表的范式，首先符合1NF，才能满足2NF，进一步满足3NF。

- 1NF：即表的列具有原子性，不可再分解，即列的信息不能再分解，只要数据库是关系型数据库（mysq/oracle/db2...），就自动满足1NF，关系型数据库中是不允许分隔列的。
- 2NF：表中的记录是唯一的，通常我们设计一个主键来实现。
- 3NF：即表中不能有冗余数据，就是说，表的信息，如果能够被推导出来就不应该单独设计一个字段来存放。
- 反3NF，没有冗余的数据未必是最好的数据库，有事为了提高运行效率，就必须降低范式标准，适当保留冗余数据；具体做法是：在概念数据模型设计时遵守第三范式，降低范式标准的工作放到物理数据模型时考虑，降低范式就是增加字段，允许冗余。
  订单和订单项、相册浏览次数和照片的浏览次数。

##4. 选择合适的数据库引擎

研发中，经常使用的存储引擎 myisam/ innodb/ memory

- **MyISAM存储引擎**
  如果表对事务要求不高，同时是以查询和添加为主的，我们考虑使用myisam存储引擎，比如：bbs中的发帖表和回复表。
- **InnoDB存储引擎**
  对事务要求高，保存的数据都是重要数据，建议使用INNODB，比如：订单表，账号表。
- **Memory 存储引擎**
  数据变化频繁，不需要入库，同时又频繁的查询和修改，考虑使用 memory，速度极快。

###MyISAM 和 INNODB 的区别

1. 事务安全：myisam 不支持事务而 innodb支持
2. 添加和查询速度：myisam 不支持事务就不用考虑同步锁，查找和添加的速度快
3. 支持全文索引： myisam支持 innodb不支持
4. 锁机制： myisam 支持表锁，innodb支持行锁（事务）
5. 外键： myisam 不支持外键，innodb 支持外键（通常不设置外键，通常在程序中保证数据一致）

|     特点     |   Myisam	|  InnoDB	|    BDB  |	Memory | Archive |
| --------     | -----:     | :----:    | :----:  |:----:  | :----:  |
| 存储限制 	   |  没有  	| 	 64TB 	|  没有   |  有    | 没有    | 
| 批量插入速度 |   高   	| 	 低   	|  高  	  |  高    |  非常高 | 
| 事务安全     |   	        |    支持   |  支持   |        |	     |    
| 锁机制       |  表锁  	|    行锁   |  页锁   |  表锁  |  行锁   | 
| B树索引      |  支持      |  支持     |  支持   |  支持  |		 |        
| 哈希索引     |     	    | 支持      |	      |  支持  |  	     |      
| 全文索引     |  支持      |      	    |   	  | 	   |         |
| 集群索引     |      	    | 支持      |    	  |	       |         |
| 数据缓存     |            |支持       |	      |  支持  |      	 | 
| 索引缓存     |  支持      |	  支持  | 	      |	 支持  |     	 |  
| 数据可压缩   | 支持       |      	    |    	  |        |  支持   | 
| 空间使用 	   | 低         |  高   	| 低  	  | N/A    |  非常低 | 
| 内存使用     | 低         |	  高   	|低  	  |中等    |  低   	 |  
| 支持外键     | 	        |    支持  	|    	  |    	   |         |



##5. 选择合适的索引

索引（Index）是帮助 DBMS 高效获取数据的数据结构。

分类：普通索引|唯一索引|主键索引|全文索引

- 普通索引：允许重复的值出现
- 唯一索引：除了不能有重复的记录外，其他和普通索引一样（用户名、用户身份证、Email、tel）
- 主键索引：是随着设定主键而创建的，也就是把某个列设为主键的时候，数据库就会给该列创建索引。这就是主键索引唯一且没有null值。
- 全文索引：用来对表中的文本域（char，vachar，text）进行索引，全文索引针对Myisam
      -- 会使用全文索引 match  和 against
      explain select * from articles where match(title,body) against('database');
  

##6. 使用索引的一些技巧

###索引弊端

1. 占用磁盘空间
2. 对dml（插入、修改、删除）操作有影响，变慢。

###使用场景

1. 肯定在 where 条件经常使用，如果不做查询就没有意义
2. 该字段的内容不是唯一的几个值（sex）
3. 字段内容不是频繁变化

###具体技巧

1. 对于创建的多列索引（复合索引），不是使用的第一部分就不会使用索引
       
```sql
alter table dept ad index my_index(dname,loc); -- dname左边的列，loc右边的列
explain select * from dept where dname='aaa'\G -- 会使用到索引
explain select * from dept where loc='aaa'\G -- 不会使用到索引
```
2.  对于使用 like 的查询，查询如果是 '%aaa'不会使用索引，而’aaa%‘ 会使用索引
```sql       
explain select * from dept where dname like '%aaa'\G -- 不会使用到索引
explain select * from dept where dname like 'aaa%'\G -- 会使用到索引
```
所以在 like 查询时，’关键字‘ 最前面不能使用 % 或者 _ 这样的字符，如果一定要前面有变化的值，则考虑使用 全文索引 -> sphinx。
3. 如果条件中有or，有条件没有使用索引，即使其中有条件带索引也不会使用，换言之，就是要求使用的所有字段，都必须单独使用时能使用索引。可以考虑用in 或 union 替换or。
4. 如果列类型是字符串，一定要在条件中将数据使用引号引用起来，否则不使用索引
```sql  
explain select * from dept where dname='1111';
explain select * from dept where dname=1111; -- 数值自动转字符串
explain select * from dept where dname=qqq; -- 报错
```
   也就是说，如果列是字符串类型，无论是不是字符串，数字一定要用 ’‘ 把它括起来。
5. 如果 MySQL 估计使用全表扫描比使用索引快，则不使用索引。
   表里面只有一条记录。

##7. 数据库优化之分表

分表分为水平（按行）分表  和 垂直（按列）分表。

根据经验，MySQL 表数据一般达到百万级别，查询效率会很低，容易造成表锁，甚至堆积很多连接，直接挂掉，水平分表能够很大程度减少这些压力。（按行数据进行分类）


如果一张表中某个字段值非常多（成文本，二进制等），而且只有在很少的情况下回查询，这时候就可以把字段单独放到一个表，通过外键关联起来。

考试详情，一般我们只关注分数，不关注详情。

### 水平分表策略

1. 按时间分表
   这种分表方式有一定的局限性，当数据有较强的时效性，如微博发送记录，微信消息记录等，这种数据很少有用户会查询几个月前的数据，如就可以按月分类。
2. 按区间范围分表
   一般有严格的自增id需求上，如按照user_id水平分表。
   table_1 user_id 从 1~100w
   table_2 user_id 从 101w~200w
   table_3 user_id 从 201w~300w
3. ☆hash分表☆
   一个原始目标的 ID 或者名称通过一定的 hash 算法计算出数据存储表的表名，然后访问相应的表。

##8. 数据库的读写分离

一台数据库支持的最大并发数是有限的，如果用户并发访问太多，一台服务器满足不了要求，就可以集群处理，MySQL的集群处理技术最常用的就是读写分离。

###主从同步

数据库最终会把数据持久化到磁盘，如果集群就必须保证每个数据库服务器的数据是一致的，能改变数据库数据的操作都往主数据库写，而其他数据库从主数据库上同步数据。

###读写分离

使用负载均衡来实现写的操作都往主数据库去，而读的数据往从数据库。

##9. 数据库优化之缓存

在持久层和数据库之前添加一个缓存层，如果用户访问的数据已经缓存起来时，在用户访问时直接从缓存中获取，不用访问数据库。而缓存是在内存级的，访问速度快。

作用

减少数据库服务器压力，减少访问时间，提高系统响应速度。

Java中常用的缓存有，hibernate 的二级缓存，但是该缓存不能完成分布式缓存。

可使用redis 作为中央缓存。



##10. SQL语句优化小技巧

###DDL优化

1. 通过禁用索引来提高导入数据性能，该操作主要针对有数据的表，追加数据。
```sql  
-- 去除键
alter table test DISABLE keys;
-- 批量插入数据
insert into test select * from test1;
-- 恢复键
alter table test ENABLE keys;
```
2. 关闭唯一校验
```sql         
set unique_checks=0 -- 关闭
set unique_checks=1 -- 开启
```
3. 修改事务提交方式（导入）[变多次提交为一次]
```sql     
set autocommit=0 -- 关闭
-- 批量插入
set autocommit=1 -- 开启
```
###DML 优化

变多次提交为一次

    insert into test values(1,2);
    insert into test values(1,3);
    insert into test values(1,4);
    -- 合并多条为一条
    insert into test values(1,2),(1,3),(1,4);



###DQL优化

- order by 优化
  1. 多用索引排序
  2. 普通结果排序（非索引排序）
- group by 优化
  使用order by null，取消默认排序
- 子查询优化
  在客户列表找到不在支付列表的用户，即查询没买过东西的客户
```sql  
  --
  explain
  select * from customer where customer_id not in (select DISTINCT customer_id from payment) ;
  -- 基于索引外链
  explain
  select * from customer c left join payment p on (c.customer_id=p.customer_id) where p.customer_id is null;
```

- or 优化
  在两个独立索引上使用or 的性能优化
  1. or 两边都是用索引字段做判断，性能好
  2. or 两边，有一边不用，性能差
  3. 如果 employee 表的 name 和 email 这两列是一个复合索引，但如果是 name='A' or email='B' 这种方式，不会用到索引
  
- limit 优化
```sql  
select a.film_id,a.description from film a order by a.title limit 50,5;
-- 优化
select a.film_id,a.description from film a inner join (selet film_id from film order by title limit 50,5) b on a.film_id=b.film_id
```
##11. JDBC批量插入几百万条数据

变多次提交为一次
```java 
    String JDBC_URL = "jdbc:mysql://localhost:3306/samp_db";
    String JDBC_USERNAME = "root";
    String JDBC_PASSWORD = "root";
    
    Connection  connection = DriverManager.getConnection(JDBC_URL + "?useServerPrepStmts=false&rewriteBatchedStatements=true",JDBC_USERNAME,JDBC_PASSWORD);
    connection.setAutoCommit(false);
    PreparedStatement cmd = connection.prepareStatement("insert into test1 values(?,?)");
            
    for (int i = 0; i < 1000000; i++) {//100万条数据
    	cmd.setInt(1, i);
    	cmd.setString(2, "test");
    	cmd.addBatch();
    	if(i%1000==0){
    		cmd.executeBatch();
    	}
    }
    cmd.executeBatch();
    connection.commit();
             
    cmd.close();
    connection.close();
```
省出的时间相当可观，像这种操作能不使用代码就不要使用代码，可以使用存储过程代替。






