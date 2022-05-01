---
title: 使用hexo框架搭建个人博客
date: 2022-04-25 16:36:01
author: Jayce he
top: true
categories: 部署
summary: 使用hexo框架搭建个人博客
img: https://jaycehe.oss-cn-hangzhou.aliyuncs.com/20220501170040.png
tags:
 - 服务器
 - github
 - 阿里云
 - https
 - hexo
 - 博客
 - Markdown
---

使用hexo框架搭建个人博客已有2个多月，是时候写一篇部署步骤以及踩坑总结了
为什么使用hexo

## 安装环境

hexo可以直接通过npm进行安装，windows同理

```
npm install -g hexo-cli
```

![image-20220427224941463](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427224941463.png)

出现此报错命令前加sudo

安装成功后，可以直接新建一个项目

![image-20220427225043033](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427225043033.png)



新建一个项目，并初始化，其中名字自定义	

```sh
hexo init blog_name
```

此时hexo会从github将框架拉到本地的项目名称文件夹下

![image-20220427225351741](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427225351741.png)

![image-20220427225423970](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427225423970.png)

使用ide打开，我使用的是webstorm，根据个人喜好即可

打开项目后，使用``npm install` 将package.json 中的包下载下来

![image-20220427225734227](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427225734227.png)

此时这个最简单的项目基本框架就算有了，生成一下静态文件，用浏览器看看吧

```sh
hexo generate  #生成静态文件   简写hexo g
hexo server    #启动服务器    简写hexo s
```

![image-20220427230214814](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427230214814.png)

打开[http://localhost:4000/](http://localhost:4000/)

即可进入欢迎界面：

![image-20220427230335376](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427230335376.png)



接着我们在github上新建一个仓库，仓库的名字必须是`用户名.github.io`格式，比如![image-20220427231524340](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427231524340.png)

将静态网页部署在此仓库即可通过上面的域名进行访问。如果觉得github的网络不好也可以使用国内的gitee，操作方法一致。

接着安装deploy-git

```sh
npm install hexo-deployer-git --save
```

在配置文件`_config.yml`中添加刚才的仓库信息，注意这里的config是根目录下的，而不是主题配置文件

```go
deploy:
  type: git
  repo: git@github.com:HeJayce/HeJayce.github.io.git
  branch: master
```

这里的repo是你仓库的git地址，branch默认master，如需要分支，可改为相应分支

使用`hexo d`即可将项目push到github仓库去





## 模板选择

在github中有许多开源的模板（主题）可供大家选择，大家也可自行百度，选择自己喜欢的模板。我选择的是[mater模板](https://github.com/blinkfox/hexo-theme-matery)，不同的模板有不同的配置，选择的时候需要仔细阅读开发者提供的使用说明

![image-20220427230711782](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427230711782.png)

将喜欢的模板下载下来后，放入themes 文件夹下，直接将整个文件夹放入就好

![image-20220427231227399](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427231227399.png)

接着在`_config.yml`中找到theme ，将theme的参数改为模板名称，注意模板名称为文件夹名称，改完后重新生成文件并启动服务：



```sh
hexo cl
hexo g
hexo s
```

![image-20220501161105372](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501161105372.png)

此时再打开localhost 4000端口，看看主题有没有配置成功

![image-20220501161217056](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501161543955-20220501171717958.png)



## 第一篇博客

框架打好后，试一下写第一篇博客吧，新建一个post，hexo会自动生成一个md文件，需要熟悉markdown这种文件格式

```
hexo new post [blog_name]
```

![image-20220501161543955](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501161543955-20220501171717958.png)

![image-20220501162931117](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501162931117-20220501171718190.png)

此时就可以编写了，随便写点内容，看看有没有发布吧。编写好后按照上面的方法生成静态文件，再次打开localhost 4000 ，看看有无文章

![image-20220501163000610](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501163000610.png) ![image-20220501163019437](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501163019437.png)



## 服务器部署

由于是静态网页，服务器部署起来非常简单，如果自己有服务器可以进行服务器部署，后续可增加可玩性。

静态网页通过nginx即可完成，没有nginx的通过yum安装即可

```sh
yum install epel-release
yum install -y nginx
```

将github上发布的代码克隆到服务器即可

![image-20220428145145675](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204281451049.png)

在nginx配置文件中修改成下面的配置，root填写项目文件夹地址

```nginx
server {
        listen       80;
        server_name  localhost;
        location / {
            root  /root/HeJayce.github.io;
            index index.html;
        }
}
```

改好别忘了重载nginx配置

这时打开浏览器访问IP:80 或域名就会打开index.html，即你的主页



## https

设置https可以到阿里云申请免费的证书，每人有20个额度

![image-20220426113211531](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204261132101.png)



![image-20220426113304547](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204261133715.png)

购买完成后，进入ssl证书管理台，选择免费证书

![image-20220426113400984](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204261134061.png)

点击创建证书，会在列表里新建一个待申请证书，点击右边证书申请，填写你的域名以及信息，CSR选择系统生成

![image-20220426113751817](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204261137893.png)

注意你的域名需要添加到DNS解析当中

创建好证书后，将申请好的证书下载下来，使用nginx代理就选择nginx证书

![image-20220427223638482](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427223638482.png)

下载下来后是两个文件

![image-20220427223736666](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220427223736666.png)

进入服务器，打开nginx的配置，添加以下内容：

```nginx
server {
        listen       443 ssl;
        server_name  jayce.icu;
        # ssl证书地址
        ssl_certificate     /usr/local/nginx/cert/jayce.pem;  # pem文件的路径
        ssl_certificate_key  /usr/local/nginx/cert/jayce.key; # key文件的路径

        # ssl验证相关配置
        ssl_session_timeout  5m;    #缓存有效期
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #加密算法
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;    #安全链接可选的加密协议
        ssl_prefer_server_ciphers on;   #使用服务器端的首选算法
        location / {
            root /var/lib/jenkins/workspace/jayce_blog;
            index index.html;
        }

}
```

其中将刚才下载的两个文件放入服务器，将配置文件中两个文件的路径改为实际路径，注意pem和key文件的位置

server_name 填写你申请证书的地址

ssl相关配置可以不用管，复制即可

重载nginx，过几分钟后验证即可

```sh
nginx -s reload
```

强制http转https:

```sh
server {
    listen 80;
    server_name  jayce.icu  www.jayce.icu;
    return 301 https://jayce.icu$request_uri;
}
```



## 个性化改装

我使用的是[matert](https://github.com/blinkfox/hexo-theme-matery)主题，主要的配置方法都在他的主页中有讲解，但如果需要更换颜色，字体，以及更加复杂的需求时，就需要对代码进行修改，充分利用F12以及IDE全局搜索，快速定位到css，js文件，进行修改。

举个例子：

我想修改原来模板的导航栏渐变颜色

![image-20220501163715336](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501163715336.png)

打开F12，看哪个文件定义了标题栏的颜色

<img src="https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501163956503.png" alt="image-20220501163956503" style="zoom: 200%;" />

从开发者模式可得，`matery.css`下面的代码是修改背景渐变颜色的

![image-20220501164233110](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501164233110.png)

找到该文件，并自行修改

![image-20220501164407058](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501164407058.png)



再看看页面，颜色就变了

![image-20220501164451931](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501164451931.png)





## 代码部署同步

使用jenkins实现服务器与github代码同步，本地部署到github后，服务器会自动拉取最新的代码

[在服务器部署Jenkins同步github代码 | Jayce's Blog](https://jayce.icu/post/jenkins-github.html)



## git 分支

有多个版本的需求，可在IDE中创建新的分支，一个是github.io的分支，一个是该项目的分支，并且在config中将master分支改为你创建的分支名

![image-20220501165147849](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501165147849.png)

![image-20220501165200028](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501165200028.png)

![image-20220501165230041](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220501165230041.png)

