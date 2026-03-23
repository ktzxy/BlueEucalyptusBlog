+++
date = '2025-08-02T21:48:20+08:00'
draft = false
title = 'Prometheus和Grafana介绍'
slug = "07rt7r21om"
description = "Prometheus和Grafana介绍"
categories = ["📒编程"]
tags = ["Go"]
summary = "Prometheus和Grafana介绍"
comments = true  
series = ["Go语言系列"]
showSeries= true
+++
# Prometheus和Grafana介绍

## 系统监控

[gopsutil](https://www.liwenzhou.com/posts/Go/go_gopsutil/)：做系统监控信息的采集，写入influxDb，使用grafana作展示

prometheus：采集性能指标数据，使用grafana作展示

## Prometheus

[普罗米修斯](https://github.com/prometheus/prometheus)：专用于服务监控，主动去拉取数据

![image-20200913185216706](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508131_c0e571.webp)

- Jobs/Exporters：任务，监控项
- Serveice Discovery：服务发现
- Short-lived jobs：短期存活的任务

### 下载普罗米修斯

去Github的[官网](https://github.com/prometheus/prometheus/releases/tag/v2.21.0)下载

![image-20200913190154123](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508132_7103cc.webp)

下载完成后解压即可

![image-20200913190213569](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508133_e22632.webp)

双击 prometheus.exe启动，然后访问下面的地址

```bash
http://localhost:9090/graph
```

进入到prometheus的图形化页面

![image-20200913190307310](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508134_1c8828.webp)

### 使用

![image-20200913190408222](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508135_cc3791.webp)

就能够看到我们的图形化信息了

![image-20200913190444467](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508136_1f5ff3.webp)

通过上图我们发现，使用 prometheus的图形化界面好像不太美观，所以就引出了下面的  [grafana](https://grafana.com/)

## grafana

grafana是采用go语言编写的，非常美观的图形化展示，我们找到[官网下载](https://grafana.com/grafana/download?platform=windows)，选择window环境

![image-20200913190726155](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508137_6ca26d.webp)

解压后的目录，如下所示

![image-20200913190824928](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508138_0c7efe.webp)

我们进入bin目录，找到 grafana-server.exe 然后启动 【首次启动比较慢，需要建立数据库】，启动成功后，访问下面的地址

```bash
http://127.0.0.1:3000
```

即可进入到grafana的图形化页面了

![image-20200913191017753](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508139_908f81.webp)

然后输入admin  admin 登录即可

![image-20200913191249800](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508140_1f60e6.webp)

然后选择普罗米修斯

![image-20200913191312189](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508141_1e6b3a.webp)

然后输入url保存

![image-20200913191338347](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508142_e87638.webp)

然后导入我们的普罗米修斯的仪表盘

![image-20200913191444113](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508143_a383f1.webp)

然后到Home目录下，选择 new board

![image-20200913191803694](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508144_0b70dc.webp)

选择刚刚的import 的 仪表信息

![image-20200913191822529](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508145_c1feaf.webp)

这样就生成了我们的仪表信息了

![image-20200913191854680](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508146_d2fca9.webp)

或者可以选择另外一个样式

![image-20200913192017515](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508147_8db87b.webp)

## 结语

如果我们要监控其它的一些服务，比如redis、mysql、Memcache等等，需要自己到官网下载对应的包

https://prometheus.io/download/

![image-20200913192232698](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291508148_1fdca3.webp)