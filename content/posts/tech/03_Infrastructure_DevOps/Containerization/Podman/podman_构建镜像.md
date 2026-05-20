+++
date = '2026-05-20T09:25:26+08:00'
draft = false
title = 'Podman_构建镜像'
slug = "df4vvu2u1n"
description = "Podman_构建镜像"
summary = "Podman_构建镜像"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# 使用 Podman 和 Dockerfile 构建 Nginx 镜像: 多阶段构建实践

> 本指南将详细介绍如何使用 Podman（或 Docker）和一份 `Dockerfile`，通过**多阶段构建**技术来创建一个轻量、安全且生产就绪的 Nginx 容器镜像。

**核心理念：多阶段构建 (Multi-stage Build)**

多阶段构建是 `Dockerfile` 中的一项高级功能，它允许我们将一个构建过程分为多个独立的“阶段”。这么做最大的好处是：

- **减小最终镜像体积**：我们可以在一个临时的“构建阶段”中安装所有重量级的编译工具（如 `gcc`, `make`），在编译完成后，只将最终生成的可执行文件和其必要的**运行依赖**复制到一个全新的、干净的“运行阶段”中。编译工具本身不会被带入最终镜像。

- **提升安全性**：最终镜像不包含任何编译工具链，减少了潜在的攻击面。

## 1. `Dockerfile` 详解

以下是我们将使用的 `Dockerfile`。相比您的原始版本，我做了一个优化：使用 `RUN wget` 代替 `ADD`，这样构建镜像时就**无需预先手动下载 Nginx 源码包**，使整个构建过程更加自包含和自动化。

```dockerfile
# Dockerfile to build a custom Nginx image using a multi-stage build.

# ----------------------------------------------------------------
# 1. 构建阶段 (命名为 "builder")
# ----------------------------------------------------------------
FROM rockylinux:9 as builder

# 设置工作目录
WORKDIR /usr/local/src

# 定义 Nginx 版本为构建参数
ARG NGX_VER=1.22.0

# 安装编译 Nginx 所需的全部依赖
RUN dnf install -y gcc gcc-c++ make tar pcre-devel zlib-devel openssl-devel wget && \
    dnf clean all

# 下载、解压并准备 Nginx 源码
RUN wget https://nginx.org/download/nginx-${NGX_VER}.tar.gz && \
    tar -xzf nginx-${NGX_VER}.tar.gz

# 进入源码目录，执行配置、编译和安装
# 所有编译产物将被安装在 /usr/local/nginx-1.22.0
WORKDIR /usr/local/src/nginx-${NGX_VER}
RUN ./configure --prefix=/usr/local/nginx-${NGX_VER} && \
    make -j$(nproc) && make install


# ----------------------------------------------------------------
# 2. 运行阶段 (最终的镜像)
# ----------------------------------------------------------------
FROM rockylinux:9

# 定义 Nginx 版本，以便后续命令使用
ENV NGX_VER=1.22.0

# 为了安全，创建一个非 root 的专用用户来运行 Nginx
RUN useradd -r -M -s /sbin/nologin nginx

# 仅安装 Nginx 运行所必需的依赖，不包含任何编译工具
RUN dnf install -y pcre zlib openssl && \
    dnf clean all

# --from=builder: 从之前的 "builder" 阶段复制编译好的 Nginx 程序到当前镜像
COPY --from=builder /usr/local/nginx-${NGX_VER} /usr/local/nginx-${NGX_VER}

# 创建一个软链接，方便路径管理
RUN ln -s /usr/local/nginx-${NGX_VER} /usr/local/nginx

# 将 Nginx 目录的所有权赋予我们创建的 nginx 用户
RUN chown -R nginx:nginx /usr/local/nginx-${NGX_VER}

# 切换工作目录
WORKDIR /usr/local/nginx

# 声明容器对外暴露的端口
EXPOSE 80
EXPOSE 443

# 指定后续命令的执行用户为非 root 的 nginx 用户
USER nginx

# 定义容器启动时的主命令 (入口点)
ENTRYPOINT ["/usr/local/nginx/sbin/nginx"]

# 定义主命令的默认参数
# -g 'daemon off;' 是让 Nginx 在前台运行的关键，这在容器中是必需的
CMD ["-g", "daemon off;"]
```

## 2. 构建与运行镜像

### 2.1 准备构建环境

- 创建一个空目录，例如 `nginx-build`。

- 进入该目录，并将上面的 `Dockerfile` 内容保存为一个名为 `Dockerfile` 的文件。

```bash
mkdir nginx-build
cd nginx-build

cat > Dockerfile << 'END'
# ... (粘贴上面的完整 Dockerfile 内容) ...
END
```

此时，您的 `nginx-build` 目录下应该只有一个 `Dockerfile` 文件。

### 2.2 构建镜像 (`podman build`)

在 `nginx-build` 目录下，执行以下命令来构建镜像：

```bash
# -t my-custom-nginx:1.22.0 为镜像打上一个易于识别的标签 (tag)
# . 表示使用当前目录的 Dockerfile
podman build -t my-custom-nginx:1.22.0 .
```

Podman 会自动执行 `Dockerfile` 中的所有步骤，包括下载源码、编译和构建最终镜像。这个过程可能需要几分钟。

### 2.3 运行容器 (`podman run`)

镜像构建成功后，使用以下命令来启动一个基于该镜像的容器：

```bash
# -d: 后台运行容器
# --name nginx-server: 为容器命名
# -p 8080:80: 将主机的 8080 端口映射到容器的 80 端口
podman run -d --name nginx-server -p 8080:80 my-custom-nginx:1.22.0
```

### 2.4 验证运行

```bash
# 查看容器是否正在运行
podman ps

# 查看容器日志
podman logs nginx-server

# 从主机访问 Nginx 默认欢迎页
curl http://localhost:8080
# 您应该能看到 Nginx 的欢迎 HTML 内容
```

## 3. (可选) 添加自定义配置

在实际应用中，您通常需要使用自己的 Nginx 配置文件。

- **准备配置文件**：在 `nginx-build` 目录下，创建一个名为 `my-nginx.conf` 的文件，并写入您的 Nginx 配置。

- **修改 `Dockerfile`**：在 `Dockerfile` 的**运行阶段 (Final Stage)**，`COPY --from=builder` 命令之后，添加一行 `COPY` 指令：

```dockerfile
...
# 拷贝构建好的 nginx 到运行环境
COPY --from=builder /usr/local/nginx-${NGX_VER} /usr/local/nginx-${NGX_VER}

# 新增：拷贝自定义配置文件，覆盖默认配置
COPY my-nginx.conf /usr/local/nginx/conf/nginx.conf

# 创建一个软链接，方便路径管理
RUN ln -s /usr/local/nginx-${NGX_VER} /usr/local/nginx
...
```

- **重新构建镜像**并运行，您的自定义配置就会生效。
