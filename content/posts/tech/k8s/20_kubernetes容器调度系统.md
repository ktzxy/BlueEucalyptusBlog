+++
date = '2026-05-12T09:47:59+08:00'
draft = false
title = '20_Kubernetes容器调度系统'
slug = "81u74xgrcl"
description = "20_Kubernetes容器调度系统"
summary = "20_Kubernetes容器调度系统"
categories = ["📒📒Kubernetes"]
tags = ["📒Kubernetes"]
comments = true
series = ["Kubernetes学习系列"]
showSeries= true  
+++

# 20_Kubernetes容器调度系统

```shell
创建 Pod
   ↓
Scheduler（调度器）
   ↓
选 Node（筛选 + 打分）
   ↓
绑定 Pod → Node
   ↓
kubelet 启动容器
```

```shell
Scheduler 的职责 = 给 Pod 找一个最合适的 Node
```

## 调度流程

完整流程

```shell
1️⃣ 监听未调度 Pod
2️⃣ 过滤节点（Filter）
3️⃣ 节点评分（Score）
4️⃣ 选择最优 Node
5️⃣ 绑定（Bind）
```

### Filter（过滤阶段）

作用：不符合条件的 Node，直接淘汰

**常见过滤条件**

```shell
资源是否足够（CPU / 内存）
NodeSelector
NodeAffinity（硬规则）
Taint（NoSchedule）
端口冲突
Volume 是否可用
```

### Score（打分阶段）

作用：在“合格节点”中选最优

常见评分策略

```shell
资源最均衡
资源最空闲
Pod 分布均匀
亲和性匹配度
```

### Bind（绑定）

Pod → Node（最终确定）

## 调度控制手段

#### NodeSelector

> 作用：强制 Pod 只能去某些 Node

示例

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

 Node 必须有：

```yaml
kubectl label node node1 disktype=ssd
```

#### NodeAffinity

硬性（必须满足）

```yaml
requiredDuringSchedulingIgnoredDuringExecution
```

 软性（尽量满足）

```yaml
preferredDuringSchedulingIgnoredDuringExecution
```

示例

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```

#### PodAffinity / PodAntiAffinity

> 作用：控制 Pod 和 Pod 的关系

PodAffinity（靠近）：让 Pod 调度到“某些 Pod 附近”

PodAntiAffinity（远离）：让 Pod 分散到不同节点

示例（反亲和）

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: web
      topologyKey: kubernetes.io/hostname
```

同一个 Node 不能有多个 web Pod

#### Taint / Toleration

> 作用：Node 拒绝 Pod

给 Node 打污点

```
kubectl taint nodes node1 key=value:NoSchedule
```

Pod 容忍

```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

污点类型

| 类型             | 作用         |
| ---------------- | ------------ |
| NoSchedule       | 不允许调度   |
| PreferNoSchedule | 尽量不调度   |
| NoExecute        | 驱逐已有 Pod |

## 调度策略总结（核心对比）

| 机制            | 作用      | 阶段   |
| --------------- | --------- | ------ |
| NodeSelector    | 简单筛选  | Filter |
| NodeAffinity    | required  | Filter |
| NodeAffinity    | preferred | Score  |
| PodAffinity     | 靠近      | Score  |
| PodAntiAffinity | 分散      | Score  |
| Taint           | 拒绝      | Filter |
| Toleration      | 允许      | Filter |

## 场景

#### 场景1：GPU 节点

```shell
#只有 AI Pod 能上

Node：打 Taint（禁止）
Pod：加 Toleration（允许）
```

场景2：数据库高可用

```shell
#多个 Pod 不能在同一个 Node

PodAntiAffinity 
```

场景3：冷热数据分离

```shell
#SSD 节点跑数据库
#普通节点跑业务

NodeAffinity 
```

## 探针（Probe）+ 生命周期

#### 为什么需要探针？

如果没有探针：

```
Pod 挂了 → 还在接流量 ❌
```

有探针：

```
自动检测 + 自动恢复 ✔
```

#### 三种探针

- livenessProbe（存活探针）

  ```shell
  判断：Pod 是否“还活着”
  失败——>重启容器
  ```

- readinessProbe（就绪探针）

  ```shell
  判断：是否可以接流量
  失败——>从 Service 移除
  ```

- startupProbe（启动探针）

  ```shell
  解决：启动慢的问题
  Java / 大应用启动慢
  ```

#### 三者关系

```shell
startupProbe 成功前
→ 不执行 liveness / readiness

顺序：启动 → startup → readiness → liveness
```

#### 探针配置方式

##### 1.HTTP

```
httpGet:
  path: /
  port: 80
```

##### 2.TCP

```
tcpSocket:
  port: 3306
```

##### 3.命令（exec）

```
exec:
  command: ["cat", "/tmp/healthy"]
```

#### 完整示例

```yaml
containers:
- name: nginx
  image: nginx
  livenessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 10
    periodSeconds: 5

  readinessProbe:
    httpGet:
      path: /
      port: 80
    initialDelaySeconds: 5
    periodSeconds: 3
```

> initialDelaySeconds：延迟多久开始检查
>
> periodSeconds：检查间隔
>
> failureThreshold：失败多少次才算失败

#### 生命周期（Lifecycle）

##### 两个 Hook

postStart：容器启动后执行

preStop：容器关闭前执行

> Pod 被杀 → 请求丢失？？？
>
> 正确流程：
>
> 1️⃣ readiness=false（不再接流量）
> 2️⃣ 执行 preStop
> 3️⃣ 等待连接结束
> 4️⃣ 容器退出

```yml
lifecycle:
  preStop:
    exec:
      command: ["sleep", "10"]
```

搭配：terminationGracePeriodSeconds: 30

