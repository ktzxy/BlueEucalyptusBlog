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

Secret的主要作用就是加密数据，然后存在etcd里面，让Pod容器以挂载Volume方式进行访问

场景：用户名 和 密码进行加密

一般场景的是对某个字符串进行base64编码 进行加密

```bash
echo -n 'admin' | base64
```

![image-20201117212037668](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528634.webp)

### 变量形式挂载到Pod

- 创建secret加密数据的yaml文件    secret.yaml

![image-20201117212124476](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528635.webp)

然后使用下面命令创建一个pod

```bash
kubectl create -f secret.yaml
```

通过get命令查看

```bash
kubectl get pods
```

![image-20201118084010980](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528636.webp)

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

### 数据卷形式挂载

首先我们创建一个 secret-val.yaml 文件

![image-20201118084321590](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528638.webp)

然后创建我们的 Pod

```bash
# 根据配置创建容器
kubectl apply -f secret-val.yaml
# 进入容器
kubectl exec -it mypod bash
# 查看
ls /etc/foo
```

![image-20201118084707478](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528639.webp)

## ConfigMap

ConfigMap作用是存储不加密的数据到etcd中，让Pod以变量或数据卷Volume挂载到容器中

应用场景：配置文件

### 创建配置文件

首先我们需要创建一个配置文件 `redis.properties`

```bash
redis.port=127.0.0.1
redis.port=6379
redis.password=123456
```

### 创建ConfigMap

我们使用命令创建configmap

```bash
kubectl create configmap redis-config --from-file=redis.properties
```

然后查看详细信息

```bash
kubectl describe cm redis-config
```

![image-20201118085503534](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528640.webp)

### Volume数据卷形式挂载

首先我们需要创建一个 `cm.yaml`

![image-20201118085847424](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291528641.webp)

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