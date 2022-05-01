---
title: 使用typora+阿里云oss打造博客写作平台
date: 2022-02-02 21:29:17
author: Jayce he
top: true
categories: 部署
summary: 在github，markdown，web等文件中使用oss取代本地图片文件
img: https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204080945215.png
tags:
  - Markdown
  - OSS
  - 阿里云
  - typora
  - 博客
---
# 使用typora+阿里云oss 打造博客写作平台

## 背景简介

我使用typora+markdown作为日常笔记的重要工具，但markdown最大的痛点是图片需要单独存放，并不能像word那样保存为一个文件，而遇到分享文件，或者整片复制的情况时，就会出现不能正常显示图片的问题。

## 方案介绍

利用阿里云oss，将涉及到的图片存放在云端，图片的url将是公网地址，不再局限于本机。使用onedrive可以对md文件进行同步，onedrive支持windows，ios，macos，Android同步，同步速度中规中矩。

## 阿里云oss

对象存储oss是阿里云提供的一种云端存储方案，它与网盘最大的区别是，它可以使用url进行查看，而且操作方便，价格实惠

阿里云oss价格参考

![image-20210819143824267](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202108191438597.png)

选择最便宜40G，存放图片完全够用。

类似的存储在不同平台也有很多，只是名字不一样而已，核心功能都一样

例如腾讯云cos

## 操作流程

### oss准备

1. 到[阿里云oss产品官网](https://www.aliyun.com/product/oss/)购买oss服务，根据自身需求购买相应的内存及性能

2. 创建Bucket，Bucket相当于仓库，是你将来要存放数据的地方

   ![image-20210819144518089](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202108191445966.png)



创建Bucket 

![image-20210819144620800](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202022133051.png)





我使用的图床软件是PicGo， 下载安装这里不多赘述，现在需要将oss的信息添加到PicGo 中，废话不多，直接上配置，打开配置文件，在阿里云的配置区域里，填写自己所买的accessKeyId和accessKeySecret，以及创建的bucket。

![image-20220202210957291](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202022133067.png)

记得在设置下面勾选阿里云oss

![image-20220202211152739](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202022111401.png)

```
  "aliyun": {
      "accessKeyId": "*******",
      "accessKeySecret": "********",
      "area": "oss-cn-hangzhou",
      "bucket": "jaycehe",
      "customUrl": "",
      "options": "",
      "path": "markdown/"
    },
```



其次是`typora`软件，虽然现在收费了，但支持多平台并且真的好用的编辑软件也找不出来了

想要配置图床，先在偏好设置-图像

![image-20220202210411391](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202022104031.png)

把PicGo的路径添加进去，点击验证图片上传选项验证oss是否能正常上传，一般上传正常就会返回成功值

![image-20220202211414361](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202022114037.png)

但一般会大大小小出现一些问题，随后会更新问题概述及解决方法，如果出现上传失败，请先检查oss配置，多半问题都出在PicGo的配置和OSS配置上，结合日志和阿里云给出的报错提示 [OSS权限相关常见错误的排查方法
](https://help.aliyun.com/document_detail/42777.html) 自己排查不出来也可以联系我。


## 使用方法

如果想用oss代替本地图片，首先正常添加图片至markdown文件后，右键即可弹出上传图片选项，点击即可替换图片的本地路径为oss在线链接

![image-20220202212256962](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202022122264.png)
