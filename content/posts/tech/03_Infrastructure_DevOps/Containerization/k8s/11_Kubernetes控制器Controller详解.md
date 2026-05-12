+++
date = '2025-01-11T21:48:10+08:00'
draft = false
title = '11 Kubernetes控制器Controller详解'
slug = "2uqd5az8zj"
description = "11 Kubernetes控制器Controller详解"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "11 Kubernetes控制器Controller详解"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes控制器Controller详解

## Statefulset

StatefulSet 是 Kubernetes 中用于管理有状态应用的控制器，能够为每个 Pod 提供稳定的网络标识、持久化存储以及有序的部署与扩缩容能力。

### 无状态应用

- Pod 无差别（可替换）
- 无需固定名称
- 无需固定存储
- 无启动顺序
- 可随意扩缩容

### 有状态应用

- Pod 有唯一身份（不可替换）
- 有固定名称（pod-0）
- 有独立存储（PVC绑定）
- 有启动顺序
- 有数据依赖关系

### 适用场景

> StatefulSet 更适用于需要“稳定身份 + 持久数据”的应用场景。相比 Deployment 的“无状态、可替换”模型，StatefulSet 更接近传统虚拟机的“有状态、不可随意替换”的特性，但通过 Kubernetes 的调度机制仍具备一定的高可用能力。

- 数据库：MySQL / PostgreSQL
- 分布式系统：Zookeeper / etcd
- 消息队列：Kafka
- 有状态服务：Redis（主从/集群）

### 部署有状态应用

无头service， ClusterIp：none

> Headless Service 是什么
>
> 不分配 ClusterIP，不做负载均衡，直接返回后端 Pod 的真实 IP。

这里就需要使用 StatefulSet部署有状态应用

![image-20201117202950336](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527792.webp)

![image-20201117203130867](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527793.webp)

然后通过查看pod，能否发现每个pod都有唯一的名称

![image-20201117203217016](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527794.webp)

然后我们在查看service，发现是无头的service

![image-20201117203245641](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527795.webp)

> 在部署有状态应用时，通常需要结合 Headless Service（clusterIP: None）与 StatefulSet 使用。Headless Service 不提供负载均衡，而是为每个 Pod 提供独立的 DNS 解析，使客户端可以直接访问指定 Pod。

> StatefulSet 会为每个 Pod 分配固定名称（如 pod-0、pod-1），并基于 Headless Service 生成稳定的域名，例如：
>
> ```
> pod-0.service-name.namespace.svc.cluster.local
> ```
>
> 这种机制保证了每个 Pod 拥有稳定的网络身份，从而满足数据库、分布式系统等有状态应用对节点身份的要求。

每个pod有唯一的主机名，并且有唯一的域名

- 格式：`<pod-name>.<service-name>.<namespace>.svc.cluster.local`
- 举例：`nginx-0.nginx.default.svc.cluster.local`

## DaemonSet

DaemonSet 是 Kubernetes 中的一种控制器，用于在每个节点上运行一个 Pod 实例，常用于部署日志采集、监控、网络等节点级系统服务。它会随着节点的加入自动创建 Pod，随着节点的移除自动删除 Pod，从而保证每个节点始终运行一个指定的服务实例。

#### 常见场景

| 类型     | 组件               |
| -------- | ------------------ |
| 日志采集 | Filebeat / Fluentd |
| 监控     | Node Exporter      |
| 网络     | Calico / Flannel   |
| 存储     | Ceph / CSI插件     |

#### 核心特征

- 每个节点一个 Pod
- 新节点自动部署
- 节点删除自动清理
- 可指定节点运行

例子：在每个node节点安装数据采集工具

![image-20201117204430836](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527796.webp)

采集 Node 上的日志文件 → 发送到 ES / Kafka

进入某个 Pod里面，进入

```bash
kubectl exec -it ds-test-cbk6v bash
```

通过该命令后，我们就能看到我们容器内映射的日志文件

![image-20201117204912838](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527799.webp)

## DaemonSet vs Deployment

| 特性     | Deployment | DaemonSet  |
| -------- | ---------- | ---------- |
| Pod数量  | 用户指定   | 每节点1个  |
| 调度方式 | 随机       | 固定每节点 |
| 应用类型 | 业务服务   | 系统服务   |

## Job和CronJob

Job 和 CronJob 是 Kubernetes 中用于执行批处理任务的控制器。Job 用于执行一次性任务，而 CronJob 用于按照时间规则周期性执行任务。

### Job（一次性任务）

#### Job 的核心特点

- 任务执行完就结束（不像 Deployment 一直运行）
- 支持重试机制
- 可以并行执行多个任务

#### 核心参数

1. completions（完成次数）

   ```bash
   completions: 3
   # 任务成功执行 3 次才算完成
   ```

2. parallelism（并行数）

   ```bash
   parallelism: 2
   # 同时最多运行 2 个 Pod
   ```

3. backoffLimit（失败重试次数）

   ```bash
   backoffLimit: 4
   # 最多重试 4 次
   ```

**标准写法:**

```shell
apiVersion: batch/v1
kind: Job
metadata:
  name: demo-job
spec:
  completions: 1
  parallelism: 1
  template:
    spec:
      containers:
      - name: job
        image: busybox
        command: ["echo", "hello job"]
      restartPolicy: Never
```

使用下面命令，能够看到目前已经存在的Job

```bash
kubectl get jobs
```

![image-20201117205948374](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527801.webp)

在计算完成后，通过命令查看，能够发现该任务已经完成

![image-20201117210031725](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527802.webp)

我们可以通过查看日志，查看到一次性任务的结果

```bash
kubectl logs pi-qpqff
```

![image-20201117210110343](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527803.webp)

### CronJob

> 按照 cron 表达式周期性创建 Job
>

核心字段：schedule

```shell
schedule: "*/1 * * * *"

# 每 1 分钟执行一次
```

#### 常见 cron 表达式

| 表达式        | 含义   |
| ------------- | ------ |
| `*/1 * * * *` | 每分钟 |
| `0 * * * *`   | 每小时 |
| `0 0 * * *`   | 每天   |
| `0 0 * * 0`   | 每周   |

**示例：**

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: demo-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron
            image: busybox
            command: ["echo", "hello cronjob"]
          restartPolicy: Never
```

我们首先用上述的配置文件，创建一个定时任务

```bash
kubectl apply -f cronjob.yaml
```

创建完成后，我们就可以通过下面命令查看定时任务

```bash
kubectl get cronjobs
```

![image-20201117210611783](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527805.webp)

我们可以通过日志进行查看

```bash
kubectl logs hello-1599100140-wkn79
```

![image-20201117210722556](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527806.webp)

然后每次执行，就会多出一个 pod

![image-20201117210751068](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291527807.webp)

## 常用点

#### 历史记录控制

```yaml
successfulJobsHistoryLimit: 3
failedJobsHistoryLimit: 1
```

#### 并发策略（CronJob）

```yaml
concurrencyPolicy: Forbid
```

| 策略    | 说明       |
| ------- | ---------- |
| Allow   | 允许并发   |
| Forbid  | 禁止并发   |
| Replace | 替换旧任务 |

#### 任务超时

```yaml
activeDeadlineSeconds: 100
```

## 删除svc 和 statefulset

使用下面命令，可以删除我们添加的svc 和 statefulset

```bash
kubectl delete svc web
kubectl delete statefulset web

# 会删掉所有，不建议
kubectl delete statefulset --all
```

## Replication Controller

> ReplicationController（RC）是 Kubernetes 早期用于保证 Pod 副本数量的控制器，通过控制循环机制确保实际运行的 Pod 数量与期望一致。由于其在标签选择能力上的局限性，RC 已被 ReplicaSet（RS）取代。

```yaml
1.selector 太弱
selector:
  app: nginx
不能支持复杂匹配

2.不支持集合表达式
RC做不到app in (nginx, apache)

3.无法支持现代部署方式
滚动升级（弱）
版本管理（几乎没有）
```

> ReplicaSet 是 RC 的增强版本，支持更灵活的标签选择器，并同样用于维持 Pod 副本数量。在实际使用中，ReplicaSet 通常不直接使用，而是作为 Deployment 的底层组件，由 Deployment 统一管理，实现滚动升级与版本控制。

支持更复杂的selector

```yaml
selector:
  matchLabels:
    app: nginx
  matchExpressions:
  - key: app
    operator: In
    values:
    - nginx
    - apache
```

