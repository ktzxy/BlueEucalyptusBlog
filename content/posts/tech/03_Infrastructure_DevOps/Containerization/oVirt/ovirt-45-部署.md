+++
date = '2026-05-20T08:50:33+08:00'
draft = false
title = 'oVirt 4.5 部署'
slug = "aico9ti3jx"
description = "oVirt 4.5 部署"
summary = "oVirt 4.5 部署"
categories = ["📒虚拟化"]
tags = ["oVirt"]
comments = true
+++

# oVirt 4.5 部署

## ovirt engine

### 1. 设置主机名

```bash
hostnamectl set-hostname ovirt-engine
```

### 2. 设置 hosts 文件

```bash
cat > /etc/hosts << 'END'
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.0.100 ovirt-engine
10.0.0.101 ovirt-node-1
10.0.0.102 ovirt-node-2
10.0.0.201 ovirt-storage-1
END
```

### 3. 配置 yum 源

```bash
mkdir /etc/yum.repos.d/bak
mv /etc/yum.repos.d/CentOS-Stream-* /etc/yum.repos.d/bak/
cp /etc/yum.repos.d/bak/* /etc/yum.repos.d/

# 更换清华源
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/$contentdir|baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
         
yum makecache

yum install -y centos-release-ovirt45
```

### 4. 加载必要的模块

```bash
yum module -y enable javapackages-tools
yum module -y enable pki-deps
yum module -y enable postgresql:12
yum module -y enable mod_auth_openidc:2.3
yum module -y enable nodejs:14
```

### 5. 安装 ovirt engine

```bash
yum install ovirt-engine -y

yum install python3.11-pip -y
pip3.11 install netaddr -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 6. 初始化配置

```bash
engine-setup
```

![image.png](https://typora.fanyops.com/documents/2023/1007/1681047490459-0bfab7ac-1868-4415-ac7d-88a4dc31a887.png)

![image.png](https://typora.fanyops.com/documents/2023/1007/1680857458562-be9352eb-c0c4-4c8f-9f4a-ecf076202283.png)

![image.png](https://typora.fanyops.com/documents/2023/1007/1680857478150-49e9bc7e-0b5c-4b0e-9107-177979588716.png)

### 6. web 登陆

![image.png](https://typora.fanyops.com/documents/2023/1007/1680857010295-1538b51d-835f-4c24-b810-5904007b41d7.png)

![image.png](https://typora.fanyops.com/documents/2023/1007/1680857022395-1f1a6263-1e02-4ca5-8e3c-4a40d1fe1d06.png)

![image.png](https://typora.fanyops.com/documents/2023/1007/1680869547602-abf5739e-23aa-40ef-8695-a8e49e78f4ec.png)

## ovirt node

### 1. 配置 node yum 源

```bash
# 从 engine 节点安装 centos-release-ovirt45
ssh ovirt-node-1 "yum install -y centos-release-ovirt45 -y"
ssh ovirt-node-2 "yum install -y centos-release-ovirt45 -y"
```

![image-20231008140440983](https://typora.fanyops.com/documents/2023/1008/image-20231008140440983.png)

### 2. 修改 node 节点的主机名

```bash
ssh ovirt-node-1 'hostnamectl set-hostname ovirt-node-1'
ssh ovirt-node-2 'hostnamectl set-hostname ovirt-node-2'
```

![image-20231008114301088](https://typora.fanyops.com/documents/2023/1008/image-20231008114301088.png)

### 3. 在 engine 添加主机

#### 3.1 配置 node1 节点

```bash
# 计算 > 主机 > 新建
```

![image-20231008113503192](https://typora.fanyops.com/documents/2023/1008/image-20231008113503192.png)

```bash
# 名称: ovirt-node-1
# 主机名/IP: ovirt-node-1
# SSH 公共密钥: 复制公钥到 node-1 节点
```

![image-20231008113958813](https://typora.fanyops.com/documents/2023/1008/image-20231008113958813.png)

#### 3.2 配置 node2 节点

```bash
# 计算 > 主机 > 新建

# 名称: ovirt-node-1
# 主机名/IP: ovirt-node-1
# SSH 公共密钥: 复制公钥到 node-1 节点
```

### 4. 安装进度排查

```bash
tail -f /var/log/ovirt-engine/engine.log
```

![image-20231008140705718](https://typora.fanyops.com/documents/2023/1008/image-20231008140705718.png)

![image-20231008140811101](https://typora.fanyops.com/documents/2023/1008/image-20231008140811101.png)

### 5. 安装成功

![eve.png](https://typora.fanyops.com/documents/2023/1008/1682165886476-71d4efa0-56fb-4897-b867-9ec2c94a17ea.png)

