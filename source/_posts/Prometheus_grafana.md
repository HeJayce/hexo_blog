---
title: 使用typora+阿里云oss打造博客写作平台
date: 2022-02-02 21:29:17
author: Jayce he
categories: 部署
summary: 使用grafana可视化仪表监控服务器数据和nginx吞吐
cover: http://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202109171525690.png
tags:
  - Prometheus
  - grafana
  - Linux
  - 可视化
---

# 使用grafana可视化仪表监控服务器数据和nginx吞吐





Prometheus 服务端负责数据的收集，然后将数据推送至grafana进行前台展示，而如何用Prometheus 采集到服务器和Nginx的数据呢，这里就需要再引入两个工具，服务器数据采用NodeExporter进行采集，Nginx数据通过nginx-module-vts插件进行采集。

为了服务器环境的整洁，此次监控都将采用docker容器，docker即简单又不影响服务器环境，越来越受欢迎。

需要在docker上安装的有
prometheus    拉取并存储数据 ，node-exporter 收集内核公开的硬件和操作系统指标 grafana  将取出的数据展示可视化  
具体流程为 node-exporter对服务器进行取数，nginx-module-vts对nginx进行取数，prometheus进行数据统一管理，由prometheus 推送至grafana进行数据展示
效果图：

### 监控服务器

![image-20220404145449706](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220404145449706.png)

### 监控nginx

![image-20220404145734567](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220404145734567.png)

## 安装prometheus

直接拉取prometheus的镜像

```
docker pull prom/prometheus
```

运行prometheus

```
docker run -d --name prometheus prom/prometheus
```

然后我们将prometheus的配置文件复制到本地目录，方便修改

```
docker cp -a prometheus:/etc/prometheus/ $PWD/prometheus
```

然后停止prometheus，重新创建并使用配置文件运行

```
docker stop prometheus
docker rm  prometheus
```

```
docker run -d --name prometheus -p 9090:9090 -v $PWD/prometheus:/etc/prometheus prom/prometheus
```

注意9090端口在安全组和防火墙开通

然后访问`ip:9090`看看是否成功

![image-20220408105846841](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081058933.png)

这里我用域名代替ip，以实际网络环境为准



查看docker运行的服务：

```
docker ps -a
```





安装node-exporter

```
docker pull prom/node-exporter
```



 运行

```
docker run -d --name node-exporter -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" --net="host" prom/node-exporter
```

修改prometheus配置文件，让node-exporter数据连接prometheus，

```
vim prometheus/prometheus.yml
```

```
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ['172.26.134.10:9090']
        labels:
          instance: prometheus
  - job_name: 'centos_1'
    static_configs:
    - targets: ['172.26.134.10:9100']
      labels:
         instance: centos_1
```

targets：选择填prometheus的ip和端口，建议选择localhost或内网地址。job_name：起个名字

保存退出，重启prometheus

```sh
docker restart prometheus
```

打开浏览器：

![image-20220413235210346](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220413235210346.png)



安装grafana

```sh
docker pull grafana/grafana
```

创建grafana目录

```sh
mkdir grafana
```

启动镜像

```shell
docker run -d --name=grafana -p 3000:3000 -v $PWD/grafana:/var/lib/grafana grafana/grafana
```

打开浏览器





## 监控nginx配置

监控nginx需要使用nginx-module-vts插件，而nginx-module-vts插件的添加又需要重新编译nginx，此方法适用于编译安装而非yum安装

安装nginx-module-vts

```sh
git clone git://github.com/vozlt/nginx-module-vts.git
```

这里会在当前目录下将nginx-module-vts克隆下来，没有git请自行百度安装

进入nginx原始的目录，准备编译

在编译前，添加参数`--add-module=/path/to/nginx-module-vts`

```
--prefix=/usr/local/nginx --with-http_gzip_static_module --with-http_stub_status_module --with-http_ssl_module --with-pcre --with-file-aio --with-http_realip_module --add-module=/usr/local/nginx-module-vts
```

```
make
```

编译完成后，将编译好的nginx可执行文件复制到原来的nginx启动目录sbin下

执行nginx -V，看看有没有nginx-module-vts参数

![image-20220402135521016](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021355092.png)

此时打开nginx配置文件，添加监控接口

![image-20220402135729259](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021357475.png)

```nginx
location /status {
        vhost_traffic_status_display;
        vhost_traffic_status_display_format html;
}
```

![image-20220402135813113](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021358160.png)

修改prometheus配置文件，添加nginx模块

```
- job_name: 'nginx'
    static_configs:
    - targets: ['172.26.134.10:9013']
      labels:
         instance: nginx
```

保存并重启

此时打开公网/内网地址，`/status`下即可显示nginx数据了

![image-20220402141230288](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021412387.png)

`/status/format/prometheus` :

![image-20220402141427152](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021414243.png)

用nginx将上面的地址转换为prometheus的地址：

```nginx
server {
        listen       9013 ;
        server_name  172.26.134.10;
location /status {
        vhost_traffic_status_display;
        vhost_traffic_status_display_format html;
        }
location /metrics{
        proxy_pass  http://172.26.134.10:9013/status/format/prometheus;
}
}
```

prometheus配置文件添加，重启prometheus

```
 - job_name: 'nginx'
    static_configs:
    - targets: ['172.26.134.10:9013']
      labels:
         instance: nginx
```

```sh
docker restart prometheus
```

此时访问IP:9013/metrics ，就可以变成prometheus需要的类型了

![image-20220402141739403](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021417501.png)

## 配置grafana

有了数据源，接下来就是配置可视化仪表了

打开grafana登录页面，默认用户名密码都是admin

![image-20220408103615852](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081036547.png)



登录后会自动提示修改密码，也可以跳过

![image-20220408103651978](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081036130.png)

> 忘记密码操作：
>
> `find / -name "grafana.db"`
>
> `sqlite3  grafana.db`
>
> `.tables` **
>
>  `select * from user;` **
>
> `update user set password = '59acf18b94d7eb0694c61e60ce44c110c7a683ac6a8f09580d626f90f4a242000746579358d77dd9e570e83fa24faa88a8a6', salt = 'F3FAxVm33R' where login = 'admin';`
>
> `.exit`
>
> 重启grafana
>
> 密码和用户名都重置为admin

修改即可

#### 配置数据源

进入主界面找到下图所示data sources

![image-20220408103844156](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081038259.png)

点击右边蓝色 add data source

![image-20220408103949639](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081039713.png)

选择第一个Prometheus源

![image-20220408104007625](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081040727.png)

填入`ip:9090`，其他可以不用管，点击最后的保存并测试

![image-20220408104436874](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081044265.png)

正确即出现下图标识：

![image-20220408104713340](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081047438.png)

#### 配置可视化

导入一个模板，模板可以在此网站挑选[Dashboards | Grafana Labs](https://grafana.com/grafana/dashboards/)

![image-20220408104801670](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081048748.png)

选择好后，将模板代号填入即可

![image-20220408105512736](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081055144.png)

![image-20220408105531459](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081055551.png)

导入成功后，配置好的模板会自动识别数据源，将数据进行展示，如果个别模块没有数据，可以手动配置数据

![image-20220408105653310](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081056466.png)

可在edit模块，选择数据源以及需要展示的数据

![image-20220408105740337](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204081057411.png)



nginx同理，但组件需要手动配置数据
