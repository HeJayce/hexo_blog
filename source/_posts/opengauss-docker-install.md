---
title: opengauss(华为高斯数据库) 在docker上的安装
date: 2024-10-23 14:14:07
author: Jayce he
categories: 部署
img: https://oss.jayce.icu/markdown/200701.webp
tags:
- opengauss
- 服务器
---

# centos系统下使用docker安装opengauss数据库（单机）



## Docker安装

## 拉取镜像

```sh
docker pull enmotech/opengauss:latest
```

## 启动容器

在基础的启动命令基础上，增加了环境变量、端口映射于持久化存储数据

```sh
docker run --name opengauss --privileged=true -d -e GS_PASSWORD=Enmo@123 -v /enmotech/opengauss:/var/lib/opengauss -u root -p 5432:5432 enmotech/opengauss:latest
```

### 环境变量

- GS_PASSWORD：设置openGauss数据库的超级用户omm以及测试用户gaussdb的密码。如果要从容器外部（其它主机或者其它容器）连接则必须要输入密码。
- GS_NODENAME：数据库节点名称，默认为gaussdb。
- GS_USERNAME：数据库连接用户名，默认为gaussdb。
- GS_PORT：数据库端口，默认为5432。

### 端口映射

openGauss的默认监听启动在容器内的5432端口上，如果想要从容器外部访问数据库，则需要在`docker run`的时候指定`-p`参数。比如以下命令将允许使用外部的5432端口访问容器数据库。

### 持久化存储数据

容器一旦被删除，容器内的所有数据和配置也均会丢失，通过在`docker run`的时候指定`-v`参数来实现。比如以下命令将会指定将openGauss的所有数据文件存储在宿主机的/enmotech/opengauss下。`-u root`参数用于指定容器启动的时候以root用户执行脚本，否则会遇到没有权限创建数据文件目录的问题。

使用`docker ps -a`查看容器是否启动



## 进入容器

```sh
docker exec -it opengauss /bin/bash
```

进入容器后是root账户，切换至omm账户

```
su - omm
```

在omm账户下即可使用连接命令进入数据库

```sh
gsql
```

![image-20231023150447811](https://oss.jayce.icu/markdown/image-20231023150447811.png)



## 出现问题



![image-20231023150634936](https://oss.jayce.icu/markdown/image-20231023150634936.png)

在远程接入数据库时出现FATAL: Forbid remote connection with initial user报错，该原因禁止omm账户登录，解决方法为创建远程账户接入。

具体步骤：

- 增加需要访问计算机的 IP 地址

- 对所有 IP 地址进行开放：0.0.0.0/0

- 修改 trust 替换成 md5 加密方式

  - ```sh
    vim /var/lib/opengauss/data/pg_hba.conf
    #在IPv4 local connections下新加一行
    host    all             all             0.0.0.0/0               md5
    ```

- 修改监听地址和加密方式

  - ```sh
    vim /var/lib/opengauss/data/postgresql.conf
    #修改成下面两行
    listen_addresses = '*'
    password_encryption_type = 0
    ```
    
  - 重启
    - ```sh
      gs_om -t stop
      gs_om -t start
      ```
  
  
  - 创建远程登录用户：
  
  	```sql
  	create user testuser password 'openGauss!666';
  	```
  
  再使用远程账户登录
  
  ![image-20231023153259026](https://oss.jayce.icu/markdown/image-20231023153259026.png)
  
  



vim提示没有该命令：

root下安装vim

```shell
apt-get update
apt -y install vim
```

