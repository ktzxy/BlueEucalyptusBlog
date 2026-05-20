+++
date = '2026-05-20T09:26:55+08:00'
draft = false
title = '部署_Postgresql'
slug = "no7h0scvdw"
description = "部署_Postgresql"
summary = "部署_Postgresql"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# 使用 Podman 和 Systemd 部署并管理 PostgreSQL

> 本指南将详细介绍如何使用 Podman 部署一个持久化的 PostgreSQL 数据库容器，并通过 Systemd 将其注册为系统服务，实现开机自启和便捷管理。

**核心理念**

- **Podman**: 一个无守护进程的容器引擎，提供了与 Docker 兼容的命令行接口，但在安全性和与 Systemd 的集成方面更具优势。
- **容器化数据库**: 将 PostgreSQL 运行在容器中，可以实现环境隔离、版本控制、快速部署和轻松迁移。
- **Systemd 集成**: 使用 `podman generate systemd` 可以为容器自动生成服务单元文件，让您能像管理普通系统服务一样（例如 `systemctl start/stop/status`）来管理容器的生命周期。

## 1. 准备工作

### 步骤 1.1：创建持久化存储目录

为了防止容器被删除后数据丢失，我们需要在主机上创建目录，用于持久化存储 PostgreSQL 的数据和配置。

```bash
# 创建数据和配置的主目录
mkdir -p /podman/podman-postgres/{data,config}
```

- `/podman/podman-postgres/data`: 将用于存储数据库的核心数据文件。
- `/podman/podman-postgres/config`: 将用于存储 PostgreSQL 的配置（虽然在此示例中未使用，但为将来扩展保留）。

### 步骤 1.2：设置 PostgreSQL 环境变量 (关键)

官方的 PostgreSQL 镜像使用环境变量来初始化数据库，其中**设置密码是必需的**。

```bash
# 设置数据库的用户名和密码
# 建议使用一个强密码
export POSTGRES_USER="myuser"
export POSTGRES_PASSWORD="YourStrongPassword123"
```

> 您也可以直接在 `podman run` 命令中使用 `-e` 参数传入这些变量。

## 2. 部署 PostgreSQL 容器

现在，我们运行 `podman run` 命令来创建容器。

```bash
# --rm 选项确保在生成 systemd 文件后，这个临时容器会被自动删除
podman run --rm -d \
--name postgres \
-p 5432:5432 \
-e POSTGRES_USER=${POSTGRES_USER} \
-e POSTGRES_PASSWORD=${POSTGRES_PASSWORD} \
--tz Asia/Shanghai \
-v /podman/podman-postgres/data:/var/lib/postgresql/data:Z \
postgres:16.2
```

**`podman run` 参数详解：**

| 参数                                | 含义                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| `--rm`                              | 容器停止后自动删除。我们用它来生成 systemd 文件，之后由 systemd 管理。 |
| `-d`                                | 后台运行容器（分离模式）。                                   |
| `--name postgres`                   | 为容器指定一个易于识别的名称。                               |
| `-p 5432:5432`                      | 将主机的 5432 端口映射到容器的 5432 端口。                   |
| `-e POSTGRES...`                    | **(关键)** 设置环境变量。`POSTGRES_PASSWORD` 首次运行时会用于设置超级用户的密码。 |
| `--tz Asia/Shanghai`                | 设置容器内的时区，确保日志和时间函数正确。                   |
| `-v ...:/var/lib/postgresql/data:Z` | **(关键)** 将主机上的数据目录挂载到容器内，实现数据持久化。`:Z` 选项会自动处理 SELinux 标签，确保容器有权限读写该目录。 |
| `postgres:16.2`                     | 要使用的镜像及其版本。                                       |

> **安全性说明**：原始脚本中的 `--privileged` 已被移除。该参数赋予容器过高的权限，通常是不必要的，并且会带来安全风险。PostgreSQL 容器在无此权限的情况下可以正常运行。

## 3. 使用 Systemd 进行服务管理

现在我们为刚刚创建的容器生成一个 Systemd 服务单元文件，使其可以被 `systemctl` 管理。

### 方法一：创建 Systemd 用户服务 (User Service)

此方法将服务安装在当前用户下，无需 `root` 权限即可管理，但默认情况下，用户必须登录后服务才会运行。

```bash
# 1. 为名为 "postgres" 的容器生成 systemd 文件
podman generate systemd --files --new --name postgres

# 2. 将生成的服务文件移动到用户服务目录下
mkdir -p ~/.config/systemd/user
mv container-postgres.service ~/.config/systemd/user/

# 3. 停止并删除我们手动创建的临时容器（因为 systemd 将接管它）
podman stop postgres

# 4. (关键步骤) 允许用户服务在系统启动时自启，即使用户未登录
loginctl enable-linger $(whoami)

# 5. 重载用户 systemd 配置，并启动/启用服务
systemctl --user daemon-reload
systemctl --user enable --now container-postgres.service
```

### 方法二：创建 Systemd 系统服务 (System Service - 推荐用于服务器)

此方法将服务安装为系统级服务，由 `root` 管理，并随系统自动启动，更适合在服务器环境中使用。

```bash
# 1. 为名为 "postgres" 的容器生成 systemd 文件
podman generate systemd --files --new --name postgres

# 2. 将生成的服务文件移动到系统服务目录下
mv container-postgres.service /etc/systemd/system/

# 3. 停止并删除我们手动创建的临时容器
podman stop postgres

# 4. 重载系统 systemd 配置，并启动/启用服务
systemctl daemon-reload
systemctl enable --now container-postgres.service
```

## 4. 验证与连接

### 步骤 4.1：检查容器状态

无论使用哪种服务方式，都可以用 `podman` 命令来检查容器的运行状态。

```bash
# 查看正在运行的容器
podman ps

# 查看 postgres 容器的日志，确认数据库已成功启动
podman logs postgres
```

### 步骤 4.2：连接到数据库

您可以使用任何标准的 PostgreSQL 客户端（如 `psql`）来连接到数据库。

```bash
# 确保您已安装 postgresql 客户端
# dnf install postgresql -y

# 从主机连接到容器中的 PostgreSQL 服务
# 使用您之前设置的用户名和密码
psql --host=localhost --port=5432 --username=${POSTGRES_USER} --dbname=postgres
```

成功连接并看到 `psql` 提示符，即表示您的 PostgreSQL 容器已部署成功并正常运行。
