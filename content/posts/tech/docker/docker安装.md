+++
date = '2026-02-14T05:06:25+08:00'
draft = false
title = 'docker安装'
slug = "ivzblwvqy0"
description = "docker安装"
categories = ["📒docker"]
tags = ["docker","docker安装"]
summary = "docker安装"
+++

# 脚本安装
```
#!/bin/bash

# 安装 Docker
echo "安装 Docker..."
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce

# 添加当前用户到 Docker 用户组
sudo usermod -aG docker $USER

# 安装 Docker Compose
echo "安装 Docker Compose..."
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 显示安装信息
echo "Docker 和 Docker Compose 安装完成。"
docker --version
docker-compose --version
```
# 手动安装

### 安装（以ubuntu）

```shell
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# step 2: 信任 Docker 的 GPG 公钥
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Step 3: 写入软件源信息
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
# Step 4: 安装Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]


# 启动 Docker 并设置开机自启
sudo systemctl start docker
sudo systemctl enable docker

# 将当前用户加入 docker 组（避免使用 sudo）
sudo usermod -aG docker $USER
newgrp docker
```

**说明**：

- `sudo apt update && sudo apt upgrade -y`：更新系统软件包列表和升级已安装的软件包
- `sudo apt install -y ...`：安装Docker所需的基础依赖包
- `curl -fsSL ... | sudo gpg --dearmor ...`：添加Docker官方GPG密钥以验证下载的包
- `echo "deb [arch=...] ..." | sudo tee ...`：添加Docker官方APT仓库
- `sudo usermod -aG docker $USER`：将当前用户添加到docker组，这样就无需每次都使用sudo运行docker命令
- `newgrp docker`：激活新的用户组权限，无需重新登录

**常见错误及解决方法**：

- 错误："Could not get lock /var/lib/dpkg/lock-frontend" - 表示其他程序正在使用apt，等待或杀死相关进程
- 错误："The following signatures couldn't be verified because the public key is not available" - 重新添加GPG密钥
- 错误："Cannot connect to the Docker daemon" - 检查Docker服务是否启动：`sudo systemctl status docker`

### 安装（以centos）

```shell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils

# Step 2: 添加软件源信息
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Step 3: 安装Docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Step 4: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

[Docker CE镜像-Docker CE镜像下载安装-开源镜像站-阿里云](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.7d911b11DLIc7d)

### 设置开机自动启动

```shell
#systemctl enable docker
```

### 查看docker服务状态

```shell
#systemctl status docker
```

### 查看docker具体信息

```shell
#docker info
```
