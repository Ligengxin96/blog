---
title: 'SQL Server中如何比较两个表中nullable(可空)字段'
tags:
  - Transact-SQL
  - SQL Server
categories:
  - Issue
---
&emsp;&emsp;今天遇到一个问题就是在两个表连接查询的时候.连接条件是A表的a字段等于B表的a字段,同时A表的b字段等于B表的b字段.因为这个b字段可为空,所以就算a字段是相同的如果b字段都是null,这条数据也是匹配不上的.但是需求是这条数据应该是匹配上的.


## 问题出现

```sql  
-- b字段可为null
select * from A,B where A.a = B.a and A.b = B.b
```
主要原因是因为SQL Server中 null = null 返回是UNKNOWN, 可以初略的理解为false.

## 解决方案
[参考资料stackoverflow的答案](https://stackoverflow.com/questions/1075142/how-to-compare-values-which-may-both-be-null-in-t-sql)
```sql  
-- b字段可为null
select * from A,B where A.a = B.a and (A.b = B.b or ISNULL(A.b, B.b) IS NULL)
```

但是这并不是最优解,因为使用上面我的方法的话可能会有索引问题,就是如果b这个字段添加的索引的话.
最优解应该是这个问题的推荐答案,但是因为我这次的话不适合,因为会把简单问题复杂化,所以就采用了上面的这个方法.
本次只讨论了SQL Server等了解了其他数据库的特点后再来补充吧。

