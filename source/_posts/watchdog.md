---
title: 服务器简单的监控脚本
date: 2022-04-08 11:08:14
img: https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202208161500819.svg
tags: 
 - shell
 - 服务器
 - Linux

---
## 服务器简单的监控脚本监控docker等进程

脚本实现监控docker 运行的各个服务和其他重要程序，当某项服务挂了，可发邮件给管理者，并自动重启

## 自动发送邮件

安装maix

![image-20220408111516967](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081115382.png)

关闭默认邮件服务器（centos7）

```sh
systemctl stop postfix
systemctl stop sendmail
```

编辑配置文件  `/etc/mail.rc`，添加下面内容

```
set from=notice@jayce.icu
set smtp=smtps://smtp.exmail.qq.com:465
set smtp-auth-user=notice@jayce.icu
set smtp-auth-password=*******
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb
```

set from  发送邮件的邮箱
set smtp  邮箱的stmp服务器地址
set smtp-auth-user   登录名
set smtp-auth-password  登录名
set smtp-auth=login  登录验证
set ssl-verify=ignore  忽略ssl证书 *重要*
set nss-config-dir=/etc/pki/nssdb  ssl证书目录

具体的邮箱服务器地址，端口，登录密码等以实际邮箱情况为准，有些密码需要授权

阿里云服务器为了防止垃圾短信，默认关闭25号端口，需要在安全组规则开通465端口

配置成功后测试一下：

![image-20220408132759402](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081327730.png)

![image-20220408132818714](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081328786.png)



## 监控docker服务

获取docker的状态

```sh
docker ps -a
```

![image-20220408133446902](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081334165.png)

从status可知具体的状态，从此可进行判断进程状态，先来一个判断是否exit的

```sh
docker ps -a |grep Exited
```

![image-20220408133617687](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081336789.png)

当此命令有返回时，即有服务退出了，利用此特性，可以将退出的进程发送邮件给管理者

取出状态为Exited的服务名：

```shell
docker_images_status_exited=$(docker ps -a | grep Exited | awk '{print $NF}')
```

![image-20220408134055200](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081340268.png)

发送邮件：

```shell
if [ "$docker_images_status_exited" ]; then
  echo  $(date +%Y-%m-%-d-%H:%M:%S) $docker_images_status_exited $status >> /root/logs/docker.log
  echo  $(date +%Y-%m-%-d-%H:%M:%S) $docker_images_status_exited $status | mailx -s "docker 异常" he@jayce.icu 2>> /dev/null
fi
```

如果需要自动启动，将  `docker start $docker_images_status_exited`放入`if`中即可

实际运行：

![image-20220408134442666](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081344811.png)



