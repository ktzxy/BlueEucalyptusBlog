+++
date = '2026-05-20T09:24:02+08:00'
draft = false
title = 'Podman 的使用指南'
slug = "t89ktss8fn"
description = "Podman 的使用指南"
summary = "Podman 的使用指南"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# Podman 的使用指南

## 1. Podman 简介

### 1.1 podman 的由来

```bash
很多用户在使用 Docker 时，可能遇到过系统重启后，Docker 守护进程（daemon）占用多核 CPU 达到 100% 的情况，导致所有容器无法启动，服务不可用。这不仅是性能问题，更是安全隐患，因为 Docker 的守护进程通常以 root 特权权限运行。
为了解决这些问题，可以选择 Podman。
官网: https://podman.io/
```

### 1.2 什么是 Podman

```bash
Podman 是一个开源的、无守护进程（daemonless）的容器引擎，兼容 Open Container Initiative（OCI）标准，可用于开发、管理和运行容器与容器镜像。
特点如下:
- 无需守护进程，使用更安全
- 与 Docker CLI 命令兼容，可以通过 alias docker=podman 实现无缝切换
- 支持 root 与非 root 用户运行容器
- 支持 pod、容器、镜像和卷等容器生态系统的管理
```

### 1.3 Podman 与 Docker 对比

| 特性               | Docker                              | Podman                        |
| ------------------ | ----------------------------------- | ----------------------------- |
| 是否依赖守护进程   | 是（docker daemon）                 | 否                            |
| 容器进程关系       | container 是 docker engine 的子进程 | container 是 podman 的子进程  |
| 支持 restart 策略  | 支持                                | 不支持（可通过 systemd 实现） |
| 是否需要 root 权限 | 是                                  | 否（支持 rootless 模式）      |

#### 详细对比说明

1. **守护进程**：
    - Docker 必须依赖 `dockerd` 守护进程运行，一旦 daemon 异常，所有容器都无法运行。
    - Podman 是无守护进程的，容器生命周期完全由命令驱动，不依赖长期运行的后台服务。
2. **容器进程关系**：
    - Docker CLI 命令通过 REST API 与 docker daemon 通信，由 docker daemon 再调用 runc 启动容器，因此容器进程是 daemon 的子进程。
    - Podman CLI 直接调用 runc 启动容器，容器进程是 podman CLI 的子进程，结构更清晰。
3. **重启策略**：
    - Docker 支持 `--restart=always` 等参数，可实现容器异常重启。
    - Podman 不支持 restart 策略，但可结合 `systemd` 管理器创建 systemd 服务文件实现自启动和异常自动恢复。
4. **权限控制**：
    - Docker 默认使用 root 用户管理容器，普通用户需要加入 `docker` 用户组才能访问 Docker socket，存在提权风险。
    - Podman 支持 rootless 模式，普通用户可以独立运行容器，且容器运行在用户权限之下，更加安全。

### 1.4 podman架构

```bash
Podman 与 Buildah、Skopeo 一起构成了一个完整的无守护进程容器工具链，由 Red Hat 主导开发。它们均符合 OCI 标准。
- Podman:  负责运行与管理容器
- Buildah: 负责构建镜像
- Skopeo:  负责镜像管理（拉取、推送、复制）
Podman 通过用户命名空间模拟容器内的 root 权限，无需连接 root socket，更加安全。
```

### 1.5 buildah

```bash
Buildah 是专门用于构建 OCI（Open Container Initiative）镜像的工具。相比 Podman 构建镜像，Buildah 构建速度更快，效率更高。
Buildah 特性梳理
- Buildah 用于构建容器镜像，而 Podman 更偏向于运行和管理容器。
- 支持使用 Dockerfile 构建镜像（通过 buildah bud 命令）。
- 支持不使用 Dockerfile，通过脚本方式一步步构建镜像，更加灵活。
- 不依赖守护进程，采用简单的 fork-exec 模型运行。
- 默认使用 overlay 存储驱动，构建速度快、磁盘空间占用少。
- 支持非 root 用户使用，适用于多用户和受限权限环境。
- 兼容 Dockerfile 中的所有构建命令，目标是成为更底层的构建工具。
- 可以与其他脚本语言集成，便于自动化构建与 CI/CD 系统对接。
```

| 功能             | Podman         | Buildah                  |
| ---------------- | -------------- | ------------------------ |
| 用途             | 管理与运行容器 | 构建容器镜像             |
| 是否构建镜像     | 可以（效率低） | 是（构建快、存储效率高） |
| 默认存储驱动     | vfs            | overlay                  |
| 支持 Dockerfile  | 是             | 是（buildah bud）        |
| 命令式构建支持   | 不支持         | 支持（一步步拼装镜像）   |
| 是否需要守护进程 | 否             | 否                       |

### 1.6 skopeo

```bash
Skopeo 是用于镜像管理的工具，支持：
- Push 镜像
- Pull 镜像
- 复制镜像（本地或远程之间）
- 不需要本地保存镜像，即可进行操作
```

## 2. Podman 安装

- RHEL ≥ 8.2 或 CentOS ≥ 8.2 默认支持
- 安装命令：```yum -y install podman```

## 3. Podman 镜像加速

Podman 默认从 `registry.redhat.io` 或 `docker.io` 拉取镜像，下载速度可能受限。如果你在中国大陆网络环境中，可以配置阿里云镜像加速，提高镜像拉取速度。

```bash
# 备份原始配置文件：
cp /etc/containers/registries.conf{,.bak}

# 重写配置文件，设置默认加速镜像地址为阿里云：
cat > /etc/containers/registries.conf << 'END'
unqualified-search-registries = ["docker.io"]
[[registry]]
prefix = "docker.io"
location = "fiyc0dbc.mirror.aliyuncs.com"
END

# 禁用 registry 短名称自动补全（防止解析出错）：
mv /etc/containers/registries.conf.d/000-shortnames.conf /etc/containers/registries.conf.d/000-shortnames.conf.bak
mv /etc/containers/registries.conf.d/001-rhel-shortnames.conf /etc/containers/registries.conf.d/001-rhel-shortnames.conf.bak
```

## 4. Podman 镜像管理

### 4.1 镜像拉取

```bash
podman pull httpd
```

### 4.2 镜像查看

```bash
podman images
```

### 4.3 镜像删除

```bash
podman image rm docker.io/library/ubuntu
```

### 4.4 镜像备份

```bash
podman save > centos-latest.tar docker.io/library/centos:latest
ls -lh centos-latest.tar
```

### 4.5 镜像导入

```bash
podman load -i centos-latest.tar
```

### 4.6 podman镜像搜索

```bash
podman search nginx
```

## 5. Podman 容器管理

Podman 提供了一套完整的容器生命周期管理命令，与 Docker 命令几乎一致，但无需守护进程（daemon）。

### 5.1 容器运行与进入

```bash
# 运行一个后台容器（-d），自动分配伪终端（-t），以 web1 命名容器
podman run -dt --name web1 httpd

# 进入容器交互式终端
podman exec -it web1 bash
```

### 5.2 停止容器

```bash
podman container stop web1
```

### 5.3 查看容器 (包括停止状态)

```bash
podman ps -a
```

### 5.4 启动 & 重启容器

```bash
podman start web1
podman restart web1
```

### 5.5 删除容器

```bash
# 正常删除容器（需先停止）
podman stop web1
podman rm web1

# 强制删除（即使容器正在运行）
podman rm -f web1
```

### 5.6 查看容器详细信息

```bash
podman inspect web1

# 可查看 IP 地址、挂载卷、环境变量、网络配置等完整 JSON 信息；
# 可搭配 jq 提取字段：
podman inspect -f '{{.NetworkSettings.IPAddress}}' web1

# 查看容器资源限制
podman inspect --format "{{ .HostConfig }}" web1
```

### 5.7 查看容器日志

```bash
podman logs web1
```

## 6. Podman 网络管理

Podman 支持自定义容器网络，默认使用 `bridge` 网络模型，也支持自定义子网、端口映射、IPv6、DNS 等功能。

### 6.1 宿主网络状态检查

```bash
nmcli device status      # 查看网卡状态
nmcli con show           # 查看所有连接
ip a show                # 查看 IP 地址分配
```

### 6.2 创建容器网络

```bash
# 会自动创建一个类似 podman1 的网络。
podman network create
podman network ls

# 为容器指定网络运行
podman run -dt --name web2 --network podman1 httpd
```

### 6.3 自定义子网（IPv4 和 IPv6）

```bash
# 创建 IPv4 网络
podman network create --subnet 192.5.0.0/16 newnet
cat /etc/containers/networks/newnet.json

# 创建 IPv6 网络
podman network create --subnet 2001:db8::/64 --ipv6 newnetv6

# 运行容器并验证 IPv6 地址：
podman run -dt --name web4 --network newnetv6 httpd
podman inspect web4 | grep GlobalIPv6Address
```

### 6.4 删除容器网络

```bash
# 注意：删除网络前必须先停止并移除使用该网络的容器。
podman network rm newnet
podman network rm newnetv6
podman network rm podman1
```

### 6.5 端口映射

```bash
podman run -dt --name web2 -p 12345:80 httpd

# 访问方式
curl localhost:12345
```

### 6.6 开启 Podman 网络 DNS

```bash
# Netavark + Aardvark
# 安装依赖组件
yum install -y netavark aardvark-dns

# 配置 Podman 使用 netavark 网络后端
sed -i 's/^#\?network_backend.*/network_backend = "netavark"/' /usr/share/containers/containers.conf

# 自定义网络配置文件示例（包含 DNS）
cat > /etc/containers/networks/podman.json << END
{
    "name": "podman",
    "id": "2f259bab93aaaaa2542ba43ef33eb990d0999ee1b9924b557b7be53c0b7a1bb9",
    "driver": "bridge",
    "network_interface": "podman0",
    "created": "2025-01-07T09:53:35.573770667+08:00",
    "subnets": [
         {
              "subnet": "10.88.0.0/16",
              "gateway": "10.88.0.1"
         }
    ],
    "ipv6_enabled": false,
    "internal": false,
    "dns_enabled": true,
    "ipam_options": {
         "driver": "host-local"
    }
}
END

# 可选查看本地网络定义：
cat ~/.local/share/containers/storage/networks/podman.json
```

## 7. Podman 持久化存储

容器的数据默认是临时的，Podman 提供了两种方式将容器与宿主机的文件系统持久挂载，避免数据丢失:

- **数据卷 (Volume)**: Podman 管理的持久性目录，推荐用于生产
- **绑定挂载 (Bind Mount)**: 将宿主机的目录挂载到容器中，更加灵活，适合开发

### 7.1 数据卷 (Volume)

```bash
# 创建与查看数据卷
podman volume ls                        # 列出所有数据卷
podman volume create volume1            # 创建数据卷 volume1
find / -name volume1                    # 搜索实际挂载路径

# 运行容器并使用数据卷
podman volume create web
podman run -dt --name centos1 -v web:/web centos
podman exec -i centos1 df -Th           # 查看挂载状态

# 查看卷和挂载信息
podman volume inspect web
podman inspect centos1 | grep web

# 挂载卷到特定容器目录
podman run -dt --name web1 -v web:/usr/local/apache2/htdocs httpd
podman exec -i web1 df -Th
```

### 7.2 绑定挂载 (Bind Mount)

```bash
# 准备宿主机目录
mkdir /web2
echo "dmxy" >> /web2/index.html

# 挂载宿主机目录到容器中
podman run -dt --name web2 -v /web2:/usr/local/apache2/htdocs httpd

# 获取容器 IP 并访问页面（容器间访问）
podman inspect web2 | grep 10.88        # 获取容器 IP
podman exec -i centos1 curl 10.88.0.12  # 其他容器访问 web2 页面
```

```bash
# SELinux 访问控制处理
# 如果启用了 SELinux（默认开启），直接绑定目录可能会失败。你需要执行：
# 临时关闭 SELinux 进行测试
setenforce 0
podman exec -i centos1 curl 10.88.0.12
```

```bash
# 恢复 Enforcing 模式
setenforce 1

# 检查 SELinux 标签差异：
ls -ldZ /web2/
ls -ldZ /var/lib/containers/storage/volumes/web/
```

```bash
# 推荐做法：添加 :Z 参数自动 relabel 挂载目录
podman run -dt --name web3 -p 22222:80 -v /web2:/usr/local/apache2/htdocs:Z httpd
curl localhost:22222

# :Z：为容器独占访问 relabel；
# :z：多容器共享挂载访问（慎用）
```

## 8. podman 容器自启动

Podman 无守护进程，不支持 `--restart=always`，因此要实现容器的 **自启动** 和 **崩溃自动恢复**，推荐用 `systemd` 管理容器服务。

### 8.1 基础方法: 为已有容器生成 systemd 服务文件

- --name表示使用容器的名字来代替容器的ID

- --files表示生成systemd的服务文件

- web1表示使用哪个容器生成systemd的服务文件

```bash
# 第一步：启动容器
podman run -dt --name web1 httpd

# 第二步：生成 systemd 服务文件
podman generate systemd --name web1 --files

# 第三步：安装 systemd 服务
mv container-web1.service /etc/systemd/system/
restorecon -RvF /etc/systemd/system/container-web1.service  # 恢复 SELinux 安全上下文
systemctl daemon-reexec                                     # 可选：重载 systemd 本体
systemctl restart container-web1.service                    # 启动容器服务
```

### 8.2 高级用法：自动创建新容器的 systemd 服务

```bash
podman run -dt --name web2 -p 88:80 -v /web2:/usr/local/apache2/htdocs:Z httpd

# 生成 systemd 服务文件(--new 表示重建容器): 
podman generate systemd --files --new --name web2
cat container-web2.service

# 移动服务文件并启用自动启动：
mv container-web2.service /etc/systemd/system/
restorecon -RvF /etc/systemd/system/container-web2.service
systemctl enable container-web2.service --now
```

## 9. 非根用户使用 podman 容器

### 9.1 创建并登录普通用户

```bash
# 注意：必须通过终端/SSH 登录该用户 才能正常运行 rootless Podman。
useradd greg
echo 123 | passwd --stdin greg
ssh greg@localhost
```

### 9.2 普通用户拉取镜像

```bash
# 每个用户拥有自己独立的容器和镜像存储空间：
podman pull httpd
podman images
```

### 9.3 启动容器并绑定挂载

```bash
mkdir ~/web1
echo web1 > ~/web1/index.html

podman run -dt --name web1 -p 54321:80 -v /home/greg/web1:/usr/local/apache2/htdocs:Z httpd
podman ps
curl localhost:54321
```

### 9.4 使用 systemd 管理用户容

```bash
# 步骤 1：准备用户级 systemd 目录
mkdir -p ~/.config/systemd/user

# 步骤 2：生成服务文件并安装
podman generate systemd --new --files --name web1
mv container-web1.service ~/.config/systemd/user/
restorecon -RvF ~/.config/systemd/user/container-web1.service

# 步骤 3：启用用户持久 systemd 会话（避免用户退出后 systemd 不生效）
loginctl enable-linger greg

# 步骤 4：启用服务并启动容器
systemctl --user daemon-reload
systemctl --user enable container-web1.service --now
```

