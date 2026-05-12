+++
date = '2026-02-01T07:29:36+08:00'
draft = false
title = '7 Kubernetes集群YAML文件详解'
slug = "0lv02an8ph"
description = "7 Kubernetes集群YAML文件详解"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "7 Kubernetes集群YAML文件详解"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes集群YAML文件详解

## 什么是 YAML

YAML 是 Kubernetes 中用于描述资源“期望状态”的声明式配置文件

本质：写 YAML → 提交给 API Server → K8s 自动创建/维护资源

### YAML 基础语法

* 使用 **空格缩进（禁止Tab）**
* 层级对齐必须一致
* #` 表示注释` 
* `---` 表示多个资源分隔

### YAML 支持的数据结构

#### 对象

键值对的集合，又称为映射(mapping) / 哈希（hashes） / 字典（dictionary）

```yaml
name: nginx
replicas: 3
```

#### 数组

```bash
containers:
- name: nginx
  image: nginx
- name: busybox
  image: busybox
```

#### 行内写法

```shell
env: {name: ENV, value: prod}
```

## Kubernetes YAML 核心结构

**标准模板**

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.25
```

### 属性说明

在一个YAML文件的控制器定义中，有很多属性名称

|  属性名称  |    介绍    |
| :--------: | :--------: |
| apiVersion |  API版本   |
|    kind    |  资源类型  |
|  metadata  | 资源元数据 |
|    spec    |  资源规格  |
|  replicas  |  副本数量  |
|  selector  | 标签选择器 |
|  template  |  Pod模板   |
|  metadata  | Pod元数据  |
|    spec    |  Pod规格   |
| containers |  容器配置  |

**完整示例**

```		bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 3

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
        image: nginx:1.25

        ports:
        - containerPort: 80
		
		# 资源控制 防止资源抢占
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
		
		# env（环境变量）
        env:
        - name: ENV
          value: prod
		
		# probes（健康检查 ⭐） liveness（存活检测）
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
		
		# readiness（就绪检测） 失败 → 不加入流量
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5

      restartPolicy: Always
```

## YAML 编写方式

#### 方式一：dry-run生成

```
kubectl create deployment nginx \
  --image=nginx \
  --dry-run=client -o yaml > nginx.yaml
```

#### 方式二：从已有资源导出

```
kubectl get deploy nginx -o yaml > nginx.yaml
```

#### 方式三：结合 explain（强烈推荐 ⭐）

```
kubectl explain deployment.spec
kubectl explain pod.spec.containers
```

# YAML 背后的“控制逻辑”

## 一、核心架构

**控制链路**

```shell
kubectl apply
    ↓
API Server
    ↓
etcd（存储期望状态）
    ↓
Controller（控制器监听变化）
    ↓
Scheduler（调度）
    ↓
Kubelet（执行）
```

> YAML 只负责“写入期望”，真正干活的是 Controller

## 二、最核心机制：Reconcile（调谐循环）

```shell
核心流程
1️⃣ 读取期望状态（Desired State）
2️⃣ 获取当前状态（Actual State）
3️⃣ 对比差异（Diff）
4️⃣ 执行动作（Create / Delete / Update）
```

## 三、用 Deployment 举例

```shell
replicas: 3

#实际发生了什么？
Step 1：写入 etcd
Deployment: replicas=3
Step 2：Deployment Controller 监听
当前没有 Pod
Step 3：创建 ReplicaSet
Deployment → ReplicaSet
Step 4：ReplicaSet 创建 Pod
ReplicaSet → Pod（3个）
Step 5：Scheduler 分配节点
Pod → Node1 / Node2 / Node3
Step 6：Kubelet 启动容器
拉镜像 → 启动容器
```

## 四、为什么 YAML 改了不会“直接删 Pod”？

```shell
改 YAML 不是直接替换Pod，而是创建一个新的ReplicaSet

原：ReplicaSet v1（nginx:1.20） → 3个Pod
改：image: nginx:1.25
实际；
创建 ReplicaSet v2（nginx:1.25）
    ↓
逐步增加 v2 Pod
    ↓
逐步删除 v1 Pod

这就是滚动更新（Rolling Update）
```

## 五、控制器本质（Controller Pattern）

每个 Controller 都在干这件事：`监听资源变化 → 对齐状态`

常见 Controller：

| 控制器      | 作用           |
| ----------- | -------------- |
| Deployment  | 管理无状态应用 |
| ReplicaSet  | 保证副本数     |
| StatefulSet | 管理有状态     |
| DaemonSet   | 每个节点一个   |
| Job         | 一次性任务     |

## 六、事件驱动 + Watch机制

Controller 不是轮询，而是：`监听 API Server 变化（Watch）`,一旦发生变化：`YAML变更 / Pod挂了 / Node挂了`，Controller立刻响应。

## 七、Pod 挂了为什么自动恢复？

当某个Pod被删除后，ReplicaSet 发现当前 Pod = 2而期望 Pod = 3，于是创建一个新的 Pod，这就是自愈能力（Self-healing）

## 八、Service 为什么能负载均衡？

Service 本身不处理流量，实际是：`kube-proxy + iptables/ipvs `通过 `Endpoints（Pod列表）` 实现转发。

## 九、核心抽象总结

#### Kubernetes 做了三层抽象：

资源定义（YAML）——>描述“我要什么”

控制器（Controller）——>保证“变成这样”

执行层（Node）——>真正运行容器





















