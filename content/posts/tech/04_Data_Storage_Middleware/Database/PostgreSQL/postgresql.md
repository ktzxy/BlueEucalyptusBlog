+++
date = '2026-05-12T11:43:36+08:00'
draft = false
title = 'PostgreSQL'
slug = "z5m1xnosye"
description = "PostgreSQL 深度部署与管理指南"
summary = "PostgreSQL 深度部署与管理指南"
categories = ["📒数据库"]
tags = ["PostgreSQL"]
comments = true
+++

# PostgreSQL 深度部署与管理指南 (含 TimescaleDB)

> 本指南旨在为数据库管理员和开发人员提供一份在 RHEL 及其衍生版系统上部署 PostgreSQL 的全面手册。内容涵盖了两种安装方式（官方源和源码编译），并详细介绍了如何集成强大的 `TimescaleDB` 时序数据库扩展。

**核心概念简介*

- **PostgreSQL**: 一款功能强大、高度可扩展的开源对象关系型数据库系统，以其稳定性、数据一致性和对 SQL 标准的严格遵循而闻名。
- **TimescaleDB**: 一个基于 PostgreSQL 的开源扩展，专门用于优化和加速时间序列数据的处理。它能将标准的 PostgreSQL 数据库转变为一个性能卓越的时序数据库。

## 1. PostgreSQL 安装

### 方法一：使用官方 YUM/DNF 源安装 (强烈推荐)

这是最简单、最安全、最易于维护的安装方式，由 PostgreSQL 全球开发组 (PGDG) 直接提供支持。

```bash
# 1. 安装 PGDG 仓库 RPM
# 访问 https://www.postgresql.org/download/linux/redhat/ 选择您的系统版本
# 以 AlmaLinux 9 为例:
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# 2. 禁用系统自带的 PostgreSQL 模块 (如果存在)
dnf -qy module disable postgresql

# 3. 安装 PostgreSQL 15 服务器和 contrib 包
dnf install -y postgresql15-server postgresql15-contrib

# 4. 初始化数据库集群
# 此命令会在 /var/lib/pgsql/15/data 目录下创建数据文件
/usr/pgsql-15/bin/postgresql-15-setup initdb

# 5. 启动并设置开机自启
systemctl enable --now postgresql-15
```

- **优点**: 安装管理简单，升级方便 (`dnf update`)，文件路径标准化，社区支持良好。

### 方法二：从源码编译安装 (高级/定制化)

源码编译提供了最大的灵活性，但过程复杂，维护成本较高。

#### 步骤 1: 安装编译依赖

```bash
yum install -y wget git cmake gcc gcc-c++ zlib-devel readline-devel openssl-devel libuuid-devel
```

#### 步骤 2: 下载并编译 PostgreSQL

```bash
# 切换到源码存放目录
cd /usr/local/src

# 下载并解压 (建议在官网检查最新版本)
wget https://ftp.postgresql.org/pub/source/v15.6/postgresql-15.6.tar.gz
tar -xzf postgresql-15.6.tar.gz && cd postgresql-15.6

# 配置、编译并安装
./configure --prefix=/opt/pgsql15 --with-openssl
make -j$(nproc)
make install
```

#### 步骤  3: 配置环境变量与用户

```bash
# 将 PostgreSQL 的 bin 目录添加到全局 PATH
grep -q "/opt/pgsql15/bin" /etc/profile || echo "export PATH=\$PATH:/opt/pgsql15/bin" >> /etc/profile
source /etc/profile

# 创建用于运行 PostgreSQL 的专用用户
id postgres &>/dev/null || useradd -m -s /bin/bash postgres
```

#### 步骤4: 初始化数据库集群

数据库集群 (Cluster) 是 PostgreSQL 存放所有数据的地方。

```bash
# 创建数据目录并授权
mkdir -p /opt/pgsql15/data
chown -R postgres:postgres /opt/pgsql15

# 切换到 postgres 用户身份，执行初始化命令
su - postgres -c "/opt/pgsql15/bin/initdb -D /opt/pgsql15/data"
```

## 2. TimescaleDB 扩展安装 (基于源码编译)

```bash
# 1. 下载 TimescaleDB 源码
cd /usr/local/src
git clone https://github.com/timescale/timescaledb.git
cd timescaledb
# 检出与您的 PostgreSQL 版本兼容的稳定分支
git checkout 2.14.2

# 2. 编译并安装
# -DPG_CONFIG 指向我们刚安装的 PostgreSQL 的配置脚本
./bootstrap -DPG_CONFIG=/opt/pgsql15/bin/pg_config
cd ./build && make -j$(nproc)
make install

# 3. 配置 PostgreSQL 加载此扩展
# 编辑 postgresql.conf，让数据库在启动时预加载 TimescaleDB 库
echo "shared_preload_libraries = 'timescaledb'" >> /opt/pgsql15/data/postgresql.conf
```

## 3. 核心配置与访问控制

新初始化的数据库默认只允许本地连接。我们需要修改配置以允许远程访问。

### 步骤 1: 监听网络连接 (`postgresql.conf`)

编辑主配置文件，让 PostgreSQL 监听所有网络接口，而不仅仅是 `localhost`。

```bash
# 将 listen_addresses 设置为 '*'
sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/" /opt/pgsql15/data/postgresql.conf
```

### 步骤 2: 配置客户端认证 (`pg_hba.conf`)

`pg_hba.conf` 文件定义了谁、从哪里、以何种方式可以连接到哪个数据库。**这是最重要的安全配置文件。**

```bash
# 示例：允许来自 192.168.1.0/24 网段的所有用户使用加密密码 (scram-sha-256) 连接所有数据库
# 在文件末尾追加此行
echo "host    all             all             192.168.1.0/24        scram-sha-256" >> /opt/pgsql15/data/pg_hba.conf
```

## 4. 服务管理与数据库初始化

### 步骤 1: 创建 Systemd 服务

```bash
cat > /etc/systemd/system/postgresql15.service << END
[Unit]
Description=PostgreSQL 15.6 database server
After=network.target

[Service]
Type=forking
User=postgres
Group=postgres
Environment=PATH=/opt/pgsql15/bin:/usr/bin:/bin
ExecStart=/opt/pgsql15/bin/pg_ctl start -D /opt/pgsql15/data -l /opt/pgsql15/data/logfile
ExecStop=/opt/pgsql15/bin/pg_ctl stop -D /opt/pgsql15/data -s -m fast
ExecReload=/opt/pgsql15/bin/pg_ctl reload -D /opt/pgsql15/data -s
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
END

systemctl daemon-reload
```

### 步骤 2: 启动服务并设置超级用户密码

```bash
# 启动并设置开机自启
systemctl enable --now postgresql15.service

# 切换到 postgres 用户并登录数据库，为自己设置一个强密码
su - postgres
psql -c "ALTER USER postgres WITH PASSWORD 'YourStrongPassword';"
exit
```

### 步骤 3: 创建应用数据库和用户 (最佳实践)

不要直接使用 `postgres` 超级用户来连接您的应用程序。为每个应用创建专用的数据库和用户。

```bash
# 以 postgres 用户身份执行
su - postgres
# 1. 创建一个新用户
createuser myapp_user
# 2. 创建一个新数据库，所有者为新用户
createdb myapp_db -O myapp_user
# 3. 登录并为新用户设置密码
psql -c "ALTER USER myapp_user WITH PASSWORD 'AppUserPassword';"
exit
```

## 5. 启用并验证 TimescaleDB

安装完扩展并重启数据库后，还需要在具体的数据库中**激活**它。

```bash
# 1. 使用新创建的应用用户连接到其数据库
psql -U myapp_user -d myapp_db

# 2. 在 psql 命令行中，执行以下 SQL 命令来创建扩展
CREATE EXTENSION IF NOT EXISTS timescaledb;

# 3. 验证扩展是否已成功安装并激活
\dx
# 您应该能在列表中看到 timescaledb
```

至此，您的 PostgreSQL + TimescaleDB 服务器已完全部署并准备就绪。

