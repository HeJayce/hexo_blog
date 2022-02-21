---
title: 阿里云等国内服务器github加速
date: 2022-01-31 15:24:52
author: Jayce he
top: true
categories: 部署
summary: 解决在阿里云等国内的服务器上从github上拉代码，但网络不通的情况
cover: http://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202109171525690.png
tags:
  - 运维
  - 服务器
  - github
  - 阿里云
  - v2ray
  - linux
---

# 阿里云等国内服务器github加速

解决在阿里云等国内的服务器上从github上拉代码，但网络不通的情况。

准备一个v2ray节点

### 步骤：

#### 安装v2ray

```sh
wget https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh  --user-agent="Mozilla/5.0"
```

如果下载不下来，请通过ftp等方式上传

#### 运行下载的脚本

```sh
bash install-release.sh
```

这个过程时间会有点长，但实测阿里云网络可以安装

#### 运行v2ray

```sh
systemctl start v2ray
```

![image-20220129233139100](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202201292331756.png)

#### 修改配置文件

在你的Windows 等v2ray客户端下，将配置文件config.json的内容复制到`/usr/local/etc/v2ray/config.json`下

   方法：

   ![image-20220129233337800](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202201292333040.png)

#### 重启v2ray

#### 使用下命名可测试节点

```sh
/usr/local/bin/v2ray -test -config /usr/local/etc/v2ray/config.json
```

![image-20220129233516568](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202201292335792.png)

检查端口是否通畅

```sh
lsof -i:10809
```

![image-20220129233619810](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202201292336185.png)

`lsof`没有命令的可yum install lsof

#### 添加github代理

   ```sh
   git config --global http.proxy http://127.0.0.1:10809
   git config --global https.proxy http://127.0.0.1:10809
   ```

此时再克隆代码速度飞快

![image-20220129233925182](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202201292339343.png)

