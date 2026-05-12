+++
date = '2026-01-28T19:23:37+08:00'
draft = false
title = '16 Kubernetes持久化存储'
slug = "pijate4jl0"
description = "16 Kubernetes持久化存储"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "16 Kubernetes持久化存储"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes持久化存储

## 前言

在 Kubernetes 中：Pod 是“短暂的”

表现：

- Pod 重建 → IP 变
- Pod 删除 → 数据丢失

 使用持久化存储（Persistent Storage）——>Pod 重建后，数据仍然存在

## 存储发展路径

### 阶段1：直接挂载（最原始）

```
Pod → NFS（写死 IP）
```

问题：

- 强耦合
- 不可迁移
- 不易维护

### 阶段2：PV / PVC（标准方案）

```
Pod → PVC → PV → 存储（NFS/云盘）
```

✔ 解耦 ✔ 标准 ✔ 可维护

### 阶段3：StorageClass（企业级）

```
Pod → PVC → StorageClass → 自动创建 PV
```

✔ 自动化 ✔ 动态供给

## NFS 持久化存储

### Step 1：搭建 NFS 服务端

使用命令安装nfs

```bash
yum install -y nfs-utils
```

首先创建存放数据的目录

```bash
mkdir -p /data/nfs
```

配置共享目录

```bash
# 打开文件
vim /etc/exports
# 添加如下内容
/data/nfs *(rw,sync,no_root_squash,no_subtree_check)
```

启动服务

```shell
systemctl start nfs-server
systemctl enable nfs-server
```

生效配置

```shell
exportfs -r
```

### Step 2：Node 节点准备

在所有 Node 上执行：

```bash
yum install -y nfs-utils
```

### Step 3：直接挂载 NFS

nfs-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: web-data
  volumes:
  - name: web-data
    nfs:
      server: 192.168.44.134
      path: /data/nfs
```

启动nfs服务端

下面我们回到nfs服务端，启动我们的nfs服务

```bash
# 启动服务
systemctl start nfs
# 或者使用以下命令进行启动
service nfs-server start
```

![image-20201119082047766](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291531221.webp)

通过这个方式，就挂载到了刚刚我们的nfs数据节点下的 /data/nfs 目录

最后就变成了：  /usr/share/nginx/html    ->  192.168.44.134/data/nfs   内容是对应的

我们通过这个 yaml文件，创建一个pod

```bash
kubectl apply -f nfs-nginx.yaml
```

创建完成后，我们也可以查看日志

```bash
kubectl describe pod nginx-dep1
```

![image-20201119083444454](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291531223.webp)

可以看到，我们的pod已经成功创建出来了，同时下图也是出于Running状态

![image-20201119083514247](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291531224.webp)

下面我们就可以进行测试了，比如现在nfs服务节点上添加数据，然后在看数据是否存在 pod中

```bash
# 进入pod中查看
kubectl exec -it nginx-dep1 bash

# NFS 数据已经挂载 
#但这种方式：不推荐（强耦合）
```

![image-20201119095847548](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291531225.webp)

## PV和PVC

在 Kubernetes 中，如果将存储（如 NFS 的 IP 和路径）直接写在 Pod 配置中，会导致应用与底层存储强耦合，不利于维护和迁移。为了解决这个问题，Kubernetes 引入了 PV（PersistentVolume）和 PVC（PersistentVolumeClaim）机制，实现存储资源的抽象与解耦。

PV（PersistentVolume）

> PV 是对底层存储资源（如 NFS、云盘等）的抽象，由集群管理员创建和维护，属于存储资源的提供者。

PVC（PersistentVolumeClaim）

> PVC 是用户对存储资源的请求，描述了所需的容量、访问模式等信息。Pod 通过 PVC 使用存储，而不需要关心底层实现细节。

### Step 4：创建 PV

 pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /data/nfs
    server: 192.168.44.134
  persistentVolumeReclaimPolicy: Retain
```

```shell
# 创建
kubectl apply -f pv.yaml
```

### Step 5：创建 PVC

pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

```shell
# 创建
kubectl apply -f pvc.yaml

# 查看绑定
kubectl get pv,pvc
```

### Step 6：Pod 使用 PVC

deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-pvc
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: web-data
      volumes:
      - name: web-data
        persistentVolumeClaim:
          claimName: nfs-pvc
```

```yaml
# 创建
kubectl apply -f deployment.yaml

# 测试
kubectl exec -it <pod-name> bash
```

## 访问模式

| 模式 | 说明              |
| ---- | ----------------- |
| RWO  | 单节点读写        |
| ROX  | 多节点只读        |
| RWX  | 多节点读写（NFS） |

## 回收策略

| 策略    | 说明           |
| ------- | -------------- |
| Retain  | 保留数据       |
| Delete  | 自动删除       |
| Recycle | 清空（已废弃） |

## 绑定方式

自动匹配（容量 + 访问模式）

## StorageClass

> StorageClass = “自动创建 PV 的模板”

不用每次都手动创建 PV，StorageClass（动态供给）

`Pod → PVC → StorageClass → 自动创建 PV`

关键组件：Provisioner

常见：

```yaml
kubernetes.io/no-provisioner（本地）
nfs-client（NFS）
ceph / aws-ebs / aliyun-disk（云盘）
```

#### StorageClass YAML

 示例（NFS）

```shell
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: example.com/nfs
reclaimPolicy: Retain
volumeBindingMode: Immediate
```

> provisioner：谁来创建存储
>
> reclaimPolicy：Retain（保留数据） Delete（自动删除）
>
> volumeBindingMode：Immediate（立即绑定） WaitForFirstConsumer（等 Pod 调度）

#### PVC 使用 StorageClass

PVC 示例

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: nfs-storage
  resources:
    requests:
      storage: 1Gi
```



