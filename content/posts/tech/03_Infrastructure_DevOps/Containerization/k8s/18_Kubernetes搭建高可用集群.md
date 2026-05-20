+++
date = '2025-07-09T00:55:05+08:00'
draft = false
title = '18 Kubernetes搭建高可用集群'
slug = "xfqke7p6so"
description = "18 Kubernetes搭建高可用集群"
categories = ["📒Kubernetes"]
tags = ["Kubernetes"]
summary = "18 Kubernetes搭建高可用集群"
series = ["Kubernetes学习系列"]
showSeries= true  
+++
# K8s使用kubeadm搭建高可用集群

## 前言

之前我们搭建的集群，只有一个master节点，当master节点宕机的时候，通过node将无法继续访问，而master主要是管理作用，所以整个集群将无法提供服务

## 高可用集群

下面我们就需要搭建一个多master节点的高可用集群，不会存在单点故障问题

但是在node 和 master节点之间，需要存在一个 LoadBalancer组件，作用如下：

- 负载
- 检查master节点的状态

对外有一个统一的VIP：虚拟ip来对外进行访问

## 高可用集群技术细节

高可用集群技术细节如下所示：

- keepalived：配置虚拟ip，检查节点的状态
- haproxy：负载均衡服务【类似于nginx】
- apiserver：
- controller：
- manager：
- scheduler：

## ip划分

| 角色         | IP              |      |
| ------------ | --------------- | ---- |
| k8s-master01 | 192.168.31.50 |      |
| k8s-master02 | 192.168.31.51 |      |
| k8s-master03 | 192.168.31.52 |      |
| /            | 192.168.31.100 | VIP  |
| k8s-node01   | 192.168.31.55 |      |
| k8s-node02   | 192.168.31.56 |      |
| Pod网段      | 172.16.0.0/16   | /    |
| Service网段  | 10.96.0.0/16    | /    |

VIP（虚拟IP）不要和公司内网IP重复，首先去ping一下，不通才可用。VIP需要和你的主机在同一个局域网内

（不是直接用我的VIP）

公有云上搭建VIP是公有云的负载均衡的IP，比如阿里云的内网SLB/NLB的地址，腾讯云内网ELB的地址。不需要再搭建keepalived 和 haproxy

如果是私有云也需要问一下私有云管理员是否支持VIP！

## 初始化操作

### 修改机器IP，变成静态IP

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33文件
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=192.168.31.50
NETMASK=255.255.255.0
GATEWAY=192.168.181.2
DNS1=192.168.181.2
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
DEVICE=ens33
ONBOOT=yes

#修改配置文件之后需要重启网络服务才能使配置生效，重启网络服务命令如下：
service network restart

注：/etc/sysconfig/network-scripts/ifcfg-ens33文件里的配置说明：
NAME=ens33    #网卡名字，跟DEVICE名字保持一致即可
DEVICE=ens33   #网卡设备名，大家ip addr可看到自己的这个网卡设备名，每个人的机器可能这个名字不一样，需要写自己的
BOOTPROTO=static   #static表示静态ip地址
ONBOOT=yes    #开机自启动网络，必须是yes
IPADDR=192.168.31.50   #ip地址，需要跟自己电脑所在网段一致
NETMASK=255.255.255.0  #子网掩码，需要跟自己电脑所在网段一致
GATEWAY=192.168.181.2   #网关，在自己电脑打开cmd，输入ipconfig /all可看到
DNS1=192.168.181.2    #DNS，在自己电脑打开cmd，输入ipconfig /all可看到 
    
各个节点执行如下命令更新yum源和操作系统：
yum update -y
```

### 更改节点主机名

```bash
hostnamectl set-hostname k8s-master01
hostnamectl set-hostname k8s-master02
hostnamectl set-hostname k8s-master03
hostnamectl set-hostname k8s-node01
hostnamectl set-hostname k8s-node02
```

### 添加hosts

```shell
cat >> /etc/hosts << EOF
192.168.31.50 k8s-master01
192.168.31.51 k8s-master02
192.168.31.52 k8s-master03
192.168.31.55 k8s-node01
192.168.31.56 k8s-node02
EOF
```

### 安装必备工具

```shell
yum install -y \
  yum-utils \
  device-mapper-persistent-data \
  lvm2 \
  ipvsadm \
  ipset \
  conntrack \
  socat \
  chrony \
  nfs-utils \
  jq \
  git \
  vim \
  wget \
  curl \
  net-tools \
  telnet \
  lrzsz \
  psmisc \
  bash-completion

#yum-utils 
#device-mapper-persistent-data  Docker 和 Containerd 存储驱动（Devicemapper）的依赖。
#lvm2  逻辑卷管理，配合上面的包，用于容器存储管理。
#ipvsadm  K8s Service 的 IPVS 模式依赖此工具。
#ipset 
#conntrack 
#socat  Kubelet 依赖它进行端口转发（kubectl port-forward）。
#chrony 
#nfs-utils 如果你使用 NFS 作为持久化存储（PV），则必须安装。
#jq  命令行 JSON 处理工具。解析 kubectl get pod -o json 的输出必备。
#git 拉取 YAML 配置文件或代码。
#vim 文本编辑和日志记录的基础工具。
#wget/curl 下载文件、测试 API 连通性、健康检查。
#net-tools 
#telnet 测试端口连通性（如测试 Master 6443 端口是否通）。
#lrzsz 包含 rz / sz 命令，用于 Xshell 等终端直接上传下载文件，运维很方便。
#psmisc 包含 killall、pstree 等进程管理工具。
#bash-completion
```

### 关闭防火墙和selinux

```shell
# 关闭防火墙
systemctl disable --now firewalld
systemctl disable --now dnsmasq

# 关闭selinux
# 永久关闭
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config 
#修改selinux配置文件之后，重启机器，selinux配置才能永久生效，重启之后，登录到机器，
# 临时关闭
setenforce 0  
```

### 关闭swap

```shell
# 1. 临时关闭并调整参数
swapoff -a
sysctl -w vm.swappiness=0

# 2. 永久关闭 (注释 fstab)
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab

# 3. 让 swappiness 参数永久生效 (推荐补充这一步)
echo "vm.swappiness = 0" >> /etc/sysctl.conf

问题1：为什么要关闭swap交换分区？
Swap是交换分区，如果机器内存不够，会使用swap分区，但是swap分区的性能较低，k8s设计的时候为了能提升性能，默认是不允许使用交换分区的。Kubeadm初始化的时候会检测swap是否关闭，如果没关闭，那就初始化失败。如果不想要关闭交换分区，安装k8s的时候可以指定--ignore-preflight-errors=Swap来解决。
```

### 时间同步

```shell
yum install chrony -y
# 备份原配置
cp /etc/chrony.conf /etc/chrony.conf.bak
# 使用阿里云 NTP 源 (如果是内网环境，请指向内网 NTP 服务器)
sed -i 's/pool 2.centos.pool.ntp.org iburst/server ntp1.aliyun.com iburst/g' /etc/chrony.conf
systemctl enable chronyd --now
chronyc sources -v
```

### 系统参数限制 (Ulimit)

```shell
cat <<EOF >> /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
* soft nproc 65535
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
EOF
```

### 验证

```shell
free -m
# Swap 一栏应该全为 0
ulimit -n
# 输出应为 65536
ulimit -u
# 输出应为 65535
sysctl net.bridge.bridge-nf-call-iptables
# 输出应为 = 1
```

### 配置 SSH 免密登录

```shell
# 1. 生成密钥 (一路回车即可)
ssh-keygen -t rsa

# 2. 分发公钥 (图片中的写法)
# 注意：图片里的写法 $i.ssh 是不对的，应该是 ~/.ssh
# 正确的循环写法如下：
for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02; do ssh-copy-id -i ~/.ssh/id_rsa.pub $i; done
```

### 安装源码文件

master01 节点下载安装所有的源码文件

```shell
cd /root/ ;git clone https://gitee.com/dukuan/k8s-ha-install.git
```

### 内核配置

所有节点

```shell
# 安装必要的软件包
yum install -y ipvsadm ipset sysstat conntrack libseccomp

# 配置内核模块
vim /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_rpfilter
ipt_REJECT
ipip

# 启动服务
systemctl enable --now systemd-modules-load.service

#验证模块是否加载成功
lsmod | grep -e ip_vs -e nf_conntrack
```

### 内核优化配置

**`kube-proxy` 模式是 `ipvs`**,**必须做这些配置**

```shell
#配置内核模块（包含存储和网络）
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
# 存储驱动
overlay
# 网桥过滤
br_netfilter
# IPVS 负载均衡
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF

# 立即加载
sudo modprobe overlay
sudo modprobe br_netfilter
sudo modprobe ip_vs
sudo modprobe nf_conntrack
```

```shell
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
# ==========================
# 基础网络 (必须)
# ==========================
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1

# ==========================
# 生产环境优化 
# ==========================
fs.may_detach_mounts = 1
net.ipv4.conf.all.route_localnet = 1
vm.overcommit_memory = 1
vm.panic_on_oom = 0
fs.inotify.max_user_watches = 89100
fs.file-max = 52706963
fs.nr_open = 52706963
net.netfilter.nf_conntrack_max = 2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF

# 1. 加载模块
sudo sysctl --system

# 2. 验证 IPVS 模块是否加载成功
lsmod | grep --color=auto -e ip_vs -e nf_conntrack
```

## 部署keepAlived

注意：如果安装的不是高可用集群，haproxy和keepalived无需安装

注意：公有云要用公有云自带的负载均衡，比如阿里云的SLB、NLB，腾讯云的ELB，用来替代haproxy和keepalived，因为公有云大部分都是不支持keepalived的。

所有master节点安装

```bash
# 安装keepalived
yum install -y keepalived haproxy
```

### 配置HAProxy

```shell
# 创建目录
mkdir -p /etc/haproxy

# 写入配置
cat <<EOF > /etc/haproxy/haproxy.cfg
global
    maxconn 2000
    ulimit-n 16384
    log 127.0.0.1 local0 err
    stats timeout 30s

defaults
    log global
    mode http
    option httplog
    timeout connect 5000
    timeout client 50000
    timeout server 50000
    timeout http-request 15s
    timeout http-keep-alive 15s

# 监控页面 (可选)
frontend monitor-in
    bind *:33305
    mode http
    option httplog
    monitor-uri /monitor

# 前端配置：监听 16443 端口
frontend k8s-master
    bind 0.0.0.0:16443
    bind 127.0.0.1:16443
    mode tcp
    option tcplog
    tcp-request inspect-delay 5s
    default_backend k8s-master

# 后端配置：转发到真实的 Master 节点
backend k8s-master
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    # 请替换为你实际的 Master 节点 IP
    server k8s-master01 192.168.31.50:6443 check
    server k8s-master02 192.168.31.51:6443 check
    server k8s-master03 192.168.31.52:6443 check
EOF
```

### 配置KeepAlived

Master01

```bash
cat <<EOF > /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_DEVEL
    script_user root
    enable_script_security
}

vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state MASTER
    interface ens160          # 注意：修改为实际网卡名
    mcast_src_ip 192.168.31.50 # 注意：修改为本机 IP
    virtual_router_id 51
    priority 101             # 注意：Master02 改为 100，Master03 改为 99
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.31.100      # 注意：这是 VIP (虚拟 IP)
    }
    track_script {
        chk_apiserver
    }
}
EOF
```

Master2

```bash
cat <<EOF > /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_DEVEL
    script_user root
    enable_script_security
}

vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160          # 注意：修改为实际网卡名
    mcast_src_ip 192.168.31.51 # 注意：修改为本机 IP
    virtual_router_id 51
    priority 100             # 注意：Master02 改为 100，Master03 改为 99
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.31.100      # 注意：这是 VIP (虚拟 IP)
    }
    track_script {
        chk_apiserver
    }
}
EOF
```

master03

```shell
cat <<EOF > /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_DEVEL
    script_user root
    enable_script_security
}

vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160          # 注意：修改为实际网卡名
    mcast_src_ip 192.168.31.52 # 注意：修改为本机 IP
    virtual_router_id 51
    priority 99             # 注意：Master02 改为 100，Master03 改为 99
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.31.100      # 注意：这是 VIP (虚拟 IP)
    }
    track_script {
        chk_apiserver
    }
}
EOF
```

### 健康检查

```shell
cat <<EOF > /etc/keepalived/check_apiserver.sh
#!/bin/bash
# 检查本地 16443 端口 (HAProxy 端口) 是否存活
err=0
for k in \$(seq 1 3)
do
    check_code=\$(pgrep haproxy)
    if [[ \$check_code == "" ]]; then
        err=\$((expr \$err + 1))
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ \$err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi
EOF

# 赋予执行权限
chmod +x /etc/keepalived/check_apiserver.sh
```

所有master节点启动haproxy和keepalived

```shell
systemctl daemon-reload
systemctl enable --now haproxy
systemctl enable --now keepalived
```

==重要：如果安装了keepalived和haproxy，需要测试keepalived是否是正常的==

所有节点测试VIP

```shell
ping 192.168.31.100 -c 4

telnet 192.168.31.100 16443
```

如果ping不通且telnet没有出现 ]，则认为VIP不可以，不可在继续往下执行，需要排查keepalived的问题，比如防火墙和 selinux，haproxy和 keepalived的状态，监听端口等

所有节点查看防火墙状态必须为disable和inactive：systemctl status firewalld

所有节点查看selinux状态，必须为disable：getenforce
master节点查看haproxy和keepalived状态：systemctl status keepalived haproxy

master节点查看监听端口：netstat -lntp
如果以上都没有问题，需要确认：

1. 是否是公有云机器
2. 是否是私有云机器（类似OpenStack）

上述公有云一般都是不支持keepalived，私有云可能也有限制，需要和自己的私有云管理员咨询

## Runtime安装

如果安装的版本低于1.24，选择Docker和Containerd均可，高于1.24建议选择Containerd作为Runtime。

### 安装Containerd

配置安装源

```shell
# 1. 安装基础工具
yum install -y yum-utils device-mapper-persistent-data lvm2 git wget jq psmisc vim net-tools telnet

# 2. 添加阿里云 Docker 源
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 3. 安装 containerd 
# 这里推荐直接安装 containerd.io，更轻量
yum install -y containerd.io
```

加载内核模块

```shell
# 1. 配置开机自动加载模块
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

# 2. 立即加载模块
sudo modprobe overlay
sudo modprobe br_netfilter
```

配置内核参数

```shell
# 1. 配置 sysctl 参数
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 2. 应用配置
sudo sysctl --system
```

生成并优化 Containerd 配置

```shell
# 1. 创建配置目录
mkdir -p /etc/containerd

# 2. 生成默认配置文件
containerd config default | tee /etc/containerd/config.toml

# 3. 修改配置文件 (关键优化)
# 使用 sed 进行替换，确保 Cgroup 驱动为 systemd，且镜像仓库为阿里云
# 注意：如果你的 k8s 版本较新，必须开启 SystemdCgroup = true

sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

# 修改 Pause 镜像地址 (沙箱镜像)
sed -i 's|registry.k8s.io/pause|registry.cn-hangzhou.aliyuncs.com/google_containers/pause|g' /etc/containerd/config.toml

# 配置镜像加速 (可选，但强烈推荐)
sed -i '/\[plugins."io.containerd.grpc.v1.cri".registry.mirrors\]/a \ \ \ \ [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]\n \ \ \ \ \ \ endpoint = ["https://<你的阿里云加速器ID>.mirror.aliyuncs.com"]' /etc/containerd/config.toml
```

配置 crictl (可选)

```shell
cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

启动服务

```shell
# 1. 重新加载配置
systemctl daemon-reload

# 2. 启动并设置开机自启
systemctl enable --now containerd
```

## 安装K8s组件

配置源（注意更改版本号）

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

```shell
# 查看可用的 kubeadm 版本，并按版本倒序排列
yum list kubeadm.x86_64 --showduplicates | sort -r
```

```shell
# 这里的版本号后面通常要加 -0，具体看 yum list 的输出
# 例如：yum install kubeadm-1.31.0-0 kubelet-1.31.0-0 kubectl-1.31.0-0 -y

# 如果你只想安装 1.31 系列的最新版，可以用通配符（对应图片2）：
yum install kubeadm-1.31* kubelet-1.31* kubectl-1.31* -y
```

```shell
# 重新加载系统服务配置
systemctl daemon-reload

# 设置 kubelet 开机自启并立即启动
# 注意：此时启动会报错，这是正常的，忽略即可
systemctl enable --now kubelet
```

```shell
# 查看版本，确认安装成功
kubeadm version
kubelet --version
kubectl version --client
```

## 集群初始化

master01

```bash
# 创建文件夹
mkdir /usr/local/kubernetes/manifests -p
# 到manifests目录
cd /usr/local/kubernetes/manifests/
# 新建yaml文件
vi kubeadm-config.yaml
```

### 编写配置文件 (kubeadm-config.yaml)

```bash
# kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  # 修改为当前 Master 节点的 IP
  advertiseAddress: 192.168.31.50
  bindPort: 6443
nodeRegistration:
  # 容器运行时 Socket 路径，Containerd 默认是这个
  criSocket: unix:///var/run/containerd/containerd.sock
  name: k8s-master01
  taints:
    # 默认给 Master 节点打上污点，不运行普通 Pod
    - effect: NoSchedule
      key: node-role.kubernetes.io/control-plane
bootstrapTokens:
  - token: "7t2weq.bjbawausm0jaxury" # 自定义 Token，格式为 [a-z0-9]{6}\.[a-z0-9]{16}
    ttl: 24h0m0s
    usages:
      - signing
      - authentication
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.31.0 # 必须与你安装的 kubeadm 版本一致
# 高可用 VIP 地址，如果是单 Master，则填 Master IP
controlPlaneEndpoint: "192.168.31.100:16443"
networking:
  dnsDomain: cluster.local
  # Pod 网段，需与 CNI 插件配置一致
  podSubnet: 172.16.0.0/16
  # Service 网段
  serviceSubnet: 10.96.0.0/16
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers # 阿里云镜像仓库
apiServer:
  certSANs:
    # 如果有 VIP，务必加在这里；如果是单节点，加本机 IP
    - 192.168.31.100
```

### 配置迁移与镜像预拉取

```shell
#查看所需的镜像列表
kubeadm config images list --config kubeadm-config.yaml
#提前拉取镜像（所有 Master 节点执行）
kubeadm config images pull --config kubeadm-config.yaml
```

### 执行初始化 (Master01 节点)

```bash
kubeadm init --config kubeadm-config.yaml --upload-certs

#--upload-certs：这个参数非常关键，它会将证书加密上传到集群中，方便后续其他 Master 节点加入时自动下载证书，而不需要你手动拷贝。
```

### 配置 kubectl (初始化成功后)

```bash
# 执行下方命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**按照提示保存以下内容，一会要使用：**

```bash
# 其余master节点加入
kubeadm join 192.168.31.100:16443 --token jv5z7n.3y1zi95p952y9p65 \
    --discovery-token-ca-cert-hash sha256:403bca185c2f3a4791685013499e7ce58f9848e2213e27194b75a2e3293d8812 \
    --control-plane 

# 其余node节点加入
kubeadm join 192.168.31.100:16443 --token jv5z7n.3y1zi95p952y9p65 \
    --discovery-token-ca-cert-hash sha256:403bca185c2f3a4791685013499e7ce58f9848e2213e27194b75a2e3293d8812 \
```

> --control-plane ： 只有在添加master节点的时候才有

### 查看集群状态

```bash
# 查看节点
kubectl get nodes
# 查看pod
kubectl get pods -n kube-system
```

### Token 过期处理

Kubeadm 生成的 Token 默认有效期为 **24小时**。如果你错过了复制 Token 的时机，或者 Token 已经失效，新的节点将无法通过旧的 Token 加入集群。此时需要重新生成。

#### **生成新的 Join 命令**

在 **Master01（主管理节点）** 上执行以下命令，系统会自动生成一个新的 Token 并打印出完整的 `kubeadm join` 命令：

```shell
kubeadm token create --print-join-command
```

#### 补充证书上传

如果你需要加入的是新的 **Master 节点**（即带有 `--control-plane` 参数），除了 Token，还需要证书密钥。如果提示证书相关错误，需要执行以下命令上传证书：

```shell
kubeadm init phase upload-certs --upload-certs
```

- **注意**：该命令会生成一个短期的证书密钥（Certificate Key），有效期通常为 2 小时。在执行 `kubeadm join` 时，需要加上 `--certificate-key <生成的Key>` 参数。

## 初始化失败排查

**当 `kubeadm init` 初始化失败时，如何正确清理环境并重新初始化**

```shell
# 1. 重置 kubeadm 初始化状态 (-f 表示强制)
kubeadm reset -f

# 2. 清理 IPVS 规则 (如果使用了 IPVS 模式)
ipvsadm --clear

# 3. 删除用户目录下的 k8s 配置文件 (非常重要，否则再次初始化可能会读取旧的配置)
rm -rf ~/.kube
```

如果重置后再次初始化依然失败，不要盲目重试，需要查看系统日志来寻找根本原因。

- **CentOS/Rocky Linux / AlmaLinux:**

  ```shell
  tail -f /var/log/messages | grep -v "not found"
  ```

- **Ubuntu / Debian:**

  ```shell
  tail -f /var/log/syslog
  ```

#### Containerd 配置错误

- **现象**：报错通常涉及 CRI（容器运行时接口）连接失败，或者找不到 `criSocket`。
- 检查：
  - 确保 `containerd` 服务正在运行：`systemctl status containerd`
  - 确保 `config.toml` 中 `SystemdCgroup = true`（如果 k8s 配置要求一致）。
  - 确保 `crictl info` 能正常输出信息。

#### 配置文件（new.yaml）端口错误

- **现象**：连接被拒绝或超时。
- **原因**：图片特别提到“非高可用集群忘记修改 16443 端口为 6443”。
- 检查：
  - 如果你是单 Master 节点（非高可用），`controlPlaneEndpoint` 应该是 `MasterIP:6443`。
  - 如果你是高可用集群（带 VIP 或负载均衡），这里才是 `VIP:16443`（或其他负载均衡端口）。

#### 网段冲突

- **现象**：报错提示路由冲突或 CIDR 无效。
- **原因**：`new.yaml` 中的 `podSubnet`（Pod 网段）或 `serviceSubnet`（Service 网段）与物理机的 IP 网段重叠，或者两者之间重叠。
- 检查：
  - 物理机 IP：`192.168.1.0/24`
  - 配置中的 `serviceSubnet`：`10.96.0.0/12` (通常不冲突)
  - 配置中的 `podSubnet`：`172.16.0.0/16` (通常不冲突)
  - **务必确保这三个网段互不干扰。**

#### VIP（虚拟 IP）不通（针对高可用集群）

- **现象**：日志中出现 VIP 超时。
- **原因**：使用了 Keepalived + Haproxy 做高可用，但 VIP 没有漂移到 Master 节点上，或者防火墙拦截了心跳包。
- 检查：
  - 在 Master 节点执行 `ip addr`，查看是否有 VIP（例如 192.168.1.236）。
  - 检查防火墙是否关闭（`systemctl stop firewalld`）或是否正确放行了 VRRP 协议和端口。
  - 检查 Keepalived 服务状态。

## **安装网络插件（CNI）**

### 防止 NetworkManager 冲突（所有节点）

```bash
# 创建配置文件，告诉 NetworkManager 不要管理 Calico 的接口
cat <<EOF | sudo tee /etc/NetworkManager/conf.d/calico.conf
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:vxlan-v6.calico;interface-name:wireguard.cali;interface-name:wg-v6.cali
EOF

# 重载配置并重启 NetworkManager
sudo systemctl daemon-reload
sudo systemctl restart NetworkManager
```

### 准备 Calico 配置文件（仅在 Master01）

```bash
# 假设你已经克隆了仓库
cd /root/k8s-ha-install
git checkout manual-installation-v1.31.x
cd calico
```

### 修改 Pod 网段并应用（关键步骤）

```shell
# 1. 从 kube-controller-manager 的静态 Pod 清单中提取 cluster-cidr
# 注意：这里假设你是用 kubeadm 默认配置，清单文件在 /etc/kubernetes/manifests/ 下
POD_SUBNET=$(cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep cluster-cidr= | awk -F '=' '{print $NF}')

echo "检测到 Pod 网段为: $POD_SUBNET"

# 2. 使用 sed 命令替换 calico.yaml 中的网段配置
# 注意：图片中的 sed 命令稍微有点复杂，这里提供一个更直观的写法
# 找到包含 "CALICO_IPV4POOL_CIDR" 的行，去掉注释并替换值
sed -i "s|#* *CALICO_IPV4POOL_CIDR.*|          - name: CALICO_IPV4POOL_CIDR\n            value: \"$POD_SUBNET\"|g" calico.yaml

# 或者如果 calico.yaml 是通过 ConfigMap 定义的，通常只需要替换 CIDR 字符串：
# sed -i "s#192.168.0.0/16#$POD_SUBNET#g" calico.yaml
# (具体取决于 calico.yaml 文件内部是如何写的，请检查文件内容)

# 3. 应用配置
kubectl apply -f calico.yaml
```

###  验证状态

```shell
#查看 Pod 状态：
watch kubectl get pods -n kube-system
# 查看节点状态：
kubectl get nodes
```

## Metrics部署

### 前置准备：证书同步

Metrics Server 需要与 Kubelet 进行安全通信，因此需要访问 Kubelet 的 CA 证书。

```shell
# 在 Master01 上执行，将 IP 或主机名替换为实际的 Node 节点信息
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node01:/etc/kubernetes/pki/front-proxy-ca.crt
# 如果有更多节点，重复上述命令
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node02:/etc/kubernetes/pki/front-proxy-ca.crt
```

### 部署 Metrics Server

```shell
cd /root/k8s-ha-install/kubeadm-metrics-server

kubectl create -f comp.yaml

#检查 Pod 状态
kubectl get po -n kube-system -l k8s-app=metrics-server
#查看资源使用率
kubectl top node
kubectl top po -A
```

## 一些必须的配置更改

**将 Kube-proxy 的代理模式从默认的 iptables 切换为 ipvs**。因为在初始化集群的时候注释了ipvs配置，所有需要自行修改一下：

在master01节点执行：

```shell
kubectl edit cm kube-proxy -n kube-system

mode: "ipvs"

#强制重启 Kube-proxy Pod
kubectl patch daemonset kube-proxy -p \
'{"spec":{"template":{"metadata":{"annotations":{"date":"'$(date +'%s')'"}}}}}' \
-n kube-system

#验证模式是否切换成功
# 在 master01 节点执行（或者任意安装了 curl 的节点）
curl 127.0.0.1:10249/proxyMode
```

## 注意事项

### 证书有效期警告

kubeadm 安装的集群，默认生成的证书有效期为 **1年**。

#### 手动续期（临时应急）

```shell
#检查证书有效期
kubeadm certs check-expiration

#备份原有的证书
cp -rp /etc/kubernetes/pki/ /opt/pki.bak

#续签所有证书
kubeadm certs renew all

#重启控制平面组件
# 删除静态 Pod，让 kubelet 自动重建
rm -f /etc/kubernetes/manifests/kube-apiserver.yaml
rm -f /etc/kubernetes/manifests/kube-controller-manager.yaml
rm -f /etc/kubernetes/manifests/kube-scheduler.yaml
rm -f /etc/kubernetes/manifests/etcd.yaml

#验证新证书有效期
kubeadm certs check-expiration
```

#### 自动续期脚本

```shell
#!/bin/bash
# /usr/local/bin/k8s-cert-renew.sh

LOG_FILE="/var/log/k8s-cert-renew.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

echo "[$DATE] 开始检查证书..." >> $LOG_FILE

# 检查是否临近过期（剩余30天以内）
if kubeadm certs check-expiration | grep -q "expires in [0-9][0-9] days"; then
    echo "[$DATE] 证书即将过期，执行续签..." >> $LOG_FILE
    kubeadm certs renew all >> $LOG_FILE 2>&1

    # 重启控制平面组件
    for pod in kube-apiserver kube-controller-manager kube-scheduler etcd; do
        rm -f /etc/kubernetes/manifests/${pod}.yaml
        sleep 5  # 等待 kubelet 重建 Pod
    done

    echo "[$DATE] 证书续签完成，组件已重启。" >> $LOG_FILE
else
    echo "[$DATE] 证书无需续签。" >> $LOG_FILE
fi

# 每月1号凌晨2点执行
crontab -e
0 2 1 * * /usr/local/bin/k8s-cert-renew.sh
```

#### cert-manager（云原生推荐）

```shell
# 安装 cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# 创建 Issuer（以自签名为例）
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
EOF
```

### 不同组件的配置与重启机制

#### Kubelet（节点代理）

- 配置文件位置：

  - `/etc/sysconfig/kubelet`（环境变量配置）
  - `/var/lib/kubelet/config.yaml`（主配置文件）

- 生效方式：修改后必须手动重启进程。

  ```
  systemctl restart kubelet
  ```

#### B. 静态 Pod（Master 组件）

- **涉及组件**：`kube-apiserver`, `kube-scheduler`, `kube-controller-manager`, `etcd`。
- **配置文件位置**：`/etc/kubernetes/manifests/` 目录下的 yaml 文件（例如 `kube-apiserver.yaml`）。
- 生效方式：自动热更新。
  - Kubelet 会监控这个目录，一旦文件发生变化，它会自动销毁旧 Pod 并启动新 Pod。
  - **警告**：图片特别强调“**不能再次创建该文件**”，意思是不要删除这些文件，也不要尝试用 `kubectl apply` 去管理它们，直接编辑文件即可。

#### C. Kube-proxy（网络代理）

- **配置位置**：存储在 Etcd 中，表现为 `kube-system` 命名空间下的 ConfigMap。

- 修改命令：

  ```
  kubectl edit cm kube-proxy -n kube-system
  ```

- **生效方式**：修改 ConfigMap 不会立即生效，需要**强制重启 Pod**（即上一节提到的 patch 命令）。

### 解除 Master 节点的调度限制

默认情况下，为了安全起见，Master 节点会被打上“污点”（Taint），禁止普通 Pod 调度到 Master 上运行。如果你想在 Master 上也部署业务 Pod（例如测试环境或资源有限时），需要移除这个限制。

**执行命令：**

```
kubectl taint node --all node-role.kubernetes.io/control-plane:NoSchedule-
```

*注意：命令末尾的减号 `-` 非常重要，它代表“删除”这个污点。*

### 如何修改每个节点的 Pod 数量上限

在 Kubernetes 中，调整每个节点（Node）能运行的 Pod 数量主要涉及修改 **Kubelet** 的配置。默认情况下，这个数量通常被限制在 **110** 个。

#### 方法一：修改 Kubelet 配置文件（推荐）

```shell
#编辑 Kubelet 配置文件。
#路径通常是 /var/lib/kubelet/config.yaml（取决于你的安装方式，kubeadm 通常用这个）。
#或者在某些系统中是 /etc/sysconfig/kubelet 或 /etc/kubernetes/kubelet.conf。

# 在 /var/lib/kubelet/config.yaml 中添加
maxPods: 200  # 将数值改为你想要的数量，例如 200

systemctl restart kubelet
```

#### 方法二：通过命令行参数启动（较旧的方式）

```shell
#如果你是直接通过二进制文件或 systemd 管理 kubelet 启动参数的：
#找到 kubelet 的启动文件（例如 /etc/systemd/system/kubelet.service.d/10-kubeadm.conf）。

--max-pods=200

systemctl daemon-reload
systemctl restart kubelet
```

理论上可以把数字设得很大，但实际上受限于一下几个瓶颈：

- 默认值：110个
- 网络插件（CNI）的限制：不同的网络插件对 IP 地址的管理方式不同，这直接决定了上限
- 节点硬件资源
- Kubelet 的性能

# 生产可用集群的 8 大验收标准

所有节点必须是 `Ready`

- kubectl get node

系统组件（如 coredns, calico, metrics-server）必须都是 `Running`

- kubectl get po -A

确保 Pod IP 和 Service IP 没有重叠，且与物理网络不冲突

- kubectl get svc
- kubectl get po -A -o wide

资源创建能力

- kubectl create deploy cluster-test --image=registry.cn-beijing.aliyuncs.com/dotbalo/debug-tools -- sleep 3600

Pod 内部必须能解析 Service 名称

- 测试同命名空间：`nslookup kubernetes`
- 测试跨命名空间：`nslookup kube-dns.kube-system`

关键端口连通性

- 每个节点都必须能访问 APIServer（通常是 6443 或 VIP 的 443）。
- 每个节点都必须能访问 CoreDNS（通常是 ClusterIP 的 53 端口）。

Pod 间要能够正常通信（同namespace和跨namespace）（同机器和跨机器）



# k8s使用二进制搭建高可用集群

## ip划分

| 角色         | IP              |      |
| ------------ | --------------- | ---- |
| k8s-master01 | 192.168.31.50 |      |
| k8s-master02 | 192.168.31.51 |      |
| k8s-master03 | 192.168.31.52 |      |
| /            | 192.168.31.100 | VIP  |
| k8s-node01   | 192.168.31.55 |      |
| k8s-node02   | 192.168.31.56 |      |
| Pod网段      | 172.16.0.0/16   | /    |
| Service网段  | 10.96.0.0/16    | /    |

## 初始化操作

### 更改节点主机名

```bash
hostnamectl set-hostname k8s-master01
hostnamectl set-hostname k8s-master02
hostnamectl set-hostname k8s-master03
hostnamectl set-hostname k8s-node01
hostnamectl set-hostname k8s-node02
```

### 添加hosts

```shell
cat >> /etc/hosts << EOF
192.168.31.50 k8s-master01
192.168.31.51 k8s-master02
192.168.31.52 k8s-master03
192.168.31.55 k8s-node01
192.168.31.56 k8s-node02
EOF
```

### 安装必备工具

```shell
yum install -y \
  yum-utils \
  device-mapper-persistent-data \
  lvm2 \
  ipvsadm \
  ipset \
  conntrack \
  socat \
  chrony \
  nfs-utils \
  jq \
  git \
  vim \
  wget \
  curl \
  net-tools \
  telnet \
  lrzsz \
  psmisc \
  bash-completion

#yum-utils 
#device-mapper-persistent-data  Docker 和 Containerd 存储驱动（Devicemapper）的依赖。
#lvm2  逻辑卷管理，配合上面的包，用于容器存储管理。
#ipvsadm  K8s Service 的 IPVS 模式依赖此工具。
#ipset 
#conntrack 
#socat  Kubelet 依赖它进行端口转发（kubectl port-forward）。
#chrony 
#nfs-utils 如果你使用 NFS 作为持久化存储（PV），则必须安装。
#jq  命令行 JSON 处理工具。解析 kubectl get pod -o json 的输出必备。
#git 拉取 YAML 配置文件或代码。
#vim 文本编辑和日志记录的基础工具。
#wget/curl 下载文件、测试 API 连通性、健康检查。
#net-tools 
#telnet 测试端口连通性（如测试 Master 6443 端口是否通）。
#lrzsz 包含 rz / sz 命令，用于 Xshell 等终端直接上传下载文件，运维很方便。
#psmisc 包含 killall、pstree 等进程管理工具。
#bash-completion
```

### 关闭防火墙和selinux

```shell
# 关闭防火墙
systemctl disable --now firewalld
systemctl disable --now dnsmasq

# 关闭selinux
# 永久关闭
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config 
#修改selinux配置文件之后，重启机器，selinux配置才能永久生效，重启之后，登录到机器，
# 临时关闭
setenforce 0  
```

### 关闭swap

```shell
# 1. 临时关闭并调整参数
swapoff -a
sysctl -w vm.swappiness=0

# 2. 永久关闭 (注释 fstab)
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab

# 3. 让 swappiness 参数永久生效 (推荐补充这一步)
echo "vm.swappiness = 0" >> /etc/sysctl.conf

问题1：为什么要关闭swap交换分区？
Swap是交换分区，如果机器内存不够，会使用swap分区，但是swap分区的性能较低，k8s设计的时候为了能提升性能，默认是不允许使用交换分区的。Kubeadm初始化的时候会检测swap是否关闭，如果没关闭，那就初始化失败。如果不想要关闭交换分区，安装k8s的时候可以指定--ignore-preflight-errors=Swap来解决。
```

### 时间同步

```shell
yum install chrony -y
# 备份原配置
cp /etc/chrony.conf /etc/chrony.conf.bak
# 使用阿里云 NTP 源 (如果是内网环境，请指向内网 NTP 服务器)
sed -i 's/pool 2.centos.pool.ntp.org iburst/server ntp1.aliyun.com iburst/g' /etc/chrony.conf
systemctl enable chronyd --now
chronyc sources -v
```

### 系统参数限制 (Ulimit)

```shell
cat <<EOF >> /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
* soft nproc 65535
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
EOF
```

### 验证

```shell
free -m
# Swap 一栏应该全为 0
ulimit -n
# 输出应为 65536
ulimit -u
# 输出应为 65535
sysctl net.bridge.bridge-nf-call-iptables
# 输出应为 = 1
```

### 配置 SSH 免密登录

```shell
# 1. 生成密钥 (一路回车即可)
ssh-keygen -t rsa

# 2. 分发公钥 (图片中的写法)
# 注意：图片里的写法 $i.ssh 是不对的，应该是 ~/.ssh
# 正确的循环写法如下：
for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02; do ssh-copy-id -i ~/.ssh/id_rsa.pub $i; done
```

### 安装源码文件

master01 节点下载安装所有的源码文件

```shell
cd /root/ ;git clone https://gitee.com/dukuan/k8s-ha-install.git
```

### 内核配置

所有节点

```shell
# 安装必要的软件包
yum install -y ipvsadm ipset sysstat conntrack libseccomp

# 配置内核模块
vim /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_rpfilter
ipt_REJECT
ipip

# 启动服务
systemctl enable --now systemd-modules-load.service

#验证模块是否加载成功
lsmod | grep -e ip_vs -e nf_conntrack
```

### 内核优化配置

**`kube-proxy` 模式是 `ipvs`**,**必须做这些配置**

```shell
#配置内核模块（包含存储和网络）
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
# 存储驱动
overlay
# 网桥过滤
br_netfilter
# IPVS 负载均衡
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF

# 立即加载
sudo modprobe overlay
sudo modprobe br_netfilter
sudo modprobe ip_vs
sudo modprobe nf_conntrack
```

```shell
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
# ==========================
# 基础网络 (必须)
# ==========================
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1

# ==========================
# 生产环境优化 
# ==========================
fs.may_detach_mounts = 1
net.ipv4.conf.all.route_localnet = 1
vm.overcommit_memory = 1
vm.panic_on_oom = 0
fs.inotify.max_user_watches = 89100
fs.file-max = 52706963
fs.nr_open = 52706963
net.netfilter.nf_conntrack_max = 2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF

# 1. 加载模块
sudo sysctl --system

# 2. 验证 IPVS 模块是否加载成功
lsmod | grep --color=auto -e ip_vs -e nf_conntrack
```

## 部署keepAlived

注意：如果安装的不是高可用集群，haproxy和keepalived无需安装

注意：公有云要用公有云自带的负载均衡，比如阿里云的SLB、NLB，腾讯云的ELB，用来替代haproxy和keepalived，因为公有云大部分都是不支持keepalived的。

所有master节点安装

```bash
# 安装keepalived
yum install -y keepalived haproxy
```

### 配置HAProxy

```shell
# 创建目录
mkdir -p /etc/haproxy

# 写入配置
cat <<EOF > /etc/haproxy/haproxy.cfg
global
    maxconn 2000
    ulimit-n 16384
    log 127.0.0.1 local0 err
    stats timeout 30s

defaults
    log global
    mode http
    option httplog
    timeout connect 5000
    timeout client 50000
    timeout server 50000
    timeout http-request 15s
    timeout http-keep-alive 15s

# 监控页面 (可选)
frontend monitor-in
    bind *:33305
    mode http
    option httplog
    monitor-uri /monitor

# 前端配置：监听 16443 端口
frontend k8s-master
    bind 0.0.0.0:16443
    bind 127.0.0.1:16443
    mode tcp
    option tcplog
    tcp-request inspect-delay 5s
    default_backend k8s-master

# 后端配置：转发到真实的 Master 节点
backend k8s-master
    mode tcp
    option tcplog
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    # 请替换为你实际的 Master 节点 IP
    server k8s-master01 192.168.31.50:6443 check
    server k8s-master02 192.168.31.51:6443 check
    server k8s-master03 192.168.31.52:6443 check
EOF
```

### 配置KeepAlived

Master01

```bash
cat <<EOF > /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_DEVEL
    script_user root
    enable_script_security
}

vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state MASTER
    interface ens160          # 注意：修改为实际网卡名
    mcast_src_ip 192.168.31.50 # 注意：修改为本机 IP
    virtual_router_id 51
    priority 101             # 注意：Master02 改为 100，Master03 改为 99
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.31.100      # 注意：这是 VIP (虚拟 IP)
    }
    track_script {
        chk_apiserver
    }
}
EOF
```

Master2

```bash
cat <<EOF > /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_DEVEL
    script_user root
    enable_script_security
}

vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160          # 注意：修改为实际网卡名
    mcast_src_ip 192.168.31.51 # 注意：修改为本机 IP
    virtual_router_id 51
    priority 100             # 注意：Master02 改为 100，Master03 改为 99
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.31.100      # 注意：这是 VIP (虚拟 IP)
    }
    track_script {
        chk_apiserver
    }
}
EOF
```

master03

```shell
cat <<EOF > /etc/keepalived/keepalived.conf
global_defs {
    router_id LVS_DEVEL
    script_user root
    enable_script_security
}

vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160          # 注意：修改为实际网卡名
    mcast_src_ip 192.168.31.52 # 注意：修改为本机 IP
    virtual_router_id 51
    priority 99             # 注意：Master02 改为 100，Master03 改为 99
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.31.100      # 注意：这是 VIP (虚拟 IP)
    }
    track_script {
        chk_apiserver
    }
}
EOF
```

### 健康检查

```shell
cat <<EOF > /etc/keepalived/check_apiserver.sh
#!/bin/bash
# 检查本地 16443 端口 (HAProxy 端口) 是否存活
err=0
for k in \$(seq 1 3)
do
    check_code=\$(pgrep haproxy)
    if [[ \$check_code == "" ]]; then
        err=\$((expr \$err + 1))
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ \$err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi
EOF

# 赋予执行权限
chmod +x /etc/keepalived/check_apiserver.sh
```

所有master节点启动haproxy和keepalived

```shell
systemctl daemon-reload
systemctl enable --now haproxy
systemctl enable --now keepalived
```

==重要：如果安装了keepalived和haproxy，需要测试keepalived是否是正常的==

所有节点测试VIP

```shell
ping 192.168.31.100 -c 4

telnet 192.168.31.100 16443
```

如果ping不通且telnet没有出现 ]，则认为VIP不可以，不可在继续往下执行，需要排查keepalived的问题，比如防火墙和 selinux，haproxy和 keepalived的状态，监听端口等

所有节点查看防火墙状态必须为disable和inactive：systemctl status firewalld

所有节点查看selinux状态，必须为disable：getenforce
master节点查看haproxy和keepalived状态：systemctl status keepalived haproxy

master节点查看监听端口：netstat -lntp
如果以上都没有问题，需要确认：

1. 是否是公有云机器
2. 是否是私有云机器（类似OpenStack）

上述公有云一般都是不支持keepalived，私有云可能也有限制，需要和自己的私有云管理员咨询

## Runtime安装

如果安装的版本低于1.24，选择Docker和Containerd均可，高于1.24建议选择Containerd作为Runtime。

### 安装Containerd

配置安装源

```shell
# 1. 安装基础工具
yum install -y yum-utils device-mapper-persistent-data lvm2 git wget jq psmisc vim net-tools telnet

# 2. 添加阿里云 Docker 源
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 3. 安装 containerd 
# 这里推荐直接安装 containerd.io，更轻量
yum install -y containerd.io
```

加载内核模块

```shell
# 1. 配置开机自动加载模块
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

# 2. 立即加载模块
sudo modprobe overlay
sudo modprobe br_netfilter
```

配置内核参数

```shell
# 1. 配置 sysctl 参数
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 2. 应用配置
sudo sysctl --system
```

生成并优化 Containerd 配置

```shell
# 1. 创建配置目录
mkdir -p /etc/containerd

# 2. 生成默认配置文件
containerd config default | tee /etc/containerd/config.toml

# 3. 修改配置文件 (关键优化)
# 使用 sed 进行替换，确保 Cgroup 驱动为 systemd，且镜像仓库为阿里云
# 注意：如果你的 k8s 版本较新，必须开启 SystemdCgroup = true

sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

# 修改 Pause 镜像地址 (沙箱镜像)
sed -i 's|registry.k8s.io/pause|registry.cn-hangzhou.aliyuncs.com/google_containers/pause|g' /etc/containerd/config.toml

# 配置镜像加速 (可选，但强烈推荐)
sed -i '/\[plugins."io.containerd.grpc.v1.cri".registry.mirrors\]/a \ \ \ \ [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]\n \ \ \ \ \ \ endpoint = ["https://<你的阿里云加速器ID>.mirror.aliyuncs.com"]' /etc/containerd/config.toml
```

配置 crictl (可选)

```shell
cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

启动服务

```shell
# 1. 重新加载配置
systemctl daemon-reload

# 2. 启动并设置开机自启
systemctl enable --now containerd
```

## 安装K8s组件etcd

Master01下载kubernetes安装包

```bash
wget https://dl.k8s.io/v1.31.0/kubernetes-server-linux-amd64.tar.gz
```

下载etcd安装包

```shell
wget https://github.com/etcd-io/etcd/releases/download/v3.5.15/etcd-v3.5.15-linux-amd64.tar.gz
```

解压kubernetes安装文件

```shell
# --strip-components=3 用于去掉压缩包内的多余目录层级，直接提取 bin 目录下的文件
tar -xf kubernetes-server-linux-amd64.tar.gz --strip-components=3 -C /usr/local/bin kubernetes/server/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy}
```

解压etcd安装文件

```shell
# --strip-components=1 用于去掉压缩包内的顶级目录
tar -zxvf etcd-v3.5.15-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin etcd-v3.5.15-linux-amd64/etcd{,ctl}
```

版本查看

```shell
kubelet --version
etcdctl version 
```

将组件发送到其他节点

```shell
# 定义变量
MasterNodes='k8s-master02 k8s-master03'
WorkNodes='k8s-node01 k8s-node02'

# 循环分发：向其他 Master 节点发送全套组件 (包括 etcd)
for NODE in $MasterNodes; do
    echo "正在分发到 Master: $NODE"
    scp /usr/local/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy} $NODE:/usr/local/bin/
    scp /usr/local/bin/etcd* $NODE:/usr/local/bin/
done

# 循环分发：向 Worker 节点发送必要组件 (kubelet 和 kube-proxy)
for NODE in $WorkNodes; do
    echo "正在分发到 Worker: $NODE"
    scp /usr/local/bin/kube{let,-proxy} $NODE:/usr/local/bin/
done
```

切换配置分支

Master01节点切换到1.31.x分支（其他版本可以切换到其他分支，.x即可，不需要更改为具体的小版本）

```shell
# 进入项目目录并切换分支
cd /root/k8s-ha-install && git checkout manual-installation-v1.31.x
```

## 生成证书

### 下载证书生成工具 (cfssl)

Master01下载生成证书工具

```shell
# 下载 cfssl 工具
wget "https://pkg.cfssl.org/R1.2/cfssl_linux-amd64" -O /usr/local/bin/cfssl

# 下载 cfssljson 工具
wget "https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson

# 赋予执行权限
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```

### 创建证书存放目录

**在所有 Master 节点执行：**

```shell
mkdir /etc/etcd/ssl -p
```

**在所有节点（Master + Node）执行：**

```shell
mkdir -p /etc/kubernetes/pki
```

### 生成 Etcd CA 根证书

这是 etcd 集群信任的源头。使用 `cfssl gencert -initca` 命令基于配置文件生成自签名的 CA 证书和私钥。

**在 Master01 执行：**

```shell
# 进入配置目录（假设配置文件已准备好）
cd /root/k8s-ha-install/pki

# 生成 etcd CA 证书 (etcd-ca.pem) 和私钥 (etcd-ca-key.pem)
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca
```

### 生成 Etcd 服务端证书

这一步是为 etcd 集群生成具体的服务器证书。关键在于 `-hostname` 参数，必须包含所有 etcd 节点的 IP 和主机名，否则其他节点连接时会报错。

**在 Master01 执行：**

```shell
cfssl gencert \
    -ca=/etc/etcd/ssl/etcd-ca.pem \
    -ca-key=/etc/etcd/ssl/etcd-ca-key.pem \
    -config=ca-config.json \
    -hostname=127.0.0.1,k8s-master01,k8s-master02,k8s-master03,192.168.31.50,192.168.31.51,192.168.31.52 \
    -profile=kubernetes \
    etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd
```

- 参数解释：
  - `-ca` / `-ca-key`: 指定刚才生成的根证书和私钥。
  - `-hostname`: **非常重要**。这里列出了所有 Master 节点的主机名和 IP，确保集群内任何节点都能验证该证书。
  - `-profile=kubernetes`: 使用配置文件中定义的 kubernetes 策略（通常包含较长的有效期和特定的用途）。
  - `etcd-csr.json`: 证书签名请求文件，定义了组织、城市等信息。

### 将证书分发到其他 Master 节点

**在 Master01 执行脚本分发：**

```shell
# 定义其他 Master 节点变量
MasterNodes='k8s-master02 k8s-master03'

# 循环复制
for NODE in $MasterNodes; do
    # 1. 在远程节点创建目录
    ssh $NODE "mkdir -p /etc/etcd/ssl"
    
    # 2. 循环复制四个关键文件 (CA证书, CA私钥, etcd证书, etcd私钥)
    for FILE in etcd-ca-key.pem etcd-ca.pem etcd-key.pem etcd.pem; do
      scp /etc/etcd/ssl/$FILE $NODE:/etc/etcd/ssl/$FILE
    done
done
```

### 生成 Kubernetes CA 根证书

这是整个 K8s 集群信任链的源头。所有组件的证书都将由这个 CA 签发。

**在 Master01 执行：**

```shell
cd /root/k8s-ha-install/pki

# 生成 K8s CA 证书 (ca.pem) 和私钥 (ca-key.pem)
cfssl gencert -initca ca-csr.json | cfssljson -bare /etc/kubernetes/pki/ca
```

### 生成 apiserver 证书

apiserver 是集群的核心入口，它的证书必须包含所有可能访问它的 IP 和域名（包括 VIP、节点 IP、Service CIDR 的第一个 IP 等）

**在 Master01 执行：**

```shell
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -hostname=10.96.0.1,192.168.31.100,127.0.0.1,kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.default.svc.cluster.local,192.168.31.50,192.168.31.51,192.168.31.52 \
    -profile=kubernetes \
    apiserver-csr.json | cfssljson -bare /etc/kubernetes/pki/apiserver
```

> **注意：**
>
> - `10.96.0.1` 是 Service CIDR 的第一个 IP（通常是 kube-dns 的地址）。
> - `192.168.31.100` 是 Master 节点的 IP（如果是高可用集群，这里应该是 VIP；如果是单 Master，就是 Master01 的 IP）。
> - 后面的 IP 列表包含了所有 Master 节点的 IP。

### 生成 apiserver 的聚合证书 (front-proxy)

用于支持 Kubernetes 的 API Aggregation 功能（如 metrics-server）。

**在 Master01 执行：**

```shell
# 生成 front-proxy CA
cfssl gencert -initca front-proxy-ca-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-ca

# 生成 front-proxy client 证书
cfssl gencert \
    -ca=/etc/kubernetes/pki/front-proxy-ca.pem \
    -ca-key=/etc/kubernetes/pki/front-proxy-ca-key.pem \
    -config=ca-config.json \
    -profile=kubernetes \
    front-proxy-client-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-client
```

### 生成 controller-manager 证书及 kubeconfig

controller-manager 需要连接 apiserver，因此需要证书和 kubeconfig 文件。

**在 Master01 执行：**

```shell
# 生成证书
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -profile=kubernetes \
    manager-csr.json | cfssljson -bare /etc/kubernetes/pki/controller-manager

# 生成 kubeconfig (设置集群参数)
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://192.168.31.100:8443 \
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=/etc/kubernetes/pki/controller-manager.pem \
    --client-key=/etc/kubernetes/pki/controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# 设置上下文
kubectl config set-context system:kube-controller-manager@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# 切换默认上下文
kubectl config use-context system:kube-controller-manager@kubernetes \
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

> **注意：** `--server=https://192.168.31.100:8443` 中的 IP 和端口需根据实际情况修改（单 Master 用本机 IP，端口默认为 6443，高可用通常用 VIP 和 8443）。

### 生成 scheduler 证书及 kubeconfig

scheduler 同样需要连接 apiserver。

**在 Master01 执行：**

```shell
# 生成证书
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -profile=kubernetes \
    scheduler-csr.json | cfssljson -bare /etc/kubernetes/pki/scheduler

# 生成 kubeconfig (设置集群参数)
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://192.168.31.100:8443 \
    --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials system:kube-scheduler \
    --client-certificate=/etc/kubernetes/pki/scheduler.pem \
    --client-key=/etc/kubernetes/pki/scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

# 设置上下文
kubectl config set-context system:kube-scheduler@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-scheduler \
    --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

# 切换默认上下文
kubectl config use-context system:kube-scheduler@kubernetes \
    --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

### 生成 admin 证书及 kubeconfig

admin 证书用于管理员通过 `kubectl` 命令行工具管理集群。

**在 Master01 执行：**

```shell
# 生成证书
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -profile=kubernetes \
    admin-csr.json | cfssljson -bare /etc/kubernetes/pki/admin

# 生成 kubeconfig (设置集群参数)
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://192.168.31.100:8443 \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials kubernetes-admin \
    --client-certificate=/etc/kubernetes/pki/admin.pem \
    --client-key=/etc/kubernetes/pki/admin-key.pem \
    --embed-certs=true \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig

# 设置上下文
kubectl config set-context kubernetes-admin@kubernetes \
    --cluster=kubernetes \
    --user=kubernetes-admin \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig

# 切换默认上下文
kubectl config use-context kubernetes-admin@kubernetes \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

### 生成 ServiceAccount 密钥对

用于签发 ServiceAccount 的 Token。

**在 Master01 执行：**

```shell
openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

### 创建登录 Token (可选)

如果需要生成一个用于 Dashboard 或其他用途的 Token。

**在 Master01 执行：**

```shell
kubectl create token admin-user -n kube-system
```

### 创建 ServiceAccount (SA) 证书

ServiceAccount 是 Kubernetes 中的一种账号类型，主要用于 Pod 内部进程访问 API Server。API Server 会使用这里的私钥来签发 Token。

**在 Master01 执行：**

- **生成私钥**
  使用 OpenSSL 生成一个 2048 位的 RSA 私钥。

```shell
openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
```

> **返回结果说明**：显示 `Generating RSA private key, 2048 bit long modulus` 表示生成成功。

- **生成公钥**
  基于私钥提取出公钥。API Server 使用私钥签名，其他组件（如 Controller Manager）使用公钥验证 Token 的合法性。

```shell
openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

### 发送证书至其他节点

```shell
for NODE in k8s-master02 k8s-master03; do
    # 分发证书文件 (排除 etcd 目录)
    for FILE in $(ls /etc/kubernetes/pki | grep -v etcd); do
        scp /etc/kubernetes/pki/${FILE} $NODE:/etc/kubernetes/pki/${FILE};
    done;

    # 分发配置文件
    for FILE in admin.kubeconfig controller-manager.kubeconfig scheduler.kubeconfig; do
        scp /etc/kubernetes/${FILE} $NODE:/etc/kubernetes/${FILE};
    done;
done
```

### 查看证书文件 (验证)

最后，通过 `ls` 命令检查 `/etc/kubernetes/pki/` 目录，确认所有必要的证书（`.pem`）和签名请求（`.csr`）都已生成。

**在 Master01 执行：**

```shell
[root@k8s-master01 pki]# ls /etc/kubernetes/pki/
admin.csr               apiserver.csr           ca.csr                  controller-manager.csr    front-proxy-client.csr  sa.key                  scheduler-key.pem
admin-key.pem           apiserver-key.pem       ca-key.pem              controller-manager-key.pem front-proxy-client-key.pem sa.pub                  scheduler.pem
admin.pem               apiserver.pem           ca.pem                  controller-manager.pem     front-proxy-client.pem   scheduler.csr

# 统计文件数量，
[root@k8s-master01 pki]# ls /etc/kubernetes/pki/ | wc -l
23
```

## Etcd配置

### 编辑 Etcd 配置文件 (Master01)

```shell
# vim /etc/etcd/etcd.config.yml

name: 'k8s-master01'  # 当前节点名称，必须与 initial-cluster 中的名称一致
data-dir: /var/lib/etcd  # 数据存放目录
wal-dir: /var/lib/etcd/wal  # WAL（预写日志）目录，用于故障恢复
snapshot-count: 5000  # 每 5000 次事务进行一次快照
heartbeat-interval: 100  # 心跳间隔（毫秒），Leader 向 Follower 发送心跳的频率
election-timeout: 1000  # 选举超时时间（毫秒），Follower 未收到心跳多久后发起选举
quota-backend-bytes: 0  # 后端存储配额，0 表示默认（通常建议设为 8GB，即 8589934592）
listen-peer-urls: 'https://192.168.31.50:2380'  # 监听其他 Etcd 节点的通信地址（当前节点 IP）
listen-client-urls: 'https://192.168.31.50:2379,http://127.0.0.1:2379'  # 监听客户端访问地址（对外 HTTPS，本地 HTTP）
max-snapshots: 3  # 保留的最大快照数量
max-wals: 5  # 保留的最大 WAL 文件数量
cors:  # 跨域资源共享设置（通常留空或按需配置）
initial-advertise-peer-urls: 'https://192.168.31.50:2380'  # 通告给集群其他成员的地址
advertise-client-urls: 'https://192.168.31.50:2379'  # 通告给客户端的地址
discovery:  # 动态发现服务地址（静态集群模式下留空）
discovery-fallback: 'proxy'  # 发现失败时的回退模式
discovery-proxy:  # 代理地址（通常留空）
discovery-srv:  # DNS SRV 记录查询（通常留空）
initial-cluster: 'k8s-master01=https://192.168.31.50:2380,k8s-master02=https://192.168.31.51:2380,k8s-master03=https://192.168.31.52:2380'  # 集群所有成员列表
initial-cluster-token: 'etcd-k8s-cluster'  # 集群唯一标识，防止误加入其他集群
initial-cluster-state: 'new'  # 集群初始化状态，'new' 表示新建集群
enable-pprof: true  # 启用性能分析接口
proxy: 'off'  # 代理模式，'off' 表示关闭
proxy-failure-wait: 5000  # 代理失败等待时间（毫秒）
proxy-refresh-interval: 30000  # 代理刷新间隔（毫秒）
proxy-dial-timeout: 1000  # 代理拨号超时（毫秒）
proxy-write-timeout: 5000  # 代理写入超时（毫秒）
proxy-read-timeout: 0  # 代理读取超时（毫秒，0 表示无限制）

# 客户端 TLS 安全配置
client-transport-security:
  cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'  # 客户端证书
  key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'  # 客户端私钥
  client-cert-auth: true  # 是否要求客户端证书认证
  trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'  # 信任的 CA 证书
  auto-tls: false  # 是否自动使用 TLS（手动指定证书时为 false）

# 节点间 TLS 安全配置
peer-transport-security:
  cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'  # 节点间通信证书
  key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'  # 节点间通信私钥
  peer-client-cert-auth: true  # 是否要求对端节点证书认证
  trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'  # 信任的 CA 证书
  auto-tls: false  # 是否自动使用 TLS

debug: false  # 是否开启调试模式
log-package-levels:  # 日志包级别（留空表示默认）
log-outputs: [default]  # 日志输出位置
force-new-cluster: false  # 是否强制创建新集群（通常用于灾难恢复，正常启动为 false）
```

### 编辑 Etcd 配置文件 (Master02)

### 编辑 Etcd 配置文件 (Master03)

在 Master02 上执行相同操作，但需修改以下字段以匹配当前节点：

- `name`: `'k8s-master02'`
- `listen-peer-urls`: `'https://192.168.31.51:2380'`
- `listen-client-urls`: `'https://192.168.31.51:2379,http://127.0.0.1:2379'`
- `initial-advertise-peer-urls`: `'https://192.168.31.51:2380'`
- `advertise-client-urls`: `'https://192.168.31.51:2379'`

> **注意**：`initial-cluster` 字段在所有节点上保持一致，无需修改。

### 创建 Systemd 服务文件

在所有 Master 节点上创建 `/usr/lib/systemd/system/etcd.service`，用于管理 Etcd 进程的启动、停止和重启。

```shell
# vim /usr/lib/systemd/system/etcd.service

[Unit]
Description=Etcd Service
Documentation=https://coreos.com/etcd/docs/latest/
After=network.target  # 在网络服务启动后启动

[Service]
Type=notify  # 通知类型，systemd 会等待 etcd 发送就绪信号
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.config.yml  # 启动命令及配置文件路径
Restart=on-failure  # 失败时自动重启
RestartSec=10  # 重启间隔 10 秒
LimitNOFILE=65536  # 最大打开文件数限制

[Install]
WantedBy=multi-user.target  # 多用户模式下启动
Alias=etcd3.service  # 服务别名
```

### 创建证书目录并建立软链接

由于 Etcd 证书已生成在 `/etc/etcd/ssl/`，但配置文件中引用的是 `/etc/kubernetes/pki/etcd/`，因此需要创建目录并建立软链接。

```shell
# 所有 Master 节点执行
mkdir /etc/kubernetes/pki/etcd  # 创建目标目录
ln -s /etc/etcd/ssl/* /etc/kubernetes/pki/etcd/  # 建立软链接，将证书映射到配置路径
```

### 启动 Etcd 服务

在所有 Master 节点上执行以下命令，重载 systemd 配置、设置开机自启并立即启动服务。

```shell
systemctl daemon-reload  # 重载 systemd 配置
systemctl enable --now etcd  # 设置开机自启并立即启动
```

### 验证 Etcd 集群状态

在任意一个 Master 节点上（如 Master01），使用 `etcdctl` 工具检查集群健康状态。

```shell
# 设置环境变量，指定 API 版本和端点
export ETCDCTL_API=3
export ENDPOINTS="192.168.31.52:2379,192.168.31.51:2379,192.168.31.50:2379"

# 检查集群成员状态
etcdctl --endpoints=$ENDPOINTS \
  --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem \
  --cert=/etc/kubernetes/pki/etcd/etcd.pem \
  --key=/etc/kubernetes/pki/etcd/etcd-key.pem \
  endpoint status --write-out=table
```

## APIServer配置

### Master01配置

```shell
vim /usr/lib/systemd/system/kube-apiserver.service

[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
      --v=2  \
      --allow-privileged=true  \
      --bind-address=0.0.0.0  \
      --secure-port=6443  \
      --advertise-address=192.168.31.50 \
      --service-cluster-ip-range=10.96.0.0/16  \
      --service-node-port-range=30000-32767  \
      --etcd-servers=https://192.168.31.50:2379,https://192.168.31.51:2379,https://192.168.31.52:2379 \
      --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem \
      --etcd-certfile=/etc/etcd/ssl/etcd.pem \
      --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \
      --client-ca-file=/etc/kubernetes/pki/ca.pem \
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem \
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem \
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem \
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem \
      --service-account-key-file=/etc/kubernetes/pki/sa.pub \
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key \
      --service-account-issuer=https://kubernetes.default.svc.cluster.local \
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota \
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem \
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem \
      --requestheader-allowed-names=aggregator \
      --requestheader-group-headers=X-Remote-Group \
      --requestheader-extra-headers-prefix=X-Remote-Extra- \
      --requestheader-username-headers=X-Remote-User \
      # --token-auth-file=/etc/kubernetes/token.csv

Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

### 核心参数解析

#### 网络与端口

- `--bind-address=0.0.0.0`: 监听所有网络接口。
- `--secure-port=6443`: HTTPS 安全端口，客户端默认连接此端口。
- `--advertise-address=192.168.31.50`: **注意修改**。这是当前 Master01 节点的 IP 地址，用于告知集群其他成员如何访问它。在 Master02 和 Master03 上需分别改为对应的 IP。

#### Service 网段

- `--service-cluster-ip-range=10.96.0.0/16`: **重要**。定义了 K8s Service（虚拟 IP）的地址池。**必须确保此网段不与宿主机物理网络或 Pod 网络冲突**。

#### NodePort 范围

- `--service-node-port-range=30000-32767`: 定义 NodePort 类型 Service 的可用端口范围。

#### Etcd 连接配置 (高可用关键)

- `--etcd-servers=https://192.168.31.50:2379,https://192.168.31.51:2379,https://192.168.31.52:2379`: 指定了 Etcd 集群的所有节点地址。API Server 会将数据写入这里。
- `--etcd-cafile`, `--etcd-certfile`, `--etcd-keyfile`: 指定连接 Etcd 所需的 TLS 证书（之前步骤生成的）。

#### TLS/SSL 证书配置

- `--client-ca-file`: CA 根证书，用于验证客户端（如 kubelet）证书。
- `--tls-cert-file`, `--tls-private-key-file`: API Server 自身的服务端证书和私钥。
- `--kubelet-client-certificate`, `--kubelet-client-key`: API Server 访问 kubelet 时使用的客户端证书（用于执行 `kubectl exec` 等操作）。

#### Service Account 配置

- `--service-account-key-file`: 公钥 (`sa.pub`)，用于验证 Token。
- `--service-account-signing-key-file`: 私钥 (`sa.key`)，用于签发 Token。
- `--service-account-issuer`: Token 的签发者标识。

#### 聚合层配置 (Aggregation Layer)

- `--proxy-client-cert-file`, `--proxy-client-key-file`: 用于聚合 API（如 metrics-server）的代理客户端证书。
- `--requestheader-*`: 一系列参数，用于配置请求头认证，允许聚合 API Server 验证用户身份。

### Master02配置

### Master03配置

**修改** `--advertise-address` 为各自节点的 IP 地址

### 启动apiserver

```shell
systemctl daemon-reload && systemctl enable --now kube-apiserver
```

**检查状态**: 使用 `systemctl status kube-apiserver` 确认服务处于 `active (running)` 状态

## Controller Manager

需要在所有 Master 节点上创建文件 `/usr/lib/systemd/system/kube-controller-manager.service`。由于 Controller Manager 是无状态的，且通过选举产生 Leader，因此所有 Master 节点的配置完全相同。

```shell
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
      --v=2 \
      --root-ca-file=/etc/kubernetes/pki/ca.pem \
      --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \
      --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \
      --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
      --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
      --authentication-kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
      --authorization-kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
      --leader-elect=true \
      --use-service-account-credentials=true \
      --node-monitor-grace-period=40s \
      --node-monitor-period=5s \
      --controllers=*,bootstrapsigner,tokencleaner \
      --allocate-node-cidrs=true \
      --cluster-cidr=172.16.0.0/16 \
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \
      --node-cidr-mask-size=24
Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

### 核心参数解析

- 认证与授权：
  - `--authentication-kubeconfig` / `--authorization-kubeconfig`: 指定用于连接 API Server 的 kubeconfig 文件，包含证书和地址信息。
- 高可用选举：
  - `--leader-elect=true`: **关键参数**。启用领导者选举机制。在多个 Master 节点中，只有一个 Controller Manager 会作为 Leader 运行，其他节点处于 standby 状态。当 Leader 故障时，其他节点会选举出新的 Leader。
- Pod 网络分配 (CIDR)：
  - `--allocate-node-cidrs=true`: 启用 Node CIDR 分配。Controller Manager 会为每个新加入的 Node 分配一个 Pod IP 子网。
  - `--cluster-cidr=172.16.0.0/16`: **重要**。定义了整个集群 Pod 的 IP 地址池。**必须确保此网段不与宿主机物理网络或 Service 网络冲突**。
  - `--node-cidr-mask-size=24`: 定义每个 Node 节点分到的 Pod 子网大小（例如 /24 表示每个节点有 254 个可用 IP）。
- 证书管理：
  - `--cluster-signing-cert-file` / `--cluster-signing-key-file`: 指定用于签署 CSR（证书签名请求）的 CA 证书和私钥。
  - `--service-account-private-key-file`: 指定用于签署 Service Account Token 的私钥。

### 启动与验证命令

```shell
systemctl daemon-reload
systemctl enable --now kube-controller-manager

systemctl status kube-controller-manager

journalctl -u kube-controller-manager -f
```

## Scheduler

需要在所有 Master 节点上创建文件 `/usr/lib/systemd/system/kube-scheduler.service`。由于 Scheduler 也是通过选举产生 Leader，因此所有 Master 节点的配置完全相同。

```shell
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
      --v=2 \
      --kubeconfig=/etc/kubernetes/scheduler.kubeconfig \
      --leader-elect=true \
      --authentication-kubeconfig=/etc/kubernetes/scheduler.kubeconfig \
      --authorization-kubeconfig=/etc/kubernetes/scheduler.kubeconfig

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

### 核心参数解析

- 认证与授权：
  - `--authentication-kubeconfig=/etc/kubernetes/scheduler.kubeconfig`: 指定用于认证 API Server 的 kubeconfig 文件。
  - `--authorization-kubeconfig=/etc/kubernetes/scheduler.kubeconfig`: 指定用于授权 API Server 的 kubeconfig 文件。
- 高可用选举：
  - `--leader-elect=true`: **关键参数**。启用领导者选举机制。在多个 Master 节点中，只有一个 Scheduler 会作为 Leader 运行，其他节点处于 standby 状态。当 Leader 故障时，其他节点会选举出新的 Leader。
- 日志级别：
  - `--v=2`: 设置日志级别为 2，输出较详细的日志信息，便于调试。
- Kubeconfig 文件：
  - `--kubeconfig=/etc/kubernetes/scheduler.kubeconfig`: 指定连接 API Server 所需的 kubeconfig 文件，包含集群地址、证书等信息。

### 启动与验证命令

```shell
systemctl daemon-reload
systemctl enable --now kube-scheduler
systemctl status kube-scheduler
```

## TLS Bootstrapping 配置

### 创建 Bootstrap Kubeconfig 文件

这一步需要在 **Master01** 上执行。我们需要创建一个名为 `bootstrap-kubelet.kubeconfig` 的文件，这个文件包含了连接 API Server 所需的基本信息（CA 证书、Server 地址）以及用于初次认证的 Token。

**注意：** 请将命令中的 IP 地址 `192.168.31.100:8443` 替换为你实际的 **VIP（虚拟IP）** 或 **Load Balancer（负载均衡器）** 的地址和端口。

```shell
# 进入工作目录（假设你之前的脚本都在这个目录下）
cd /root/k8s-ha-install/bootstrap

# 1. 设置集群参数
# --certificate-authority: 指定 CA 根证书，用于验证 API Server 的身份
# --embed-certs=true: 将证书内容直接嵌入到 kubeconfig 文件中，而不是引用路径
# --server: 指定 API Server 的地址（这里通常是 VIP + 端口）
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://192.168.31.100:8443 \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig

# 2. 设置客户端认证参数
# --token: 指定之前生成的 Bootstrap Token。
# 注意：这里的 token=c8ad9c.2e4d610cf3e7426e 必须与你之前在 bootstrap.secret.yaml 中定义的 token 一致。
kubectl config set-credentials tls-bootstrap-token-user \
    --token=c8ad9c.2e4d610cf3e7426e \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig

# 3. 设置上下文参数
# 将用户和集群关联起来
kubectl config set-context tls-bootstrap-token-user@kubernetes \
    --cluster=kubernetes \
    --user=tls-bootstrap-token-user \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig

# 4. 设置默认上下文
kubectl config use-context tls-bootstrap-token-user@kubernetes \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

### 准备管理员 Kubeconfig (可选但推荐)

为了方便后续操作，通常会将 admin 的 kubeconfig 复制到 root 用户的默认目录下，这样执行 `kubectl` 命令时就不需要每次都指定 `--kubeconfig` 参数。

```shell
mkdir -p /root/.kube
cp /etc/kubernetes/admin.kubeconfig /root/.kube/config
```

### 验证集群核心组件状态

在继续配置 Node 节点之前，必须确保 Master 节点的核心组件（Controller Manager, Scheduler, Etcd）已经正常运行。

```shell
kubectl get cs
```

### 创建 Bootstrap 相关资源 (RBAC 授权)

为了让 Kubelet 能够使用上面的 Token 成功申请证书，我们需要在集群中创建相应的 Secret 和 RBAC 规则。这通常通过一个 YAML 文件来完成。

**创建文件 `bootstrap.secret.yaml`：**

```shell
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-c8ad9c
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  description: "The default bootstrap token generated by 'kubeadm init'."
  token-id: c8ad9c
  token-secret: 2e4d610cf3e7426e
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
  auth-extra-groups: system:bootstrappers:default-node-token,system:bootstrappers:worker,system:bootstrappers:ingress
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubelet-bootstrap
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node-bootstrapper
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:default-node-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-autoapprove-bootstrap
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:certificates.k8s.io:certificatesigningrequests:nodeclient
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:default-node-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-autoapprove-certificate-rotation
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:certificates.k8s.io:certificatesigningrequests:selfnodeclient
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:nodes
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kube-apiserver
```

```shell
kubectl create -f bootstrap.secret.yaml
```

## Node节点配置

### 复制证书与配置文件 (仅在 Master01 执行)

为了让其他节点能够加入集群，需要将 Master01 上的 CA 证书、Front-Proxy CA 证书以及之前生成的 Bootstrap Kubeconfig 文件分发到所有其他节点（Master02, Master03, Node01, Node02）。

```shell
# 切换到 kubernetes 配置目录
cd /etc/kubernetes/

# 使用循环脚本批量分发文件
# 注意：请确保你的主机名与脚本中的 k8s-master02 等一致，或者替换为实际 IP
for NODE in k8s-master02 k8s-master03 k8s-node01 k8s-node02; do
    # 在远程节点创建存放证书的目录
    ssh $NODE mkdir -p /etc/kubernetes/pki

    # 遍历需要分发的文件列表
    for FILE in pki/ca.pem pki/ca-key.pem pki/front-proxy-ca.pem bootstrap-kubelet.kubeconfig; do
        # 使用 scp 将文件从本地复制到远程节点的对应路径
        scp /etc/kubernetes/$FILE $NODE:/etc/kubernetes/${FILE}
    done
done
```

### 创建 Kubelet 相关目录 (所有节点执行)

在所有节点（Master 和 Node）上创建 Kubelet 运行所需的数据目录、日志目录以及 systemd 的扩展配置目录。

```shell
mkdir -p /var/lib/kubelet /var/log/kubernetes /etc/systemd/system/kubelet.service.d /etc/kubernetes/manifests/
```

### 配置 Kubelet Systemd Service (所有节点执行)

创建 Kubelet 的 systemd 启动文件。这个文件定义了 Kubelet 进程的基本行为（如重启策略）。

**文件路径：** `/usr/lib/systemd/system/kubelet.service`

```shell
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### 配置 Kubelet 启动参数 (所有节点执行)

这是最关键的一步。通过 systemd 的 drop-in 文件来传递具体的启动参数。这里使用的是 `containerd` 作为容器运行时。

**文件路径：** `/etc/systemd/system/kubelet.service.d/10-kubelet.conf`

```shell
[Service]
# 1. 引导参数：指定 bootstrap kubeconfig（用于初次申请证书）和正式 kubeconfig 的路径
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"

# 2. 系统参数：指定容器运行时 socket 地址 (containerd)
Environment="KUBELET_SYSTEM_ARGS=--container-runtime-endpoint=unix:///run/containerd/containerd.sock"

# 3. 配置参数：指定 Kubelet 的主配置文件路径
Environment="KUBELET_CONFIG_ARGS=--config=/etc/kubernetes/kubelet-conf.yml"

# 4. 额外参数：可以在这里添加节点标签等
Environment="KUBELET_EXTRA_ARGS=--node-labels=node.kubernetes.io/node=''"

# 5. 最终启动命令：组合上述所有变量
ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_SYSTEM_ARGS $KUBELET_EXTRA_ARGS
```

### 创建 Kubelet 主配置文件 (所有节点执行)

创建 YAML 格式的配置文件，定义 Kubelet 的详细行为（如认证授权方式、端口等）。

**注意：** 如果你的 Service 网段不是默认的 `10.96.0.0/12`，请务必修改 `clusterDNS` 为你们网段的第 10 个 IP（例如 `10.96.0.10`）。

**文件路径：** `/etc/kubernetes/kubelet-conf.yml`

```shell
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: systemd  # 必须与 containerd/docker 的 cgroup driver 一致
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
```

### 启动 Kubelet (所有节点执行)

重新加载 systemd 配置并启动 Kubelet。

```shell
systemctl daemon-reload
systemctl enable --now kubelet
```

###  验证状态与排错

启动后，查看系统日志。此时因为还没有安装网络插件（如 Calico），会出现 CNI 相关的报错，这是**正常现象**。

**查看日志命令：**

```shell
tail -f /var/log/messages
# 或者
journalctl -u kubelet -f
```

## kube-proxy配置

### 生成 kube-proxy 证书 (仅在 Master01 执行)

kube-proxy 需要证书来与 API Server 进行安全通信。这里使用 `cfssl` 工具生成证书。

**注意：** 请确保你已经有了 `ca-config.json` 和 `ca.pem`/`ca-key.pem` 文件（通常在之前的步骤中已生成）。

```shell
# 进入 PKI 目录
cd /root/k8s-ha-install/pki

# 生成 kube-proxy 的 CSR (证书签名请求) 并签署证书
# -ca: CA 根证书
# -ca-key: CA 私钥
# -config: CA 配置文件
# -profile: 使用的证书配置文件 (kubernetes)
# 最后通过 cfssljson 生成 pem 和 key 文件
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -profile=kubernetes \
    kube-proxy-csr.json | cfssljson -bare /etc/kubernetes/pki/kube-proxy
```

### 创建 kube-proxy.kubeconfig 文件 (仅在 Master01 执行)

这个文件包含了 kube-proxy 连接集群所需的认证信息。

**注意：** 请将 `192.168.31.100:8443` 替换为你实际的 **VIP（虚拟IP）** 或 **Load Balancer** 的地址和端口。如果是单 Master 非高可用集群，请改为 Master01 的 IP，端口通常为 6443。

```shell
# 1. 设置集群参数
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://192.168.31.100:8443 \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig

# 2. 设置客户端认证参数 (使用刚才生成的证书)
kubectl config set-credentials system:kube-proxy \
    --client-certificate=/etc/kubernetes/pki/kube-proxy.pem \
    --client-key=/etc/kubernetes/pki/kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig

# 3. 设置上下文参数
kubectl config set-context system:kube-proxy@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-proxy \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig

# 4. 设置当前上下文
kubectl config use-context system:kube-proxy@kubernetes \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

### 3. 分发 kubeconfig 文件 (在 Master01 执行)

将生成的 `kube-proxy.kubeconfig` 发送到所有其他 Master 和 Node 节点。

```shell
# 发送给其他 Master 节点
for NODE in k8s-master02 k8s-master03; do
    scp /etc/kubernetes/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig
done

# 发送给 Worker 节点
for NODE in k8s-node01 k8s-node02; do
    scp /etc/kubernetes/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig
done
```

### 创建 kube-proxy systemd 服务文件 (所有节点执行)

在所有节点上创建 `/usr/lib/systemd/system/kube-proxy.service` 文件。

```shell
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/etc/kubernetes/kube-proxy.yaml \
  --v=2

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

### 创建 kube-proxy 配置文件 (所有节点执行)

创建 `/etc/kubernetes/kube-proxy.yaml` 文件。这是 kube-proxy 的核心配置，决定了它使用什么模式（iptables 或 ipvs）以及连接哪个集群。

```shell
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
bindAddress: 0.0.0.0
clientConnection:
  kubeconfig: /etc/kubernetes/kube-proxy.kubeconfig
clusterCIDR: 172.16.0.0/12  # 注意：这里必须与你集群规划的 Pod CIDR 一致
hostnameOverride: ""       # 如果需要指定节点名，可在此修改，通常留空自动获取
mode: "ipvs"               # 推荐使用 ipvs 模式，性能优于 iptables
ipvs:
  scheduler: "rr"          # 调度算法，rr 为轮询
  excludeCIDRs: null
  strictARP: false
  tcpTimeout: 0
  tcpFinTimeout: 0
  udpTimeout: 0
iptables:
  masqueradeAll: false
  masqueradeBit: 14
  minSyncPeriod: 0s
  syncPeriod: 30s
ipvs:
  strictARP: false
  tcpTimeout: 0
  tcpFinTimeout: 0
  udpTimeout: 0
metricsBindAddress: 127.0.0.1:10249
nodePortAddresses: null
oomScoreAdj: -999
portRange: ""
udpIdleTimeout: 250ms
winkernel:
  enableDSR: false
  networkName: ""
  sourceVip: ""
```

### 启动 kube-proxy (所有节点执行)

配置完成后，启动服务并设置开机自启。

```shell
# 重新加载 systemd 配置
systemctl daemon-reload

# 启动 kube-proxy 并设置开机自启
systemctl enable --now kube-proxy

# 查看日志
journalctl -u kube-proxy -f
```

## 安装Calico

### 安装 Calico 网络插件

Calico 负责为 Pod 分配 IP 地址并建立节点间的网络连接。**注意：** 以下操作仅在 `master01` 执行。

#### 修改 Pod 网段

你需要将配置文件中的默认网段替换为你规划的实际 Pod 网段（例如 `172.16.0.0/16`）。

```shell
# 进入 calico 目录
cd /root/k8s-ha-install/calico/

# 使用 sed 命令批量替换网段
# 注意：请将 172.16.0.0/16 替换为你实际规划的 POD_CIDR
sed -i "s#POD_CIDR#172.16.0.0/16#g" calico.yaml
```

#### 验证网段配置

在应用配置前，务必检查替换是否成功。

```shell
# 检查 CALICO_IPV4POOL_CIDR 的值
grep "CALICO_IPV4POOL_CIDR" calico.yaml -A 1
```

> **预期输出：** 应显示 `value: "172.16.0.0/12"` (或者你设置的 /16)，确保这与你的规划一致。

```shell
kubectl apply -f calico.yaml
```

#### 故障排查

如果 Calico Pod 状态异常（如 `CrashLoopBackOff` 或 `Init:Error`），可以使用以下命令查看日志：

```shell
# 查看特定 Pod 的日志
kubectl logs -f <POD_NAME> -n kube-system

# 查看 init 容器日志（常见错误点）
kubectl logs -f <POD_NAME> -c upgrade-ipam -n kube-system
```

## 安装 CoreDNS

CoreDNS 是集群的内部 DNS 服务器，负责 Service 名称到 IP 的解析。

#### 获取 Service IP 并修改配置

CoreDNS 需要知道 Kubernetes API Server 的 Service IP（通常是 Service 网段的第 10 个 IP）。

```shell
cd /root/k8s-ha-install/

# 自动获取 kubernetes service 的 IP
COREDNS_SERVICE_IP=$(kubectl get svc | grep kubernetes | awk '{print $3}')

# 将获取到的 IP 替换到 CoreDNS 配置文件中
# 注意：这里是将占位符 KUBEDNS_SERVICE_IP 替换为实际 IP
sed -i "s#KUBEDNS_SERVICE_IP#${COREDNS_SERVICE_IP}#g" CoreDNS/coredns.yaml
```

```shell
kubectl create -f CoreDNS/coredns.yaml
```

## 部署 Metrics Server

Metrics Server 用于采集节点和 Pod 的 CPU、内存等监控数据，是 `kubectl top` 命令的基础。

#### 分发证书

Metrics Server 需要访问 Kubelet 的 HTTPS 接口，因此需要信任 Kubelet 的证书。这里使用的是 `front-proxy-ca.crt`。

```shell
# 在 master01 上执行，将证书复制到所有 Node 节点
# 请根据实际节点名称修改
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node01:/etc/kubernetes/pki/front-proxy-ca.crt
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node02:/etc/kubernetes/pki/front-proxy-ca.crt
# ... 其他节点自行拷贝
```

#### 安装 Metrics Server

```shell
cd /root/k8s-ha-install/kubeadm-metrics-server

# 应用组件配置
kubectl create -f comp.yaml
```

