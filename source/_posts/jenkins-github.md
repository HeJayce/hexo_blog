---
title: 在服务器部署Jenkins同步github代码
date: 2022-02-05 04:10:46
author: Jayce he
top: true
categories: 服务器
summary: 在服务器部署Jenkins同步github代码
# cover: http://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202109171525690.png
tags:
  - Jenkins
  - github
  - 阿里云
  - hexo 
---

# 在服务器部署Jenkins同步github代码

## 背景

使用hexo在github托管个人网站后，如果想要在自己的服务器上部署并且代码与github一致，可以使用Jenkins来同步

Jenkins可以监控github的动作，设置当代码push后，Jenkins监控到这一动作，自动拉取最新的代码

## [Jenkins安装]()



## 获取hook

在安装Jenkins时，已经默认安装了github的插件，首先需要配置全局的github，在管理Jenkins中点击Configure System，进入设置。这里的入口与其他博客可能会有不同，请根据自己安装的Jenkins版本自行判断

![image-20220205131509975](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051315671.png)

找到github的配置块

![image-20220205131557625](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051315820.png)

Jenkins在关联Github前，需要准备好Hook，这个Hook需要点击高级，找到为 Github 指定另外一个 Hook URL，勾选这个CheckBox（一时想不起来中文叫啥了)，获取到这个URL

![image-20220205132303844](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051323334.png)

![image-20220205132229972](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051323137.png)

## github添加hook

到github，选择需要同步的项目，进入设置界面，点击Webhooks

![image-20220205132622973](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051326764.png)

新建一个webhooks（右上角Add Webhooks）

![image-20220205132829053](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051328277.png)

这里的Payload URL填写刚才在Jenkins中的URL`http://ip:port/github-webhook/`

下面选择第三个

![image-20220205132951899](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051329104.png)

在下面勾选Pushes![image-20220205133026618](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051330792.png)

最后点击添加即可

![image-20220205133125556](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051331682.png)

## 生成sercret text

添加好后，先别急关掉，此时还需要生成一个sercret text，进入github的设置，进入Developer setting，选择Personal Access Token 点击 Generate new token

![image-20220205133943847](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051339773.png)

选择这两个

![image-20220205134047182](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051340362.png)

生成即可

## Jenkins配置github

返回Jenkins，这里的名称可以随便写

API URL保持默认https://api.github.com就行

这里需要添加凭据

![image-20220205131650075](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051316492.png)

点击高亮的Jenkins进入添加凭据界面

![image-20220205134249370](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051342576.png)

类型选择sercret text，把刚才生成的sercret text添加到Secret中，id可不写，描述可简单描述即可

填完后点击下面添加，此时凭据界面就会有刚才添加的sercret text了

![image-20220205134501227](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051345485.png)

最后保存

## 创建项目

回到主界面，新建一个项目，![image-20220205134552671](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051345969.png)

名称自己填，项目类型选择第一个Freestyle project![image-20220205134639318](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051346408.png)

点击确定添加

## 配置github仓库

进入配置界面，选择Github项目，填写仓库地址

![image-20220205134743946](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051347122.png)

源码管理使用Git

输入仓库的Git地址，例如`git@github.com:HeJayce/HeJayce.github.io.git`

此时需要添加git的私钥，如果服务器已经配置好，可以直接在`.ssh/`文件夹找到，复制私钥，点击添加凭据，与刚才类似，但这里类型需要选择SSH Username with private key，其他可以适当写写，但重要的是在下面把私钥复制进去

![image-20220205135210530](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051352246.png)

分支根据自己情况填写，源码库浏览器选择githubweb，URL填写仓库地址

![image-20220205135326208](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051403853.png)

其他安装下图选择即可

![image-20220205135438447](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051354450.png)

最后点击保存即可，此时返回项目界面，点击立即构建，Jenkins就会把仓库代码clone下来，当出现绿色对勾即表示正常，前面的叉都是踩的坑，搞到4点多，终于搞定了

![image-20220205135614058](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202051356625.png)

由于我的项目是静态网页，只需要在Nginx中配置下路径即可

```nginx
server {
        listen       80;
        server_name  填你的公网ip;
        location / {
            root /var/lib/jenkins/workspace/jayce_blog;
            index index.html;
        }
    }
```



