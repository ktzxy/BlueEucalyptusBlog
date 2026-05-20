+++
date = '2026-05-20T09:30:09+08:00'
draft = false
title = 'Podman_Certimate部署'
slug = "19fu0yb7wd"
description = "Podman_Certimate部署"
summary = "Podman_Certimate部署"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# 使用 Podman 无根用户部署 Certimate

## 1. 目录准备

```bash
mkdir -p ~/podman/podman-certimate/pb_data
```

## 2. 容器运行流程

### 2.1 启用 linger

```bash
# 启用 linger（允许非 root 用户后台运行服务）
loginctl enable-linger
```

### 2.2 停止并移除旧容器（若存在）

```bash
systemctl --user stop container-certimate.service &>/dev/null
podman stop certimate &>/dev/null && podman rm certimate &>/dev/null
```

### 2.3 启动 Certimate 容器

```bash
podman rm -f certimate

podman run -d --name certimate \
  -p 8090:8090 \
  --tz Asia/Shanghai \
  -v ~/podman/podman-certimate/pb_data:/app/pb_data:Z \
  certimate/certimate
```

### 2.4 初始化 Certimate

```bash
# 创建新账号
podman exec -it certimate ./certimate superuser create greg@local.com MySecretPass

# 删除默认的 admin 账号
podman exec -it certimate ./certimate superuser delete admin@certimate.fun
```

## 3. 配置 Systemd 自启动服务

### 3.1 生成 Systemd 服务文件

```bash
mkdir -p ~/.config/systemd/user && cd ~/.config/systemd/user
podman generate systemd --files --new --name certimate
```

### 3.2 停止容器

```bash
# 停止容器（释放占用，交由 systemd 管理）
podman stop certimate && podman rm certimate
```

### 3.3 启动并设置开机自启

```bash
systemctl --user daemon-reload
systemctl --user enable container-certimate.service --now
```

### 3.4 放行防火墙

```bash
firewall-cmd --add-port=8090/tcp
firewall-cmd --add-port=8090/tcp --per
```

