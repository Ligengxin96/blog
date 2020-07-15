---
layout: post
title: '存储过程入参默认值问题'
tags:
  - issue
hero: https://source.unsplash.com/collection/145131/
overlay: red
---
&emsp;&emsp;简单记录下一个因为存储过程默认值问题导致浪费时间在一段没问题的存储过程中找问题的事情.
<!–-break-–>

## 问题出现
&emsp;&emsp;因为一个表数据不对,所以开始排除问题,最终定位到某个存储过程.于是在一系列的测试过后,终于找到了问题.定位问题和不停的调试存储过程排除了一个个问题.最终发现了一个地方有问题,到这里感觉已经99%找到问题了,一切似乎都很顺利,但是我改完后运行下存储过程看看最终效果的时候,就是数据还是不对.这我就纳闷了啊,因为部分数据不对,部分数据是正确的啊(最后发现原因是因为在测试过程中,修改了部分数据,而这部分修改后的数据是正确的).这个时候难免不怀疑自己.于是在这个正确的存储过程中,继续找问题.

## 解决方案
&emsp;&emsp;如果我没偶然看你的表中的最后一列的数据最后刷新时间的话.我估计能在这个正确的存储过程调试一天.因为偶然看到表的数据的最后刷新时间,发现不是刚刚刷新的,而是几天前.这个时候我就意识到.存储过程没问题,而是没有执行更新数据.为什么没更新数据呢,首先看下存储过程入参的定义

{% highlight sql %}
CREATE PROCEDURE [dbo].[uspMonthlySwaggerKPICalculation]
	(
	@env nvarchar(50) = 'prod',
	@endDate Date
)
{% endhighlight %}

很明显,env 这个参数有默认值 'prod',而我在执行的时候却输入的是NULL,为什么会输入NULL呢,是因为我是SSMS里面右键运行存储过程,然后他有个弹窗输入这个存储过程的参数.当时我就想偷个懒.因为env 这个参数有默认值,所以我就不输入,我就勾选了 pass null value(允许空值)选项.这个时候SSMS自动生成了如下的代码并执行.

{% highlight sql %}
DECLARE	@return_value int

EXEC	@return_value = [dbo].[uspMonthlySwaggerKPICalculation]
		@env = NULL,
		@endDate = '2020-03-31'

SELECT	'Return Value' = @return_value

GO
{% endhighlight %}

于是乎,env 的默认值并没有取到 'prod',而是被覆盖为了NULL.所以导致你看起来允许了存储过程,但是他缺没查询到任何数据,逻辑里面也就导致没更新任何数据.

&emsp;&emsp;存储过程有默认值的情况下,如果不传那么正确的写法不是传NULL而是这样的

{% highlight sql %}
DECLARE	@return_value int

EXEC	@return_value = [dbo].[uspMonthlySwaggerKPICalculation]
		--@env = NULL, 这里直接不传
		@endDate = '2020-03-31'

SELECT	'Return Value' = @return_value

GO
{% endhighlight %}

## 总结
&emsp;&emsp;直接用js的理解来说就是不传就是undefined这个时候sql server就会取默认值.而你如果传了NULL,那就是有值,值是NULL,所以会覆盖掉默认值.
还有就是少偷懒,按规范来,也是偷懒的教训.
