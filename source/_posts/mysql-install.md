---
title: mysql 在ubuntu和centos上的安装
date: 2022-03-09 15:14:07
author: Jayce he
categories: 部署
img: https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204080944267.jpeg
tags:
- mysql
- 服务器
---

# mysql 在ubuntu和centos上的安装

## ubuntu

系统：Ubuntu16.04

```
apt-get update
apt-get install mysql-server
```

需要注意的是，在安装mysql 后是不可以使用navicat等远程工具进行连接的

默认安装后，3306端口只绑定给了localhost，所以外网是无法访问的

解决方法：

```sh
vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

将bind-address   = 127.0.0.1注释掉

![image-20210721175850771](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202203091514554.png)

重启即可

```
systemctl restart mysql 
service mysql restart
```

 接着授予权限：

## centos

系统：centos7

下载mysql5.7的本地仓库文件

```sh
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

将仓库信息添加至yum

```shell
yum -y install mysql57-community-release-el7-10.noarch.rpm
```

接着在通过yum进行安装

```shell
yum -y install mysql-community-server --nogpgcheck
```

此时mysql会取代mariadb

启动mysql服务

```sh
systemctl start mysqld
systemctl status mysqld
```

此时的登陆密码存放在文件里，查看密码，用该初始化密码登录

```
grep "A temporary password is generated" /var/log/mysqld.log|awk '{print $11}'
```

![image-20220309150733851](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202203091514491.png)

使用命令登录数据库后，修改该默认密码，否则无法使用

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '$Password';
```

其中，新密码必须使用大写，小写，数字和特殊字符组成

开启远程登录

```sql
grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;
flush privileges; 
```

其中%代表所有ip，password替换为数据库密码。



## centos一键部署脚本

**[Rate limit · GitHub](https://github.com/HeJayce/linux-and-shell/blob/main/mysql/mysql_install.sh)，初来乍到，有错请指教，默认密码为`Abc123!@#`**
