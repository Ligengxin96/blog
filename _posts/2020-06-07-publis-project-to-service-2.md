---
layout: post
title: '发布项目到服务器-2'
tags:
  - issue
hero: https://source.unsplash.com/collection/145102/
overlay: green
---
&emsp;&emsp;解决了 [昨天的问题](/posts/publis-project-to-service-1),成功把项目发布到了80端口,可是发现进来默认是我第一个玩具项目,
于是尝试把项目切换到一个新端口上,可是问题又出现了.
<!–-break-–>

## 问题出现
&emsp;&emsp;和前一样要把项目发布到特定的端口上,就需要配置nginx.很快找到了发布项目到服务器上的 [参考资料](https://segmentfault.com/a/1190000019442994) 
然后,根据教程做完后(我设置是是发布的到8000端口),访问服务器ip:8000.发现并没有用.页面返回'无法访问此网站,服务器响应时间过长'.
当时看到这不是和昨天一个情况吗.果断在在华为云控制台的访问控制中把8000端口加上.然后刷新页面,结果还是没用.因为把项目发布到80端口能正常访问,
但是发布到8000端口不能访问,首先排除项目代码的问题.然后排查是否是nginx配置文件的问题,发现 nginx 配置也没问题.
 
## 解决方案
&emsp;&emsp;没办法,查下资料看看是什么原因导致的服务器返回'无法访问此网站,服务器响应时间过长'的提示.好在浏览器给出的解决办法有下面3个,
可以尝试用给出的解决办法尝试解决问题.
1. 检查网络连接
2. 检查代理服务器和防火墙
3. 运行 Windows 网络诊断

1和3可以排除了.剩下就是代理服务器的问题和防火墙的问题了.尝试关闭代理后问题依然存在,那么问题只剩下防火墙的问题了.遂查看防火墙状态,
发现服务器防火墙是开着的.然后运行 `sudo ufw disable` 关闭防火墙,刷新页面,成功出现了项目首页.问题解决.果然是防火墙问题.
接着运行`sudo ufw enable` 开启防火墙,然后运行 `iptables -A INPUT -p tcp --dport 8000 -j ACCEPT` 让8000端口能通过防火墙.

## 附上我的Nginx的配置文件
这是我有别于 [参考资料](https://segmentfault.com/a/1190000019442994) 的nginx配置文件,其他配置和其一致
```
server {

  listen 8000; // 端口号

  server_name  localhost;

  root /opt/fontEnd/bikeManagement; // index.html 文件所在目录

  index index.html index.htm;

  location / {
    try_files $uri $uri/ =404;
  }
}
```

