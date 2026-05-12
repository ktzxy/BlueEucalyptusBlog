+++
date = '2025-03-01T04:48:32+08:00'
draft = false
title = '6 Kubernetes集群管理工具kubectl'
slug = "ge9oetn7no"
description = "6 Kubernetes集群管理工具kubectl"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "6 Kubernetes集群管理工具kubectl"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes集群管理工具kubectl

## 概述

#### kubectl 是什么

kubectl 是 Kubernetes 的命令行工具，本质上是一个 **API 客户端**，用于通过调用 API Server 来管理集群资源。

#### 工作原理

```shell
kubectl → kubeconfig → API Server → etcd
```

- kubectl 读取 kubeconfig
- 与 API Server 通信（HTTP/HTTPS）
- 所有操作最终作用于 etcd

## 命令格式

命令格式如下

```bash
kubectl [command] [type] [name] [flags]
```

### 参数说明：

- command：操作类型
  - get / create / apply / delete / describe
- type：资源类型（通常小写）
  - pod / deployment / service
  - 支持缩写（po / deploy / svc）
- **name**：资源名称（区分大小写）
- **flags**：附加参数，例如，可用 -s 或者 -server参数指定Kubernetes API server的地址和端口

例如：

```bash
kubectl get pod pod1
kubectl get pods pod1
kubectl get po pod1
```

## 常见命令

####  一、资源操作

**声明式**

```shell
kubectl apply -f xxx.yaml
kubectl delete -f xxx.yaml
```

**命令式**

```shell
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
```

#### 二、查看资源

```bash
kubectl get pods
kubectl get pods -o wide
kubectl get pods -A
kubectl get pod <name> -o yaml
```

- `-o wide`：看IP/节点
- `-o yaml`：看完整配置

#### 三、排障命令

```bash
# describe
kubectl describe pod <pod>

# logs
kubectl logs <pod>
kubectl logs -f <pod>

多容器：kubectl logs <pod> -c <container>

# exec(进入容器排查)
kubectl exec -it <pod> -- /bin/bash
```

#### 四、网络调试

```shell
# 排查 Service 是否正常
kubectl port-forward pod/<pod> 8080:80
kubectl get svc
kubectl get endpoints
```

#### 五、扩缩容 / 发布

```shell
# 滚动发布核心命令
kubectl scale deployment nginx --replicas=3
kubectl rollout status deployment nginx
kubectl rollout undo deployment nginx
```

#### 六、节点管理

```bash
kubectl get nodes
kubectl describe node <node>
```

下线节点：

```shell
kubectl cordon <node>
kubectl drain <node> --ignore-daemonsets
kubectl uncordon <node>
```

#### 七、权限与配置

```shell
kubectl config view
kubectl config use-context xxx
kubectl auth can-i create pods
```

#### 补充

```shell
#1.watch（实时观察）(看 Pod 启动过程;看滚动发布)
kubectl get pods -w


#2.-n（命名空间）(排查系统组件（CoreDNS、CNI）)
kubectl get pods -n kube-system

#3.--all-namespaces (全局排查（最常用之一）)
kubectl get pods -A

#4.--show-labels (排查 Service 匹配问题)
kubectl get pods --show-labels

#5.label（动态打标签） (Service 选中 Pod;环境区分)
kubectl label pod nginx env=prod

#6.annotate（注解） (存储额外信息（监控/运维）)
kubectl annotate pod nginx description="test"

#7.patch（非常重要 ⭐）(快速修改资源（不改 YAML）;patch 是局部更新，apply 是声明式全量管理)
kubectl patch deployment nginx -p '{"spec":{"replicas":3}}'

#8.api-resources (查看支持的资源类型)
kubectl api-resources

#9.api-versions (版本兼容问题)
kubectl api-versions

#10.explain（神器） (写 YAML 不会时)
kubectl explain deployment
kubectl explain deployment.spec

#高级
#1.debug（新版本神器 ⭐） （Pod 内排查网络问题）
kubectl debug pod/nginx -it --image=busybox

#2.events（关键排查）（看集群异常；Pod 调度失败）
kubectl get events

#3.top（监控） （CPU / 内存排查）
kubectl top pod
kubectl top node

#4.cp（文件拷贝）
kubectl cp pod:/tmp/a.txt ./a.txt

#5.port-forward（开发神器） （本地访问 Pod）
kubectl port-forward pod/nginx 8080:80

#6.config（多集群管理） （切换环境（dev / prod））
kubectl config get-contexts
kubectl config use-context xxx

#7.auth can-i（权限排查）（RBAC问题排查）
kubectl auth can-i create pods

#8.drain 高级用法 （节点维护；滚动升级）
kubectl drain node1 --ignore-daemonsets --delete-emptydir-data

#9.rollout history（版本回溯） （查看历史版本）
kubectl rollout history deployment nginx

#10.diff（高级用法 ⭐） （发布前对比变更）
kubectl diff -f deployment.yaml
```

