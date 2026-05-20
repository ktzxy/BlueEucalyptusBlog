+++
date = '2026-05-20T10:33:40+08:00'
draft = false
title = '26_使用kubeadm方式搭建K8S集群-基于ubuntu'
slug = "kxy0n6sosn"
description = "26_使用kubeadm方式搭建K8S集群-基于ubuntu"
summary = "26_使用kubeadm方式搭建K8S集群-基于ubuntu"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
comments = true
series = ["Kubernetes学习系列"]
showSeries= true  
+++

# 26_使用kubeadm方式搭建K8S集群-基于ubuntu

# 使用kubeadm方式搭建K8S集群-基于ubuntu

分区如下（40G）

| 挂载点 | 大小     | 用途           |
| ------ | -------- | -------------- |
| /boot  | 1G       | 引导           |
| swap   | 4G       | 交换           |
| /      | 20G      | 系统           |
| /var   | 剩余全部 | Docker/K8s数据 |

安装的环境信息：

| 主机名   | 配置 | IP            |
| :------- | :--- | :------------ |
| master01 | 2C4G | 192.168.31.51 |
| node01   | 2C3G | 192.168.31.52 |
| node02   | 2C3G | 192.168.31.53 |

```shell
# 版本兼容性查询
https://github.com/kubesphere/kubekey/blob/master/docs/kubernetes-versions.md
https://github.com/kubeedge/kubeedge
https://github.com/kubeedge/kubeedge/blob/master/CHANGELOG/CHANGELOG-1.17.md
https://github.com/kubeedge/edgemesh
```

### 1 配置镜像源

配置 阿里云 ubuntu 22.04 镜像源并进行更新

```shell
sudo bash -c "cat << EOF > /etc/apt/sources.list && apt update
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
EOF"
```

### 2 安装前准备

#### 1）设置主机名称

```txt
hostnamectl set-hostname k8smaster
hostnamectl set-hostname k8snode1
hostnamectl set-hostname k8snode2
```

#### 2）配置 hosts 映射

```txt
cat >> /etc/hosts << EOF
192.168.31.51 k8smaster
192.168.31.52 k8snode1
192.168.31.53 k8snode2
EOF
```

#### 3）关闭防火墙（ufw替代firewalld）

```txt
sudo ufw status
sudo ufw disable
```

#### 4）关闭 selinux（ubuntu无）

```bash
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

#### 5）关闭交换分区

```bash
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```

#### 6）开启 IPV4 转发

```bash
# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

```bash
# 应用 sysctl 参数而不重新启动
sysctl --system
```

#### 7）加载 overlay/netfilter 模块

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```bash
modprobe overlay
modprobe br_netfilter

#通过运行以下指令确认 br_netfilter 和 overlay 模块被加载：
lsmod | grep br_netfilter
lsmod | grep overlay
```

```shell
#通过运行以下指令确认 net.bridge.bridge-nf-call-iptables、net.bridge.bridge-nf-call-ip6tables 和 net.ipv4.ip_forward 系统变量在你的 sysctl 配置中被设置为 1：
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

#### 8）配置时间同步

```bash
# 设置为亚洲的上海时区
sudo timedatectl set-timezone Asia/Shanghai
# 重启时间同步服务
sudo systemctl restart systemd-timesyncd.service
# 确保时间同步服务正常运行
timedatectl status
```

### 3 k8s 组件安装

#### 1）containerd 安装

```shell
wget https://github.com/containerd/containerd/releases/download/v1.7.25/containerd-1.7.25-linux-amd64.tar.gz

tar -zxvf containerd-1.7.25-linux-amd64.tar.gz -C /usr/local
#实际安装在/usr/local/bin目录下

root@k8snode2:~# containerd -v
containerd github.com/containerd/containerd v1.7.25 bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
```

通过 systemd 启动 containerd：

下载地址：[https://raw.githubusercontent.com/containerd/containerd/refs/heads/main/containerd.service](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fraw.githubusercontent.com%2Fcontainerd%2Fcontainerd%2Frefs%2Fheads%2Fmain%2Fcontainerd.service&objectId=2486361&objectType=1&contentType=undefined)

```shell
# 直接下载 containerd.service，拷贝到 /etc/systemd/system/ 目录下
sudo wget -O /etc/systemd/system/containerd.service https://raw.githubusercontent.com/containerd/containerd/refs/heads/main/containerd.service


# 可以通过拷贝方式创建
vim /etc/systemd/system/containerd.service
```

```shell
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
#uncomment to enable the experimental sbservice (sandboxed) version of containerd/cri integration
#Environment="ENABLE_CRI_SANDBOXES=sandboxed"
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

加载配置、启动：

```txt
systemctl daemon-reload
systemctl enable --now containerd
```

```txt
# 查看 containerd 状态
systemctl status containerd
```

```shell
# 查看安装的 containerd 版本
ctr version

root@k8snode2:~# ctr version
Client:
  Version:  v1.7.25
  Revision: bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
  Go version: go1.22.10

Server:
  Version:  v1.7.25
  Revision: bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
  UUID: d1fc7fe8-1fcf-4034-853f-caa2a2faa012
```

生成配置文件：

```shell
mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml

# 重启 containerd
systemctl restart containerd
```

#### 2）runc 安装

```shell
root@k8smaster:~# runc -v
Command 'runc' not found, but can be installed with:
apt install runc                                   # version 1.3.4-0ubuntu1~22.04.1, or
apt install golang-github-opencontainers-runc-dev  # version 1.1.7-0ubuntu1~22.04.6
root@k8smaster:~# apt install runc
```

验证:

```shell
root@k8smaster:~# runc -v
runc version 1.3.4-0ubuntu1~22.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.3
root@k8smaster:~# sudo containerd config dump | grep -A 5 "runtimes.runc.options"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""   #BinaryName = "" 表示 containerd 会使用系统默认的 runc 路径
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0

```

#### 3）CNI 安装

```txt
wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz

mkdir -p /opt/cni/bin

sudo tar -xzvf cni-plugins-linux-amd64-v1.6.2.tgz -C /opt/cni/bin
```

#### 4）配置加速器(无用)

阿里云的镜像加速，自2024年07月02日起，ACR对镜像加速功能的使用范围进行了调整：

- 仅限阿里云用户在具备公网访问的阿里云产品上使用该镜像加速能力。
- 仅支持通过镜像加速器拉取限定范围内的[容器镜像](https://cloud.tencent.com/product/tcr?from_column=20065&from=20065)。

会导致部分镜像拉取不到，可以不配置阿里云镜像加速；

```txt
mkdir -p /etc/containerd/certs.d/docker.io
```

添加 config_path = "/etc/containerd/certs.d"

```txt
sed -i 's/config_path\ =.*/config_path = \"\/etc\/containerd\/certs.d\"/g' /etc/containerd/config.toml

# 或者直接在文件中进行修改
# vim /etc/containerd/config.toml
```

```txt
cat > /etc/containerd/certs.d/docker.io/hosts.toml << EOF
server = "https://docker.io"
[host."https://51bhdfwh.mirror.aliyuncs.com"]
  capabilities = ["pull", "resolve"]
EOF
```

```txt
systemctl daemon-reload
systemctl restart containerd
```

#### 5）配置 CGroup 驱动

把 SystemdCgroup = false 修改为：SystemdCgroup = true

```txt
sed -i 's/SystemdCgroup\ =\ false/SystemdCgroup\ =\ true/g' /etc/containerd/config.toml
```

把 sandbox_image = "k8s.gcr.io/pause:3.6" 修改为：sandbox_image="registry.aliyuncs.com/google_containers/pause:3.8"

```bash
sed -i 's/sandbox_image\ =.*/sandbox_image\ =\ "registry.aliyuncs.com\/google_containers\/pause:3.9"/g' /etc/containerd/config.toml|grep sandbox_image
```

```bash
systemctl daemon-reload
systemctl restart containerd
```

#### 6）安装 crictl

```bash
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.25.0/crictl-v1.25.0-linux-amd64.tar.gz

sudo tar -xzvf crictl-v1.25.0-linux-amd64.tar.gz -C /usr/local/bin/
```

```bash
# 配置 crictl 使用 containerd 的 socket
cat >>  /etc/crictl.yaml << EOF
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
debug: true
EOF

#crictl ps 查看容器，或者 crictl images 查看镜像
```

```txt
systemctl restart containerd
```

### 4 k8s 集群部署

#### 1）更换阿里云 k8s 镜像源

```txt
echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"  | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```txt
apt-get update
```

问题1：更新的时候回出现如下错误

解决：安装 安装 GPG 秘钥

The following signatures couldn't be verified because the public key is not available: NO_PUBKEY B53DC80D13EDEF05

```txt
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg |sudo apt-key add -
```

#### 2）安装 kubeadm、kubelet、kubectl

配置并安装apt包

以下内容直接在官网复制，安装的 1.28 版本：

更新 `apt` 包索引并安装使用 Kubernetes `apt` 仓库所需要的包：

```txt
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

```shell
#下载用于 Kubernetes 软件包仓库的公共签名密钥。所有仓库都使用相同的签名密钥，因此你可以忽略URL中的版本：
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```shell
#添加 Kubernetes apt 仓库。 请注意，此仓库仅包含适用于 Kubernetes 1.28 的软件包； 对于其他 Kubernetes 次要版本，则需要更改 URL 中的 Kubernetes 次要版本以匹配你所需的次要版本 
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```shell
#更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

```shell
#通过下面命令查看安装的kubeadm版本：
root@k8smaster:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"28", GitVersion:"v1.28.15", GitCommit:"841856557ef0f6a399096c42635d114d6f2cf7f4", GitTreeState:"clean", BuildDate:"2024-10-22T20:33:16Z", GoVersion:"go1.22.8", Compiler:"gc", Platform:"linux/amd64"}
```

#### 3）master 节点安装

拉取镜像：

```shell
sudo kubeadm config images pull \
--image-repository=registry.aliyuncs.com/google_containers \
--kubernetes-version=v1.28.15 \
--cri-socket=unix:///run/containerd/containerd.sock
```

`--cri-socket=unix:///run/containerd/containerd.sock`==主要是为了**消除歧义**和**确保命令在非默认环境下能正确执行**。==

通过下面命令初始化集群(注意修改主节点IP地址和版本号)：

```bash
sudo kubeadm init \
--apiserver-advertise-address=192.168.31.51 \
--image-repository=registry.aliyuncs.com/google_containers \
--kubernetes-version=v1.28.15 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--cri-socket=unix:///run/containerd/containerd.sock
```

拷贝 config 文件：

```txt
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 4）node 节点安装

加入集群：

然后在所有工作节点上执行这行命令（注意修改为自己的token），注意后面拼接上 --cri-socket=unix:///var/run/containerd/containerd.sock 参数：

```txt
kubeadm join 192.168.31.51:6443 --token ox9dtv.agq25e3r6cv3l8bs \
        --discovery-token-ca-cert-hash sha256:872c1f6ec872cf014e455140e390d60763ba4f1269e2fdb9e7524e02d4ca02c4 \
--cri-socket=unix:///run/containerd/containerd.sock
```

查看节点：

```txt
kubectl get nodes -o wide
```

此时节点状态是 NotReady 的状态；

#### 5）网络插件部署

这里 flannel 和 calico 任选其一即可；

1）创建 flannel.yaml 配置文件：

```txt
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

2）创建 calico.yaml

安装Tigera Calico操作符和自定义资源定义：

```txt
wget https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/tigera-operator.yaml

kubectl create -f tigera-operator.yaml
```

接下来需要安装必须的客户端资源，因为我们pod的网段与calico官网不相同，所以先将这个文件下载下来然后更改一下网段地址：

```txt
wget https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/custom-resources.yaml


# 修改pod的网段地址
sed -i 's/cidr: 192.168.0.0/cidr: 10.244.0.0/g' custom-resources.yaml

kubectl create -f custom-resources.yaml
```

然后可以开始使用下面这行命令监控创建过程：

```shell
watch kubectl get all -o wide -n calico-system
```

由于 calico 的镜像拉取问题，这里可以先将镜像拉到本地，再进行导入；

```shell
# 下载对应版本的 calico 镜像
wget https://github.com/projectcalico/calico/releases/download/v3.29.1/release-v3.29.1.tgz
tar -zxf release-v3.29.1.tgz

# 导入镜像
ctr -n k8s.io images import calico-cni.tar 
ctr -n k8s.io images import calico-dikastes.tar
ctr -n k8s.io images import calico-flannel-migration-controller.tar
ctr -n k8s.io images import calico-kube-controllers.tar
ctr -n k8s.io images import calico-node.tar
ctr -n k8s.io images import calico-pod2daemon.tar
ctr -n k8s.io images import calico-typha.tar

# 拉取缺失的镜像
docker pull docker.io/calico/csi:v3.29.1
docker pull docker.io/calico/node-driver-registrar:v3.29.1
# 注意：CSI 通常还需要一个 registrar 边车容器
crictl pull harbor.ktzxy.top/calico/node-driver-registrar:v3.29.1

crictl pull harbor.ktzxy.top/calico/csi:v3.29.1

ctr -n k8s.io images tag harbor.ktzxy.top/calico/node-driver-registrar:v3.29.1 docker.io/calico/node-driver-registrar:v3.29.1

ctr -n k8s.io images tag harbor.ktzxy.top/calico/csi:v3.29.1 docker.io/calico/csi:v3.29.1 
```

查看集群导入的镜像列表：

```txt
crictl image list
```

清理镜像：

```txt
crictl rmi imageID
```

#### 6）查看部署的集群

```txt
kubectl get node -owide
```

```shell
kubectl get pod -A -owide
```

