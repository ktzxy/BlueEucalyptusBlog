+++
date = '2026-02-27T07:35:51+08:00'
draft = false
title = '13 Kubernetes集群安全机制'
slug = "f3dz6yr0vg"
description = "13 Kubernetes集群安全机制"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "13 Kubernetes集群安全机制"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# Kubernetes集群安全机制

## 概述

> Kubernetes 的安全机制由认证（Authentication）、鉴权（Authorization）和准入控制（Admission）三部分组成。所有请求必须通过 kube-apiserver 统一入口处理。

> - 认证用于确认请求者身份（如证书、Token）
> - 鉴权用于判断请求者是否有权限执行操作（如 RBAC）
> - 准入控制用于对请求进行最终校验或修改（如安全策略、资源限制）

```shell
客户端请求
   ↓
Authentication（认证）
   ↓
Authorization（鉴权）
   ↓
Admission（准入控制）
   ↓
etcd（持久化）

所有请求必须经过 kube-apiserver，依次完成身份确认、权限校验和请求合法性校验，最终才会生效。
```

kube-apiserver 是 Kubernetes 的统一入口，所有资源操作（kubectl / API / UI）都必须经过它，是集群的安全与控制核心。

- 访问过程中，需要证书、token、或者用户名和密码
- 如果访问pod需要serviceAccount

### 认证

K8s **只使用 HTTPS（6443端口）**，没有“明文 HTTP 认证”

常见认证方式

| 方式            | 说明                     |
| --------------- | ------------------------ |
| 证书认证（TLS） | 最常用（kubectl）        |
| Bearer Token    | ServiceAccount / API访问 |
| OIDC            | 接入企业身份系统（SSO）  |

### 鉴权

常见鉴权方式:

| 模式    | 说明         |
| ------- | ------------ |
| RBAC    | 主流（推荐） |
| ABAC    | 旧方案       |
| Webhook | 自定义       |
| Node    | 节点权限控制 |

### 准入控制

准入控制是在请求通过认证和鉴权之后，对资源进行“最后的校验和修改”。

两类控制器：Validating（校验）；Mutating（修改）

## RBAC介绍

> RBAC（Role-Based Access Control）是 Kubernetes 中基于角色的访问控制机制，通过定义角色（Role/ClusterRole）和权限规则，并将其绑定到用户或服务账号，从而实现对资源访问的精细化控制。
>

![image-20201118093949893](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291529045.webp)

k8s中有默认的几个角色

- role：只能作用在一个 namespace
- ClusterRole：可以是集群资源;也可以被 RoleBinding 限制在某个 namespace

角色绑定

- roleBinding：把 Role 或 ClusterRole 绑定到某个 namespace 下的用户
- ClusterRoleBinding：在整个集群范围内生效

主体

- user：用户
- group：用户组
- serviceAccount：服务账号

**权限规则**

权限由 rules 定义

```yaml
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

| 字段      | 说明     |
| --------- | -------- |
| apiGroups | 资源组   |
| resources | 资源类型 |
| verbs     | 操作类型 |

**完整实例：**

定义 Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

绑定用户

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: alice
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## RBAC实现鉴权

```yaml
1️⃣ 创建 namespace
2️⃣ 创建资源（Pod）
3️⃣ 定义 Role（权限规则）
4️⃣ 创建 RoleBinding（绑定用户）
5️⃣ 创建用户身份（证书 / kubeconfig）
6️⃣ 使用该用户访问验证权限
```

### 创建命名空间

我们可以首先查看已经存在的命名空间

```bash
kubectl get namespace
```

![image-20201118094516426](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291529046.webp)

然后我们创建一个自己的命名空间  roledemo

```bash
kubectl create ns roledemo
```

### 命名空间创建Pod

为什么要创建命名空间？因为如果不创建命名空间的话，默认是在default下

```bash
kubectl run nginx --image=nginx -n roledemo
```

### 创建角色

我们通过 rbac-role.yaml进行创建

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: roledemo
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

只允许：
✔ 查看 Pod
❌ 不能删除 / 创建 / 修改

```bash
# 创建
kubectl apply -f rbac-role.yaml
# 查看
kubectl get role -n roledemo
```

### 创建角色绑定

我们还是通过 role-rolebinding.yaml 的方式，来创建我们的角色绑定

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: roledemo
subjects:
- kind: User
  name: mary   # 用户名必须和证书一致
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

然后创建我们的角色绑定

```bash
# 创建角色绑定
kubectl apply -f rbac-rolebinding.yaml
# 查看角色绑定
kubectl get role, rolebinding -n roledemo
```

### 使用证书识别身份

准备 CSR 文件

```yaml
cat > mary-csr.json <<EOF
{
  "CN": "mary",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing"
    }
  ]
}
EOF
```

生成证书

```yaml
cfssl gencert \
  -ca=/etc/kubernetes/pki/ca.pem \
  -ca-key=/etc/kubernetes/pki/ca-key.pem \
  -config=/etc/kubernetes/pki/ca-config.json \
  -profile=kubernetes \
  mary-csr.json | cfssljson -bare mary
```

**生成 kubeconfig**

设置集群信息

```yaml
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.pem \
  --server=https://<API_SERVER>:6443 \
  --kubeconfig=mary.kubeconfig
```

设置用户

```yaml
kubectl config set-credentials mary \
  --client-certificate=mary.pem \
  --client-key=mary-key.pem \
  --kubeconfig=mary.kubeconfig
```

设置上下文

```yaml
kubectl config set-context mary@kubernetes \
  --cluster=kubernetes \
  --user=mary \
  --namespace=roledemo \
  --kubeconfig=mary.kubeconfig
```

使用该用户

```yaml
kubectl config use-context mary@kubernetes \
  --kubeconfig=mary.kubeconfig
```

### 验证权限

```bash
# 方法1
kubectl get pods --kubeconfig=mary.kubeconfig 
成功
kubectl get svc --kubeconfig=mary.kubeconfig 
拒绝

# 方法2
kubectl auth can-i get pods \
  --as=mary -n roledemo

kubectl auth can-i get svc \
  --as=mary -n roledemo
  
# 用get命令查看 pod 【有权限】
kubectl get pods -n roledemo
# 用get命令查看svc 【没权限】
kubectl get svc -n roledmeo
```

