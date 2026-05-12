+++
date = '2025-03-03T16:51:12+08:00'
draft = false
title = '3 使用kubeadm方式搭建K8S集群'
slug = "ag31mut3sc"
description = "3 使用kubeadm方式搭建K8S集群"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "3 使用kubeadm方式搭建K8S集群"
series = ["Kubernetes学习系列"]
showSeries= true  

+++
# 使用kubeadm方式搭建K8S集群

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

```bash
# 创建一个 Master 节点
kubeadm init

# 将一个 Node 节点加入到当前集群中
kubeadm join <Master节点的IP和端口 >
```

## Kubeadm方式搭建K8S集群

使用kubeadm方式搭建K8s集群主要分为以下几步

- 准备三台虚拟机，同时安装操作系统CentOS 7.x
- 对三个安装之后的操作系统进行初始化操作
- 在三个节点安装 docker kubelet kubeadm kubectl
- 在master节点执行kubeadm init命令初始化
- 在node节点上执行 kubeadm join命令，把node节点添加到当前集群
- 配置CNI网络插件，用于节点之间的连通【失败了可以多试几次】
- 通过拉取一个nginx进行测试，能否进行外网测试

## 安装要求

在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多【注意master需要两核】
- 可以访问外网，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止swap分区

## 创建虚拟机的参数

Master：1 CPU × 2核

Worker：1 CPU × 2核

### Master节点

- CPU：**1 × 2核（起步）**
- 内存：3GB

### Worker节点

- CPU：**1 × 2核**
- 内存：2~2.5GB

硬盘40G分区方案

| 挂载点 | 大小 | 用途                  |
| ------ | ---- | --------------------- |
| /boot  | 1GB  | 启动                  |
| /      | 10GB | 系统                  |
| /var   | 20GB | ⭐ Docker / K8s / 日志 |
| swap   | 2GB  | 交换分区              |
| 剩余   | ~7GB | LVM预留               |

## 准备环境

| 角色   | IP            |
| ------ | ------------- |
| master | 192.168.31.51 |
| node1  | 192.168.31.52 |
| node2  | 192.168.31.53 |

然后开始在每台机器上执行下面的命令

```bash
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
# 永久关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config  
# 临时关闭
setenforce 0  

# 关闭swap
# 临时
swapoff -a 
# 永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 根据规划设置主机名【master节点上操作】
hostnamectl set-hostname k8smaster
# 根据规划设置主机名【node1节点操作】
hostnamectl set-hostname k8snode1
# 根据规划设置主机名【node2节点操作】
hostnamectl set-hostname k8snode2

# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.31.51 k8smaster
192.168.31.52 k8snode1
192.168.31.53 k8snode2
EOF


# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# 生效
sysctl --system  

# 时间同步 临时同步
# 原因：K8s 极度依赖“时间一致性 ；
# 证书（TLS）问题；etcd 一致性问题；日志 / 排障
yum install ntpdate -y
ntpdate time.windows.com

# 时间同步 生产推荐（长期）
yum install -y chrony
systemctl enable chronyd
systemctl start chronyd
# 查看状态
chronyc sources
```

[Docker 常见疑难杂症解决方案 | 🏘️Home](https://ktzxy.top/posts/7k38hq60mo/#20yum源报错)

## 安装Docker/kubeadm/kubelet

所有节点安装Docker/kubeadm/kubelet ，Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker

### 安装Docker（新版不推荐）

首先配置一下Docker的阿里yum源

```bash
cat >/etc/yum.repos.d/docker.repo<<EOF
[docker-ce-edge]
name=Docker CE Edge - \$basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/\$basearch/edge
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF
```

然后yum方式安装docker

```bash
sudo yum install -y yum-utils

# Step 2: 添加软件源信息
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Step 3: 安装Docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Step 4: 开启Docker服务
sudo service docker start
```

配置docker的镜像源

```bash
cat >> /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```

然后重启docker

```bash
systemctl restart docker
```

### 安装containerd

```shell
#安装容器运行时（containerd 推荐）
# 可以不安装docker，而是只安装containerd
# 安装
yum install -y containerd.io

#报错
[root@k8smaster ~]# yum install -y containerd
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
No package containerd available.
Error: Nothing to do

#解决
cat > /etc/yum.repos.d/docker-ce.repo <<EOF
[docker-ce-stable]
name=Docker CE Stable - Aliyun
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/
enabled=1
gpgcheck=0
EOF

yum clean all
yum makecache



# 生成配置
containerd config default > /etc/containerd/config.toml

# 修改
SystemdCgroup = true

# 添加镜像
[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
#加入下列
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
    endpoint = ["https://registry.cn-hangzhou.aliyuncs.com"]

  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
    endpoint = ["https://registry.aliyuncs.com/google_containers"]

  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."ghcr.io"]
    endpoint = ["https://ghcr.nju.edu.cn"]

#启动
systemctl restart containerd
systemctl enable containerd


containerd 和 Docker 关系
Docker = containerd + CLI + 构建工具
```

### 加kubernetes软件源

然后我们还需要配置一下yum的k8s软件源

```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

###  安装kubeadm，kubelet和kubectl

由于版本更新频繁，这里指定版本号部署：

```bash
# 安装kubelet、kubeadm、kubectl，同时指定版本
yum install -y kubelet-1.28.2 kubeadm-1.28.2 kubectl-1.28.2
# 设置开机启动
systemctl enable kubelet
```

## 部署Kubernetes Master【master节点】

在   192.168.31.51  执行，也就是master节点

```bash
kubeadm init --apiserver-advertise-address=192.168.31.51 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.28.2 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16

#如果卡在拉取镜像那里，可以手动拉取镜像
[root@k8smaster ~]# kubeadm config images list \
> --kubernetes-version v1.28.2 \
> --image-repository registry.aliyuncs.com/google_containers
registry.aliyuncs.com/google_containers/kube-apiserver:v1.28.2
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.28.2
registry.aliyuncs.com/google_containers/kube-scheduler:v1.28.2
registry.aliyuncs.com/google_containers/kube-proxy:v1.28.2
registry.aliyuncs.com/google_containers/pause:3.9
registry.aliyuncs.com/google_containers/etcd:3.5.9-0
registry.aliyuncs.com/google_containers/coredns:v1.10.1
[root@k8smaster ~]# kubeadm config images pull \
> --kubernetes-version v1.28.2 \
> --image-repository registry.aliyuncs.com/google_containers
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.28.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.28.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.28.2
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.28.2
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.9
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.5.9-0
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:v1.10.1



#如果初始化有误，重置，然后初始化
kubeadm reset -f

rm -rf /etc/kubernetes/
rm -rf /var/lib/kubelet/
rm -rf /var/lib/etcd/

rm -rf /var/lib/cni/
rm -rf /var/lib/containerd/*

systemctl restart containerd
systemctl restart kubelet

########################################
#参数详解
--apiserver-advertise-address API Server 对外“广播”的地址
--apiserver-advertise-address=192.168.31.51  表示所有节点通过这个IP访问 master

--image-repository   镜像仓库地址
[root@k8snode1 ~]# kubeadm config images list
I0503 20:48:11.711560   14155 version.go:256] remote version is much newer: v1.36.0; falling back to: stable-1.28
registry.k8s.io/kube-apiserver:v1.28.15
registry.k8s.io/kube-controller-manager:v1.28.15
registry.k8s.io/kube-scheduler:v1.28.15
registry.k8s.io/kube-proxy:v1.28.15
registry.k8s.io/pause:3.9
registry.k8s.io/etcd:3.5.9-0
registry.k8s.io/coredns/coredns:v1.10.1


可以提前拉 kubeadm config images pull

--kubernetes-version 指定安装的 K8s 版本

--service-cidr Service 虚拟IP网段 
10.96.0.0/12 默认值（推荐不改）

--pod-network-cidr  Pod 网络网段
10.244.0.0/16  这是 Flannel 默认网段
这个参数必须和你用的网络插件匹配！

6443（API Server）
2379（etcd）
10250（kubelet）

##生产写法了解：
kubeadm init \
--apiserver-advertise-address=192.168.31.51 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.28.2 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--control-plane-endpoint=192.168.31.100 \
--upload-certs

--control-plane-endpoint  VIP（高可用入口）
--upload-certs  多 master 时共享证书
```

### 报错

```shell
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

#关闭swap
free -m
swapoff -a
sed -i '/ swap / s/^/#/' /etc/fstab #不管用
```



```shell
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.

Unfortunately, an error has occurred:
        timed out waiting for the condition

This error is likely caused by:
        - The kubelet is not running
        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
        - 'systemctl status kubelet'
        - 'journalctl -xeu kubelet'

Additionally, a control plane component may have crashed or exited when started by the container runtime.
To troubleshoot, list all containers using your preferred container runtimes CLI.
Here is one example how you may list all running Kubernetes containers by using crictl:
        - 'crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock ps -a | grep kube | grep -v pause'
        Once you have found the failing container, you can inspect its logs with:
        - 'crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock logs CONTAINERID'
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher

原因：控制平面（apiserver等）没起来 → kubeadm 等超时了
排查：
1.systemctl status kubelet
如果是 failed / restarting：
journalctl -xeu kubelet -n 50
常见：failed to run Kubelet: cgroup driver mismatch
改SystemdCgroup = true
systemctl restart containerd
systemctl restart kubelet

2.systemctl status containerd
常见containerd 配置不对：
containerd config default > /etc/containerd/config.toml
再改：
SystemdCgroup = true

3.看控制面容器 
crictl ps -a | grep kube
正常：
kube-apiserver
kube-controller-manager
kube-scheduler
etcd

报错：
[root@k8smaster ~]# crictl ps -a | grep kube
WARN[0000] runtime connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
E0501 14:20:28.004631   62932 remote_runtime.go:390] "ListContainers with filter from runtime service failed" err="rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory\"" filter="&ContainerFilter{Id:,State:nil,PodSandboxId:,LabelSelector:map[string]string{},}"
FATA[0000] listing containers: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"
说明：
❌ crictl 试图连接 Docker
❌ 但你机器上没有 Docker

K8s 1.24+ 已经移除 dockershim
👉 默认运行时 = containerd

journalctl -u kubelet -n 50
得到下列：
kuberuntime_manager.go:1166] "CreatePodSandbox for pod failed" err="rpc error: code = DeadlineExceeded
May 01 14:27:22 k8smaster kubelet[60125]: E0501 14:27:22.294032   60125 pod_workers.go:1300] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"kube-apise
May 01 14:27:23 k8smaster kubelet[60125]: W0501 14:27:23.609419   60125 reflector.go:535] vendor/k8s.io/client-go/informers/factory.go:150: failed to list *v1.Node: Get "https
May 01 14:27:23 k8smaster kubelet[60125]: E0501 14:27:23.609484   60125 reflector.go:147] vendor/k8s.io/client-go/informers/factory.go:150: Failed to watch *v1.Node: failed to
May 01 14:27:24 k8smaster kubelet[60125]: E0501 14:27:24.859727   60125 eviction_manager.go:258] "Eviction manager: failed to get summary stats" err="failed to get node info:
May 01 14:27:25 k8smaster kubelet[60125]: W0501 14:27:25.129489   60125 reflector.go:535] vendor/k8s.io/client-go/informers/factory.go:150: failed to list *v1.CSIDriver: Get "
May 01 14:27:25 k8smaster kubelet[60125]: E0501 14:27:25.129603   60125 reflector.go:147] vendor/k8s.io/client-go/informers/factory.go:150: Failed to watch *v1.CSIDriver: fail
May 01 14:27:25 k8smaster kubelet[60125]: E0501 14:27:25.326026   60125 controller.go:146] "Failed to ensure lease exists, will retry" err="Get \"https://192.168.31.51:6443/ap
May 01 14:27:25 k8smaster kubelet[60125]: I0501 14:27:25.795630   60125 kubelet_node_status.go:70] "Attempting to register node" node="k8smaster"
May 01 14:27:25 k8smaster kubelet[60125]: E0501 14:27:25.796202   60125 kubelet_node_status.go:92] "Unable to register node with API server" err="Post \"https://192.168.31.51:
May 01 14:27:28 k8smaster kubelet[60125]: W0501 14:27:28.905314   60125 reflector.go:535] vendor/k8s.io/client-go/informers/factory.go:150: failed to list *v1.RuntimeClass: Ge
May 01 14:27:28 k8smaster kubelet[60125]: E0501 14:27:28.905466   60125 reflector.go:147] vendor/k8s.io/client-go/informers/factory.go:150: Failed to watch *v1.RuntimeClass: f
May 01 14:27:30 k8smaster kubelet[60125]: E0501 14:27:30.129295   60125 event.go:289] Unable to write event: '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMet
May 01 14:27:32 k8smaster kubelet[60125]: E0501 14:27:32.330126   60125 controller.go:146] "Failed to ensure lease exists, will retry" err="Get \"https://192.168.31.51:6443/ap
May 01 14:27:32 k8smaster kubelet[60125]: I0501 14:27:32.799305   60125 kubelet_node_status.go:70] "Attempting to register node" node="k8smaster"

containerd 创建 Pod 沙箱超时（根本没成功）
crictl images

#真实原因：containerd 仍在访问国外 registry（registry.k8s.io）失败
最终查看
cat /etc/containerd/config.toml | grep sandbox_image
看到
sandbox_image = "registry.k8s.io/pause:3.9"

#解决：
修改 sandbox 镜像
vi /etc/containerd/config.toml
找到：sandbox_image = "registry.k8s.io/pause:3.9"
改成：sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
重启 containerd（必须）
systemctl restart containerd
清理环境
kubeadm reset -f
rm -rf /etc/kubernetes/*
rm -rf /var/lib/etcd
重新初始化
```

报错二

```shell
[root@k8smaster ~]# kubeadm init --apiserver-advertise-address=192.168.31.51 .28.2 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.28.2
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /pr
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4
[preflight] If you know what you are doing, you can make a check non-fatal wi
To see the stack trace of this error execute with --v=5 or higher


解决：
1. bridge-nf-call-iptables 不存在
/proc/sys/net/bridge/bridge-nf-call-iptables does not exist
说明：br_netfilter 模块没有加载
2. ip_forward 没开
/proc/sys/net/ipv4/ip_forward contents are not set to 1
说明：Linux 没开启路由转发（K8s 必须开启）

##
① 加载内核模块
modprobe br_netfilter
② 开启内核参数（临时）
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/ipv4/ip_forward
③ 永久生效（关键！）

创建配置文件：

cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

应用配置：
sysctl --system
验证一下
ls /proc/sys/net/bridge/
cat /proc/sys/net/ipv4/ip_forward

应该看到：
bridge 目录存在
ip_forward = 1
```



由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址，【执行上述命令会比较慢，因为后台其实已经在拉取镜像了】，我们 docker images 命令即可查看已经拉取的镜像

![image-20200929094302491](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520817.webp)

当我们出现下面的情况时，表示kubernetes的镜像已经安装成功

![image-20200929094620145](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520818.webp)

使用kubectl工具 【master节点操作】

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

执行完成后，我们使用下面命令，查看我们正在运行的节点

```bash
kubectl get nodes
```

![image-20260501150605324](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260501150605324.png)

能够看到，目前有一个master节点已经运行了，但是还处于未准备状态

下面我们还需要在Node节点执行其它的命令，将node1和node2加入到我们的master节点上

## 加入Kubernetes Node【Slave节点】

下面我们需要到 node1 和 node2服务器，执行下面的代码向集群添加新节点

执行在kubeadm init输出的kubeadm join命令：

> 注意，以下的命令是在master初始化完成后，每个人的都不一样！！！需要复制自己生成的

```bash
kubeadm join 192.168.31.51:6443 --token qw723k.qv8r5vopayulrq2x \
        --discovery-token-ca-cert-hash sha256:fc19d3cdb16c7a38a21add21a1f1a953320755f1d14933656d9f7648171603a1
```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

```
kubeadm token create --print-join-command
```

当我们把两个节点都加入进来后，我们就可以去Master节点 执行下面命令查看情况

```bash
kubectl get node
```

![image-20201113165358663](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520820.webp)

## 部署CNI网络插件

上面的状态还是NotReady，下面我们需要网络插件，来进行联网访问

```bash
# 添加
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 查看状态 【kube-system是k8s中的最小单元】
kubectl get pods -n kube-system

#或者
kubectl delete -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

报错
Warning  FailedMount  20s (x10 over 4m30s)  kubelet            MountVolume.SetUp failed for volume "bpffs" : hostPath type check failed: /sys/fs/bpf is not a directory

```

运行后的结果

![image-20201113165929510](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520821.webp)

运行完成后，我们查看状态可以发现，已经变成了Ready状态了

![image-20201113194557147](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520822.webp)

如果上述操作完成后，还存在某个节点处于NotReady状态，可以在Master将该节点删除

```bash
# master节点将该节点删除

##20210223 yan 查阅资料添加###kubectl drain k8snode1 --delete-local-data --force --ignore-daemonsets

kubectl delete node k8snode1
 
# 然后到k8snode1节点进行重置
 kubeadm reset
# 重置完后在加入
kubeadm join 192.168.31.51:6443 --token qw723k.qv8r5vopayulrq2x \
        --discovery-token-ca-cert-hash sha256:fc19d3cdb16c7a38a21add21a1f1a953320755f1d14933656d9f7648171603a1
        

#添加后，发现主节点ready，node节点还是处于NotReady
且node节点得到如下
[root@k8snode1 ~]# kubectl get nodes E0501 15:16:53.952084 43193 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?
解决：
Step 1：Master 删除 Node
在 master 上执行：
kubectl delete node k8snode1
kubectl delete node k8snode2
Node 执行 reset（非常关键）
在每个 node 上执行：
kubeadm reset -f
清理残留：
rm -rf /etc/kubernetes/
rm -rf /var/lib/kubelet/
rm -rf /var/lib/cni/
rm -rf /run/flannel
确认 containerd 配置正确
检查：
cat /etc/containerd/config.toml | grep sandbox
应该是：
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
然后重启：
systemctl restart containerd
systemctl restart kubelet
重新 join 集群

然后
kubectl get node
发现还是处于NotReady
kubectl desctibe node k8snode1
得打如下错误
reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized

master检查
[root@k8smaster ~]# kubectl get pods -n kube-flannel -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP              NODE        NOMINATED NODE   READINESS GATES
kube-flannel-ds-c7469   1/1     Running   0          9m1s    192.168.31.52   k8snode1    <none>           <none>
kube-flannel-ds-qfzln   1/1     Running   0          8m54s   192.168.31.53   k8snode2    <none>           <none>
kube-flannel-ds-tmkw8   1/1     Running   0          24m     192.168.31.51   k8smaster   <none>           <none>
再检查node，应该没问题了
kubectl get nodes
有时候 Node 刚加入不会立刻调度：
kubectl delete pod -n kube-flannel --all

##报错
[root@k8smaster ~]# kubectl get pod -n kube-system                           NAME                                READY   STATUS    RESTARTS   AGE
coredns-66f779496c-jq9hr            0/1     Pending   0          5m5s
coredns-66f779496c-q2chh            0/1     Pending   0          5m5s

```

## 测试kubernetes集群

我们都知道K8S是容器化技术，它可以联网去下载镜像，用容器的方式进行启动

在Kubernetes集群中创建一个pod，验证是否正常运行：

```bash
# 下载nginx 【会联网拉取nginx镜像】
kubectl create deployment web --image=nginx
# 查看状态
kubectl get pod
```

### 镜像拉取解决方案

```shell
#拉取镜像报错
1.ImagePullBackOff
# 注意，所有节点都要修改配置
vi /etc/containerd/config.toml
 [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
        endpoint = [
         "https://registry.aliyuncs.com",
         "https://mirror.ccs.tencentyun.com"
        ]
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
  endpoint = ["https://registry.aliyuncs.com/google_containers"]
  
systemctl restart containerd
crictl pull nginx
```

如果我们出现Running状态的时候，表示已经成功运行了

![image-20201113203537028](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520823.webp)

下面我们就需要将端口暴露出去，让其它外界能够访问

```bash
# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
# 查看一下对外的端口
kubectl get pod,svc
```

我们到我们的宿主机浏览器上，访问如下地址

```bash
http://192.168.31.52:30633/
```

发现我们的nginx已经成功启动了

![image-20201113204056851](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520825.webp)

到这里为止，我们就搭建了一个单master的k8s集群

![image-20201113204158884](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520826.webp)



## 错误汇总

### 错误一

在执行Kubernetes  init方法的时候，出现这个问题

```bash
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
```

是因为VMware设置的核数为1，而K8S需要的最低核数应该是2，调整核数重启系统即可

### 错误二

我们在给node1节点使用 kubernetes join命令的时候，出现以下错误

```bash
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR Swap]: running with swap on is not supported. Please disable swap
```

错误原因是我们需要关闭swap

```bash
# 关闭swap
# 临时
swapoff -a 
# 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 错误三

在给node1节点使用 kubernetes join命令的时候，出现以下错误

```bash
The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp [::1]:10248: connect: connection refused
```

解决方法，首先需要到 master 节点，创建一个文件

```bash
# 创建文件夹
mkdir /etc/systemd/system/kubelet.service.d

# 创建文件
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# 添加如下内容
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --fail-swap-on=false"

# 重置
kubeadm reset
```

然后删除刚刚创建的配置目录

```bash
rm -rf $HOME/.kube
```

然后 在master重新初始化

```bash
kubeadm init --apiserver-advertise-address=202.193.57.11 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
```

初始完成后，我们再到 node1节点，执行 kubeadm join命令，加入到master

```bash
kubeadm join 202.193.57.11:6443 --token c7a7ou.z00fzlb01d76r37s \
    --discovery-token-ca-cert-hash sha256:9c3f3cc3f726c6ff8bdff14e46b1a856e3b8a4cbbe30cab185f6c5ee453aeea5
```

添加完成后，我们使用下面命令，查看节点是否成功添加

```bash
kubectl get nodes
```

### 错误四

我们再执行查看节点的时候，  kubectl get nodes 会出现问题

```bash
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

这是因为我们之前创建的配置文件还存在，也就是这些配置

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

我们需要做的就是把配置文件删除，然后重新执行一下

```bash
rm -rf $HOME/.kube
```

然后再次创建一下即可

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

这个问题主要是因为我们在执行 kubeadm reset 的时候，没有把 $HOME/.kube 给移除掉，再次创建时就会出现问题了

### 错误五

安装的时候，出现以下错误

```bash
Another app is currently holding the yum lock; waiting for it to exit...
```

是因为yum上锁占用，解决方法

```bash
yum -y install docker-ce
```

### 错误六

在使用下面命令，添加node节点到集群上的时候

```bash
kubeadm join 192.168.31.51:6443 --token qw723k.qv8r5vopayulrq2x \
        --discovery-token-ca-cert-hash sha256:fc19d3cdb16c7a38a21add21a1f1a953320755f1d14933656d9f7648171603a1
```

然后出现了这个错误

```bash
W1117 06:55:11.220907   11230 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

出于安全考虑，Linux系统**默认是禁止数据包转发**的。所谓**转发即当主机拥有多于一块的网卡时，其中一块收到数据包，根据数据包的目的ip地址将包发往本机另一网卡，该网卡根据路由表继续发送数据包**。这通常就是路由器所要实现的功能。也就是说  **/proc/sys/net/ipv4/ip_forward** 文件的值不支持转发

- 0：禁止
- 1：转发

所以我们需要将值修改成1即可

```bash
echo “1” > /proc/sys/net/ipv4/ip_forward
```

修改完成后，重新执行命令即可