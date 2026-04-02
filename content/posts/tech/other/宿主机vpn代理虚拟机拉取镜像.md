+++
date = '2026-04-02T17:10:38+08:00'
draft = false
title = '宿主机vpn代理虚拟机拉取镜像'
slug = "kithu0hblk"
description = "虚拟机设置vpn代理拉取镜像"
summary = "虚拟机设置vpn代理拉取镜像"
categories = ["📒问题集锦"]
tags = ["vpn"]
comments = true
+++

# 宿主机vpn代理虚拟机拉取镜像

```shell
#桥接模式

vim /etc/systemd/system/docker.service.d/proxy.conf
[Service]
# 将 192.168.31.99 替换为你真实的宿主机 IP
Environment="HTTP_PROXY=http://宿主机ip:7897"  #宿主机ip+vpn端口
Environment="HTTPS_PROXY=http://宿主机ip:7897"
# 本地地址不走代理，防止死循环
Environment="NO_PROXY=localhost,127.0.0.1,.local,.internal,192.168.31.0/24"
sudo systemctl daemon-reload
sudo systemctl restart docker
```

宿主vpn系统代理，局域网开启
