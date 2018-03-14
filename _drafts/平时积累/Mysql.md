#分页
select * from table limit (start-1)*limit,limit; 其中start是页码，limit是每页显示的条数


#性能调优
## 1. 表设计注意事项
>1. 数据行的长度不要超过 8020 字节，如果超过这个长度的话在物理页中这条数据会占用两行从而造成存储碎片，降低查询效率。
>2. 能够用数字类型的字段尽量选择数字类型而不用字符串类型的（电话号码），这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接回逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
>3. 字段的长度在最大限度的满足可能的需要的前提下，应该尽可能的设得短一些，这样可以提高查询的效率，而且在建立索引的时候也可以减少资源的消耗。

## 2. 查询优化

```sql
1. 保证在实现功能的基础上，尽量减少对数据库的访问次数 (可以用缓存保存查询结果，减少查询次数)
2. 通过搜索参数，尽量减少对表的访问行数, 最小化结果集，从而减轻网络负担
3. 能够分开的操作尽量分开处理，提高每次的响应速度
4. 在数据窗口使用 SQL 时，尽量把使用的索引放在选择的首列
5. 算法的结构尽量简单
6. 在查询时，不要过多地使用通配符如 SELECT * FROM T1 语句，要用到几列就选择几列
7. 在可能的情况下尽量限制结果集行数如：SELECT TOP 300 COL1,COL2,COL3 FROM T1, 因为某些情况下用户是不需要那么多的数据的
```

1.  应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：
```sql
    select id from t where num is null
```
可以在 num 上设置默认值 0，确保表中 num 列没有 null 值，然后这样查询：
```sql
    select id from t where num=0
```
2.  应尽量避免在 where 子句中使用!= 或 <> 操作符，否则将引擎放弃使用索引而进行全表扫描。优化器将无法通过索引来确定将要命中的行数, 因此需要搜索该表的所有行。

3. 应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：
```sql
    select id from t where num=10 or num=20
```
可以这样查询：
```sql
    select id from t where num=10
    union all
    select id from t where num=20
```
4. in 和 not in 也要慎用，因为 IN 会使系统无法使用索引, 而只能直接搜索表中的数据。如：
```sql
    select id from t where num in(1,2,3)
```
对于连续的数值，能用 between 就不要用 in 了：
```sql
    select id from t where num between 1 and 3
```
5. 尽量避免在索引过的字符数据中，使用非打头字母搜索。这也使得引擎无法利用索引。

见如下例子： 
```sql
    SELECT * FROM T1 WHERE NAME LIKE ‘%L%’ 
    
    SELECT * FROM T1 WHERE SUBSTING(NAME,2,1)=’L’ 
    
    SELECT * FROM T1 WHERE NAME LIKE ‘L%’ 
```
即使 NAME 字段建有索引，前两个查询依然无法利用索引完成加快操作，引擎不得不对全表所有数据逐条操作来完成任务。而第三个查询能够使用索引来加快操作。

6.必要时强制查询优化器使用某个索引，如在 where 子句中使用参数，也会导致全表扫描。因为 SQL 只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：
```sql
    select id from t where num=@num
```
可以改为强制查询使用索引：
```sql
    select id from t with(index(索引名)) where num=@num
```
7.应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：
```sql
    SELECT * FROM T1 WHERE F1/2=100 
```
应改为: 
```sql
    SELECT * FROM T1 WHERE F1=100*2

    SELECT * FROM RECORD WHERE SUBSTRING(CARD_NO,1,4)=’5378’ 
```
应改为: 
```sql
    SELECT * FROM RECORD WHERE CARD_NO LIKE ‘5378%’

    SELECT member_number, first_name, last_name FROM members WHERE DATEDIFF(yy,datofbirth,GETDATE()) > 21 
```
应改为: 
```sql
    SELECT member_number, first_name, last_name FROM members WHERE dateofbirth <DATEADD(yy,-21,GETDATE())
```
即：任何对列的操作都将导致表扫描，它包括数据库函数、计算表达式等等，查询时要尽可能将操作移至等号右边。

8.  应尽量避免在 where 子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：
```sql
    select id from t where substring(name,1,3)='abc'  --name 以 abc 开头的 id

    select id from t where datediff(day,createdate,'2005-11-30')=0  --‘2005-11-30’生成的 id
```
应改为:
```sql
    select id from t where name like 'abc%'

    select id from t where createdate>='2005-11-30' and createdate<'2005-12-1'
```
9.不要在 where 子句中的 “=” 左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。

10.  在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。

11.  很多时候用 exists 是一个好的选择：
```sql
    select num from a where num in(select num from b)
```
用下面的语句替换：
```sql
    select num from a where exists(select 1 from b where num=a.num)

    SELECT SUM(T1.C1)FROM T1 WHERE(
    (SELECT COUNT(*)FROM T2 WHERE T2.C2=T1.C2>0) 

    SELECT SUM(T1.C1) FROM T1 WHERE EXISTS( 
    SELECT * FROM T2 WHERE T2.C2=T1.C2) 
```
两者产生相同的结果，但是后者的效率显然要高于前者。因为后者不会产生大量锁定的表扫描或是索引扫描。

如果你想校验表里是否存在某条纪录，不要用 count(*) 那样效率很低，而且浪费服务器资源。可以用 EXISTS 代替。如：
```sql
    IF (SELECT COUNT(*) FROM table_name WHERE column_name = 'xxx') 
```
可以写成： 
```sql
    IF EXISTS (SELECT * FROM table_name WHERE column_name = 'xxx')
```
经常需要写一个 T_SQL 语句比较一个父结果集和子结果集，从而找到是否存在在父结果集中有而在子结果集中没有的记录，如：
```sql
    SELECT a.hdr_key FROM hdr_tbl a---- tbl a 表示 tbl 用别名 a 代替 
    WHERE NOT EXISTS (SELECT * FROM dtl_tbl b WHERE a.hdr_key = b.hdr_key)

    SELECT a.hdr_key FROM hdr_tbl a 
    
    LEFT JOIN dtl_tbl b ON a.hdr_key = b.hdr_key WHERE b.hdr_key IS NULL

    SELECT hdr_key FROM hdr_tbl 
    WHERE hdr_key NOT IN (SELECT hdr_key FROM dtl_tbl) 
```
12. 尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。

13. 避免频繁创建和删除临时表，以减少系统表资源的消耗。

14. 临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件，最好使用导出表。

15. 在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先 create table，然后 insert。

16.  如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。

17. 在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。

18. 尽量避免大事务操作，提高系统并发能力。

19. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。 

20. 避免使用不兼容的数据类型。例如 float 和 int、char 和 varchar、binary 和 varbinary 是不兼容的（条件判断时）。数据类型的不兼容可能使优化器无法执行一些本来可以进行的优化操作。例如:
```sql
SELECT name FROM employee WHERE salary > 60000 
```
在这条语句中, 如 salary 字段是 money 型的, 则优化器很难对其进行优化, 因为 60000 是个整型数。我们应当在编程时将整型转化成为钱币型, 而不要等到运行时转化。

21. 充分利用连接条件（条件越多越快），在某种情况下，两个表之间可能不只一个的连接条件，这时在 WHERE 子句中将连接条件完整的写上，有可能大大提高查询速度。

例： 
```sql
    SELECT SUM(A.AMOUNT) FROM ACCOUNT A,CARD B WHERE A.CARD_NO = B.CARD_NO
    
    SELECT SUM(A.AMOUNT) FROM ACCOUNT A,CARD B WHERE A.CARD_NO = B.CARD_NO AND A.ACCOUNT_NO=B.ACCOUNT_NO
```
第二句将比第一句执行快得多。

22.  当只要一行数据时使用 LIMIT 1

当你查询表的有些时候，你已经知道结果只会有一条结果，但因为你可能需要去fetch游标，或是你也许会去检查返回的记录数。 　　

在这种情况下，加上 LIMIT 1 可以增加性能。这样一样，MySQL数据库引擎会在找到一条数据后停止搜索，而不是继续往后查少下一条符合记录的数据

23. 使用 ENUM 而不是 VARCHAR

ENUM 类型是非常快和紧凑的。在实际上，其保存的是 TINYINT，但其外表上显示为字符串。这样一来，用这个字段来做一些选项列表变得相当的完美。 

如果你有一个字段，比如“性别”，“国家”，“民族”，“状态”或“部门”，你知道这些字段的取值是有限而且固定的，那么，你应该使用 ENUM 而不是 VARCHAR。使用 PROCEDURE ANALYSE() 你可以得到相关的建议。

24. **使用PROCEDURE ANALYSE()  获取优化建议**

PROCEDURE ANALYSE() 会让 MySQL 去分析字段和其实际的数据，并会给一些有用的建议。只有表中有实际的数据，这些建议才会变得有用，因为要做一些大的决定是需要有数据作为基础的。
PROCEDURE ANALYSE的语法如下：
SELECT ... FROM ... WHERE ... PROCEDURE ANALYSE([max_elements,[max_memory]])

max_elements （默认值256） analyze查找每一列不同值时所需关注的最大不同值的数量.
analyze还用这个值来检查优化的数据类型是否该是ENUM,如果该列的不同值的数量超过了
max_elements值ENUM就不做为建议优化的数据类型。
max_memory （默认值8192） analyze查找每一列所有不同值时可能分配的最大的内存数量
样例:
```sql
SELECT * FROM gateway_order_pay_detail procedure analyse();
SELECT * FROM gateway_order_pay_detail procedure analyse(16,256);
// 告诉PROCEDURE ANALYSE 不要为那些包含值多余16个或256个字节的ENUM类型提优化建议！
```
25. **☆EXPLAIN 你的 SELECT 查询**

Explain + select语句
或
Desc + select语句
使用explain 可以知道mysql是如何处理sql语句的，可以帮你分析你的查询语句或是表结构的性能瓶颈。EXPLAIN 的查询结果还会告诉索引主键被如何利用的，数据表是如何被搜索和排序的……等等。

###EXPLAIN列的解释
**table**
显示这一行的数据是关于哪张表的

**type**
 这是重要的列，显示连接使用了何种类型。从`最好到最差`的连接类型为`const`、`eq_reg`、`ref`、`range`、`indexhe` 和  `ALL`

**possible_keys**
显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句

**key**
实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用 `USE INDEX（indexname）`来强制使用一个索引或者用 `IGNORE INDEX（indexname）`来强制MYSQL忽略索引

**key_len**
使用的索引的长度。在不损失精确性的情况下，长度越短越好

**ref**
显示索引的哪一列被使用了，如果可能的话，是一个`常数`

**rows**
MYSQL 认为必须检查的用来返回请求数据的行数

**Extra**
关于MYSQL如何解析查询的额外信息。**

####extra 列返回的描述的意义

**Distinct**
 一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

**Not exists**
MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了

**Range checked for each**
Record（index map:#）没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

**Using filesort**
 看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行

**Using index**
列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候

**Using temporary** 看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

**Where used**
使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题

####不同连接类型的解释（按照效率高低的顺序排序）
**system**
 表只有一行：system表。这是const连接类型的特殊情况

**const**
 表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待

**eq_ref**
 在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用

**ref**
 这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好

**range**
 这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西时发生的情况

**index**
 这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为索引一般小于表数据）

**ALL**
 这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免

26. 当子查询速度慢时，可用JOIN来改写一下该查询来进行优化

但有时候也不一定，需经过测试再定。


