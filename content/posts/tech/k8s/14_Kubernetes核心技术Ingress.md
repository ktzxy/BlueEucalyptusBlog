+++
date = '2024-12-26T00:53:12+08:00'
draft = false
title = '14 Kubernetes核心技术Ingress'
slug = "7kw46ku3pk"
description = "14 Kubernetes核心技术Ingress"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "14 Kubernetes核心技术Ingress"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes核心技术Ingress

## 前言

在 Kubernetes 中，早期对外暴露服务通常通过 Service 的 NodePort 或 LoadBalancer 实现，但这些方式在实际生产中存在一定局限性，例如端口管理复杂、访问方式不友好等。因此，引入了 Ingress 作为统一的七层流量入口，实现基于域名和路径的流量转发。

## Ingress 是什么

Ingress 是 Kubernetes 中用于管理外部访问服务的资源对象，通过定义域名和路径规则，将外部流量转发到内部 Service。Ingress 本身只是规则定义，实际的流量转发由 Ingress Controller 实现，常用于实现统一入口、域名路由和 HTTPS 支持。

## Ingress 能解决什么问题

- 基于域名访问
- 基于路径路由
- HTTPS 支持
- 统一入口

## Ingress Controller

### 常见实现

- Nginx Ingress（最常用）
- Traefik
- HAProxy
- 云厂商 LB（阿里云、AWS）

## Ingress和Pod关系

Ingress 并不直接关联 Pod，而是通过 Service 进行间接关联。Service 负责维护 Pod 列表并提供负载均衡能力，Ingress 作为统一入口，根据域名或路径规则将请求转发到对应的 Service，再由 Service 将流量分发到具体的 Pod 实例。

![image-20201118102637839](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291529675.webp)

## Ingress工作流程

在 Kubernetes 中，Ingress 通过定义域名和路径规则，将外部请求统一接入到集群内部，并根据规则转发到对应的 Service，再由 Service 将流量分发到后端 Pod。

**实际请求流程**

```shell
用户访问 a.com / b.com
   ↓
DNS 解析 → Ingress Controller IP
   ↓
Ingress Controller（如 Nginx）
   ↓
匹配 Ingress 规则（host/path）
   ↓
转发到对应 Service
   ↓
Service 负载均衡
   ↓
Pod 处理请求
```

## 场景

```yaml
a.com → 服务A
b.com → 服务B
```

Ingress 规则示例

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host
spec:
  rules:
  - host: a.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-a
            port:
              number: 80
  - host: b.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-b
            port:
              number: 80
```

需要注意，ingress不是内置的组件，需要我们单独的安装

## 使用Ingress

步骤如下所示

- 部署ingress Controller【需要下载官方的】
- 创建ingress规则【对哪个Pod、名称空间配置规则】

1.部署 Ingress Controller

推荐使用官方 Nginx Ingress

官方安装（推荐）

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

```yaml
# 查看
kubectl get pods -n ingress-nginx
```

部署业务应用

### 创建Nginx Pod

创建一个nginx应用，然后对外暴露端口

```bash
# 创建pod
kubectl create deployment web --image=nginx
# 查看
kubectl get pods
```

对外暴露端口

```bash
kubectl expose deployment web --port=80 --target-port=80 --type=ClusterIP
```

### 创建ingress规则文件

创建ingress规则文件，ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: web.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
```

```yaml
kubectl apply -f ingress.yaml
```

### 添加域名访问规则

在windows 的 hosts文件，添加域名访问规则【因为我们没有域名解析，所以只能这样做】

```shell
C:\Windows\System32\drivers\etc\hosts
```

## 补充

### Ingress Controller 暴露方式

| 方式         | 说明           |
| ------------ | -------------- |
| NodePort     | 本地测试       |
| LoadBalancer | 云环境（推荐） |
| hostNetwork  | 测试/特殊场景  |

### HTTPS

```yaml
tls:
- hosts:
  - web.local
  secretName: tls-secret
```

