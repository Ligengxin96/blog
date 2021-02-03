---
title: 'wsl使用过程中遇到的问题和解决办法'
tags:
  - WSL
  - WSL2
categories:
  - Issue
---
&emsp;&emsp;使用wsl(Window Subsystem for linux)过程中遇到的问题和解决办法


## git 授权问题
&emsp;&emsp;今天在wsl中打开项目git pull一个在 Azure DevOps平台的repo的时候.在账号和密码输入正确的情况下竟然提示我没授权
(fatal: Authentication failed for '仓库的url').我的解决办法,wsl中输入如下命令后重新git pull
```bash  
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-manager.exe"
```

## git diff 混乱问题
&emsp;&emsp;wsl中打开项目发现几乎所以文件都是modified状态.想看差异的时候会发现没有差异.
我的解决办法,执行以下命令然后刷新下git状态. --gloabl 可加可不加,如果你只想变更这个仓库的配置就不加
```bash  
git config --global core.autocrlf true
git config --global core.filemode false
```
输入这两个命令就应该解决了这个问题,如果还没解决了的话.可以检查下wsl里的git的版本和windows中git版本是否一致
如果还没解决,可以点击这个 [issue](https://github.com/microsoft/WSL/issues/184) 找找是否有新的解决方案.
