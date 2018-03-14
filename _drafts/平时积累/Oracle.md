## 分页
```
-- 无 order by 的情况，建议使用 2
-- 1、在查询的最外层控制分页的最小值和最大值
select * from 
(select t1.*, rownum rn
from XSH_DIDI_CONSUME t1
) t2
 where t2.rn between 10 and 15
 ;
 
 
 -- 2、内层控制最大，外层控制最小,效率较高
 select * from 
(select t1.*, rownum rn
from XSH_DIDI_CONSUME t1
where rownum <=15
) t2
 where t2.rn >= 10 
 
 ;
 
 -- 有 order by 的情况，建议使用 2
 -- 1、在查询的最外层控制分页的最小值和最大值
 select * from 
 (select t2.*, rownum rn from 
         ( select *
                from XSH_DIDI_CONSUME t1 
                -- where 
                order by t1.datetime         
         ) t2
 ) t3
 where t3.rn between 10 and 15
 ;
 
 -- 2、内层控制最大，外层控制最小,效率较高
  select * from 
 (select t2.*, rownum rn from 
         ( select *
                from XSH_DIDI_CONSUME t1 
                -- where 
                order by t1.datetime         
         ) t2
   where rownum <=15
 ) t3
 where t3.rn >=10
 ```
