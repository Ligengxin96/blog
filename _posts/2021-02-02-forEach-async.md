---
title: 'forEach中使用async/await'
tags:
  - JavaScript
categories:
  - Technology
---
## 1.问题出现
&emsp;&emsp;今天在写Azure Function的时候发现了一个问题,就是为了方便定位问题,于是给下面这段代码添加了log信息

```js
// 大致是需要请求一个接口获取数据然后在把结果发送给另外一个接口
let count = 0;
const result = await fetchDataFromDataBase();
result.forEach(async (item) => {
  await request(item);
  count++;
});
console.log('发送次数', count);
```

&emsp;&emsp;预期结果就是想看看有没有正确的发送请求的数量.当我调试的时候发现结果很奇怪,理论上来说,result.length和count应该是相等的,但是这段代码表现出来的确是count永远输出的都是0.这就让我很好奇了,这到底是为什么.

## 2.解决办法

&emsp;&emsp;查阅资料发现了其实forEach的内部简单来看大概是这样的([forEach源码](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach#polyfill)).

```js
Array.prototype.forEach = function (callback) {
  for (let index = 0; index < this.length; index++) {
    callback(this[index], index, this);
  }
};
```

&emsp;&emsp;可以看到实际上callback并没有await,所以导致了前一次的callback执行后没有等待callback得到返回值就执行下一次的迭代.所以到这里,我们大概知道了怎么解决问题.我们可以自己定义一个函数来实现同步.

```js
async function asyncForEach(array, callback) {
  for (let index = 0; index < array.length; index++) {
    await callback(array[index], index, array);
  }
}
```

&emsp;&emsp;于是就有了如下代码,可是执行后,发现效果还是和以前一样,count始终都是0.

```js
let count = 0;
async function asyncForEach(array, callback) {
  for (let index = 0; index < array.length; index++) {
    await callback(array[index], index, array);
  }
}
const result = await fetchDataFromDataBase();
asyncForEach(result, async (item) => {
  await request(item);
  count++;
});
console.log('发送次数', count);
```

&emsp;&emsp;检查下发现 asyncForEach返回的是一个promise对象,我们没有等待asyncForEach执行完毕就输出了count,所以只需要再在调用asyncForEach的时候前面加上await,但是注意在使用await的时候一定需要被async包裹着,所以我们需要把我们的代码包裹在一个函数里面.下面的代码就正确输出了count的值.

```js
let count = 0;

async function asyncForEach(array, callback) {
  for (let index = 0; index < array.length; index++) {
    await callback(array[index], index, array);
  }
}
const result = await fetchDataFromDataBase();

const start = async () => {
  await asyncForEach(result, async (item) => {
    await request(item);
    count++;
  });

  console.log('发送次数', count);
}

start();

```

## 3.总结

&emsp;&emsp;虽然我查阅资料发现使用的[`for await...of`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of)就能解决这个问题.但是问题的本质我们还是需要去窥探的,这一行最怕的就是浅尝辄止,在深挖问题的过程中我们才会思考,才能成长.

&emsp;&emsp;比如在这个过程中我看了[forEach源码](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach#polyfill)中的这行代码

`if (this == null) { throw new TypeError('Array.prototype.forEach called on null or undefined'); }`

&emsp;&emsp;我就学到了,以后可以偷懒使用`== null`来判断来处理为null 或者 undefined的数据.

```js
// 最终实现代码
let count = 0;

const result = await fetchDataFromDataBase();

const start = async () => {
 for await (const item of result) {
    await request(item);
    count++;
  }

  console.log('发送次数', count);
}

start();
```

## 4.Ref
1. [medium](https://codeburst.io/javascript-async-await-with-foreach-b6ba62bbf404)