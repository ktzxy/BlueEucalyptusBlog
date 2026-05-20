+++
date = '2026-05-20T09:27:53+08:00'
draft = false
title = 'Podman_Nginx部署'
slug = "u9zq8yndu1"
description = "Podman_Nginx部署"
summary = "Podman_Nginx部署"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# Podman 启动 Nginx 容器服务

## 1. 前置准备

### 1.1 允许非 root 用户监听 80 端口

```bash
grep "^net.ipv4.ip_unprivileged_port_start" /etc/sysctl.conf || echo "net.ipv4.ip_unprivileged_port_start = 80" >> /etc/sysctl.conf
sysctl -p
```

### 1.2 开启 SELinux 允许网络访问

```bash
sudo setsebool -P httpd_can_network_connect 1
```

## 2. 目录结构与软连接

### 2.1 创建容器使用的挂载目录

```bash
mkdir -p /podman/podman-nginx/conf/conf.d
mkdir -p /podman/podman-nginx/{logs,html}
mkdir -p /qnap/{iso,cdrom,mirror,soft}
```

### 2.2 建立静态资源目录的软连接

```bash
ls /podman/podman-nginx/html/iso    &>/dev/null || ln -s /qnap/iso /podman/podman-nginx/html/
ls /podman/podman-nginx/html/cdrom  &>/dev/null || ln -s /qnap/cdrom /podman/podman-nginx/html/
ls /podman/podman-nginx/html/mirror &>/dev/null || ln -s /qnap/mirror /podman/podman-nginx/html/
ls /podman/podman-nginx/html/soft   &>/dev/null || ln -s /qnap/soft /podman/podman-nginx/html/
```

## 3. Nginx 配置文件

```bash
cat > /podman/podman-nginx/conf/conf.d/web.conf << 'END'
server {
    listen       80;
    server_name  localhost;
    charset utf-8;
    autoindex_exact_size off;
    autoindex_localtime on;
    location / {}

    location /web {
        alias html;
        autoindex on;
    }
    location /video {
        alias /video;
        autoindex on;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {}
}
END
```

## 4. 启动 Nginx 容器

### 4.1 停止旧服务

```bash
loginctl enable-linger
systemctl --user stop container-nginx.service
podman stop nginx &>/dev/null && podman rm nginx &>/dev/null
```

### 4.2 启动新容器

```bash
podman run --privileged -dt \
    --name nginx \
    -p 80:80 \
    --tz Asia/Shanghai \
    -v /podman/podman-nginx/conf/conf.d:/etc/nginx/conf.d:Z \
    -v /podman/podman-nginx/logs:/var/log/nginx:Z \
    -v /podman/podman-nginx/html:/etc/nginx/html:Z \
    -v /qnap/iso:/etc/nginx/html/iso \
    -v /qnap/mirror:/etc/nginx/html/mirror \
    -v /qnap/software:/etc/nginx/html/software \
    -v /cdrom:/etc/nginx/html/cdrom:Z \
    -v /video:/video:Z \
    nginx:1.22.0
```

## 5. 注册 systemd 服务

### 5.1 生成并部署 service 文件

```bash
mkdir -p ~/.config/systemd/user
cd ~/.config/systemd/user

podman generate systemd --files --new --name nginx

# 停止容器以避免冲突
podman stop nginx && podman rm nginx
```

### 5.2 启动并设置自启

```bash
systemctl --user daemon-reload
systemctl --user enable container-nginx.service --now
```
