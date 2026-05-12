+++
date = '2025-01-21T22:36:08+08:00'
draft = false
title = '5 Kubeadm和二进制方式对比'
slug = "f6fxremm4l"
description = "5 Kubeadm和二进制方式对比"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "5 Kubeadm和二进制方式对比"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubeadm和二进制方式对比

## Kubeadm方式搭建K8S集群

#### 1.环境准备

- 准备 3 台机器（1 Master + 2 Node）
- 系统建议：Ubuntu / Rocky / AlmaLinux

#### 2.系统初始化（关键步骤）

```shell
# 关闭 swap
swapoff -a

# 关闭 selinux
setenforce 0

# 开启桥接
modprobe br_netfilter
```

#### 3.安装 containerd

> 为什么用 containerd 而不是 Docker？两者区别
>
> Kubernetes 已经不再直接支持 Docker（从 v1.24 开始）
>
> Kubernetes 不再直接使用 Docker，而是使用 containerd 作为容器运行时，因为 containerd 更轻量、架构更简单，并且符合 CRI 标准。

```shell
yum install -y containerd
systemctl enable containerd
systemctl start containerd
```

#### 4.安装 kubeadm / kubelet / kubectl

```shell
yum install -y kubelet-1.28.2 kubeadm-1.28.2 kubectl-1.28.2
systemctl enable kubelet
```

#### 5.初始化 Master

```shell
kubeadm init \
  --apiserver-advertise-address=192.168.1.100 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.28.2 \
  --pod-network-cidr=192.168.0.0/16
```

> --apiserver-advertise-address （控制平面对外通信IP）  Master 节点的内网IP
>
> --image-repository （镜像仓库地址）
>
> --kubernetes-version （指定版本）
>
> --pod-network-cidr （Pod 网段） 这个必须和网络插件匹配！
>
> | 插件    | 默认 CIDR          |
> | ------- | ------------------ |
> | Flannel | 10.244.0.0/16      |
> | Calico  | 192.168.0.0/16     |
> | Cilium  | 10.0.0.0/8（常见） |
>
> 为什么必须初始化？不做会怎样？
>
> 本质：把一个普通Linux机器变成“控制平面节点”
>
> ### ① 生成证书（超级重要）
>
> - API Server 证书
> - kubelet 证书
>
> ### ② 启动核心组件（以Pod形式）
>
> - API Server
> - Scheduler
> - Controller Manager
> - etcd
>
> ### ③ 初始化 etcd
>
> 👉存储集群数据
>
> ### ④ 生成 join token
>
> 👉 给 Node 用来加入集群
>
> ### ⑤ 配置 kubeconfig
>
> 👉 让 kubectl 能操作集群
>
> 如果某一步失败：
>
> | 步骤失败   | 后果             |
> | ---------- | ---------------- |
> | 证书       | 所有组件通信失败 |
> | etcd       | 集群数据无法存储 |
> | API Server | 集群不可访问     |
> | kubelet    | Pod起不来        |

#### 6.配置 kubectl

```shell
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

#### 7.安装网络插件（CNI）

```shell
kubectl apply -f calico.yaml
```

> Calico / Cilium / Flannel 区别
>
> 本质：它们都是 CNI（网络插件）
>
> 作用：让 Pod 之间可以通信
>
> | 特性     | Flannel  | Calico       | Cilium |
> | -------- | -------- | ------------ | ------ |
> | 难度     | 简单     | 中等         | 高     |
> | 性能     | 一般     | 高           | 非常高 |
> | 网络策略 | ❌ 不支持 | ✔ 支持       | ✔ 强大 |
> | 原理     | overlay  | BGP/iptables | eBPF   |
> | 企业使用 | 少       | ⭐主流        | ⭐高端  |

#### 8.Node 加入集群

```shell
kubeadm join <master-ip>:6443 --token xxx --discovery-token-ca-cert-hash xxx
```

#### 9.验证集群

```shell
kubectl get nodes
kubectl get pods -A
```

## 二进制方式搭建K8S集群

> 手动搭建：证书体系 + etcd + 控制平面 + 节点组件

#### 1.环境初始化

- 关闭 swap
- 配置主机名
- 配置 hosts
- 时间同步

#### 2.生成证书

> 使用 cfssl：CA → 签发所有组件证书

包括：

- etcd
- apiserver
- kubelet
- kubectl

#### 3.部署 etcd 集群

> 3节点 etcd + TLS通信

```shell
systemctl start etcd
```

#### 4.部署 Control Plane

##### kube-apiserver（最核心）

必须指定：

- etcd 地址
- TLS证书
- service CIDR

##### controller-manager

- 控制循环
- 需要 kubeconfig

##### scheduler

- 调度 Pod

#### 5.部署 Node

### 安装：

- containerd
- kubelet
- kube-proxy

kubelet 加入流程

```shell
kubelet → 请求证书 → CSR → 手动批准 → Node Ready
```

批准：

```shell
kubectl get csr
kubectl certificate approve xxx
```

#### 6.安装 CNI

Calico / Flannel

#### 7.验证集群

```shell
kubectl get nodes
kubectl get pods -A
```

## kubeadm vs 二进制部署 对比

| 对比维度            | kubeadm 部署             | 二进制部署          |
| ------------------- | ------------------------ | ------------------- |
| **本质**            | 自动化部署工具           | 手动部署所有组件    |
| **复杂度**          | ⭐ 简单（几条命令）       | ❗ 非常复杂          |
| **学习价值**        | 入门/生产实用            | 深入理解原理        |
| **部署速度**        | 快（分钟级）             | 慢（小时级）        |
| **证书管理**        | 自动生成 ✔               | 手动生成（cfssl）❗  |
| **组件配置**        | 自动生成 ✔               | 手动编写所有配置 ❗  |
| **etcd部署**        | 自动 ✔                   | 手动搭建集群 ❗      |
| **控制平面组件**    | 自动以 Pod 运行 ✔        | 手动 systemd 管理 ❗ |
| **kubelet加入集群** | 自动 join ✔              | 手动 CSR 审批 ❗     |
| **出错排查**        | 简单（标准化）           | 困难（自由度高）    |
| **可控性**          | 中等                     | ⭐ 极高              |
| **灵活性**          | 一般                     | ⭐ 极强              |
| **升级维护**        | 简单（kubeadm upgrade）✔ | ❗ 非常复杂          |
| **生产环境使用**    | ⭐⭐⭐⭐⭐ 主流               | ⭐ 少数（定制场景）  |
| **适合人群**        | 运维 / DevOps            | 架构师 / 深度学习   |
| **是否推荐**        | ✅ 强烈推荐               | ❗ 学习用            |

kubeadm 本质上是对二进制部署流程的封装，它自动完成证书生成、组件配置和集群初始化，而二进制部署则需要手动实现这些步骤，因此二进制方式更灵活但运维成本更高。

# 故障模拟解决

**通用问题解决框架：**

① 节点（Node）是否正常
② 组件（Control Plane / kubelet）是否正常
③ 网络（CNI / DNS / Service）是否正常
④ 资源对象（Pod / Deployment）是否异常

```shell
kubectl get nodes
kubectl get pods -A
kubectl describe xxx
kubectl logs xxx
```

## 场景1：kubeadm init 卡住

### 拉镜像失败（最常见）

```
卡在：pulling images

排查：kubeadm config images pull
```

### 2.containerd / CRI 问题

```
卡在：waiting for kubelet

#排查：
systemctl status containerd
systemctl status kubelet

常见原因：
- containerd 没启动
- cgroup 不一致

# 修复
containerd config default | tee /etc/containerd/config.toml
systemctl restart containerd
```

### 3.端口被占用

```
netstat -lntp | grep 6443

# 常见端口：6443（API Server）;2379（etcd）
```

### 4. swap 没关

```
free -h

# 如果有 swap：
swapoff -a
```

## 场景2：Node NotReady（超高频）

------

## 一步判断

```
kubectl get nodes

kubectl describe node <node>
```

## 常见原因

### 1. CNI没装（最常见）

```
NotReady
Reason: NetworkPluginNotReady

解决：kubectl apply -f calico.yaml
```

### 2.kubelet异常

```
systemctl status kubelet
journalctl -u kubelet -f

常见问题：配置错误；证书问题
```

### 3.containerd异常

```
systemctl status containerd
```

### 4.节点资源不足

```bash
#  CPU / 内存不够
top
free -h
```

### 5.时间不同步

```bash
# TLS认证失败
timedatectl
```

## 场景3：Pod Pending

### 常见原因

### 1.没有可用节点（调度失败）

```bash
0/2 nodes are available

# 原因：Node NotReady;资源不足
```

### 2.资源不足

```
Insufficient CPU / memory
# 解决：减少资源请求;增加节点
```

### 3.PVC 未绑定（存储问题）

```bash
PersistentVolumeClaim is not bound

# 排查：kubectl get pvc
```

### 4.网络问题（CNI）

Pod一直 Pending 或 ContainerCreating

### 5.镜像拉取失败（有时是 Pending → ImagePullBackOff）

```bash
kubectl describe pod

# 原因：私有仓库没认证;网络问题
```


