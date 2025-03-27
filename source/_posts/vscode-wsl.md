---
title: windows环境使用vscode在linux环境编写shell和python脚本
date: 2022-11-28 15:07:34
author: Jayce he
categories: 工具
top: true
summary: 使用vscode在linux环境编写shell和python脚本
img: https://oss.jayce.icu/markdown/vscode-svgrepo-com.svg
tags:
- WSL
- vscode
- linux
- shell
- python
- 脚本
---

# 使用vscode在linux环境编写shell和python脚本
对于经常编写脚本的人来说，编写shell一般都会在vim或记事本中，这样的方式没什么问题，但遇到需要测试的时候，就没有IDE那样的方便，而且使用IDE对shell的自动补全会让效率翻倍。
此文章将讲述如何在windows vscode中配置Linux运行环境，让你run code即可看到输出。

## 安装vscode
安装vscode的方式在这就不做大篇的叙述了，百度vscode，下载安装包，点击下一步安装即可
[下载地址点我](https://code.visualstudio.com/)    




## 安装WSL
简单讲下WSL，这里我们将使用WSL作为脚本的运行环境
WSL是一个在Windows 10\11上能够运行原生Linux二进制可执行文件的兼容层。注意它并不是虚拟机。
步骤
右键单击windows徽标，打开Powershell（管理员），执行

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
即可将WSL功能激活。

接着在微软商店中挑一个你想用的Linux发行版

![image-20221128152641580](https://oss.jayce.icu/markdown/image-20221128152641580.png)

这里我选择了Ubuntu 20，直接点击安装即可。安装成功后，在开始菜单中点击运行，看看是否正常，一般第一次都需要设置用户和密码

![image-20221128152909237](https://oss.jayce.icu/markdown/image-20221128152909237.png)

这就是wsl Ubuntu的界面，当你正常设置好用户名和密码，出现这个界面后，说明你的WSL安装正常了![image-20221128152945459](https://oss.jayce.icu/markdown/image-20221128152945459.png)



## 安装插件

想必大家肯定不是第一次使用vscode的人了，就直接开门见山，直说需要安装什么插件好了

- Code Runner
- WSL
- Remote - SSH
- ShellCheck
- Shellman
- Shell-format

更多好用的脚本可自行安装

这里核心的插件是WSL，这里的wsl只是一个插件，和刚才安装的不一样，这个是用来连接WSL的

此时，在新建终端时，就可以看到刚才安装的WSL，这里我把它设置成了默认

![image-20221128154132874](https://oss.jayce.icu/markdown/image-20221128154132874.png)

新建一个终端看看是不是WSL

![image-20221128154228391](https://oss.jayce.icu/markdown/image-20221128154228391.png)

此时就可以在这里的终端直接输入各类命令使用了，目录也是windows下的目录，比如上图中当前的目录为`/mnt/e`，就代表现在在windows 中的E盘，这里需要注意的是，访问windows的文件管理器时，都要在mnt目录下寻找，WSL将windows 下的各个盘挂载在mnt下，当然你也可以使用WSL环境的linux目录结构。

vscode中setting.json部分配置如下

![image-20221128161645440](https://oss.jayce.icu/markdown/image-20221128161645440.png)



## Code Runner插件配置

在vscode安装好linux环境后还不能优雅的调试写好的脚本，我们希望在点击run code按钮后，终端直接执行脚本，但现在还无法做到，现在执行脚本会有如下报错：

![image-20221128155757738](https://oss.jayce.icu/markdown/image-20221128155757738.png)

这是因为code runner 在执行shell时不能识别WSL，这时我们就要对code runner做一些修改。

找到code runner的安装目录，插件一般放在`C:\Users\用户名\.vscode\extensions`

code runner的文件名是formulahendry.code-runner，进入out\src 下，找到codeManager.js，在编辑器中打开。

找到

``` js
else if (windowsShell && windowsShell.toLowerCase().indexOf("bash") > -1 && windowsShell.toLowerCase().indexOf("windows") > -1) {
    command = command.replace(/([A-Za-z]):\\/g, this.replacer).replace(/\\/g, "/");}
```

这段代码需改改成

``` js
else if (windowsShell && windowsShell.toLowerCase().indexOf("wsl") > -1) {
	command = command.replace(/([A-Za-z]):/g, this.replacer).replace(/\\/g, "/");}
```

保存重启vscode即可

此时我们再运行code runner，即可正常在WSL里执行脚本了

![image-20221128161103550](https://oss.jayce.icu/markdown/image-20221128161103550.png)



## Python脚本

只需要在WSL中安装python即可

![image-20221128161529394](https://oss.jayce.icu/markdown/image-20221128161529394.png)





