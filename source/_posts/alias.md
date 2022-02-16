---
title: aalias的详解与运维
date: 2022-02-14 00:35:16
tags:
  - 运维
  - 服务器
  - linux
---
# alias的详解

alias 可以为命令指定别名，所谓别名可以省去一长串命令的麻烦

## 查看别名

直接使用命令`alias`可以查看所有的别名，如果想看某一命令，在alias后跟命令即可

![image-20220214152355809](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202141523138.png)

## 创建别名

例如：

```shell
alias newcmd='cmd'
alias install='sudo apt-get	install'
```

上述方式一旦关闭终端，设置的别名就失效了

可以将命令放入 ~/.bashrc 中，当每个进程生成时，都要执行`~/.bashrc`中的命令

## 删除别名

`unalias` 命令可以将之前的别名删除

```sh
unalias vi
```

使用`-a`参数可以清除所有的别名



## 不使用别名

当我们为命令加上日常参数后，但有时需要使用原始的命令时，有三种方法可以调用原始命令

1. 使用命令的绝对路径
2. 切换命令所在目录，使用`./cmd`
3. 在命令前使用反斜线`\`

别名



