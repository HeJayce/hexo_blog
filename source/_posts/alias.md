---
title: alias的详解与运维
date: 2022-02-14 00:35:16
categories: 文章
tags:
  - 运维
  - 服务器
  - linux
---
# alias的详解

alias 可以为命令指定别名，所谓别名可以省去一长串命令的麻烦

## 查看别名

直接使用命令`alias`可以查看所有的别名，如果想看某一命令，在alias后跟命令即可

![image-20220214152355809](https://oss.jayce.icu/markdown/202202141523138.png)

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



## 别名永久生效

我们通过 alias 命令设置的别名，仅限于在当前的 Shell 中使用，如果系统重启了，那么新设置的别名就失效了。

如果想让别名永久有效的话，就需要把所有的别名设置方案加入到（$HOME）目录下的 .alias 文件中（如果系统中没有这个文件，你可以创建一个），然后在 .bashrc 文件中增加这样一段代码：

```
# Aliases
if [ -f ~/.alias ]; then
  . ~/.alias
fi
```

.alias文件：

```shell
vim .alias
######
alias tf='tail -f'  #动态查看文件变化
```

执行`source ~/.bashrc`



## 安全删除

```sh
alias rm='saferm(){ /bin/cp -a $@ ~/backup;rm $@; };saferm $@'
```

