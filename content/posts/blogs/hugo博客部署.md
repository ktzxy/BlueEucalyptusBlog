+++
date = '2026-01-04T11:02:13+08:00'
draft = false
title = 'hugo博客部署'
description = "使用docker部署hugo，部署到云服务器"  
summary = "使用docker部署hugo，部署到云服务器"         
categories = ["📒博客部署"]                                      
tags = ["hugo"]                                                  
+++
# **一、准备 SSH 密钥**

GitHub Actions 要通过 SSH 上传文件到服务器，需要 **一个公私钥对**：

1. **在本地生成 SSH 密钥**（推荐单独为部署创建，不覆盖现有 SSH）：

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/myblog_deploy_key
```

- `-f ~/.ssh/myblog_deploy_key` → 保存路径和文件名
- 会生成两个文件：

```
~/.ssh/myblog_deploy_key        # 私钥
~/.ssh/myblog_deploy_key.pub    # 公钥
```

- **注意不要设置 passphrase**，GitHub Actions 无法交互输入密码

1. **把公钥上传到阿里云服务器**

假设服务器 IP：`123.45.67.89`，用户名 `root` 或普通用户：

```
ssh root@123.45.67.89
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "<myblog_deploy_key.pub 内容>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

- 这样 GitHub Actions 用私钥就可以 SSH 登录服务器，无需密码。

1. **私钥内容放到 GitHub Secrets**

- 打开 GitHub 仓库 → `Settings → Secrets and variables → Actions → New repository secret`
- Secret 名称：`ALI_KEY`
- 内容就是 `~/.ssh/myblog_deploy_key` 文件里的完整内容（保持换行）

> 注意：不要在 public key 中放入 GitHub Secrets，只放私钥。

或者可以直接将本地的公钥上传到服务器，私钥放在github Secrets中

# **二、阿里云服务器初始化**

假设服务器全新（Ubuntu 22.04 举例）：

1. **更新系统**

```
sudo apt update && sudo apt upgrade -y
```

1. **安装必需工具**

```
sudo apt install -y git rsync curl
```

1. **创建博客目录**

```
sudo mkdir -p /var/www/myblog
sudo chown -R $USER:$USER /var/www/myblog
```

1. **允许防火墙端口 80/443**

```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

# **三、Hugo 构建部署准备**

1. 安装 Hugo Extended（支持 PaperMod SCSS）

```
wget https://github.com/gohugoio/hugo/releases/download/v0.115.0/hugo_extended_0.157.0_Linux-64bit.tar.gz
tar -xzf hugo_extended_0.157.0_Linux-64bit.tar.gz
sudo mv hugo /usr/local/bin/

hugo version
# 输出下列
hugo v0.157.0-7747abbb316b03c8f353fd3be62d5011fa883ee6+extended linux/amd64 BuildDate=2026-02-25T16:38:33Z VendorInfo=gohugoio
```

# 安装docker 



## 创建docker-compose

```shell
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: myblog_nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./public:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
```

# **四、配置 ssl**

## 安装 Nginx + Certbot

```
sudo apt install -y nginx certbot python3-certbot-nginx
```

## 申请 SSL 证书（Let’s Encrypt）

使用 certbot 自动签发并配置：

```
sudo certbot --nginx -d ktzxy.top -d www.ktzxy.top
```

按照提示：

- 输入邮箱
- 同意条款
- 选择是否强制 HTTPS（选 2 强制跳转）

成功后会自动生成：

```
/etc/letsencrypt/live/ktzxy.top/
```

------

## 自动续期（重要）

测试自动续期：

```
sudo certbot renew --dry-run
```

Certbot 会自动加入系统定时任务，无需手动操作。

## 创建 Nginx 站点配置

创建文件：

```
sudo vim /var/www/myblog/nginx.conf
```

写入下面完整配置：

```
# ==============================
# HTTP 自动跳转 HTTPS
# ==============================
server {
    listen 80;
    server_name ktzxy.top www.ktzxy.top;
    return 301 https://$host$request_uri;
}

# ==============================
# HTTPS 主站配置
# ==============================
server {
    listen 443 ssl http2;
    server_name ktzxy.top www.ktzxy.top;

    # ⚠ 关键修改
    root /usr/share/nginx/html;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/ktzxy.top/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ktzxy.top/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header Referrer-Policy no-referrer-when-downgrade;

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_min_length 1024;

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|webp|woff2?)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ /\. {
        deny all;
    }

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;
}

```

------

## 启用站点

```
sudo ln -s /etc/nginx/sites-available/ktzxy.top /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 最终检查

查看状态：

```
sudo systemctl status nginx
```

访问：

```
https://ktzxy.top
```

应该看到你的 Hugo 博客。



