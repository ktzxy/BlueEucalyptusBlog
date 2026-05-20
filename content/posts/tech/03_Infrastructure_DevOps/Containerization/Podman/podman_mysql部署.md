+++
date = '2026-05-20T09:29:43+08:00'
draft = false
title = 'Podman_MySql部署'
slug = "9iszw0who5"
description = "Podman_MySql部署"
summary = "Podman_MySql部署"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# 使用 Podman 无根用户部署 MySQL

## 1. 目录准备

```bash
mkdir -p /podman/podman-mysql/{data,etc/conf.d,log,mysql-files}
```

## 2. 配置文件 `my.cnf`

> 可根据实际需求取消注释 `skip-grant-tables` 用于跳过认证（恢复密码时使用）。

```bash
cat > /podman/podman-mysql/etc/conf.d/my.cnf << 'END'
[mysqld]
port = 3306
bind-address = 0.0.0.0
character-set-server = utf8mb4
max_connections = 1024
default-time_zone = '+8:00'
slow-query-log = on
long_query_time = 2
skip-log-bin
# skip-grant-tables

[client]
default-character-set = utf8mb4
END
```

## 3. 容器运行流程

### 3.1 启用 linger

```bash
# 启用 linger（允许非 root 用户后台运行服务）
loginctl enable-linger
```

### 3.2 停止并移除旧容器（若存在）

```bash
systemctl --user stop container-mysql-8.0.33.service &>/dev/null
podman stop mysql-8.0.33 &>/dev/null && podman rm mysql-8.0.33 &>/dev/null
```

### 3.3 启动 MySQL 容器

```bash
podman rm -f mysql-8.0.33

podman run -d --name mysql-8.0.33 \
  -p 3306:3306 \
  --tz Asia/Shanghai \
  -v /podman/podman-mysql/etc/conf.d:/etc/mysql/conf.d:Z \
  -v /podman/podman-mysql/data:/var/lib/mysql:Z \
  -v /podman/podman-mysql/mysql-files:/var/lib/mysql-files:Z \
  -v /podman/podman-mysql/log:/var/log/mysql:Z \
  -e MYSQL_ROOT_PASSWORD=123 \
  mysql:8.0.33
```

## 4. 配置 Systemd 自启动服务

### 4.1 生成 Systemd 服务文件

```bash
mkdir -p ~/.config/systemd/user && cd ~/.config/systemd/user
podman generate systemd --files --new --name mysql-8.0.33
```

### 4.2 停止容器

```bash
# 停止容器（释放占用，交由 systemd 管理）
podman stop mysql-8.0.33 && podman rm mysql-8.0.33
```

### 4.3 启动并设置开机自启

```bash
systemctl --user daemon-reload
systemctl --user enable container-mysql-8.0.33.service --now
```

## 5. 常用命令

### 5.1 进入容器 shell

```bash
podman exec -it mysql-8.0.33 bash
```

### 5.2 登录 MySQL

```bash
podman exec -it mysql-8.0.33 mysql -uroot -p123
```

### 5.3 查看运行日志

```bash
podman logs mysql-8.0.33
```
