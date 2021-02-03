---
title: '初识node.js并发'
tags:
  - Node.js
  - TypeScript
categories:
  - Technology
---
&emsp;&emsp;最近的一个任务的部分内容是需要去读取多个文件中的数据(10个文件,平均每个文件250,000行数据,一共2500,000行数据),然后将这些数据作为payload发送给一个服务器.


## 问题出现
&emsp;&emsp;开始的时候还以为很简单,不就是读取文件数据然后用fetch或者axios把数据发送给服务器吗?但是当我代码写好后,发现了一个很严重的问题.因为每一次发送请求都需要1s左右,也就是说发送完全部数据需要2500,000s也就是需要大约700小时,将近一个月的时间。

&emsp;&emsp;这么长的时间谁能接受呢,仔细想想,问题可能出现在哪里.读取数据太慢了?不一会我就否定了这个问题,因为测试发现读取文件的速度是很快的.再想想,现代计算机的性能瓶颈应该出现在IO上.而目前IO没问题.所以感觉唯一可以优化的地方就是降低每次请求的时间和提高每秒发送的请求数.

## 解决方案
&emsp;&emsp;1. 降低每次请求的时间.配置一个https Agent,设置 keepAlive: true 这样就可以避免每次连接需要进行三次握手节约而降低每次请求的时间.配置结束后测试发现现在一次请求只需要0.3s了.但是这样做显然还是不够,只提升了3倍性能.跑完全部的数据还是需要很长时间.接下来只能提升每秒发送请求的数据了.
```js
const sslConfiguredAgent = new https.Agent({
        cert: fs.readFileSync(process.env.HTTPS_CERT_PATH!),
        key: fs.readFileSync(process.env.HTTPS_KEY_PATH!),
        rejectUnauthorized: false,
        keepAlive: true,
      });
```

2. 提高每秒发送请求的数,这个问题可触及到我的知识盲区了,因为以前就写过前端,对服务端的并发了解并不是很多.不过经过不断的努力的查询资料,终于让我找到了解决办法.使用promise.all 提高每秒发送请求的数.这里promise.all 中存放了100个promise对象,也就是说并发数为100.因为readFilePayload是一个generator函数,所以就算100个并发的去读取文件内容也是能够保证同步的不会读取到用一行的数据.经过这次改进测试发现250,000行数据总用时只需要15分钟.(250000 / 15 / 60 ≈ 278,大致符合提升300倍效率理论数据)

```js
const dataSource = utils.readFilePayload(filePath);
const allPromises: Array<Promise<{resultAry: string[]; errorAry: string[]}>> = [];
for (let i = 0; i < 100; ++i) {
allPromises.push(utils.sendRequest(dataSource));
}
const results = await Promise.all(allPromises);
```

## 总结
&emsp;&emsp;总体来说,这次这个任务让我认识到了node.js的强大,也让我明白了我要走的路还很长.因为我感觉这些代码如果我不去参考别人的内容是否可以写出这样的代码？现在肯定不行.


## 剩余部分关键代码
worker.ts(执行函数)
```js 
import * as utils from './utils';

const loadFiles = async() => {
  const filesAry = fs.readdirSync(directoryPath);
  let allFilesTotalOps: number = 0;
  for (const fileName of filesAry) {
    allFilesTotalOps += await main(directoryPath + '/' + fileName);
  }
} 

const main = async(filePath: string): Promise<number> => {
  const testStart = Date.now();
  const dataSource = utils.readFilePayload(filePath);
  const allPromises: Array<Promise<{resultAry: string[]; errorAry: string[]}>> = [];
  for (let i = 0; i < 100; ++i) {
    allPromises.push(utils.sendRequest(dataSource));
  }
  const results = await Promise.all(allPromises);
  let successOps: number = 0;
  let errorOps: number = 0;
  let allErrorInfo: string[] = [];
  for (const result of results) {
    successOps += result.resultAry.length;
    errorOps += result.errorAry.length;
    allErrorInfo = allErrorInfo.concat(result.errorAry);
  }
  return successOps + errorOps;
};

loadFiles();
```

utils.ts(工具函数)
```js
const endpoint = process.env.endpoint;
const sslConfiguredAgent = new https.Agent({
        cert: fs.readFileSync(process.env.HTTPS_CERT_PATH!),
        key: fs.readFileSync(process.env.HTTPS_KEY_PATH!),
        rejectUnauthorized: false,
        keepAlive: true,
      });

export const sendRequest = async (
    source: AsyncGenerator<[string, RequestResponsePair, RequestResponseResult]>,
  ): Promise<{resultAry: string[]; errorAry: string[]}> => {
    const resultAry: string[] = [];
    const errorAry: string[] = [];
    for await (const [id, req] of source) {
      try {
        const handle = await fetch(endpoint, {
          method: 'POST',
          agent: sslConfiguredAgent,
          body: JSON.stringify(req),
          headers: {
            "Content-Type": "application/json",
          },
        });
        const result = await handle.json();
        if (result.id) {
          resultAry.push(result.id);
        } else {
          throw new Error(JSON.stringify(result));
        }
      } catch (e) {
        console.error(id);
        console.error(e);
        errorAry.push(JSON.stringify({id, e: e.toString()}));
        continue;
      } 
    }
    return {resultAry, errorAry};
  };

export const readFilePayload = async function* (
    filePath: string
  ): AsyncGenerator<[string, RequestResponsePair, RequestResponseResult]> {
    const parser = csvParse({ delimiter: "," });   //import csvParse from "csv-parse";
    const iter = fs.createReadStream(filePath).pipe(parser);
    const pendingRequest = new Map<string, RequestResponsePair>();
    const pendingResponse = new Map<string, RequestResponseResult>();
  
    for await (const r of iter) {
      const [id, payloadStr] = r;
      let payload: RequestResponsePair | RequestResponseResult;
      try {
        payload = JSON.parse(payloadStr);
      } catch (e) {
        console.error(e);
        continue;
      }
  
      if ("someProperty" in payload) {
        const pendingPair = pendingResponse.get(id);
        if (pendingPair === undefined) {
          pendingRequest.set(id, payload);
        } else {
          pendingResponse.delete(id);
          yield [id, payload, pendingPair];
        }
      } else {
        const pendingPair = pendingRequest.get(id);
        if (pendingPair === undefined) {
          pendingResponse.set(id, payload);
        } else {
          pendingRequest.delete(id);
          yield [id, pendingPair, payload];
        }
      }
    }
  }
```
