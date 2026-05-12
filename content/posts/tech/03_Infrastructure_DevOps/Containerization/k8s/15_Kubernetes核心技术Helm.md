+++
date = '2025-03-17T01:27:55+08:00'
draft = false
title = '15 Kubernetes核心技术Helm'
slug = "zjoinuc8y7"
description = "15 Kubernetes核心技术Helm"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "15 Kubernetes核心技术Helm"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes核心技术Helm

Helm就是一个包管理工具【类似于npm】

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291530850.webp)

## 为什么引入Helm

在 Kubernetes 中，应用通常通过多个 YAML 文件（Deployment、Service、Ingress 等）进行部署。当系统演进为微服务架构后，一个应用往往由多个组件组成，每个组件都需要维护独立的 YAML 文件，导致部署复杂、依赖难以管理、版本控制困难。因此，引入 Helm 作为 Kubernetes 的包管理工具，用于统一管理、打包和发布应用。

### 传统 YAML 部署的核心问题

1. 文件碎片化严重

   ```shell
   一个服务：Deployment + Service + Ingress
   多个服务：几十甚至上百 YAML
   
   难维护 ❌
   难查找 ❌
   难统一管理 ❌
   ```

2. 无法描述依赖关系

   ```shell
   先部署 DB？
   再部署后端？
   最后部署前端？
   ```

3. 环境差异难处理

   ```shell
   dev / test / prod 配置不同
   YAML 需要复制多份修改
   ```

4. 缺乏版本管理

   ```shell
   YAML 需要复制多份修改
   
   无法快速回滚
   ```

5. 复用能力差

   ```shell
   多个项目用同一套组件（如 Redis）
   
   只能复制粘贴
   ```

### Helm 解决了什么问题

- 应用级打包
-  模板化
- 环境隔离
- 版本管理
- 依赖管理

## Helm介绍

> Helm 是 Kubernetes 的包管理工具，用于将多个资源对象（Deployment、Service、Ingress 等）打包为一个 Chart，通过模板化机制实现配置复用，并提供应用级的版本管理、依赖管理和一键部署能力，大幅提升 Kubernetes 应用的交付效率。
>

Helm有三个重要概念

- helm：一个命令行客户端工具，主要用于Kubernetes应用chart的创建、打包、发布和管理
- Chart：应用描述，一系列用于描述k8s资源相关文件的集合
- Release：基于Chart的部署实体，一个chart被Helm运行后将会生成对应的release，将在K8S中创建出真实的运行资源对象。也就是应用级别的版本管理
- Repository：用于发布和存储Chart的仓库

## Helm组件及架构

Helm采用客户端/服务端架构，有如下组件组成

- Helm CLI是Helm客户端，可以在本地执行
- Tiller是服务器端组件，在Kubernetes集群上运行，并管理Kubernetes应用程序
- Repository是Chart仓库，Helm客户端通过HTTP协议来访问仓库中Chart索引文件和压缩包

![image-20201119095458328](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291530851.webp)

## Helm v3变化

> Helm v3 采用纯客户端架构，移除了服务端组件 Tiller。Helm CLI 直接通过 Kubernetes API 与集群通信，所有操作基于当前 kubeconfig 权限执行，从而简化架构并提升安全性。

2019年11月13日，Helm团队发布了Helm v3的第一个稳定版本

该版本主要变化如下

- 架构变化

  - 最明显的变化是Tiller的删除
  - V3版本删除Tiller
  - relesase可以在不同命名空间重用
  - 支持将Chart推送至Docker 镜像仓库中
  - 使用JSONSchema验证 chart values

![image-20220815224715228](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291538105.webp)

### Helm v2 vs v3 对比

| 对比点   | Helm v2             | Helm v3               |
| -------- | ------------------- | --------------------- |
| 架构     | C/S（有 Tiller）    | 纯客户端              |
| 安全性   | 低（Tiller 高权限） | 高（基于 kubeconfig） |
| 复杂度   | 高                  | 低                    |
| 权限控制 | 难                  | 统一 RBAC             |

## helm配置

首先我们需要去 [官网下载](https://helm.sh/docs/intro/quickstart/)

- 第一步，[下载helm](https://github.com/helm/helm/releases)安装压缩文件，上传到linux系统中
- 第二步，解压helm压缩文件，把解压后的helm目录复制到 usr/bin 目录中
- 使用命令：helm

我们都知道yum需要配置yum源，那么helm就就要配置helm源

## helm仓库

添加仓库

```bash
helm repo add 仓库名  仓库地址 
```

例如

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 更新
helm repo update
# 查看仓库
helm repo list

# 搜索 Chart
helm search repo nginx

# 删除仓库
helm repo remove bitnami
```

安装

```yaml
# 下载
wget https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz

# 解压
tar -zxvf helm-v3.14.0-linux-amd64.tar.gz

# 移动
mv linux-amd64/helm /usr/local/bin/helm

# 验证
helm version
```

## helm基本命令

| 命令       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| create     | 创建一个chart并指定名称                                      |
| dependency | 管理chart依赖                                                |
| get        | 下载release，可用子命令：all、hooks、manifest、notes、value  |
| history    | 获取release历史                                              |
| install    | 安装一个chart                                                |
| list       | 列出release                                                  |
| package    | 将chart目录打包到chart存档文件中                             |
| pull       | 从远程仓库中下载chart并解压到本地； helm pull stable/mysql --untar |
| repo       | 添加、列出、移除、更新和索引chart仓库                        |
| rollback   | 从之前版本回滚                                               |
| search     | 根据关键字搜索Chart                                          |
| show       | 查看chart详细信息                                            |
| template   | 本地呈现模板                                                 |
| uninstall  | 卸载一个release                                              |
| upgrade    | 更新一个release                                              |
| version    | 查看helm客户端版本                                           |

```shell
# 安装
helm install my-nginx bitnami/nginx
# 查看
helm list
# 升级
helm upgrade my-nginx bitnami/nginx
# 回滚
helm rollback my-nginx 1
#卸载
helm uninstall my-nginx
```

## values.yaml（Helm 核心）

参数化配置

示例

```shell
helm install my-nginx bitnami/nginx \
  --set service.type=NodePort
  
或
helm install my-nginx bitnami/nginx -f values.yaml
```

## 使用helm快速部署应用

### Step 1：搜索应用

```bash
# 搜索 weave仓库
helm search repo weave
```

### Step 2：查看 Chart 详情

```shell
helm show values aliyun/weave-scope
```

用来找

- 端口
- 副本数
- 资源限制

### Step 3：安装（带参数）

```bash
helm install ui aliyun/weave-scope \
  --set service.type=NodePort
 
或使用 values.yaml
helm install ui aliyun/weave-scope -f values.yaml
```

### Step 4：查看状态

```bash
helm list
helm status ui
```

### Step 5：查看资源

```shell
kubectl get pods,svc
```

###  Step 6：访问服务

```shell
NodeIP + NodePort
```

Helm 已经部署了资源，但参数没配对（service.type 默认是 ClusterIP）

```shell
# 为什么不能直接 kubectl edit
kubectl edit svc ui-weave-scope
虽然能改成功，但有问题：
❌ Helm 不知道你改了什么
❌ 下次 helm upgrade 会被覆盖
❌ 配置不可追踪（不符合 DevOps）

正确做法:
方式1：直接升级
helm upgrade ui aliyun/weave-scope \
  --set service.type=NodePort

方式2：用 values.yaml（推荐）
① 导出默认配置
helm show values aliyun/weave-scope > values.yaml
② 修改
service:
  type: NodePort
③ 升级
helm upgrade ui aliyun/weave-scope -f values.yaml

方式3：强制修改
kubectl patch svc ui-weave-scope \
  -p '{"spec":{"type":"NodePort"}}'
```

### 如果自己创建Chart

使用命令，自己创建Chart

```bash
helm create mychart
```

创建完成后，我们就能看到在当前文件夹下，创建了一个 mychart目录

```shell
mychart/
  Chart.yaml        # 描述信息
  values.yaml       # 全局变量（核心）
  templates/        # 模板文件
  charts/           # 依赖
```

### 在templates文件夹创建两个文件

我们创建以下两个

- deployment.yaml
- service.yaml

我们可以通过下面命令创建出yaml文件

```bash
# 导出deployment.yaml
kubectl create deployment web1 --image=nginx --dry-run -o yaml > deployment.yaml
# 导出service.yaml 【可能需要创建 deployment，不然会报错】
kubectl expose deployment web1 --port=80 --target-port=80 --type=NodePort --dry-run -o yaml > service.yaml
```

### 安装mychart

执行命令创建

```bash
helm install web1 mychart
```

![image-20201118213120916](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291530861.webp)

### 应用升级

当我们修改了mychart中的东西后，就可以进行升级操作

```bash
helm upgrade web1 mychart
```

## chart模板使用

通过传递参数，动态渲染模板，yaml内容动态从传入参数生成

刚刚我们创建mychart的时候，看到有values.yaml文件，这个文件就是一些全局的变量，然后在templates中能取到变量的值，下面我们可以利用这个，来完成动态模板

- 在values.yaml定义变量和值
- 具体yaml文件，获取定义变量值
- yaml文件中大体有几个地方不同
  - image
  - tag
  - label
  - port
  - replicas

> Helm Chart 提供了一套默认配置（values.yaml），但在实际部署中通常需要根据环境进行调整。可以通过 `-f values.yaml` 或 `--set` 覆盖默认配置，其中 `--set` 优先级更高。推荐在生产环境中使用 values.yaml 文件统一管理配置，以保证可维护性和可复用性。

### 定义变量和值

在values.yaml定义变量和值

```shell
replicaCount: 2

image:
  repository: nginx
  tag: "1.25"

service:
  type: ClusterIP
  port: 80
```

### 获取变量和值

我们通过表达式形式 使用全局变量  `{{.Values.变量名称}} `

例如： `{{.Release.Name}}`

deployment.yaml（模板）

```shell
replicas: {{ .Values.replicaCount }}

containers:
- name: nginx
  image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

service.yaml

```shell
type: {{ .Values.service.type }}
port: {{ .Values.service.port }}
```

### 安装应用

在我们修改完上述的信息后，就可以尝试的创建应用了

```bash
# 带调试
helm install web2 mychart --dry-run --debug
```

![image-20201118214727058](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291530866.webp)