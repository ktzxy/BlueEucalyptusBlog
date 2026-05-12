+++
date = '2026-01-24T20:10:13+08:00'
draft = false
title = '12 Kubernetes配置管理'
slug = "z2xptdy6ns"
description = "12 Kubernetes配置管理"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "12 Kubernetes配置管理"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes配置管理

## Secret

> Secret 是 Kubernetes 中用于存储敏感信息（如密码、Token、证书）的资源对象，支持以环境变量或文件挂载的方式注入到 Pod 中使用。
>

场景：用户名 和 密码进行加密

一般场景的是对某个字符串进行base64编码 进行加密

```bash
echo -n 'admin' | base64

# 还原
echo YWRtaW4= | base64 -d
```

![image-20201117212037668](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528634.webp)

### 变量形式挂载到Pod

示例 YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=
  password: MTIzNDU2
```

然后使用下面命令创建一个pod

```bash
kubectl create -f secret.yaml
```

```yaml
env:
- name: SECRET_USERNAME
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: username
- name: SECRET_PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: password
```

通过get命令查看

```bash
kubectl get pods
```

然后我们通过下面的命令，进入到我们的容器内部

```bash
kubectl exec -it mypod bash
```

然后我们就可以输出我们的值，这就是以变量的形式挂载到我们的容器中

```bash
# 输出用户
echo $SECRET_USERNAME
# 输出密码
echo $SECRET_PASSWORD
```

![image-20201118084137942](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528637.webp)

最后如果我们要删除这个Pod，就可以使用这个命令

```bash
kubectl delete -f secret-val.yaml
```

==缺点：环境变量容易被泄露（ps / 日志）==

### 数据卷形式挂载

首先我们创建一个 secret-val.yaml 文件

![image-20201118084321590](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528638.webp)

```yaml
volumeMounts:
- name: secret-vol
  mountPath: /etc/secret

volumes:
- name: secret-vol
  secret:
    secretName: my-secret
```

然后创建我们的 Pod

```bash
# 创建 Secret
kubectl apply -f secret.yaml

# 查看
kubectl get secret

# 创建 Pod
kubectl apply -f pod.yaml

# 进入容器
kubectl exec -it mypod -- bash

# 查看
ls /etc/foo
```

![image-20201118084707478](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528639.webp)

**优点：**

- 更安全
- 支持权限控制
- 可动态更新（部分场景）

**总结：**

> Secret 是 Kubernetes 中用于存储敏感信息的资源对象，支持以环境变量或文件挂载的方式注入到 Pod 中使用。其数据通常以 Base64 编码存储，但安全性主要依赖于 etcd 加密和访问控制机制。在生产环境中，推荐使用 Volume 挂载方式以提高安全性。

## ConfigMap

> ConfigMap 是 Kubernetes 中用于存储非敏感配置数据的资源对象，可以以环境变量或文件的形式注入到 Pod 中使用，实现配置与应用解耦。
>

应用场景：配置文件

### 创建配置文件

首先我们需要创建一个配置文件 `redis.properties`

```bash
redis.host=127.0.0.1
redis.port=6379
redis.password=123456
```

### 创建ConfigMap

我们使用命令创建configmap

```bash
# 方式一 从文件创建
kubectl create configmap redis-config --from-file=redis.properties

# 方式二 从字面值创建
kubectl create configmap redis-config \
  --from-literal=redis.host=127.0.0.1 \
  --from-literal=redis.port=6379
  
# 方式三 使用 YAML（推荐生产）
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
data:
  redis.host: "127.0.0.1"
  redis.port: "6379"
  redis.password: "123456"
```

然后查看详细信息

```bash
kubectl describe cm redis-config
```

![image-20201118085503534](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528640.webp)

### Volume数据卷形式挂载

首先我们需要创建一个 `cm.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod
spec:
  containers:
  - name: test
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: redis-config
```



然后使用该yaml创建我们的pod

```bash
# 创建
kubectl apply -f cm.yaml
# 查看
kubectl get pods
```

![image-20201118090634869](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528642.webp)

最后我们通过命令就可以查看结果输出了

```bash
kubectl logs mypod
```

![image-20201118090712780](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528643.webp)

**优点：**

- 支持热更新（部分场景）
- 结构清晰
- 适合配置文件

### 以变量的形式挂载Pod

首先我们也有一个 myconfig.yaml文件，声明变量信息，然后以configmap创建

![image-20201118090911260](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528644.webp)

然后我们就可以创建我们的配置文件

```bash
# 创建pod
kubectl apply -f myconfig.yaml
# 获取
kubectl get cm
```

![image-20201118091042287](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528645.webp)

然后我们创建完该pod后，我们就需要在创建一个  config-var.yaml 来使用我们的配置信息

![image-20201118091249520](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528646.webp)

最后我们查看输出

```bash
kubectl logs mypod
```

![image-20201118091448252](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528647.webp)

## ConfigMap vs Secret

| 特性     | ConfigMap | Secret    |
| -------- | --------- | --------- |
| 数据类型 | 普通配置  | 敏感数据  |
| 是否加密 | ❌         | ✔（可选） |
| 用途     | 配置文件  | 密码/证书 |