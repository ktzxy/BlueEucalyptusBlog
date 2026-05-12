+++
date = '2026-02-24T01:43:11+08:00'
draft = false
title = '10 Kubernetes核心技术Service'
slug = "agbyh9mwpz"
description = "10 Kubernetes核心技术Service"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "10 Kubernetes核心技术Service"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes核心技术Service

Service 是 Kubernetes 中的网络抽象层，用于将动态变化的 Pod 集合映射为一个稳定的访问入口，并通过节点上的 kube-proxy 生成转发规则，实现对后端实例的透明负载均衡与服务发现。

## Service存在的意义

### 服务发现（Service Discovery）

Pod 的 IP 是动态变化的，无法作为稳定访问入口。Service 通过提供一个固定的虚拟 IP（ClusterIP）和 DNS 名称，使客户端无需感知后端 Pod 的变化，即可访问服务，实现服务发现能力。

### 负载均衡（Load Balancing）

Service 会将客户端请求通过 kube-proxy 转发到后端多个 Pod，实现流量分发，从而提升系统的可用性与扩展性。

Service 的负载均衡由 kube-proxy 实现，在 iptables 模式下基于规则转发（近似随机），在 IPVS 模式下支持多种调度算法（如轮询、最少连接等）。

## Pod和Service的关系

Pod 和 Service 之间通过 label 和 selector 建立关联。Service 根据 selector 动态选择一组 Pod，并生成对应的 Endpoints 列表作为后端实例。客户端通过访问 Service 的虚拟 IP（ClusterIP），由 kube-proxy 将请求转发到后端 Pod，从而实现服务发现与负载均衡。

![image-20201117094142491](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291525478.webp)

## Service常用类型

Service常用类型有三种

- ClusterIp：默认类型，仅在集群内部访问，用于微服务之间通信
- NodePort：在每个节点上开放一个端口（30000-32767），可通过 NodeIP:NodePort 从集群外访问
- LoadBalancer：依赖云厂商提供外部负载均衡器，对外暴露一个公网 IP

### 举例

我们可以导出一个文件 包含service的配置信息

```bash
kubectl expose deployment web --port=80 --target-port=80 --dry-run=client -o yaml > service.yaml
```

service.yaml 如下所示

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web
status:
  loadBalancer: {}
```

如果我们没有做设置的话，默认使用的是第一种方式 ClusterIp，也就是只能在集群内部使用，我们可以添加一个type字段，用来设置我们的service类型

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web
  type: NodePort
status:
  loadBalancer: {}
```

修改完命令后，我们使用创建一个Service

```bash
kubectl apply -f service.yaml
```

## NodePort 与 LoadBalancer 的访问方式

在 Kubernetes 集群中，节点（Node）通常部署在内网环境，外部用户无法直接访问 Pod。因此，需要通过 Service 将应用对外暴露。

### NodePort 方式（基础对外访问）

当 Service 类型为 NodePort 时，Kubernetes 会在每个节点上开放一个端口（30000-32767），外部可以通过以下方式访问服务：

```bash
NodeIP:NodePort
```

但在实际环境中，节点通常位于内网，因此需要额外的入口来接入外部流量，例如：

- 在一台具备公网访问能力的服务器上部署 Nginx
- 通过反向代理将请求转发到集群节点的 NodePort
- 手动维护后端节点列表

👉 这种方式本质是：

```bash
外部流量 → Nginx → NodePort → Service → Pod
```

### LoadBalancer 方式（云原生推荐）

当 Service 类型为 LoadBalancer 时，Kubernetes 会调用云厂商（如阿里云 / AWS）的 API，自动创建一个外部负载均衡器，并分配公网 IP。

该负载均衡器会将流量转发到集群内部的 NodePort，再由 Service 分发到后端 Pod。

👉 实际访问链路：

```bash
公网IP
   ↓
云负载均衡（SLB / ELB）
   ↓
NodePort
   ↓
Service
   ↓
Pod
```

## kube-proxy

> kube-proxy = 每个 Node 上的“转发代理”

监听 Service / Endpoints 变化
→ 写入转发规则（iptables / ipvs）

当你访问 Service IP 时，比如：10.96.0.10:80（ClusterIP），实际命中 iptables / ipvs 规则
→ 被转发到某个 Pod IP

过程：

```
访问 VIP
   ↓
内核规则匹配
   ↓
随机选择 Pod
   ↓
转发过去
```

#### kube-proxy 三种模式

- userspace（已淘汰）

- iptables（默认）

- IPVS（推荐生产）

  ````
  基于 Linux IPVS（四层负载均衡）
  高性能 
  支持多种算法 
  
  支持策略：
  轮询（RR）
  最少连接（LC）
  加权轮询（WRR）
  ````

Pod 扩容后为什么自动负载均衡？

Endpoints 更新
→ kube-proxy 更新规则
→ 自动加入转发

Pod 挂了为什么不会访问？

被移出 Endpoints

为什么 Service IP 不会变？

是虚拟IP（不是 Pod）
