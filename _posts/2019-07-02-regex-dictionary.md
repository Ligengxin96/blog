---
layout: post
title: '常用正则'
tags:
  -
hero: https://source.unsplash.com/collection/145105/
overlay: gray
---
&emsp;&emsp;记录下工作中用到的或者其他常用的正则
<!–-break-–>
```
/reg/.test(value) => reg(正则表达式), value(需要检测的值)

获取字符串中的数字存入数组: /\d+/g
整个字符串是存数字: ^\d+$
获取字符串中的数字部分: const num = value.replace(/[^0-9]/ig, '');
获取字符串中的字母部分: const word=value.replace(/[^a-z]+/ig, '');
匹配介于${ 和 } 直接的字符串: (?<=\${)(.*?)(?=}) => (?<=AAA)(.*?)(?=BBB)  AAA是开始位置 BBB是结束位置 匹配介于AAA和BBB之间字符串
删除字符串中的所有特殊字符: replace(/[^\w\s]/gi, '')
删除字符串中的所有空格: replace(/\s/g,'')
```