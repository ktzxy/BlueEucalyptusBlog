+++
date = '2025-09-01T19:08:44+08:00'
draft = true
title = 'halo快速部署个人博客'
slug = "bbezvn095x"
description = "halo快速部署个人博客"
categories = ["📒博客部署"]
tags = ["halo"]
summary = "halo快速部署个人博客"
+++

# halo快速部署个人博客

## 技术方案

docker+docker-compose+nginx+postgressql

## halo简介

Halo是一款现代化的开源博客/CMS系统，所有代码开源在GitHub上且处于积极维护状态。它是基于 Java Spring Boot 构建的，易于部署，支持REST API、模板系统、附件系统和评论系统等功能。

Halo 作为一款好用又强大的开源建站工具，配合上不同的模板与插件，可以很好地帮助你构建你心中的理想站点。它可以是你公司的官方网站，可以是你的个人博客，也可以是团队共享的知识库，甚至可以是一个论坛、一个商城。

## 相关地址

### 开源仓库

[halo-dev/halo: 强大易用的开源建站工具。 (github.com)](https://github.com/halo-dev/halo)

[halo: 强大易用的开源建站工具。 (gitee.com)](https://gitee.com/halo-dev/halo)

### 中文文档

[Halo 文档](https://docs.halo.run/)

### 社区

[Halo 社区](https://bbs.halo.run/)

### 官网

[Halo 建站 - 强大易用的开源建站工具](https://www.halo.run/)

### 主题仓库

[应用市场 - Halo 建站 - 强大易用的开源建站工具](https://www.halo.run/store/apps)

## 基础设置

关闭防火墙（docker启动之前关闭）

```
systemctl status firewalld
systemctl stop firewalld
```

## docker安装

https://ktzxy.top/posts/tech/docker/ivzblwvqy0/

## 目录结构规划

```shell
mkdir -p ~/halo/{nginx/conf.d,nginx/ssl,nginx/log}
cd ~/halo

halo/
 ├── docker-compose.yml
 └── nginx/
     ├── nginx.conf
     ├── conf.d/
     ├── ssl/
     └── log/
```

# 申请 SSL 证书

安装：

```
sudo apt install -y certbot
```

申请：

```
sudo certbot certonly --standalone \
-d your-domain.com -d www.your-domain.com
```

证书路径：

```
/etc/letsencrypt/live/your-domain.com/
```

复制到 nginx 目录：

```
cp -r /etc/letsencrypt/live/your-domain.com ./nginx/ssl/
```

自动续期测试：

```
sudo certbot renew --dry-run
```

# Nginx 主配置

## nginx/nginx.conf

```shell
worker_processes auto;

events {
    worker_connections 1024;
}

http {

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    keepalive_timeout 30;

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log warn;

    include /etc/nginx/conf.d/*.conf;
}
```

# 站点反向代理配置

## nginx/conf.d/halo.conf

```shell
# 上游服务
upstream halo_backend {
    server halo:8090;
    keepalive 32;
}

# HTTP 自动跳转 HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name your-domain.com www.your-domain.com;

    location /.well-known/acme-challenge/ {
        root /usr/share/nginx/html;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS 生产配置
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate     /etc/nginx/ssl/your-domain/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/your-domain/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    ssl_prefer_server_ciphers on;
    ssl_ciphers HIGH:!aNULL:!MD5;

    add_header X-Frame-Options SAMEORIGIN always;
    add_header X-Content-Type-Options nosniff always;

    location / {
        proxy_pass http://halo_backend;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Connection "";
    }
}
```

# Docker Compose 

## docker-compose.yml

```shell
version: "3.8"

services:

  nginx:
    image: nginx:stable
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/ssl:/etc/nginx/ssl
      - ./nginx/log:/var/log/nginx
    networks:
      - halo_network

  halo:
    image: registry.fit2cloud.com/halo/halo:2.22
    container_name: halo
    restart: unless-stopped
    depends_on:
      - halodb
    networks:
      - halo_network
    volumes:
      - ./halo2:/root/.halo2
    expose:
      - "8090"
    environment:
      JVM_OPTS: "-Xms256m -Xmx512m -XX:+UseG1GC"
    command:
      - --spring.r2dbc.url=r2dbc:pool:postgresql://halodb:5432/halo
      - --spring.r2dbc.username=halo
      - --spring.r2dbc.password=StrongPasswordHere
      - --spring.sql.init.platform=postgresql
      - --halo.external-url=https://your-domain.com

  halodb:
    image: postgres:15
    container_name: halodb
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: halo
      POSTGRES_PASSWORD: StrongPasswordHere
      POSTGRES_DB: halo
    networks:
      - halo_network

networks:
  halo_network:
    driver: bridge
```

# 启动服务

```
docker compose up -d
docker ps
```

## 访问测试

http://【服务器或虚拟机ip】

初次访问有点慢

账号密码为compose.yml中填写的超级管理员的账号密码

### 备份数据库

```
docker exec -t halodb pg_dump -U halo halo > backup.sql
```

## 基础操作

[参考用户指南 | Halo 文档](https://docs.halo.run/category/用户指南/)

### 主题

2.10版本加入了应用市场，可在其中下载安装喜欢的主题

# 优化

```shell
建议加 1G swap：

fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```



# 番外：有界面的反向代理神器 Nginx Proxy Manager（自行测试）

## 创建一个docker-compose.yml 文件

```
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '8080:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

## 一键启动

```
docker-compose up -d
```

## 登录到管理界面

当docker容器跑起来后， 连接到81端口即可访问管理界面。有时候需要稍等一会儿。

访问：http://你的服务器IP:80

第一次登录的默认管理员账户：

```
Email:    admin@example.com
Password: changeme
```

第一次登陆后，你会立刻被要求修改登录密码和一些重要信息。

# ssl证书

cer和key

https://freessl.cn/

[ACME v2证书自动化快速入门 (freessl.cn)](https://blog.freessl.cn/acme-quick-start/)

参考文章

[Let’s Encrypt泛域名证书生成 acme.sh免费申请使用_letsencrypt泛域名-CSDN博客](https://blog.csdn.net/csdn_life18/article/details/128743730)

https://guozh.net/acme-sh-gen-lets-encrypt-domain-ssl-tutorial/

# 番外：错误集锦

下面两处错误主要是nginx和nginx proxy端口冲突

Error response from daemon: driver failed programming external connectivity on endpo                                 int nginx-proxy-manager-app-1 (b39ad26f01f7599bcee2fc11c042ea357c4b33369ccdd9effcdb7                                 97a92ce94e4): Bind for 0.0.0.0:80 failed: port is already allocated

```
查看docker 代理占用的端口
ps -aux | grep -v grep | grep docker-proxy
关闭docker 服务
sudo service docker stop
### 重启docker服务

sudo service docker start
还不行的话就
docker rm $(docker ps -aq)
sudo rm /var/lib/docker/network/files/local-kv.db
```

Error response from daemon: failed to update store for object type *libnetwork.endpointCnt: Key not found in store

```
# pkill docker
# iptables -t nat -F
# ifconfig docker0 down
# brctl delbr docker0
# service docker restart
```

