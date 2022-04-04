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



## 架构简介

Prometheus 服务端负责数据的收集，然后将数据推送至grafana进行前台展示，而如何用Prometheus 采集到服务器和Nginx的数据呢，这里就需要再引入两个工具，服务器数据采用NodeExporter进行采集，Nginx数据通过nginx-module-vts插件进行采集。

为了服务器环境的整洁，此次监控都将采用docker容器，docker即简单又不影响服务器环境，越来越受欢迎。

需要在docker上安装的有
prometheus    拉取并存储数据 ，node-exporter 收集内核公开的硬件和操作系统指标 grafana  将取出的数据展示可视化  
具体流程为 node-exporter对服务器进行取数，nginx-module-vts对nginx进行取数，prometheus进行数据统一管理，由prometheus 推送至grafana进行数据展示
效果图：

### 监控服务器

![image-20220404145449706](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220404145449706.png)

监控nginx

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

![image-20220404145617171](/Users/jayce/github/hexo_blog/source/_posts/Prometheus_grafana.assets/image-20220404145617171-9055485.png)

这里我用域名代替ip，以实际网络环境为准



查看docker运行的服务：

```
docker ps -a
```

![image-20220404144850405](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220404144850405.png)

## 安装node-exporter

```sh
docker pull prom/node-exporter
```



 运行

```shell
docker run -d --name node-exporter -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" --net="host" prom/node-exporter
```

修改prometheus配置文件，让node-exporter数据连接prometheus，

```
vim prometheus/prometheus.yml
```

```properties
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

```
docker restart prometheus
```

打开浏览器：

![image-20220404145831640](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220404145831640.png)



## 安装grafana

拉取镜像

```shell
docker pull grafana/grafana
```

```shell
 mkdir grafana
```

创建grafana目录

启动镜像

```shell
docker run -d --name=grafana -p 3000:3000 -v $PWD/grafana:/var/lib/grafana grafana/grafana
```

打开浏览器

![image-20220404150145690](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/image-20220404150145690.png)



监控nginx

监控nginx需要使用nginx-module-vts插件，而nginx-module-vts插件的添加又需要重新编译nginx，此方法适用于编译安装而非yum安装

安装nginx-module-vts

```
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

```
location /status {
        vhost_traffic_status_display;
        vhost_traffic_status_display_format html;
}
```

![image-20220402135813113](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021358160.png)

此时打开公网/内网地址，`/status`下即可显示nginx数据了

![image-20220402141230288](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021412387.png)

`/status/format/prometheus` :

![image-20220402141427152](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021414243.png)

用nginx将上面的地址转换为prometheus的地址：

```
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

```
docker restart prometheus
```

此时访问IP:9013/metrics ，就可以变成prometheus需要的类型了

![image-20220402141739403](https://jaycehe.oss-cn-hangzhou.aliyuncs.com/markdown/202204021417501.png)









