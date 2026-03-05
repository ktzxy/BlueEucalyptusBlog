+++
date = '2026-02-14T05:06:25+08:00'
draft = false
title = 'docker安装'
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
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装 Docker
# 安装依赖
sudo apt install -y ca-certificates curl gnupg

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 添加仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

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

### docker-compose

离线安装

[所有版本预览Releases · docker/compose (github.com)](https://github.com/docker/compose/releases/)

```
#最好是进入到/usr/local/bin/目录下安装，我第一次在根目录下安装，报错docker-compose: command not found，第二次在上述目录下安装可以成功
#将文件上传到linux后，重命名
mv docker-compose-linux-x86_64.docker-compose-linux-x86_64 docker-compose


#添加可执行权限
chmod +x /usr/local/bin/docker-compose

#测试
docker-compose version
```

在线安装

```shell
# 下载 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 设置执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

**说明**：

- `$(uname -s)`：获取操作系统类型（如Linux）
- `$(uname -m)`：获取硬件架构（如x86_64）
- `sudo chmod +x`：给文件添加执行权限
- `docker-compose --version`：验证Docker Compose是否正确安装

**常见错误及解决方法**：

- 错误："Permission denied" - 确保使用sudo执行命令
- 错误："command not found" - 检查docker-compose是否在PATH中，或尝试使用完整路径`/usr/local/bin/docker-compose --version`
