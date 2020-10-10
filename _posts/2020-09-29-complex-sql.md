---
layout: post
title: '一次复杂的需求的存储过程实现代码'
tags:
  - tutorial
  - technology
hero: https://source.unsplash.com/collection/145171/
overlay: orange
---
&emsp;&emsp;这次需求类似于需要对比某个产品的不同月份的销售数据,生成一个趋势的新字段,也就是说如果你的这个产品有4个月的数据的话,那么就应该有在这个趋势的字段里面就应该有3个值来描述4个月的趋势变化.难点应该在于如何对比并把生成趋势这个字段.
<!–-break-–>
 
## 具体代码
{% highlight sql %}
WITH source AS (
SELECT column1,
       column2,
       column3,
       column4,
       column5,
       column6,
       column7,
       a.column8,
       b.column9
FROM table1 AS a
LEFT JOIN 
(   
    SELECT column9, column10, column8 FROM table2) AS b
    ON a.column8 = b.column8 AND a.column3 = b.column10
    WHERE column2 = 'prod' AND  column1 in ('last1week', 'last2week', 'last3week', 'last4week') AND column4 != 'N/A'
), 
sourceWithTrend AS (
SELECT *,
(
	CASE
    WHEN DATEDIFF(DAY, (LAG(column5 + column6, 1) over (PARTITION BY column3, column4, column8 ORDER BY column1 DESC)), column5 + column6) > 0 then nchar(8599) 
	WHEN DATEDIFF(DAY, (LAG(column5 + column6, 1) over (PARTITION BY column3, column4, column8 ORDER BY column1 DESC)), column5 + column6) = 0 then nchar(8594) 
	WHEN DATEDIFF(DAY, (LAG(column5 + column6, 1) over (PARTITION BY column3, column4, column8 ORDER BY column1 DESC)), column5 + column6) < 0 then nchar(8600)
	ELSE nchar(8594) 
	END
) AS missingTrend,
(
	CASE 
    WHEN DATEDIFF(DAY, (LAG(column7, 1) over (PARTITION BY column3, column4, column8 ORDER BY column1 DESC)), column7) > 0 then nchar(8599) 
	WHEN DATEDIFF(DAY, (LAG(column7, 1) over (PARTITION BY column3, column4, column8 ORDER BY column1 DESC)), column7) = 0 then nchar(8594) 
	WHEN DATEDIFF(DAY, (LAG(column7, 1) over (PARTITION BY column3, column4, column8 ORDER BY column1 DESC)), column7) < 0 then nchar(8600)
	ELSE nchar(8594) 
	END
) AS incorrectTrend
FROM source
)
, sourceWihtTrendFilter AS (
SELECT * 
FROM sourceWithTrend
WHERE column1 != 'last4week'
)

SELECT DISTINCT column3, column4, column8, column9,
(   
    SELECT ' ' + missingTrend 
    FROM sourceWihtTrendFilter AS a 
    WHERE a.column3 = b.column3 AND a.column4 = b.column4 AND a.column8 = b.column8 for xml path('')
) AS missingTrend,
(
    SELECT ' ' + incorrectTrend 
    FROM sourceWihtTrendFilter AS a 
    WHERE a.column3 = b.column3 AND a.column4 = b.column4 AND a.column8 = b.column8 for xml path('')
) AS incorrectTrend
FROM sourceWihtTrendFilter AS b 
GROUP BY column3, column4, column1, column8, column9, column5, column6, column7
{% endhighlight %}