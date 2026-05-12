+++
date = '2026-05-12T09:46:54+08:00'
draft = false
title = '21_Kubernetes HPA'
slug = "24cewk6vmg"
description = "21_Kubernetes HPA"
summary = "21_Kubernetes HPA"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
comments = true
series = ["Kubernetes学习系列"]
showSeries= true  
+++

# 21_Kubernetes HPA

## HPA 是什么

```shell
HPA = 根据指标（CPU/内存等）自动调整 Pod 副本数
```

## HPA 工作链路

```shell
Pod 运行
   ↓
metrics-server 收集指标（CPU/内存）
   ↓
HPA 定期获取指标
   ↓
计算是否需要扩/缩容
   ↓
修改 Deployment 副本数
```

## HPA 核心计算逻辑

## 官方核心公式

![image-20260430112505224](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260430112505224.png)

```shell
# 举例
当前副本数：2
当前 CPU：80%
目标 CPU：40%

计算：2 × (80 / 40) = 4 
扩容到 4 个 Pod 
```

## HPA 支持哪些指标

#### Resource Metrics

```shell
CPU ✔（最稳定）
Memory ✔
```

#### Custom Metrics

```shell
QPS
请求数
队列长度

需要：Prometheus Adapter
```

#### External Metrics

```shell
Kafka lag
云监控指标
```

## HPA 前置条件

```yaml
# 必备组件
metrics-server 
# 检查
kubectl get deployment metrics-server -n kube-system

# 下载
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## HPA YAML 实战配置

```shell
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

> scaleTargetRef: 目标对象，控制哪个 Deployment
>
> 副本范围：
>
> ​ minReplicas: 2
> ​ maxReplicas: 10
>
> averageUtilization: 50  指标规则，平均 CPU 使用率目标 = 50%

## HPA 扩缩容策略

```shell
behavior:
  scaleUp:
    stabilizationWindowSeconds: 0
  scaleDown:
    stabilizationWindowSeconds: 300
```

> 缩容前等待 5 分钟
>
> 防止流量抖动导致频繁扩缩容

## HPA + 资源管理

```yaml
# HPA 依赖 requests
CPU 利用率计算：实际使用 / requests

```

举例

```
requests = 500m
实际使用 = 500m
利用率：100%
requests = 1000m
实际使用 = 500m
利用率：50%
requests 设置不合理 → HPA 完全失效
```


