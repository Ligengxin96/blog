---
title: 'ubuntu18.04上安装mysql'
tags:
  - tutorial
categories:
  - tutorial
---
&emsp;&emsp;最近工作是写存储过程.但是又不敢在微软他们数据库中乱操作.虽然有备份,但是搞坏了总不好.刚好最近又买了服务器,那总得用起来嘛.于是网上找教程开始,但是没有一篇文章完整了记录了安装mysql(5.7版本)到ssh远程连接数据库.这又是一篇自己乱撞踩坑的记录.

 
## 卸载mysql
&emsp;&emsp;为什么首先讲卸载mysql.因为我乱撞乱试后没法恢复了就卸载了.而且卸载过程也遇到了问题.估计是没卸载完全,导致后续没办法安装.讲下我的操作过程如何彻底卸载mysql.
```bash  
# 开始卸载,这条卸载命令运行应该没问题
sudo apt-get autoremove --purge mysql-server

# 这条命令我不记得我是否有报错,不过不管报不报错都没影响
sudo apt-get remove mysql-common 

# 下面这条命令据说是清除数据,但是我运行后报错了，如果你没报错,重新安装了
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P

# 如果你上面执行完后没出错了并且可以重新安装,那么接下来就不需要看了.
# 接下来需要手动删除一些文件, 首先找到这些文件
sudo find  / -name mysql -print 
# 运行完应该会显示找到mysql相关的文件(我这是安装后查询的情况,如果你执行完卸载命令应该没这么多文件)
/etc/init.d/mysql
/etc/mysql
/usr/share/mysql
/usr/bin/mysql
/usr/lib/mysql
/var/log/mysql
/var/lib/mysql
/var/lib/mysql/mysql

# 接下来就手动一个个直接删除
rm -rf 查询出来的文件名
```

## 安装mysql和配置远程ssh访问

&emsp;&emsp;安装其实很简单.安装完毕之后的配置最麻烦也是踩坑最多的地方.
```bash  
# 首先输入
sudo apt-get update
# 安装 中间需要输入yes 的一路yes
sudo apt-get install mysql-server

# 安装完毕之后开始配置了
# 进入etc/mysql
cd etc/mysql
# 查看debian.cnf文件
vi debian.cnf
# 会看到差不多如下信息.这样得到了一个用户名和密码.就可以使用这个用户名(user)和密码(password)进入mysql数据库中
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = LZxaHXvRygXuffn5
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = LZxaHXvRygXuffn5
socket   = /var/run/mysqld/mysqld.sock

# 刚刚看到我的密码 LZxaHXvRygXuffn5 所以输入(-u 和-p后不要有空格)
# 登录mysql
mysql -udebian-sys-maint -pLZxaHXvRygXuffn5
# 给root账户设置密码
show databases;
use mysql;
update user set authentication_string=password("你的密码") where user='root';
update user set plugin="mysql_native_password";
flush privileges;
quit;
# 接下来我们在新建一个用户，因为如果用root的话，我遇到了一个问题就是在配置外网访问的时候
# 由于原本mysql中存在一个root的本地访问，会冲突。如果你有别的解决办法也可以。

# username是你新建的用户名 host 我改成了 %,就是允许那些ip可以访问，设置成%是任何ip都能访问  password 是你设置的密码
set user 'username'@'host' identified by 'password';

# 授权刚刚创建的用户 all 是crud的全部权限，也可以自定义crud的部分权限 username 是上一步创建的用户名
grant all on *.* to 'username'@'%';

# 设置和更改用户密码 username 创建的用户名 host 替换为 %  newpasswordss 是你设置的密码
set password for 'username'@'host' = password('newpassword');

# 退出mysql后重启下下mysql
/etc/init.d/mysql restart
```

## 接下来设置mysql的一个配置文件
```bash  
# 进入目录
cd etc/mysql/mysql.conf.d/
# 修改 mysqld.cnf 
vi mysqld.cnf 
# 可以看到这一行信息 只需要前面加个#号注释就行了.
bind-address = 127.0.0.1
# 保险起见在重启下
/etc/init.d/mysql restart
```

## 最后别忘记了放行3306 端口
1. 首先需要在你买的云服务器的控制台设置安全规则放行3306, 具体操作可以点击查看 [发布项目到服务器_1](/posts/publis-project-to-service-1)
2. 然后是iptables放行 3306端口
```bash  
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
```

## 远程终端访问命令(前提需要安装mysql的客户端，这个安装很简单自己百度下就ok)
```bash  
mysql -u 数据库用户名 -p数据库用户密码 -h 服务器ip -P 3306
# 例如 我刚刚创建的用户名是 mycount, 密码是 123456  服务器ip是 123.123.123.123 那么我的命令如下
mysql -u mycount -p123456 -h 123.123.123.123 -P 3306
```