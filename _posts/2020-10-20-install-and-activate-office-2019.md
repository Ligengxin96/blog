---
layout: post
title: '不需要下载激活软件,免费激活office 2019'
tags:
  - tutorial
  - tools
hero: https://source.unsplash.com/collection/145177/
overlay: green
---
&emsp;&emsp;在公司办公office 365用起来是真的舒服,今天在家突然发现电脑上没有,果断下载一个office 2019,本着能省则省的原则.Google了一个白嫖的方式,总结下写了一个傻瓜教程.
<!–-break-–>
 
## 1.下载offcie 2019 
使用迅雷下载office 2019, 下载链接如下
{% highlight bash%}
ed2k://|file|cn_office_professional_plus_2019_x86_x64_dvd_5e5be643.iso|3775004672|1E4FFA5240F21F60DC027F73F1C62FF4|/
{% endhighlight %}

## 2.激活使用的命令
管理员运行cmd, 一定要是管理员运行cmd.按步骤运行如下命令

1.进入office的安装目录
{% highlight bash%}
cd /d %ProgramFiles%\Microsoft Office\Office16
cd /d %ProgramFiles(x86)%\Microsoft Office\Office16
{% endhighlight %}

2.转换许可证
{% highlight bash%}
for /f %x in ('dir /b ..\root\Licenses16\ProPlus2019VL*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%x"
{% endhighlight %}

3.激活
{% highlight bash%}
cscript ospp.vbs /setprt:1688
cscript ospp.vbs /unpkey:6MWKP >nul
cscript ospp.vbs /inpkey:NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP
cscript ospp.vbs /sethst:kms8.msguides.com
cscript ospp.vbs /act
{% endhighlight %}

4.如下信息表示激活成功
![avatar](/assets/img/2020/10-20/2020-10-20.png)