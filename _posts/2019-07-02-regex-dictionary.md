---
title: '常用正则'
tags:
  - Regex
categories:
  - Technology
---
记录下工作中用到的或者其他常用的正则


```js
// => reg(正则表达式), value(需要检测的值)
/reg/.test(value)
```
```js
// 获取字符串中的数字存入数组
/\d+/g 
```
```js
// 整个字符串是存数字
^\d+$
```
```js
// 获取字符串中的数字部分
const num = value.replace(/[^0-9]/ig, '');
```
```js
// 获取字符串中的字母部分
const word = value.replace(/[^a-z]+/ig, ''); 
```
```js
// 匹配介于${ 和 } 直接的字符串, AAA是开始位置 BBB是结束位置 匹配介于AAA和BBB之间字符串
(?<=\${)(.*?)(?=}) => (?<=AAA)(.*?)(?=BBB) 
```
```js
// 删除字符串中的所有特殊字符
replace(/[^\w\s]/gi, '') 
```
```js
// 删除字符串中的所有空格
replace(/\s/g,'') 
```
```js
// 匹配任意字符串开头数字结尾的字符串
/^\w+\d+$/ 
```
