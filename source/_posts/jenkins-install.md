---

title: Jenkins安装
date: 2022-02-05 14:16:50
categories: 部署
tags:
  - Jenkins
  - 阿里云
  - linux
---
# Jenkins 安装
## 安装JDK
### 下载jdk
进入[JDK下载官网](https://www.oracle.com/java/technologies/downloads/#java8) 下载JDK

这里选择下载131版本，如果版本较新也没问题，[老版本下载链接](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html) 

在服务器下创建安装的文件夹，在`usr`文件夹下创建java文件夹
``` shell
mkdir -p /usr/java
```
将下载好的JDK解压至该文件夹
``` shell
tar -zvxf jdk-8u131-linux-x64.tar.gz -C /usr/java
```
### 配置环境变量
与windows类似，Linux下配置环境变量只需修改配置文件即可
```shell
vim /etc/profile
```
在文件下添加：
```
#java
export JAVA_HOME=/usr/java/jdk1.8.0_131
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
执行`source /etc/profile`生效

执行`java -version` 如果返回版本号则安装成功


## 安装Jenkins

- yum源导入

```shell
#添加Yum源
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```
如果该步骤无法下载，报错ERROR: cannot verify pkg.jenkins.io's certificate, issued by ‘/C=US/O=Let's Encrypt/CN=R3’:

这是因为wget命令下载不安全的https 域名下的内容，此时需要安装ca-certificates

```shell
yum install -y ca-certificates
```

```shell
#导入密钥
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

- 安装

```shell
yum install -y jenkins
```

如果缺少daemonize，则同样yum安装daemonize



访问IP+8080 端口即可访问Jenkins界面

![image-20220308142711995](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202203081427668.png)

如果不能访问请检查防火墙，添加8080端口

```sh
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```



第一次打开需要解锁Jenkins，查看web端提示的密码文件

![image-20220308142939118](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202203081429423.png)

将此行密码复制进去即可

接着选择安装推荐的插件

![image-20220308143042461](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202203081430531.png)

等待即可

![image-20220308143106111](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202203081431177.png)

创建管理员

![image-20220308144503922](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202203081445918.png)

URL可自愿修改

![image-20220308144542253](C:\Users\EB\AppData\Roaming\Typora\typora-user-images\image-20220308144542253.png)
