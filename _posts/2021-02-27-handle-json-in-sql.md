---
title: 'SQL Server中中处理JSON数据'
tags:
  - SQL Server
categories:
  - Technology
---
&emsp;&emsp;这次的任务是需要计算Azure中每个API访问的数量以及访问的方式,为了避免数据库数据冗余,所以就将source这个字段在kusto的查询中处理成了一个JSON.然后在存入到了数据库中.具体需求如下图

![avatar](/assets/img/2021/02-27/02-27-1.png)

## 1.具体代码
- 我们首先纵向展开上图的原始数据,可以得到下图的数据
```sql
SELECT 
  t1.apiPath,
  t1.operation,
  t1.[source],
  t1.[sourceCount]
FROM (
  SELECT apiPath, operation, impact.[key] as source, SUM(CAST(ISNULL(impact.[value], 0) AS BIGINT)) AS sourceCount
  FROM @tempTable1
  CROSS APPLY OPENJSON(impact) as impact
  GROUP BY apiPath, operation, impact.[key]
) AS t1
```
![avatar](/assets/img/2021/02-27/02-27-2.png)

- 然后合并成数据生成目标数据
```sql
with tempTable as (
SELECT 
			t1.apiPath,
			t1.[source],
			t1.[sourceCount]
		FROM (
      SELECT apiPath, impact.[key] as source, SUM(CAST(ISNULL(impact.[value], 0) AS BIGINT)) AS sourceCount
      FROM gengxintest
      CROSS APPLY OPENJSON(impact) as impact
      GROUP BY apiPath, operation, impact.[key]
    ) AS t1
)
SELECT a.apiPath, SUM(a.[sourceCount]) as occurrence,
  '{' + STUFF(
          (
            SELECT DISTINCT ',' + '"' + source + '"' + ':' + CAST(sourceCount AS nvarchar)
            FROM tempTable
            WHERE apiPath = a.apiPath 
            FOR XML PATH ('')
          )
        , 1, 1, '')  + '}' AS impact
FROM tempTable AS a
GROUP BY apiPath
```

## 2.遇到的问题
1. 纵向展开原始数据导致数据丢失问题
因为impact这列是后续加入的列,所以导致历史数据这列都是NULL,那么在执行这个SQL纵向展开原始数据过程中,会把impact为NULL的数据都丢失,解决办法是
`CROSS APPLY` 换成`OUTER APPLY`

2. 性能问题
由于在合并目标数据的时候使用了字符串拼接,导致执行速度很慢,所以最终是没有采用字符串拼接的方式.最终代码如下,最终数据和目标数据有点差异,但并不影响最终使用

```sql
with tempTable as (
SELECT 
			t1.apiPath,
			t1.[source],
			t1.[sourceCount]
		FROM (
      SELECT apiPath, impact.[key] as source, SUM(CAST(ISNULL(impact.[value], 0) AS BIGINT)) AS sourceCount
      FROM gengxintest
      CROSS APPLY OPENJSON(impact) as impact
      GROUP BY apiPath, operation, impact.[key]
    ) AS t1
)
SELECT a.apiPath, SUM(a.[sourceCount]) as occurrence,
    (
      SELECT source, sourceCount AS [count]
      FROM tempTable
      WHERE apiPath = a.apiPath 
      FOR JSON PATH
  ) AS impact
FROM tempTable AS a
GROUP BY apiPath
```
![avatar](/assets/img/2021/02-27/02-27-3.png)

## 3.Ref
1.[Transact-SQL
	- SQL Server文档](https://docs.microsoft.com/en-us/sql/relational-databases/json/json-data-sql-server?view=sql-server-ver15)
