+++
date = '2026-03-15T19:38:46+08:00'
draft = true
title = 'Typecho博客部署记录'
description = "typecho博客部署记录"  
summary = ""         
categories = ["📒博客部署"]                                      
tags = ["typecho"]                                                  
+++

## dockercompose

```shell
root@iZwz9cs9md1nr2tk2k4hyzZ:/var/www/typecho# cat docker-compose.yml
version: "3.8"

services:

  nginx:
    image: crpi-7jaksta3zhkkymyt.cn-shenzhen.personal.cr.aliyuncs.com/common_imgage/nginx:alpine
    container_name: typecho-nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    mem_limit: 100m
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    volumes:
      - ./typecho:/var/www/html:cached
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
    depends_on:
      - php
    networks:
      - blog-net

  php:
    build: .
    container_name: typecho-php
    restart: always
    mem_limit: 300m
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    volumes:
      - ./typecho:/var/www/html:cached
    depends_on:
      - db
    networks:
      - blog-net

  db:
    image: crpi-7jaksta3zhkkymyt.cn-shenzhen.personal.cr.aliyuncs.com/common_imgage/mariadb:10.6
    container_name: typecho-db
    restart: always
    mem_limit: 500m
    logging:
      driver: json-file
      options:
        max-size: "50m"
        max-file: "3"
    env_file:
      - .env
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
      --innodb-buffer-pool-size=512M
      --innodb-log-file-size=128M
    volumes:
      - ./mysql:/var/lib/mysql
    networks:
      - blog-net

  redis:
    image: crpi-7jaksta3zhkkymyt.cn-shenzhen.personal.cr.aliyuncs.com/common_imgage/redis:7-alpine
    container_name: typecho-redis
    restart: always
    mem_limit: 200m
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    command: >
      redis-server
      --appendonly yes
      --appendfsync everysec
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
    volumes:
      - ./redis:/data
    networks:
      - blog-net

networks:
  blog-net:
    driver: bridge

```

## Dockerfile

```shell
root@iZwz9cs9md1nr2tk2k4hyzZ:/var/www/typecho# cat Dockerfile
FROM crpi-7jaksta3zhkkymyt.cn-shenzhen.personal.cr.aliyuncs.com/common_imgage/php:8.2-fpm

# 工作目录
WORKDIR /var/www/html

# 安装 PHP 扩展
RUN docker-php-ext-install pdo_mysql mysqli \
    && docker-php-ext-enable opcache

# 复制 PHP 配置
COPY php/opcache.ini /usr/local/etc/php/conf.d/opcache.ini
COPY php/custom.conf /usr/local/etc/php-fpm.d/custom.conf

# 创建 uploads 目录并设置权限
RUN mkdir -p /var/www/html/usr/uploads

```

## nginx.conf

```shell
root@iZwz9cs9md1nr2tk2k4hyzZ:/var/www/typecho# cat nginx.conf
worker_processes auto;

events {
    worker_connections 10240;
}

http {

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    keepalive_timeout 65;

    client_max_body_size 50m;

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml;

    server {

        listen 80;
        server_name ktzxy.top www.ktzxy.top;

        return 301 https://$host$request_uri;
    }

    server {

        listen 443 ssl http2;
        server_name ktzxy.top www.ktzxy.top;

        root /var/www/html;
        index index.php;

        ssl_certificate /etc/letsencrypt/live/ktzxy.top/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/ktzxy.top/privkey.pem;

        location / {

            if (!-e $request_filename){
                rewrite ^(.*)$ /index.php$1 last;
                break;
            }
            # 如果文件存在，直接访问
            try_files $uri $uri/ =404;

        }

        location ~ \.php$ {

            fastcgi_pass php:9000;
            fastcgi_index index.php;

            include fastcgi_params;

            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

            # 增加以下行以防止某些 PATH_INFO 问题
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_param PATH_INFO $fastcgi_path_info;
            fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        }

        location ~* \.(jpg|jpeg|png|gif|css|js|ico|svg|woff2?)$ {

            expires 30d;
            access_log off;

            try_files $uri =404;

        }

        location ~ /\. {
            deny all;
        }

    }

}

```

