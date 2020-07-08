---
layout: post
title: 'wsl使用过程中遇到的问题和解决办法'
tags:
  - issue
hero: https://source.unsplash.com/collection/145140/
overlay: green
---
&emsp;&emsp;使用wsl(Window Subsystem for linux)过程中遇到的问题和解决办法
<!–-break-–>

## git 授权问题
&emsp;&emsp;今天在wsl中打开项目git pull一个在 Azure DevOps平台的repo的时候.在账号和密码输入正确的情况下竟然提示我没授权
(fatal: Authentication failed for '仓库的url').我的解决办法,wsl中输入如下命令后重新git pull
{% highlight bash %} 
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-manager.exe"
{% endhighlight %}