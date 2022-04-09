---
title: 在服务器部署jupyter notebook，实现服务器环境运算
date: 2022-02-16 00:38:48
categories: 部署
img: https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204080955588.png
tags:
 - python
 - 服务器
 - jupyter notebook
---

# 在服务器部署jupyter notebook，实现服务器环境运算
>大学时期为了借用服务器环境，已踩坑无数，今天突然想起来，部署着玩玩，python初学者也可使用jupyter notebook，ipython对python初学者很友好，也很适合做深度学习，数据分析等

能实现随时随地在浏览器上进行Python处理数据等程序

效果：

![image-20220216013107482](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160131561.png)

## 简单说下jupyter notebook

Jupyter notebook是一种 Web 应用，能让用户将说明文本、数学方程、代码和可视化内容全部组合到一个易于共享的文档中。与常规的py文件不同的是，Jupyter notebook主要运行的是ipython，ipython是一个python的交互式shell，非常适合进行科学计算和交互可视化。

看下图就能明白区别

iPython(Jupyter notebook)

![image-20220216010041321](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160100134.png)

Python：

![image-20220216012827342](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160128853.png)



## 开始部署

直接使用` pip install jupyter` 即可安装

但这里我们还是用anaconda，anaconda虽然很臃肿，但包含了许多的库，可以节省许多精力，**即使不安装anaconda也是可以安装**

### anaconda安装

在[Anaconda官网](https://www.anaconda.com/)寻找适合linux的安装脚本，这里直接给出地址https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh

由于软件更新，地址可能出现失效，实际已官网提供为准。

![image-20220216014226572](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160142772.png)

下载好脚本后，用scp`或`ftp`上传至服务器`，当然也可直接在服务器`wget`![image-20220216014144845](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160141546.png)

运行安装脚本:

```sh
bash Anaconda3-2021.11-Linux-x86_64.sh
```

这里的文件名以自己下载的为准

运行脚本后，为了方便基本可以全程回车和yes，但建议还是看每项是否都需要yes，一切根据自身情况判断

安装好后，需要配置环境变量，修改`/etc/profile`或者`/etc/bashrc`的配置信息来设置环境变量

在`/etc/profile` 中添加：

```sh
export PATH=$PATH:/root/anaconda3/bin
```

这里的路径为你安装anaconda的目录

保存退出，执行`source /etc/profile`

输入python3 ，看看环境：

![image-20220216014727419](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160147985.png)

如果是上图这样，则不成功，系统调用了原装的python3.6

执行python3.9，运行anaconda环境：

![image-20220216014906051](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160149595.png)

多版本python的情况可以利用别名进行区分：

```sh
alias python3=python3.9
```

![image-20220216015210129](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160152179.png)

 安装好anaconda和python环境后，需要对jupyter进行配置

进入python，执行以下命令

```python
from notebook.auth import passwd 
passwd()
```

![image-20220216094836772](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160948965.png)

此时提示需要输入密码，设置一个进入jupyter notebook的密码，系统会输出一个sha1加密的字符串，这个要保存下来，后面要用

![image-20220216095052523](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202160950884.png)



在终端执行jupyter 配置文件生成命令，如果是root用户执行的，加上`加上`--allow-config``

```sh
 jupyter notebook --generate-config
 jupyter notebook --generate-config --allow-config
```

如果提示找不到命令，请进入`anaconda3/etc/jupyter/` 目录下执行

此时会生成一个配置文件jupyter_notebook_config.py ，接着用编辑器打开（命令执行后有目录位置的提示）

```sh
vim ~/.jupyter/jupyter_notebook_config.py
```

在配置文件中添加以下参数：

```python
# 允许访问此服务器的 IP，星号表示任意 IP
c.NotebookApp.ip = "*"
# 运行时不打开本机浏览器
c.NotebookAPp.open_browser = False
# 之前生成的密码 hash 字串
c.NotebookApp.password = 'sha1:781d82cc6080:e047e581703e4f0c6cd6ec4d037b244315aebe9c'
# 使用的端口，记得在安全组中开放
c.NotebookApp.port= 8000
#指定后续程序的目录，创建好再启动
c.NotebookApp.notebook_dir = "/root/jupyter_project"
# 禁止使用终端
c.NotebookApp.terminals_enabled = True
```

其余注释的配置，感兴趣的可以自行查询效果，根据自己意愿进行配置

保存并退出

启动jupyter notebook

```sh
jupyter notebook --allow-root
```

其中root用户才需要`--allow-root`

此时系统会出现运行日志，如果报错可根据日志查询报错原因

如果成功运行，此时，用ip或域名加上刚才配置文件指定的ip，看看能不能访问到

如果成功进入，输入刚才你设置的密码即可

需要进行反向代理的，也可以配置nginx，这里不深入了

测试jupyter notebook

右上角new一个python3 ，此时会进入jupyter notebook编辑页面，尝试写一些代码，看看python和各类库是否正常

![image-20220216100627203](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202161006258.png)

![image-20220216102155441](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202202161021586.png)
