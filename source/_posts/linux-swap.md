---
title: linux添加虚拟内存交换内存
date: 2023-05-15 15:54:57
author: Jayce he
categories: 问题解决
img: 
tags:
- linux
- 服务器
---

## linux添加虚拟内存交换内存

当Linux内存不够的情况下，可以适当增加swap内存来分担

新建交换分区文件夹

```
dd if=/dev/zero of=/tmp/swapfile bs=1024 count=1024k
```

![image-20230515155918499](https://oss.jayce.icu/markdown/image-20230515155918499.png)

格式化该分区为交换分区

```
mkswap /tmp/swapfile
```

开启该交换分区

```
swapon /tmp/swapfile
```

如果报错提示权限不足的话，还需添加权限	

`swapon: /tmp/swapfile: insecure permissions 0644, 0600 suggested.`

```
 chmod 600 /tmp/swapfile
```

 接着启用sawp

```
swapon /tmp/swapfile
```

如果提示报错`swapon: /tmp/swapfile: swapon failed: Device or resource busy`

先关闭在打开

```
swapoff /tmp/swapfile
swapon /tmp/swapfile
```

 此时free查看swap内存已添加

设置

`vim /etc/fstab`

文件新加

```
vm.swappiness=10      #当内存在使用到100-10=90%的时候，交换分区启动
vm.vfs_cache_pressure=50  #虚拟内存回收directory和inode缓冲的倾向，这个值越大。越易回收
```

重启生效