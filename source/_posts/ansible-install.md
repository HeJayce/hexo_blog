---
title: ansible的部署配置与简单使用
date: 2023-02-15 15:32:39
author: Jayce he
top: false
categories: 部署
summary: 部署ansible，实现自动化运维，批量系统操作
img: https://oss.jayce.icu/markdown/ansible-svgrepo-com.svg
tags:
- 运维
- 服务器
- 部署
- linux
- ansible
---

# ansible的部署配置与简单使用

## 安装部署

分别在centos的主机和docker中的centos部署ansible

### 物理主机部署

```shell
yum -y install epel-release
yum -y install ansible
```

如果安装失败，尝试配置一下yum源

配置yum：

进入`/etc/yum.repos.d/`文件夹下创建一个阿里的镜像地址配置文件`aliBase.repo`,内容如下

```repo
[base]
name=aliBase
baseurl=https://mirrors.aliyun.com/centos/$releasever/os/$basearch/
gpgcheck=0

[aliEpel]
name=aliEpel
baseurl=https://mirrors.aliyun.com/epel/$releasever\Server/$basearch/
gpgcheck=0

[epel]
name=epel
baseurl=https://mirrors.aliyun.com/epel-archive/7/x86_64/
gpgcheck=0
```

然后再执行`yum -y install ansible`即可

安装成功后执行`ansible --version`验证

![image-20230215161647664](https://oss.jayce.icu/markdown/image-20230215161647664.png)



## 机器配置与分组

主机的信息配置在`/etc/ansible/hosts` 文件中，想要批量操作主机，就需要提前对主机的连接参数进行配置，根据业务再把不同的主机分组，来执行各项命令

打开hosts文件已经有一些配置的样例在里面，如果想添加主机，需要先设置一个组，在这个组里填写每台主机的连接参数，实例如下：

```
[webservers]
192.168.42.131 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass="12345"
192.168.42.132 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass="12345"
```

该配置将两台主机分到webservers这一组下，配置了ip，ssh端口，用户名及密码，此时保存退出，使用ansible webservers -m ping 来试一下连通性

如果网络正常的话，可能会出现下面的报错

![image-20230215162530953](https://oss.jayce.icu/markdown/image-20230215162530953.png)

原因是该主机没有连接过配置中的主机，导致宿主机的known_hosts中没有相关记录，简单的方法是直接使用ssh命令连一下

![image-20230215162756540](https://oss.jayce.icu/markdown/image-20230215162756540.png)

输入yes后即可更新known_hosts

如果不想每台主机都ssh连一下，可以修改配置文件关闭验证

在`/etc/ansible/ansible.cfg` 文件下将`host_key_checking = False` 前的注释去掉

测试成功的输出如下

![image-20230215163151910](https://oss.jayce.icu/markdown/image-20230215163151910.png)



除了最简单的ip加用户名密码这种最简单的方式外，多个ip还支持合并，比如

`192.168.42.13[1:3]` 代表`192.168.42.131` `192.168.42.132` `192.168.42.133` 这三个ip

端口也可以直接跟在ip后面

```
192.168.42.131:22 ansible_ssh_user=root ansible_ssh_pass="12345"
192.168.42.13[1:3]:22 ansible_ssh_user=root ansible_ssh_pass="12345"
```





## 批量操作



