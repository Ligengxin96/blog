---
layout: post
title: '使用frp实现windows远程桌面连接'
tags:
  - tools
  - technology
hero: https://source.unsplash.com/collection/145114/
overlay: blue
---
&emsp;&emsp;公司不允许安装非授权的软件,所以迫于无奈,只能放弃teamviewer这些远程软件,只能使用window自带的远程桌面.
<!–-break-–>

网上结合了N个教程后,踩坑并汇总出来的傻瓜式教程.

1.下载frps 

&emsp;&emsp;下载地址 https://github.com/fatedier/frp/releases
服务器自己查看自己的系统是32位的还是64位的.我的是64位的ubuntu所以下载了[frp_0.33.0_linux_amd64.tar.gz](https://github-production-release-asset-2e65be.s3.amazonaws.com/48378947/03a3af00-88ad-11ea-91e9-21b33a8d8ff6?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200610%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200610T125729Z&X-Amz-Expires=300&X-Amz-Signature=93b7ae7f061249fa0ccfbbae17fa2b1d738a14a34cc5814d0cfcc310fd4c9851&X-Amz-SignedHeaders=host&actor_id=46650314&repo_id=48378947&response-content-disposition=attachment%3B%20filename%3Dfrp_0.33.0_linux_amd64.tar.gz&response-content-type=application%2Foctet-stream)客户端就是我们自己的电脑就下载[frp_0.33.0_windows_amd64.zip](https://github-production-release-asset-2e65be.s3.amazonaws.com/48378947/6137fb80-88ad-11ea-8bce-d161b9764100?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200609%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200609T125432Z&X-Amz-Expires=300&X-Amz-Signature=6e530f9428d7628becf44347cbe0f5b9bbf68d92c98dfe3a8f7c085d636e9dd4&X-Amz-SignedHeaders=host&actor_id=46650314&repo_id=48378947&response-content-disposition=attachment%3B%20filename%3Dfrp_0.33.0_windows_amd64.zip&response-content-type=application%2Foctet-stream)

2.配置

服务器端: frp_0.33.0_linux_amd64.tar.gz 解压缩来后,进入解压出来的文件夹,ls 命令应该会出现如下的文件
![avatar](/assets/img/2020/06-09/2020-06-09.png)
最主要的是两个配置文件frps.ini 和 frpc.ini 这两个文件(复制的时候请删除注释,注释只是说明这个的应该写什么数据)

frps.ini 的配置如下(其实我感觉这个配置没啥用,但是因为别的教程都配置了,也跟着写了下,最后能用就没动)
{% highlight bash %}
  [common]
  bind_port = 7000 // 绑定的端口
  dashboard_user = username // 这个用户名
  dashboard_port = 7500 // 不知道是什么,完全没用到7500端口
{% endhighlight %}
frpc.ini 的配置如下
{% highlight bash %}
  [common]
  server_addr = 服务器ip地址 
  server_port = 7000 // 服务器端口

  [ssh]
  type = tcp
  local_ip = 127.0.0.1
  local_port = 22
  remote_port = 6001
{% endhighlight %}
客户端: frp_0.33.0_windows_amd64.zip 解压出来的文件和上面解压出来的差不多,只需要关注配置文件即可
frps.ini 的配置如下
{% highlight bash %}
  [common]
  bind_port = 7000
{% endhighlight %}
frpc.ini 的配置如下
{% highlight bash %}
  [common]
  server_addr = 服务器ip
  server_port = 7000

  [ssh]
  type = tcp
  local_ip = 127.0.0.1
  local_port = 3389  // 3389 远程桌面的端口 这个好像不能随便改
  remote_port = 7004  // 这个是远程桌面连接主机的时候输入的ip后加上的端口
{% endhighlight %}

3.启动运行
服务端: 进入解压出来的文件夹, 输入(二选一,推荐第一种后台运行)
{% highlight bash %} 
  // 后台运行进程
  nohup ./frps -c ./frps.ini & 
  // 不后台运行
  ./frps -c ./frps.ini
{% endhighlight %}
  
客户端: 进入解压出来的文件夹
{% highlight bash %} 
  // cdm的话就 输入
  frpc -c frpc.ini
  // power shell的话就 输入
  ./frpc -c frpc.ini
{% endhighlight %}
4.上面步骤结束后,就可以通过输入 服务器ip:端口, 假如你的服务器ip是 123:123:123:123,根据我的配置就应该输入 123:123:123:123:7004 (冒号要用英文的)
![avatar](/assets/img/2020/06-09/2020-06-09_1.png)
如果不行的话请往下看步骤6

5.我的附加的便捷操作

在客户端,即我们的电脑

(1).让电脑每天定时开机自启,这个需要设置bios.自行百度解决

(2).开机自启运行一个bat文件,让我们客户端的程序开机自启,在
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp 目录下写一个bat文件,bat文件内容如下
{% highlight bash %} 
  cmd /k "cd /d 你的解压的文件路径 && frpc -c frpc.ini"
{% endhighlight %}
  我的路径是 D:\Downloads\frp_0.33.0_windows_amd64 所以我的bat文件内容如下
{% highlight bash %} 
  cmd /k "cd /d D:\Downloads\frp_0.33.0_windows_amd64 && frpc -c frpc.ini"  
{% endhighlight %}
6.一些注意的地方

(1) 配置文件中的端口号需要在云服务的安全策略中放行,具体操作方式可以看[发布项目到服务器-1](/posts/publis-project-to-service-1).
如果你的服务器开启的防火墙的话(自行百度下查看防火墙状态命令),还需要放行这些端口,服务器放行端口命令如下
{% highlight bash %} 
  iptables -A INPUT -p tcp --dport 端口号 -j ACCEPT
  // 举个例子 放行我配置中的 7004 端口
  iptables -A INPUT -p tcp --dport 7004 -j ACCEPT
  // 顺便放行下 配置文件中出现的 7000 端口
  iptables -A INPUT -p tcp --dport 7000 -j ACCEPT
{% endhighlight %}
(2) 服务器重启后需要重新启动下服务,重复步骤3.还有可能还需要重新输入 放行端口的命令

