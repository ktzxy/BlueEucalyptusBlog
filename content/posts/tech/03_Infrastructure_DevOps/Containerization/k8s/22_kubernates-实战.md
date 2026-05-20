+++
date = '2026-05-12T09:47:16+08:00'
draft = false
title = '22_Kubernates 实战'
slug = "s3vzauc9bl"
description = "22_Kubernates 实战"
summary = "22_Kubernates 实战"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
comments = true
series = ["Kubernetes学习系列"]
showSeries= true  
+++

# 22_Kubernates 实战

![ChatGPT Image 2026430171301](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/ChatGPT%20Image%202026430171301.png)

# 前期（基础掌握｜能跑起来）

## 🚀 集群 & 基础

------

### 实战一：部署 K8s 集群（1 master + 2 node）

```shell
kubectl create deployment web --image=nginx

kubectl expose deployment web --port=80 --target-port=80 --type=NodePort

kubectl get pod,svc
```

### 实战二：安装网络插件（Calico）

✅ 预期

```
kubectl get pods -n kube-system
```

💣 故障

- Pod 卡在 ContainerCreating
- Pod 无法通信

问题

```shell
[root@k8smaster ~]# kubectl get pods -n kube-flannel -o wide
NAME                    READY   STATUS             RESTARTS      AGE     IP              NODE        NOMINATED NODE   READINESS GATES
kube-flannel-ds-4qksn   0/1     CrashLoopBackOff   4 (37s ago)   2m20s   192.168.31.53   k8snode2    <none>           <none>
kube-flannel-ds-bps76   0/1     CrashLoopBackOff   4 (49s ago)   2m20s   192.168.31.52   k8snode1    <none>           <none>
kube-flannel-ds-pm5k7   0/1     CrashLoopBackOff   4 (37s ago)   2m20s   192.168.31.51   k8smaster   <none>           <none>
[root@k8smaster ~]# kubectl logs -n kube-flannel kube-flannel-ds-4qksn
......
I0504 07:25:27.158332       1 main.go:255] Installing signal handlers
I0504 07:25:27.158760       1 main.go:534] Found network config - Backend type: vxlan
E0504 07:25:27.158824       1 main.go:289] Failed to check br_netfilter: stat /proc/sys/net/bridge/bridge-nf-call-iptables: no such file or directory


#永久加载模块
echo "br_netfilter" > /etc/modules-load.d/k8s.conf
```

## 🚀 工作负载

### 实战三：Deployment 部署 nginx

### 实战四：副本扩缩容

```shell
kubectl scale deployment web --replicas=3

kubectl get pods -o wide
```

### 实战五：滚动升级 + 回滚

```shell
kubectl set image deployment web nginx=nginx:1.25

kubectl rollout status deployment web

kubectl rollout undo deployment web
```

## 🚀 Service

### 实战六：ClusterIP 访问

```shell
kubectl expose deployment web \
  --port=80 \
  --target-port=80 \
  --type=ClusterIP
```

### 实战七：NodePort 对外暴露

💣 故障

- NodePort 无法访问
- Pod 正常但访问失败

kubectl edit svc web

```
type: NodePort
```

或者直接重建：

```
kubectl expose deployment web \
  --type=NodePort \
  --port=80 \
  --target-port=80
```

```
kubectl get svc
```

```
http://NodeIP:NodePort
```

## 🚀 Ingress

### 实战八：安装 Ingress Controller

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/cloud/deploy.yaml

ImagePullBackOff

vi /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
  endpoint = ["https://registry-1.docker.io"]

kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/cloud/deploy.yaml

kubectl get pods -n ingress-nginx

kubectl get svc -n ingress-nginx
```

```shell
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/cloud/deploy.yaml

#一键替换
sed -i 's#registry.k8s.io/ingress-nginx/controller:v1.11.2#harbor.ktzxy.top/k8s/controller:v1.11.2#g' deploy.yaml

sed -i 's#registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.3#harbor.ktzxy.top/k8s/kube-webhook-certgen:v1.4.3#g' deploy.yaml

#验证（harbor仓库公开不用配置）
=======================
kubectl create namespace ingress-nginx
kubectl create secret docker-registry harbor-secret \
  --docker-server=harbor.ktzxy.top \
  --docker-username=admin \
  --docker-password=你的密码 \
  --docker-email=test@test.com \
  -n ingress-nginx
  
spec:
  template:
    spec:
      imagePullSecrets:   #添加这一行
      - name: harbor-secret   #添加这一行
=================================     
kubectl apply -f deploy.yaml

[root@k8smaster ~]# grep image deploy.yaml
        image: registry.k8s.io/ingress-nginx/controller:v1.11.2@sha256:d5f8217feeac4887cb1ed21f27c2674e58be06bd8f5184cacea2a69abaf78dce
        imagePullPolicy: IfNotPresent
        image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.3@sha256:a320a50cc91bd15fd2d6fa6de58bd98c1bd64b9a6f926ce23a600d87043455a3
        imagePullPolicy: IfNotPresent
        image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.3@sha256:a320a50cc91bd15fd2d6fa6de58bd98c1bd64b9a6f926ce23a600d87043455a3
        imagePullPolicy: IfNotPresent
[root@k8smaster ~]# grep -E "image:" deploy.yaml | awk '{print $2}' | sort -u
registry.k8s.io/ingress-nginx/controller:v1.11.2@sha256:d5f8217feeac4887cb1ed21f27c2674e58be06bd8f5184cacea2a69abaf78dce
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.3@sha256:a320a50cc91bd15fd2d6fa6de58bd98c1bd64b9a6f926ce23a600d87043455a3

wget https://github.com/containerd/nerdctl/releases/download/v1.7.6/nerdctl-full-1.7.6-linux-amd64.tar.gz

tar -xvf nerdctl-full-*.tar.gz -C /usr/local


#!/bin/bash

images=(
registry.k8s.io/ingress-nginx/controller:v1.11.2
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.3
)

for image in "${images[@]}"
do
  nerdctl pull $image

  new_image=$(echo $image | sed 's#registry.k8s.io/ingress-nginx#harbor.ktzxy.top/k8s#g')

  nerdctl tag $image $new_image
  nerdctl push $new_image
done
```

### 实战九：域名访问 nginx

💣 故障

- 404
- 域名无法解析

```shell
vi test-ingress.yml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: web.com
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

```shell
kubectl apply -f ingress.yaml

#查看是哪个node节点
kubectl get pod -n ingress-nginx -o wide

echo "192.168.31.53 web.com" >> /etc/hosts

http://web.com:31772

[root@k8smaster ingress-nginx]# kubectl get pod -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-6r7p5        0/1     Completed   0          78m
ingress-nginx-admission-patch-cw6zg         0/1     Completed   2          78m
ingress-nginx-controller-6c4b58cb44-j5g5h   1/1     Running     0          78m
[root@k8smaster ingress-nginx]# kubectl get pod,svc -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-6r7p5        0/1     Completed   0          78m
pod/ingress-nginx-admission-patch-cw6zg         0/1     Completed   2          78m
pod/ingress-nginx-controller-6c4b58cb44-j5g5h   1/1     Running     0          78m

NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.101.73.42   <pending>     80:31772/TCP,443:30385/TCP   78m
service/ingress-nginx-controller-admission   ClusterIP      10.96.159.19   <none>        443/TCP                      78m

```

### 实战：Ingress 灰度发布（Canary）

```shell
# 删除
kubectl delete deploy,svc,ingress -l app=web

kubectl create deployment web-v1 --image=nginx
kubectl expose deployment web-v1 --port=80 --name=web-v1

kubectl create deployment web-v2 --image=nginx
kubectl expose deployment web-v2 --port=80 --name=web-v2

kubectl exec -it deploy/web-v2 -- /bin/bash
echo "this is v2" > /usr/share/nginx/html/index.html
exit
```

创建主 Ingress（v1）

```shell
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: web.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-v1
            port:
              number: 80
```

```shell
kubectl apply -f ingress-v1.yaml
```

创建 Canary Ingress

```shell
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  ingressClassName: nginx
  rules:
  - host: web.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-v2
            port:
              number: 80
```

```shell
kubectl apply -f ingress-canary.yaml
```

连续请求：

```
for i in {1..10}; do curl http://web.com:31772; done
```

你会看到：

```
Welcome to nginx!   ← v1
this is v2          ← v2
```

👉 大概比例：

```
80% v1
20% v2
```

#### 什么是灰度发布（Canary）

> 灰度发布是指：**将新版本逐步放量给一部分用户，而不是一次性全量上线**

```shell
旧版本（稳定） → 大部分用户
新版本（测试） → 小部分用户
```

#### 应用场景

- 新功能上线：先放 5% → 没问题 → 逐步放大
- 风险控制（最重要）：发现 bug → 只影响少量用户
- A/B 测试：不同用户看到不同版本
- 性能验证：新版本是否更耗资源？
- 热更新（不停机发布）：无感升级

常见 3 种

| 方式                | 工具          | 难度  |
| ------------------- | ------------- | ----- |
| Ingress 灰度        | ingress-nginx | ⭐⭐    |
| Service Mesh        | Istio         | ⭐⭐⭐⭐⭐ |
| Deployment 滚动更新 | 原生 K8s      | ⭐     |

```shell
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"   #标识这是灰度 ingress
    nginx.ingress.kubernetes.io/canary-weight: "20"  #20% 流量走 v2 0 ~ 100
spec:
  ingressClassName: nginx
  rules:
  - host: web.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-v2
            port:
              number: 80

#参数详解
nginx.ingress.kubernetes.io/canary-by-header: "version"
nginx.ingress.kubernetes.io/canary-by-header-value: "v2"
请求头带 version=v2 → 才走新版本


nginx.ingress.kubernetes.io/canary-by-cookie: "user"
带 cookie 的用户走新版本
```

nginx.ingress.kubernetes.io/canary-weight

| 值   | 效果     |
| ---- | -------- |
| 0    | 不走灰度 |
| 10   | 10%      |
| 50   | 一半     |
| 100  | 全量     |

三种灰度策略对比

| 方式   | 特点             |
| ------ | ---------------- |
| weight | 简单随机         |
| header | 精准控制（推荐） |
| cookie | 用户维度         |

> ingress-nginx 通过在 Nginx 层实现流量分流，根据请求特征（权重、Header、Cookie）决定流量走向，从而实现应用版本的灰度发布，这种方式无需修改应用代码，适合大多数微服务场景。

## 🚀 存储

------

### 实战十：NFS 挂载

```shell
yum install -y nfs-utils

创建共享目录
mkdir -p /data/nfs
chmod 777 /data/nfs

配置共享
vim /etc/exports
写入：
/data/nfs *(rw,sync,no_root_squash,no_subtree_check)

启动服务
systemctl enable rpcbind --now
systemctl enable nfs-server --now

重新加载配置
exportfs -r

验证是否成功
showmount -e localhost

创建 PV（核心）
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany   # 多节点读写
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs
    server: 192.168.31.51
    
kubectl apply -f pv.yaml    

创建 PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi

kubectl apply -f pvc.yaml

kubectl get pv,pvc

[root@k8smaster pv_pvc]# kubectl get pvc,pv
NAME                            STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/nfs-pvc   Bound    nfs-pv   1Gi        RWX                           14s

NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
persistentvolume/nfs-pv   1Gi        RWX            Retain           Bound    default/nfs-pvc                           57s


Pod 挂载 PVC（重点🔥）
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: nfs-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: nfs-volume
    persistentVolumeClaim:
      claimName: nfs-pvc
应用：
kubectl apply -f pod.yaml


进入 Pod
kubectl exec -it nfs-test -- /bin/bash
 写文件
echo "hello nfs" > /usr/share/nginx/html/index.html
 在 NFS 服务器看
cat /data/nfs/index.html

应该看到：
hello nfs
说明成功
Pod ↔ NFS 打通 ✔

重要参数详解

```

accessModes

| 模式          | 说明                    |
| ------------- | ----------------------- |
| ReadWriteOnce | 单节点                  |
| ReadOnlyMany  | 多节点只读              |
| ReadWriteMany | 多节点读写（NFS用这个） |

### 实战十一：PV + PVC

💣 故障

- PVC Pending
- Pod 挂载失败

## 🚀 配置管理

------

### 实战十二：ConfigMap 挂载

```shell
#文件方式
创建 ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  index.html: |
    <h1>This is ConfigMap Page</h1>
应用：
kubectl apply -f configmap.yaml
查看：
kubectl get cm
kubectl describe cm nginx-config
✅ Step 2：Pod 挂载 ConfigMap（重点🔥）
apiVersion: v1
kind: Pod
metadata:
  name: cm-test
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
应用：
kubectl apply -f pod.yaml
🎯 验证
kubectl exec -it cm-test -- curl localhost

👉 输出：

<h1>This is ConfigMap Page</h1>

#环境变量
```

### 实战十三：Secret 注入

💣 故障

- 环境变量为空
- 文件不存在

```shell
#Secret 注入
Secret 默认是 Base64（不是加密，只是编码）

创建 Secret
方法一（推荐）
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=123456
方法二（YAML）
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=   # admin
  password: MTIzNDU2   # 123456
查看：
kubectl get secret
kubectl describe secret my-secret

Step 2：Pod 使用 Secret（两种方式）
🔥 方式1：环境变量（最常用）
apiVersion: v1
kind: Pod
metadata:
  name: secret-test
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
验证：
kubectl exec -it secret-test -- env | grep USERNAME
🔥 方式2：文件挂载
volumes:
- name: secret-volume
  secret:
    secretName: my-secret

👉 挂载后：

/目录/username
/目录/password


```

ConfigMap vs Secret 对比

| 项目     | ConfigMap  | Secret       |
| -------- | ---------- | ------------ |
| 用途     | 普通配置   | 敏感数据     |
| 是否加密 | ❌          | Base64（弱） |
| 挂载方式 | 文件 / env | 文件 / env   |
| 安全性   | 低         | 相对高       |

## 🚀 探针

------

### 实战十四：readiness 探针

> 判断：**容器能不能对外提供服务？**

```
失败 → 从 Service 中摘除（不接流量）
```

```shell
模拟服务不可用
kubectl exec -it <pod> -- /bin/bash
mv /usr/share/nginx/html/index.html /tmp/
🎯 观察
kubectl get endpoints

👉 你会看到：

Pod 被从 Service 移除 ❗
```

三种探针方式

HTTP（最常用）

```
httpGet:
  path: /
  port: 80
```

 TCP

```
tcpSocket:
  port: 80
```

只检查端口是否通

Exec（最灵活）

```
exec:
  command:
  - cat
  - /tmp/healthy
```

### 实战十五：liveness 探针

> 判断：**容器还活不活？**

```
失败 → kubelet 重启容器
```

💣 故障

- 容器疯狂重启
- 服务不可用

```shell
#HTTP 探针（最常用）
创建 Deployment（带探针）
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe
  template:
    metadata:
      labels:
        app: probe
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

        # 🔥 Readiness 探针
        readinessProbe:
          httpGet:
            path: /ready  #写成这样会报错，要进入容器，echo ok > /usr/share/nginx/html/ready 然后才能变成ready
            port: 80
          initialDelaySeconds: 5   #容器启动后多久开始检测
          periodSeconds: 5   #每隔多久检测一次

        # 🔥 Liveness 探针
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10


应用：
kubectl apply -f probe.yaml
查看：
kubectl get pods
kubectl describe pod <pod-name>
🎯 三、验证效果（必须做）
👉 模拟故障（让 liveness 触发）
kubectl exec -it <pod> -- /bin/bash

删除首页：

rm -f /usr/share/nginx/html/index.html
🎯 结果：
Liveness 探针失败 → Pod 自动重启 ✔


[root@k8smaster probe]# kubectl describe pod probe-demo-85bb66d8d4-tbhpr
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m38s               default-scheduler  Successfully assigned default/probe-demo-85bb66d8d4-tbhpr to k8snode1
  Normal   Pulled     2m36s               kubelet            Successfully pulled image "nginx" in 503ms (503ms including waiting)
  Normal   Pulling    8s (x2 over 2m37s)  kubelet            Pulling image "nginx"
  Warning  Unhealthy  8s (x3 over 28s)    kubelet            Liveness probe failed: HTTP probe failed with statuscode: 403
  Warning  Unhealthy  8s (x7 over 28s)    kubelet            Readiness probe failed: HTTP probe failed with statuscode: 403
  Normal   Killing    8s                  kubelet            Container nginx failed liveness probe, will be restarted
  Normal   Created    7s (x2 over 2m36s)  kubelet            Created container nginx
  Normal   Started    7s (x2 over 2m36s)  kubelet            Started container nginx
  Normal   Pulled     7s                  kubelet            Successfully pulled image "nginx" in 502ms (502ms including waiting)

```

```shell
timeoutSeconds:检测超时时间
failureThreshold:连续失败几次 → 判定失败
successThreshold（readiness 才有意义）:连续成功几次 → 判定恢复
```

## 🚀 生命周期

------

### 实战十六：preStop 优雅终止

> `preStop` 是容器生命周期钩子，在 **容器被终止前执行一段逻辑**

优雅下线（Graceful Shutdown）

```shell
Pod 删除 / 扩缩容 / 滚动更新
        ↓
执行 preStop
        ↓
发送 SIGTERM 给容器
        ↓
等待宽限期（默认 30s）
        ↓
强制杀死（SIGKILL）
```

```shell
#preStop + readiness（标准组合🔥）

apiVersion: apps/v1
kind: Deployment
metadata:
  name: graceful-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: graceful
  template:
    metadata:
      labels:
        app: graceful
    spec:
      terminationGracePeriodSeconds: 30   # 优雅终止时间
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

        # 🔥 readiness 探针
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          periodSeconds: 5

        # 🔥 preStop 钩子
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "echo stopping... && sleep 20"]
              
              

#验证
开两个终端
👉 终端1（持续访问）
while true; do curl http://web.com:31772; sleep 1; done
👉 终端2（删除 Pod）
kubectl delete pod <pod-name>
🎯 观察现象
❌ 没 preStop
突然中断 ❌
✅ 有 preStop
请求还能继续一会儿 ✔
然后才停止 ✔

```

preStop 常见用法

```shell
1.sleep（最简单）
command: ["sh", "-c", "sleep 10"]
2.调接口（企业常用）
command: ["sh", "-c", "curl http://localhost:8080/shutdown"]

👉 应用自己做：

关闭连接 / 停止接单
3.标记不可用
rm /tmp/ready

配合 readiness：

立即摘流量
```

```shell
执行流程
kubectl delete pod xxx

① 执行 preStop（sleep 20）
② readiness 失败 → 从 Service 摘除
③ 不再接新流量
④ 等待现有请求处理完
⑤ 容器退出
```

💣 故障

- 请求丢失
- Pod 被直接 kill

------

------

## 🚀 调度

------

### 实战十七：nodeSelector

```shell
Step 1：给节点打标签
kubectl label nodes k8snode1 env=prod
kubectl label nodes k8snode2 env=test
查看：
kubectl get nodes --show-labels
✅ Step 2：Pod 指定调度节点
apiVersion: v1
kind: Pod
metadata:
  name: node-selector-demo
spec:
  nodeSelector:
    env: prod   # 👈 指定节点
  containers:
  - name: nginx
    image: nginx
应用：
kubectl apply -f node-selector.yaml
🎯 验证
kubectl get pod -o wide

👉 会看到：

NODE: k8snode1 ✔
```

### 实战十八：Taint/Toleration

💣 故障

- Pod Pending

```shell
Step 1：给节点加污点
kubectl taint nodes k8snode2 key=test:NoSchedule
🎯 含义
不允许 Pod 调度到这个节点 ❗
查看：
kubectl describe node k8snode2 | grep Taint
✅ Step 2：测试效果

创建普通 Pod：

apiVersion: v1
kind: Pod
metadata:
  name: taint-test
spec:
  containers:
  - name: nginx
    image: nginx

👉 结果：

不会调度到 k8snode2 ✔
```

Toleration（容忍污点）

```shell
让 Pod 能调度到 taint 节点
apiVersion: v1
kind: Pod
metadata:
  name: toleration-demo
spec:
  tolerations:
  - key: "test"
    operator: "Equal"
    value: "test"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx

👉 结果：

可以调度到 k8snode2 ✔
```

nodeSelector vs Taint 对比

| 项目   | nodeSelector | Taint    |
| ------ | ------------ | -------- |
| 控制方 | Pod          | Node     |
| 类型   | 主动选择     | 被动限制 |
| 用途   | 精准调度     | 隔离节点 |
| 强度   | 强           | 更强     |

## 🚀 HPA

> 根据指标（CPU/内存等）**自动增加或减少 Pod 副本数**

### 实战十九：CPU 自动扩容

```shell
流量上涨 → CPU 高 → 自动扩容
流量下降 → CPU 低 → 自动缩容
```

```shell
#前期准备
kubectl top nodes

👉 如果报错：

metrics API not available ❌
🚀 安装（常用国内环境）
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


##创建可扩容应用
Step 1：Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa
  template:
    metadata:
      labels:
        app: hpa
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 500m
应用：
kubectl apply -f hpa-deploy.yaml
✅ Step 2：暴露服务
kubectl expose deployment hpa-demo --port=80 --type=NodePort
✅ Step 3：创建 HPA
kubectl autoscale deployment hpa-demo \
  --cpu-percent=50 \
  --min=1 \
  --max=5

👉 含义：

CPU > 50% → 扩容
最少 1 个
最多 5 个


##验证
方法1：用 busybox 打压测
kubectl run -it --rm load-generator --image=busybox /bin/sh

然后：

while true; do wget -q -O- http://hpa-demo; done

观察
kubectl get hpa -w
CPU ↑ → REPLICAS 从 1 → 2 → 3 ...

[root@k8smaster hpa]# kubectl get hpa -w
NAME       REFERENCE             TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-demo   Deployment/hpa-demo   0%/50%    1         5         1          17m
hpa-demo   Deployment/hpa-demo   5%/50%    1         5         1          20m
hpa-demo   Deployment/hpa-demo   70%/50%   1         5         1          20m
hpa-demo   Deployment/hpa-demo   76%/50%   1         5         2          20m
hpa-demo   Deployment/hpa-demo   42%/50%   1         5         2          20m
hpa-demo   Deployment/hpa-demo   27%/50%   1         5         2          21m
```

```shell
期望副本数 = 当前副本数 × (当前CPU / 目标CPU)

当前 1 Pod
CPU 使用 100%
目标 50%

→ 1 × (100 / 50) = 2 个 Pod
```

```shell
[root@k8smaster hpa]# kubectl get hpa
NAME       REFERENCE             TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
hpa-demo   Deployment/hpa-demo   <unknown>/50%   1         5         1          8m9s

#查看
[root@k8smaster hpa]# kubectl get pods -n kube-system | grep metrics
metrics-server-76466b47bd-v4bg9     0/1     ImagePullBackOff   0              26m
metrics-server-845d86dc79-vb2qz     0/1     ImagePullBackOff   0              28m

ImagePullBackOff--->镜像拉不下来

kubectl edit deployment metrics-server -n kube-system
找到 image：
image: registry.k8s.io/metrics-server/metrics-server:v0.7.0
改成👇（推荐）
image: registry.aliyuncs.com/google_containers/metrics-server:v0.7.0
```

💣 故障

- HPA 不生效
- metrics-server 问题

------

------

## 🚀 Helm

------

### 实战二十：使用 Helm 部署应用

```shell
#安装 Helm（所有节点都可以，通常 master）
#下载（Linux）
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
#验证
helm version

#添加仓库（相当于软件源）
#推荐用阿里云（你这个环境更稳）
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo update

export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
export ALL_PROXY=socks5://127.0.0.1:7897

#用 Bitnami（最主流）
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
#查看：
helm repo list
```

```shell
#实战一：部署 nginx（最简单上手）
搜索应用
helm search repo nginx
安装
helm install my-nginx bitnami/nginx


####报错
[root@k8smaster ~]# helm install my-nginx bitnami/nginx
Error: INSTALLATION FAILED: failed to perform "FetchReference" on source: Get "https://registry-1.docker.io/v2/bitnamicharts/nginx/manifests/23.0.3": dial tcp 157.240.1.33:443: i/o timeout

#使用代理
HTTP_PROXY=http://192.168.31.99:7897 \
HTTPS_PROXY=http://192.168.31.99:7897 \
NO_PROXY=localhost,127.0.0.1,.svc,.cluster.local \
helm install my-nginx bitnami/nginx \
  --set image.registry=docker.io \
  --set image.repository=bitnami/nginx \
  --set image.tag=latest
=====================
查看
kubectl get pods
kubectl get svc


✅ 卸载
helm uninstall my-nginx
🎯 五、实战二：部署复杂应用（Redis）
✅ 安装 Redis
  
HTTP_PROXY=http://192.168.31.99:7897 \
HTTPS_PROXY=http://192.168.31.99:7897 \
NO_PROXY=localhost,127.0.0.1,.svc,.cluster.local \
helm install my-redis bitnami/redis \
  --set image.registry=docker.io \
  --set image.repository=bitnami/redis \
  --set image.tag=latest \
  --set master.persistence.enabled=false \
  --set replica.persistence.enabled=false

✅ 查看密码（重点）
[root@k8smaster ~]# kubectl get secret my-nginx-redis -o yaml
apiVersion: v1
data:
  redis-password: dGZ2U25iWUFZcA==
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: my-nginx
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2026-05-05T09:37:44Z"
  labels:
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: redis
    app.kubernetes.io/version: 8.6.2
    helm.sh/chart: redis-25.4.1
  name: my-nginx-redis
  namespace: default
  resourceVersion: "57898"
  uid: 35a69eff-cd16-483b-85a4-c02e7dc4b4a0
type: Opaque
[root@k8smaster ~]# echo dGZ2U25iWUFZcA== | base64 -d
tfvSnbYAYp
[root@k8smaster ~]# kubectl exec -it my-nginx-redis-master-0 -- redis-cli -a tfvSnbYAYp
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379>

实战三：自定义部署（重点🔥）
👉 下载 chart
helm pull bitnami/nginx --untar
cd nginx
👉 修改 values.yaml
比如：
replicaCount: 3
service:
  type: NodePort
👉 安装
helm install my-nginx ./nginx
👉 结果：
3 个 Pod ✔
NodePort 服务 ✔
升级（非常重要🔥）
helm upgrade my-nginx ./nginx
👉 修改 values 后：
自动滚动更新 ✔
🚀 九、回滚（企业必备）
查看版本
helm history my-nginx
回滚
helm rollback my-nginx 1
👉 秒回滚 ✔
```

```shell
##报错
[root@k8smaster ~]# helm install my-redis aliyun/redis
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "my-redis-redis" namespace: "" from "": no matches for kind "Deployment" in version "extensions/v1beta1"
ensure CRDs are installed first


no matches for kind "Deployment" in version "extensions/v1beta1"
👉 说明：
❌ 这个 Redis chart 还在用 旧 API（extensions/v1beta1）
❌ 但你当前 K8s 版本已经移除了这个 API
```

```shell
##报错
[root@k8smaster ~]# crictl pull nginx
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
E0505 16:50:29.822515  104508 remote_image.go:171] "PullImage from image service failed" err="rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory\"" image="nginx"
FATA[0000] pulling image: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"


crictl 默认配置：
/var/run/dockershim.sock ❌
但你实际应该是：
/run/containerd/containerd.sock ✔

解决：
配置 crictl 指向 containerd
1.创建或修改：
vi /etc/crictl.yaml
写入：
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false

2. 重启 containerd（建议）
systemctl restart containerd

然后可以正常crictl nginx
```

💣 故障

- values.yaml 错误
- 安装失败

# 代理

```shell
##操作系统层
写入 /etc/profile：

vi /etc/profile
追加：
export HTTP_PROXY=http://代理IP:端口
export HTTPS_PROXY=http://代理IP:端口
export NO_PROXY=localhost,127.0.0.1,10.0.0.0/8,192.168.0.0/16,.svc,.cluster.local

生效：
source /etc/profile

##systemd 全局层
很多服务不会读 /etc/profile

配置：
/etc/systemd/system.conf
/etc/systemd/user.conf

添加：

DefaultEnvironment="HTTP_PROXY=http://IP:PORT"
DefaultEnvironment="HTTPS_PROXY=http://IP:PORT"
DefaultEnvironment="NO_PROXY=localhost,127.0.0.1"

然后：

systemctl daemon-reexec

👉 作用：

kubelet
containerd
docker（如果有）



##配置 containerd 代理

编辑：

mkdir -p /etc/systemd/system/containerd.service.d
vi /etc/systemd/system/containerd.service.d/http-proxy.conf

写入：

[Service]
Environment="HTTP_PROXY=http://代理IP:端口"
Environment="HTTPS_PROXY=http://代理IP:端口"
Environment="NO_PROXY=localhost,127.0.0.1,10.244.0.0/16,10.96.0.0/12,.svc,.cluster.local"
重启：
systemctl daemon-reexec
systemctl restart containerd
验证 containerd：
crictl pull nginx:latest

##kubelet
路径：

/etc/systemd/system/kubelet.service.d/10-proxy.conf
[Service]
Environment="HTTP_PROXY=http://IP:PORT"
Environment="HTTPS_PROXY=http://IP:PORT"
Environment="NO_PROXY=127.0.0.1,localhost,10.0.0.0/8,10.96.0.0/12,10.244.0.0/16,.svc,.cluster.local"

重启：

systemctl daemon-reexec
systemctl restart kubelet

👉 作用：

Pod DNS / API Server 通信
镜像拉取间接链路

##Helm
Helm 需要：

export HTTP_PROXY=...
export HTTPS_PROXY=...

或者：

helm repo add ...

👉 注意：
Helm 只吃环境变量，不吃 systemd

##containerd 镜像拉取链路
除了 systemd proxy，还要：

/etc/containerd/config.toml（重点）

有些环境需要加 mirror：

[plugins."io.containerd.grpc.v1.cri".registry]
  config_path = "/etc/containerd/certs.d"

或者：

[plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
  endpoint = ["https://mirror.aliyuncs.com"]
```

# 🟡 二、中期（系统能力｜能跑业务）

👉 目标：**完整业务 + 稳定性 + 自动化**

------

## 🚀 存储进阶

------

### 实战二十一：StorageClass 动态供给

在传统方式里：

- 先手动创建 PV（磁盘）
- 再创建 PVC 绑定

👉 问题：**运维成本高，不灵活**

动态供给（核心思想）

 PVC 一创建
 👉 Kubernetes 自动帮你创建 PV（底层存储自动分配）

provisioner

决定谁来“自动创建 PV”

常见：

| 存储类型 | provisioner                     |
| -------- | ------------------------------- |
| NFS      | nfs-subdir-external-provisioner |
| Ceph     | rook-ceph.rbd.csi.ceph.com      |
| 云盘     | ebs.csi.aws.com / alicloud-disk |
| 本地     | local-path-provisioner          |

reclaimPolicy

| 值     | 含义                       |
| ------ | -------------------------- |
| Delete | PVC 删除 → PV 删除（常用） |
| Retain | 保留数据                   |

volumeBindingMode

| 值                   | 含义                         |
| -------------------- | ---------------------------- |
| Immediate            | 立即绑定                     |
| WaitForFirstConsumer | Pod 调度后再创建（推荐生产） |

实战：NFS 动态供给（最经典）

```shell
前提：
NFS 已搭好
✔ /data/nfs 已共享
✔ showmount -e 正常
```

```shell
完整 YAML（直接保存为 nfs-provisioner.yaml）
---
apiVersion: v1
kind: Namespace
metadata:
  name: nfs-provisioner

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
  namespace: nfs-provisioner

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nfs-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: nfs-provisioner-runner
subjects:
  - kind: ServiceAccount
    name: nfs-provisioner
    namespace: nfs-provisioner
roleRef:
  kind: ClusterRole
  name: nfs-provisioner-runner
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-provisioner
  namespace: nfs-provisioner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-provisioner
  template:
    metadata:
      labels:
        app: nfs-provisioner
    spec:
      serviceAccountName: nfs-provisioner
      containers:
        - name: nfs-provisioner
          image: registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-volume
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: 192.168.31.100   # ❗改成你的NFS IP
            - name: NFS_PATH
              value: /data/nfs
      volumes:
        - name: nfs-volume
          nfs:
            server: 192.168.31.100   # ❗改成你的NFS IP
            path: /data/nfs

---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
reclaimPolicy: Delete
volumeBindingMode: Immediate
parameters:
  archiveOnDelete: "false"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-storage
  resources:
    requests:
      storage: 1Gi
🚀 四、执行
kubectl apply -f nfs-provisioner.yaml
🧪 五、验证（一步步看）
1️⃣ 看 provisioner

[root@k8smaster storageclass]# kubectl get pod -n nfs-provisioner
NAME                               READY   STATUS    RESTARTS   AGE
nfs-provisioner-58c8868f58-brrcm   1/1     Running   0          6m17s

2️⃣ 看 PVC
kubectl get pvc
[root@k8smaster storageclass]# kubectl get pvc
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test-pvc   Bound    pvc-6e328302-5af9-41f3-9fb2-7e1baa70f606   1Gi        RWX            nfs-storage    88m



K8s 动态存储三大核心坑
1.CNI 网络问题
2.镜像问题
3.RBAC 权限问题

3️⃣ 看 PV
kubectl get pv
[root@k8smaster storageclass]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
pvc-6e328302-5af9-41f3-9fb2-7e1baa70f606   1Gi        RWX            Delete           Bound    default/test-pvc   nfs-storage             3m36s


4️⃣ 看 NFS 目录（关键验证）
ls /data/nfs
[root@k8smaster storageclass]# ls /data/nfs
default-test-pvc-pvc-6e328302-5af9-41f3-9fb2-7e1baa70f606  index.html


###查看日志
kubeclt logs -n nfs-provisioner deploy/nfs-provisioner
```

```shell
###镜像拉取失败 导出导入

##

docker pull registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2

docker save -o nfs.tar registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2


scp nfs.tar root@192.168.0.53:/root/

ctr -n k8s.io images import /root/nfs.tar

ctr -n k8s.io images tag \
k8sminikube/nfs-subdir-external-provisioner:v4.0.2 \
registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2

kubectl delete pod -n nfs-provisioner --all
```

```shell
StorageClass 为什么重要？

👉 解耦存储管理
👉 自动化 PV 生命周期
👉 云原生存储核心
```

### 实战二十二：MySQL 持久化部署

```shell
MySQL Pod
   ↓
PVC
   ↓
StorageClass (nfs-storage)
   ↓
NFS 动态创建目录
```

```shell
#1.Secret（密码）
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdDEyMw==   # root123
#2. Service（稳定访问）
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - port: 3306
  clusterIP: None   # Headless（给 StatefulSet 用）
  selector:
    app: mysql
#3. StatefulSet（核心）
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql

  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes: ["ReadWriteMany"]
        storageClassName: nfs-storage
        resources:
          requests:
            storage: 1Gi            
```

```shell
kubectl apply -f mysql.yaml

#验证
Pod 状态
kubectl get pod
kubectl get pvc
ls /data/nfs

#测试数据持久化
kubectl exec -it mysql-0 -- mysql -uroot -p
create database testdb;
kubectl delete pod mysql-0
kubectl exec -it mysql-0 -- mysql -uroot -p
show databases;
testdb 🎉
```

💣 故障

- 数据丢失
- PVC 不绑定

------

------

## 🚀 有状态服务

------

### 实战二十三：StatefulSet 部署 MySQL 主从

```shell
mysql-0  → 主库（master）
mysql-1  → 从库（slave）
mysql-2  → 从库（slave）
```

### StatefulSet 带来的能力

- Pod 有固定名字：`mysql-0 / mysql-1`
- 有序启动（先 0，再 1）
- 每个 Pod 独立 PVC（数据隔离）

```shell
mysql/
├── configmap.yaml
├── secret.yaml
├── service.yaml
├── statefulset.yaml
├── init-script.sql
```

```shell
#configmap.yaml（MySQL 配置 + 初始化脚本）
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  master.cnf: |
    [mysqld]
    log-bin=mysql-bin
    server-id=1
    binlog_format=ROW

  slave.cnf: |
    [mysqld]
    server-id=2
    relay-log=mysql-relay-bin

  init.sh: |
    #!/bin/bash
    set -ex

    HOSTNAME=$(hostname)

    if [[ "$HOSTNAME" == "mysql-0" ]]; then
      echo "I am master"
    else
      echo "I am slave"

      until mysql -h mysql-0.mysql -uroot -p${MYSQL_ROOT_PASSWORD} -e "select 1"; do
        echo "waiting for master..."
        sleep 5
      done

      mysql -uroot -p${MYSQL_ROOT_PASSWORD} <<EOF
      CHANGE MASTER TO
        MASTER_HOST='mysql-0.mysql',
        MASTER_USER='root',
        MASTER_PASSWORD='${MYSQL_ROOT_PASSWORD}',
        MASTER_AUTO_POSITION=1;
      START SLAVE;
EOF
    fi
```

```shell
#secret.yaml（数据库密码）
# echo -n "root123" | base64   得到cm9vdDEyMw==
# echo cm9vdDEyMw== | base64 -d 得到root123
# 可以直接写明文密码，不用手动 base64，K8s 自动帮你转
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdDEyMw==  # root123
```

```shell
#service.yaml（Headless Service）
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      name: mysql
```

```shell
#statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 2
  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql
    spec:

      initContainers:
      - name: init-mysql
        image: mysql:5.7
        command:
        - bash
        - "-c"
        - |
          set -ex
          if [[ $(hostname) == "mysql-0" ]]; then
            cp /config/master.cnf /etc/mysql/conf.d/
          else
            cp /config/slave.cnf /etc/mysql/conf.d/
          fi
        volumeMounts:
        - name: config
          mountPath: /config
        - name: conf
          mountPath: /etc/mysql/conf.d

      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD

        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        - name: conf
          mountPath: /etc/mysql/conf.d
        - name: config
          mountPath: /docker-entrypoint-initdb.d

      volumes:
      - name: config
        configMap:
          name: mysql-config
          defaultMode: 0755
      - name: conf
        emptyDir: {}

  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

```shell
#init-script.sql
-- 创建复制用户（推荐用专用用户，不用 root）
CREATE USER 'repl'@'%' IDENTIFIED BY 'repl123';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

```shell
kubectl apply -f secret.yaml
kubectl apply -f configmap.yaml
kubectl apply -f service.yaml
kubectl apply -f statefulset.yaml
```

```shell
验证主从是否成功
进入从库：
kubectl exec -it mysql-1 -- mysql -uroot -proot123
show slave status\G

关键字段：

Slave_IO_Running: Yes
Slave_SQL_Running: Yes

模拟主库挂掉
kubectl delete pod mysql-0
观察：
mysql-0 是否重建
slave 是否恢复连接
```

💣 故障

- 主从不同步

实战

**企业级 MySQL 高可用（K8s 终极版）**

- 自动主从切换
- VIP 漂移
- 读写分离
- 故障自动恢复

## 🚀 网络深入

------

### 实战二十四：跨节点 Pod 通信验证

### 实战二十五：Service 负载均衡验证

💣 故障

- 只访问一个 Pod

### 实战二十六：蓝绿发布

### 实战二十七：金丝雀发布（Ingress）

💣 故障

- 流量不均

## 🚀 安全

### 实战二十八：RBAC 限权

💣 故障

- 权限 denied

------

------

## 🚀 资源管理

------

### 实战二十九：requests / limits

### 实战三十：制造 OOMKilled

💣 故障

- Pod 被杀

------

------

## 🚀 日志 & 监控

------

### 实战三十一：部署 Prometheus

### 实战三十二：Grafana 看监控

### 实战三十三：日志收集（EFK）

💣 故障

- 指标为空
- 日志丢失

------

------

## 🚀 故障专项训练（核心）

------

### 实战三十四：Pod Pending 排查

```shell
Pending = 已创建，但还没被调度到节点
本质：scheduler 没找到“合适节点”


##Pod Pending 5大原因
1.节点不可用 / NotReady
Node NotReady / unreachable
检查：
kubectl get nodes

2. taint / toleration 不匹配（你已经遇到）
node-role.kubernetes.io/control-plane:NoSchedule
没加 toleration 就会 Pending
解决：去掉 taint；或加 toleration
tolerations:
- key: "node-role.kubernetes.io/control-plane"
  operator: "Exists"
  effect: "NoSchedule"
  
3. 资源不够（CPU / 内存）
Insufficient cpu / memory
检查：
kubectl describe node k8snode1
解决：减少 requests/limits
4. nodeSelector / affinity 不匹配
节点标签不符合 Pod 要求
解决：删掉或匹配节点标签：
kubectl label node k8snode1 app=web
5. PVC / 存储卡住（很常见）
pod 卡在 ContainerCreating / Pending

##排查流程
1.看 Pod 事件
2.看节点状态
kubectl get nodes -o wide
3.看是否有 taint
kubectl describe node k8smaster | grep Taints
4.看资源
kubectl top node
```

```shell
#Pod Pending / Scheduling 相关 Events 关键词大全
节点不满足调度（最常见）
👉 关键词：
0/3 nodes are available
no nodes available to schedule pods
didn't match Pod's node affinity/selector
👉 含义：
没有任何节点满足条件（综合原因）
🧱 2️⃣ taint / toleration 问题（非常高频）
👉 关键词：
had untolerated taint
node(s) had taint
Taint {node-role.kubernetes.io/control-plane}
👉 含义：
节点被“污点保护”，Pod 没有容忍
🧱 3️⃣ 节点不可用（NotReady）
👉 关键词：
node(s) were not ready
node(s) unreachable
node status is NotReady
👉 含义：
节点宕机 / kubelet异常 / 网络断开
🧱 4️⃣ 资源不足（CPU / 内存）
👉 关键词：
Insufficient cpu
Insufficient memory
node(s) had no available resources
👉 含义：
节点资源不够调度 Pod
🧱 5️⃣ nodeSelector / affinity 不匹配
👉 关键词：
didn't match node selector
node(s) didn't match node affinity rules
👉 含义：
标签 / 亲和性规则不匹配节点
🧱 6️⃣ PVC / 存储问题（Pending/ContainerCreating）
👉 关键词：
waiting for a volume to be created
persistentvolumeclaim not bound
failed to attach volume
👉 含义：
存储没准备好，Pod卡住
🧱 7️⃣ 调度器预留 / 抢占问题
👉 关键词：
preemption is not helpful
no preemption victims found
👉 含义：
调度器尝试抢占，但也没有资源
```

### 实战三十五：CrashLoopBackOff 排查

```shell
容器启动 → 退出 → 重启 → 再退出 → 无限循环
#排查思路
看日志 + 看退出原因 + 看启动命令

####标准排查 5 步法
Step 1：看 Pod 状态
kubectl get pod
重点看：
RESTARTS > 0
STATUS = CrashLoopBackOff
Step 2：看详细事件（非常重要）
kubectl describe pod <pod名>
👉 重点看：
Last State: Terminated
Reason: XXX
Exit Code: XXX
Step 3：看日志（核心中的核心🔥）
kubectl logs <pod名>
👉 如果容器重启太快：
kubectl logs <pod名> --previous
Step 4：看 exit code（关键判断）
```

常见退出码：

| Exit Code | 含义                     |
| --------- | ------------------------ |
| 0         | 正常退出（但不应该退出） |
| 1         | 通用错误                 |
| 137       | OOMKilled（内存不足）    |
| 139       | Segfault（程序崩溃）     |
| 143       | 被 kill（SIGTERM）       |

```shell
#CrashLoopBackOff 常见原因
1.应用启动失败（最常见）
👉 关键词：
application error
connection refused
config not found
👉 原因：
配置文件错
环境变量缺失
服务依赖没启动
2. 启动命令错误（非常常见）
👉 关键词：
executable file not found
permission denied
👉 原因：
CMD / ENTRYPOINT 写错
3. 端口冲突 / 服务起不来
👉 关键词：
address already in use
bind: permission denied
4. 健康检查失败（你刚学过 probe）
👉 关键词：
liveness probe failed
readiness probe failed
👉 结果：
K8s 直接 kill 容器 → 重启
5. OOM（内存不足🔥）
👉 关键词：
OOMKilled
👉 典型：
Java
Redis
Node.js
6. 配置错误 / Secret / ConfigMap
👉 关键词：
configmap not found
secret not found
```

### 实战三十六：ImagePullBackOff 排查

```shell
ImagePullBackOff = 镜像拉不下来 + Kubelet反复重试失败

排查思路：看事件 → 看镜像 → 看仓库 → 看认证 → 看网络

####标准排查 6 步法
#Step 1：看 Pod 状态
kubectl get pod -o wide
重点：STATUS = ImagePullBackOff / ErrImagePull
#Step 2：看详细事件（最关键🔥）
kubectl describe pod <pod名>
👉 重点看：
Events:
  Failed to pull image
  ErrImagePull
  ImagePullBackOff
#Step 3：直接看错误类型（关键判断）
3.1 镜像不存在
manifest unknown
not found
✔ 含义：
3.2 认证失败
unauthorized: authentication required
✔ 含义：
私有仓库没配 secret
3.3 网络问题（你现在最可能）
i/o timeout
dial tcp ...:443 timeout
✔ 含义：
无法访问 registry（docker.io / harbor）
3.4 DNS问题
no such host
#Step 4：验证镜像是否能拉（关键）
✔ 手动测试（containerd / docker）
ctr images pull <image>
或：
docker pull <image>
#Step 5：检查 imagePullSecrets（私有仓库）
kubectl get secret
Pod 是否有：
imagePullSecrets:
#Step 6：检查节点网络（你现在很关键）
curl https://registry-1.docker.io
或：
ping registry-1.docker.io
```

### 实战三十七：网络不通排查

```shell
K8s 网络不通分 4 层

① 节点系统网络（Linux）
② DNS解析
③ Pod网络（CNI：flannel/calico）
④ 出口网络（外网/registry）
```

```shell
####标准排查 7 步法
#Step 1：节点是否能上网（最关键）
ping 8.8.8.8
curl https://registry-1.docker.io
#Step 2：DNS 是否正常
nslookup google.com
或：
cat /etc/resolv.conf
👉 常见问题：
nameserver 不可用 / 114.114.114.114
#Step 3：检查路由
ip route
👉 必须有：
default via 192.168.x.x
#Step 4：检查防火墙
systemctl status firewalld
或直接：
iptables -L -n
#Step 5：检查 CNI（flannel/calico）
kubectl get pods -n kube-flannel
或：
ip a | grep cni
👉 如果 CNI 异常：
Pod 网络完全不通 ❌
#Step 6：Pod 内测试网络（非常重要🔥）
kubectl run test --rm -it --image=busybox -- sh

然后：

ping 8.8.8.8
wget www.baidu.com
#Step 7：测试 Kubernetes DNS
nslookup kubernetes.default
```

结果判断：

| 结果                | 含义              |
| ------------------- | ----------------- |
| ❌ ping 不通         | 节点无外网        |
| ❌ curl 超时         | 出口被拦 / 防火墙 |
| ✔ ping 通 curl 不通 | HTTPS被封         |

### K8s 生产故障全景（7大类）

```shell
① 调度与资源类（Scheduler层）
🔥 常见故障
Pod Pending
Insufficient CPU/MEM
No nodes available
preemption not helpful
📌 深层问题
节点资源碎片化
requests/limits 设计错误
HPA + 资源冲突
优先级抢占失效
🧠 本质
👉 Scheduler 找不到“合适机器”
🧱 ② Pod生命周期类（运行时层）
🔥 常见故障
CrashLoopBackOff
OOMKilled
Error
Completed异常退出
📌 深层问题
应用本身 bug
内存泄漏（Java/Node）
probe 配置错误
initContainer 卡死
🧠 本质
👉 容器“活不过启动阶段”
🧱 ③ 镜像与仓库类（Image层）
🔥 常见故障
ImagePullBackOff
ErrImagePull
unauthorized
manifest unknown
📌 深层问题
私有仓库认证失效
Harbor TLS / DNS 问题
镜像版本管理混乱
CI/CD tag 错误
🧠 本质
👉 容器还没启动就死在“拉镜像阶段”
🧱 ④ 网络与服务发现（CNI / DNS / Service）
🔥 常见故障
Pod互访失败
Service不可达
DNS解析失败
Ingress 404 / 502
📌 深层问题
flannel / calico 异常
kube-proxy 规则错
CoreDNS 崩溃
iptables 混乱
🧠 本质
👉 Pod之间“看不见对方”
🧱 ⑤ 存储类（PV / PVC）
🔥 常见故障
Pending（PVC）
ContainerCreating卡住
MountVolume失败
ReadWriteOnce冲突
📌 深层问题
NFS/CEPH 挂载失败
权限问题（UID/GID）
存储未绑定
多节点写冲突
🧠 本质
👉 数据卷“挂不上 / 挂错 / 不能写”
🧱 ⑥ 控制面组件（Control Plane）
🔥 常见故障
API Server不可用
etcd slow / corrupt
Controller异常
Scheduler不工作
📌 深层问题
etcd 磁盘IO瓶颈
证书过期
控制面资源不足
多 master 不一致
🧠 本质
👉 集群“大脑出问题”
🧱 ⑦ 安全与权限类（RBAC / Admission）
🔥 常见故障
forbidden: User cannot create
RBAC denied
Admission webhook failed
📌 深层问题
role/clusterrole 错误
serviceAccount 缺失权限
webhook crash
policy 拦截（OPA / Kyverno）
🧠 本质
👉 “你有权限，但系统不让你干”
```

## 🚀 自动化

------

### 实战三十八：CI/CD + Helm 部署

------

------

## 🚀 DaemonSet

------

### 实战三十九：日志采集（Filebeat）

------

------

## 🚀 Job

------

### 实战四十：定时任务（CronJob）

------

------

# 🔴 三、后期（生产级｜架构能力）

👉 目标：**高可用 + 大规模 + 企业能力**

------

## 🚀 高可用

------

### 实战四十一：多 master 高可用

💣 故障

- etcd 挂掉

------

------

## 🚀 网络高级

------

### 实战四十二：Calico 网络策略（NetworkPolicy）

💣 故障

- 服务互相访问失败

------

------

## 🚀 Ingress 进阶

------

### 实战四十三：HTTPS + 证书

### 实战四十四：多域名路由

------

------

## 🚀 扩缩容进阶

------

### 实战四十五：HPA + 自定义指标

------

------

## 🚀 Operator

------

### 实战四十六：部署 Operator（如 MySQL）

------

------

## 🚀 存储高级

------

### 实战四十七：Ceph / 云盘存储

------

------

## 🚀 故障极限挑战（重点）

------

### 实战四十八：节点宕机演练

### 实战四十九：Pod 大规模重启

### 实战五十：网络分区故障

------

------

## 🚀 混沌工程（高阶）

------

### 实战五十一：Chaos Mesh 故障注入

------

------

## 🚀 性能压测

------

### 实战五十二：压测 + HPA 自动扩容验证

------

------

## 🚀 多租户

------

### 实战五十三：Namespace + RBAC 隔离

------

------

## 🚀 安全

------

### 实战五十四：Pod 安全策略（PSA）

------

------

## 🚀 服务网格（加分项）

------

### 实战五十五：Istio 入门

------

------

# 🎯 四、最终 Boss 关（真正毕业）

------

### 💀 实战五十六：完整企业级系统

```
Ingress + Helm + MySQL + Redis + HPA + StorageClass + 监控
```

### 💀 实战五十七：全链路故障演练

```
OOM + 网络断开 + 节点宕机
```

------

### 💀 实战五十八：从0恢复系统

```
删除所有资源 → 恢复
```

# 网络改动

```shell
彻底清理旧集群（非常关键）
kubeadm reset -f

然后手动清理残留（必须做）：

rm -rf /etc/kubernetes/*
rm -rf /var/lib/etcd
rm -rf /var/lib/kubelet/*
rm -rf ~/.kube

清理网络规则：

iptables -F
iptables -t nat -F
iptables -t mangle -F
iptables -X

然后重新kubeadm init....
```

