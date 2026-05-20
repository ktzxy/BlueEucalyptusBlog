+++
date = '2026-05-20T10:36:25+08:00'
draft = false
title = 'Grafana部署'
slug = "z20m1x80ps"
description = "Grafana部署"
summary = "Grafana部署"
categories = ["📒运维"]
tags = ["Grafana"]
comments = true
+++

# Grafana 可视化展示平台

## 1. 部署 Grafana

```bash
# 创建工作目录
mkdir -p /podman/podman-grafana/data
chown 472 /podman/podman-grafana/data

# 拉起 Grafana
podman run -dt \
--name=grafana \
-p 3000:3000 \
--tz Asia/Shanghai \
-v /podman/podman-grafana/data:/var/lib/grafana:Z \
grafana/grafana:11.1.4

# 注册系统服务
loginctl enable-linger
mkdir -p ~/.config/systemd/user && cd ~/.config/systemd/user
podman generate systemd --files --new --name grafana
podman stop grafana && podman rm grafana

# 启动并设置开机启动
systemctl --user daemon-reload
systemctl --user enable container-grafana.service --now

# 防火墙放行
firewall-cmd --add-port=3000/tcp
firewall-cmd --add-port=3000/tcp --per

# 浏览器访问
http://127.0.0.1:3000/login
username: admin
password: admin
```
