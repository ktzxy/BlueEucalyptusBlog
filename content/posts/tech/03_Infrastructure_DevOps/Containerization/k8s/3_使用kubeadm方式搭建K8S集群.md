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
# 检查
free -m

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

#加载模块（解决“找不到文件”的问题）
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# 立即加载
sudo modprobe overlay
sudo modprobe br_netfilter

----------------------------------

#配置参数（解决“网络不通”的问题）
cat <<EOF | tee /etc/sysctl.d/k8s.conf
# 开启网桥流量过滤，使 iptables 能看到桥接流量 (K8s 网络插件依赖)
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
# 开启 IP 转发
net.ipv4.ip_forward                 = 1
# 内存溢出时保护机制
vm.swappiness = 0
EOF

# 加载配置
sysctl --system
# 验证一下
ls /proc/sys/net/bridge/
cat /proc/sys/net/ipv4/ip_forward

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

```shell
问题1：sysctl是做什么的？
在运行时配置内核参数
  -p   从指定的文件加载系统参数，如不指定即从/etc/sysctl.conf中加载

问题2：为什么要执行modprobe br_netfilter？
修改/etc/sysctl.d/k8s.conf文件，增加如下三行参数：
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

sysctl -p /etc/sysctl.d/k8s.conf出现报错：

sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-ip6tables: No such file or directory
sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-iptables: No such file or directory

解决方法：
modprobe br_netfilter

问题3：为什么开启net.bridge.bridge-nf-call-iptables内核参数？
在centos下安装docker，执行docker info出现如下警告：
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled

解决办法：
vim  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1

问题4：为什么要开启net.ipv4.ip_forward = 1参数？
kubeadm初始化k8s如果报错：
就表示没有开启ip_forward，需要开启。

net.ipv4.ip_forward是数据包转发：
出于安全考虑，Linux系统默认是禁止数据包转发的。所谓转发即当主机拥有多于一块的网卡时，其中一块收到数据包，根据数据包的目的ip地址将数据包发往本机另一块网卡，该网卡根据路由表继续发送数据包。这通常是路由器所要实现的功能。
要让Linux系统具有路由转发功能，需要配置一个Linux的内核参数net.ipv4.ip_forward。这个参数指定了Linux系统当前对路由转发功能的支持情况；其值为0时表示禁止进行IP转发；如果是1,则说明IP转发功能已经打开。
```



yum源报错：[Docker 常见疑难杂症解决方案 | 🏘️Home](https://ktzxy.top/posts/7k38hq60mo/#20yum源报错)

## 安装Containerd/kubeadm/kubelet

所有节点安装Docker/kubeadm/kubelet ，Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker

### 安装containerd

```shell
#安装容器运行时（containerd 推荐）
# 可以不安装docker，而是只安装containerd
# 添加docker-ce 软件源
cat > /etc/yum.repos.d/docker-ce.repo <<EOF
[docker-ce-stable]
name=Docker CE Stable - Aliyun
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/
enabled=1
gpgcheck=0
EOF

yum clean all
yum makecache
# 安装
yum install -y containerd.io

# 生成配置
containerd config default > /etc/containerd/config.toml

# 修改一
SystemdCgroup = true

# 修改二
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"

# 修改三 添加镜像地址 
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


#containerd 和 Docker 关系
#Docker = containerd + CLI + 构建工具
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

```shell
kubeadm init --apiserver-advertise-address=192.168.31.51 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.28.2 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
```

```bash
#参数详解
--apiserver-advertise-address API Server 对外“广播”的地址
--apiserver-advertise-address=192.168.31.51  表示所有节点通过这个IP访问 master
--image-repository   镜像仓库地址
--kubernetes-version 指定安装的 K8s 版本
--service-cidr Service 虚拟IP网段  10.96.0.0/12 默认值（推荐不改）
--pod-network-cidr  Pod 网络网段   10.244.0.0/16  这是 Flannel 默认网段
这个参数必须和你用的网络插件匹配！
# 各个组件的端口
6443（API Server）
2379（etcd）
10250（kubelet）



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

#可以提前拉 kubeadm config images pull
```

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
```

运行完成后，我们查看状态可以发现，已经变成了Ready状态了

![image-20201113194557147](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202507291520822.webp)

如果上述操作完成后，还存在某个节点处于NotReady状态，可以在Master将该节点删除

```bash
# master节点将该节点删除
kubectl delete node k8snode1
 
# 然后到k8snode1节点进行重置
 kubeadm reset
# 重置完后在加入
kubeadm join 192.168.31.51:6443 --token qw723k.qv8r5vopayulrq2x \
        --discovery-token-ca-cert-hash sha256:fc19d3cdb16c7a38a21add21a1f1a953320755f1d14933656d9f7648171603a1
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
kubeadm init --apiserver-advertise-address=192.168.31.51 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.28.2 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
```

初始完成后，我们再到 node1节点，执行 kubeadm join命令，加入到master

```bash
kubeadm join 192.168.31.51:6443 --token qw723k.qv8r5vopayulrq2x \
        --discovery-token-ca-cert-hash sha256:fc19d3cdb16c7a38a21add21a1f1a953320755f1d14933656d9f7648171603a1
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