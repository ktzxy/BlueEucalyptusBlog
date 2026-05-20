+++
date = '2026-05-20T09:29:10+08:00'
draft = false
title = 'Podman_Nextcloud部署'
slug = "lge7b48ive"
description = "Podman_Nextcloud部署"
summary = "Podman_Nextcloud部署"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# 使用 Podman 无根用户部署 Nextcloud

## 1. 环境准备

### 1.1 创建非 root 用户

```bash
useradd greg
echo 123 | passwd --stdin greg
```

### 1.2 创建工作目录

```bash
mkdir -p /podman/podman-nextcloud/{db,html,nginx,data,config}
chown greg:greg -R /podman/
```

### 1.3 SELinux 与端口调整（启用 443）

```bash
grep "^net.ipv4.ip_unprivileged_port_start" /etc/sysctl.conf || echo "net.ipv4.ip_unprivileged_port_start = 443" >> /etc/sysctl.conf
sysctl -p
setsebool -P httpd_can_network_connect 1
```

### 1.4 开放防火墙端口

```bash
firewall-cmd --add-service=https
firewall-cmd --add-service=https --per
```

## 2. 容器部署 (greg 用户下)

### 2.1 登录新用户并启用 linger 支持

```bash
ssh greg@172.24.1.105
loginctl enable-linger
```

### 2.2 创建 Pod

```bash
podman pod create --hostname nextcloud --name nextcloud -p 443:443
```

### 2.3 MariaDB 容器

```bash
podman rm -f nextcloud-mariadb

podman run -d --pod=nextcloud --name nextcloud-mariadb \
  --tz Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  -e MYSQL_DATABASE=nextcloud \
  -e MYSQL_USER=nextcloud \
  -e MYSQL_PASSWORD=123 \
  -v /podman/podman-nextcloud/db:/var/lib/mysql:Z \
  mariadb:10.6 \
  --transaction-isolation=READ-COMMITTED --binlog-format=ROW
```

### 2.4 Redis 容器

```bash
podman rm -f nextcloud-redis

podman run -d --pod=nextcloud --name nextcloud-redis \
  redis:alpine redis-server --requirepass 123
```

### 2.5 Nextcloud FPM 容器

```bash
podman rm -f nextcloud-fpm

podman run -d --pod=nextcloud --name nextcloud-fpm \
  -e REDIS_HOST="localhost" \
  -e REDIS_HOST_PASSWORD="123" \
  -e MYSQL_HOST="localhost" \
  -e MYSQL_USER="nextcloud" \
  -e MYSQL_PASSWORD="123" \
  -e MYSQL_DATABASE="nextcloud" \
  -v /podman/podman-nextcloud/html:/var/www/html:Z \
  -v /podman/podman-nextcloud/data:/var/www/html/data:Z \
  -v /podman/podman-nextcloud/config:/var/www/html/config:Z \
  docker.io/library/nextcloud:fpm-alpine
```

### 2.6 Cron 容器

```bash
podman rm -f nextcloud-cron

podman run -d --pod=nextcloud --name nextcloud-cron \
  -v /podman/podman-nextcloud/html:/var/www/html:Z \
  --entrypoint=/cron.sh \
  docker.io/library/nextcloud:fpm-alpine
```

## 3. Nginx 部署与配置

### 3.1 创建自签名证书

```bash
cd /podman/podman-nextcloud/nginx
openssl genrsa 2048 > nextcloud.example.com.key

cat > csr.cnf <<EOF
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[dn]
C = CN
ST = shanxi
L = xi'an
O = fanyang
OU = fanyang
CN = nextcloud.example.com
EOF

openssl req -new -key nextcloud.example.com.key -config csr.cnf -out nextcloud.example.com.csr
openssl x509 -req -days 3650 -in nextcloud.example.com.csr -signkey nextcloud.example.com.key -out nextcloud.example.com.crt
```

### 3.2 编写 Nginx 配置文件 (nginx.conf)

```bash
cat > /podman/podman-nextcloud/nginx/nginx.conf << 'END'
worker_processes auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include mime.types;
    default_type  application/octet-stream;
    types {
        text/javascript mjs;
    }

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    # Prevent nginx HTTP Server Detection
    server_tokens   off;

    keepalive_timeout  65;

    # Set the `immutable` cache control options only for assets with a cache busting `v` argument
    map $arg_v $asset_immutable {
        "" "";
    default ", immutable";
    }

    #gzip  on;

    upstream php-handler {
        server nextcloud-fpm:9000;
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        ssl_certificate /etc/ssl/certs/nextcloud.example.com.crt;
        ssl_certificate_key /etc/ssl/private/nextcloud.example.com.key;

        # HSTS settings
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;

        # set max upload size and increase upload timeout:
        client_max_body_size 512M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;

        # The settings allows you to optimize the HTTP2 bandwidth.
        # See https://blog.cloudflare.com/delivering-http-2-upload-speed-improvements/
        # for tuning hints
        client_body_buffer_size 512k;

        # Enable gzip but do not remove ETag headers
        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml text/javascript application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

        # Pagespeed is not supported by Nextcloud, so if your server is built
        # with the `ngx_pagespeed` module, uncomment this line to disable it.
        #pagespeed off;

        # HTTP response headers borrowed from Nextcloud `.htaccess`
        add_header Referrer-Policy                      "no-referrer"       always;
        add_header X-Content-Type-Options               "nosniff"           always;
        add_header X-Frame-Options                      "SAMEORIGIN"        always;
        add_header X-Permitted-Cross-Domain-Policies    "none"              always;
        add_header X-Robots-Tag                         "noindex, nofollow" always;
        add_header X-XSS-Protection                     "1; mode=block"     always;

        # Remove X-Powered-By, which is an information leak
        fastcgi_hide_header X-Powered-By;

        # Path to the root of your installation
        root /var/www/html;

        # Specify how to handle directories -- specifying `/index.php$request_uri`
        # here as the fallback means that Nginx always exhibits the desired behaviour
        # when a client requests a path that corresponds to a directory that exists
        # on the server. In particular, if that directory contains an index.php file,
        # that file is correctly served; if it doesn't, then the request is passed to
        # the front-end controller. This consistent behaviour means that we don't need
        # to specify custom rules for certain paths (e.g. images and other assets,
        # `/updater`, `/ocm-provider`, `/ocs-provider`), and thus
        # `try_files $uri $uri/ /index.php$request_uri`
        # always provides the desired behaviour.
        index index.php index.html /index.php$request_uri;

        # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
        location = / {
            if ( $http_user_agent ~ ^DavClnt ) {
                return 302 /remote.php/webdav/$is_args$args;
            }
        }

        location = /robots.txt {
            allow all;
            log_not_found off;
            access_log off;
        }

        # Make a regex exception for `/.well-known` so that clients can still
        # access it despite the existence of the regex rule
        # `location ~ /(\.|autotest|...)` which would otherwise handle requests
        # for `/.well-known`.
        location ^~ /.well-known {
            # The rules in this block are an adaptation of the rules
            # in `.htaccess` that concern `/.well-known`.

            location = /.well-known/carddav { return 301 /remote.php/dav/; }
            location = /.well-known/caldav  { return 301 /remote.php/dav/; }

            location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
            location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

            # Let Nextcloud's API for `/.well-known` URIs handle all other
            # requests by passing them to the front-end controller.
            return 301 /index.php$request_uri;
        }

        # Rules borrowed from `.htaccess` to hide certain paths from clients
        location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
        location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)                { return 404; }

        # Ensure this block, which passes PHP files to the PHP process, is above the blocks
        # which handle static assets (as seen below). If this block is not declared first,
        # then Nginx will encounter an infinite rewriting loop when it prepends `/index.php`
        # to the URI, resulting in a HTTP 500 error response.
        location ~ \.php(?:$|/) {
            # Required for legacy support
            rewrite ^/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|ocs-provider\/.+|.+\/richdocumentscode(_arm64)?\/proxy) /index.php$request_uri;

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;
            fastcgi_param HTTPS on;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass php-handler;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;

            fastcgi_max_temp_file_size 0;
        }

        # Serve static files
        location ~ \.(?:css|js|mjs|svg|gif|ico|jpg|png|webp|wasm|tflite|map|ogg|flac)$ {
            try_files $uri /index.php$request_uri;
            add_header Cache-Control "public, max-age=15778463$asset_immutable";
            add_header Referrer-Policy                   "no-referrer"       always;
            add_header X-Content-Type-Options            "nosniff"           always;
            add_header X-Frame-Options                   "SAMEORIGIN"        always;
            add_header X-Permitted-Cross-Domain-Policies "none"              always;
            add_header X-Robots-Tag                      "noindex, nofollow" always;
            add_header X-XSS-Protection                  "1; mode=block"     always;
            access_log off;     # Optional: Don't log access to assets

            location ~ \.wasm$ {
                default_type application/wasm;
            }
        }

        location ~ \.woff2?$ {
            try_files $uri /index.php$request_uri;
            expires 7d;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        # Rule borrowed from `.htaccess`
        location /remote {
            return 301 /remote.php$request_uri;
        }

        location / {
            try_files $uri $uri/ /index.php$request_uri;
        }
    }
}
END
```

### 3.3 启动 Nginx 容器

```bash
podman rm -f nextcloud-nginx

podman run -d --pod=nextcloud --name nextcloud-nginx \
  -v /podman/podman-nextcloud/html:/var/www/html:ro,Z \
  -v /podman/podman-nextcloud/nginx/nginx.conf:/etc/nginx/nginx.conf:ro,Z \
  -v /podman/podman-nextcloud/nginx/nextcloud.example.com.crt:/etc/ssl/certs/nextcloud.example.com.crt:Z \
  -v /podman/podman-nextcloud/nginx/nextcloud.example.com.key:/etc/ssl/private/nextcloud.example.com.key:Z \
  docker.io/library/nginx:alpine
```

## 4. 初始化 Nextcloud

### 4.1 执行安装

```bash
podman exec -it -u www-data nextcloud-fpm /bin/sh
php occ maintenance:install \
  --database "mysql" \
  --database-host "127.0.0.1" \
  --database-name "nextcloud" \
  --database-user "nextcloud" \
  --database-pass "123" \
  --admin-user "test" \
  --admin-pass "password"
```

### 4.2 设置受信任域名与其它参数

```bash
# 配置一些设置，例如受信任的域
php occ config:system:set trusted_domains 1 --value=172.24.1.105
php occ config:system:set trusted_domains 2 --value=nextcloud.example.com
php occ config:system:set overwrite.cli.url --value "https://nextcloud.example.com"
php occ config:system:set overwriteprotocol --value "https"

# 设置维护时段
php occ config:system:set maintenance_window_start --value=2 --type=integer
php occ config:system:set maintenance_window_end --value=5 --type=integer

# 设置默认国际区号
php occ config:system:set default_phone_region --value="CN"

# 执行 MIME 类型迁移
php occ maintenance:repair --include-expensive

# 添加缺失索引
php occ db:add-missing-indices
```

### 4.3 容器自启动配置

```bash
podman generate systemd --files --new --name nextcloud
mkdir -p ~/.config/systemd/user
mv container-nextcloud-* pod-nextcloud.service ~/.config/systemd/user
systemctl --user enable --now pod-nextcloud.service
```
