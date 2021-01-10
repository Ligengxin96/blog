---
layout: post
title: 'Azure 踩坑合集'
tags:
  - issue
hero: https://source.unsplash.com/collection/145183/
overlay: blue
---
&emsp;&emsp;记录在使用Azure过程中遇到的的一系列问题和解决办法.
<!–-break-–>
 
## 1.Azure Function

* Invalid ELF header from build/Release/nodegit.node

![avatar](/assets/img/2020/12-25/2020-12-25-01.jpg)

错误信息: 'Error: /home/site/wwwroot/node_modules/nodegit/build/Release/nodegit.node: invalid ELF header'

解决办法: 不要用windows环境部署Azure Function, 可以用WSL

* 不要在Data Factory trigger pipeline 的时候debug本地的Azure Funtion.

Data Factory 执行pipeline 有两种方式一个是trigger方式一个是debug方式,所以我遇到过一个问题就是debug运行这个pipeline(这个pipeline执行Azure Funtion)是正常的,trigger是不正常的.后来才知道debug会跑来调用我本地的服务,而trigger不会,最终原因还是因为云端和本地的Azure Funtion配置不同步(我以为同步了)

## 2.Data Factory

* Data Factory Pipeline 调用sql server存储过程性能不稳定

![avatar](/assets/img/2020/12-25/2020-12-25-02.png)

在修改数据表的索引后,性能提升并不明显,目前处在在最坏的情况和最好的情况的平均值. 

* Debug 运行Azure Funtion正常但是 Trigge
