+++
date = '2026-05-20T09:51:28+08:00'
draft = false
title = 'RedHat_RH436'
slug = "vfn3evs1kn"
description = "RedHat_RH436"
summary = "RedHat_RH436"
categories = ["📒考试"]
tags = ["RedHat"]
comments = true
series = ["RedHat 认证考试"]
showSeries = true
+++

# RH436 试题解法 - 高可用集群

## 0. 模拟环境制作

```bash
# 创建虚拟机
for i in node1 node2 node3 control; do
qemu-img create -f qcow2 /virtual/rh436-${i}-vda.qcow2 40G
sudo virt-install --name rh436-${i} --memory 512 --vcpus 4 --import --os-variant rhel8.7 --disk path=/virtual/rh436-${i}-vda.qcow2,bus=virtio --location http://10.173.28.219/web/cdrom/RedHat/8.9/ --network bridge=net-172.24.1.0,model=virtio --extra-args ks=http://10.173.28.219/web/ks/rhel-8.9.cfg --graphics vnc --noautoconsole
done

# 为 control 节点添加一块硬盘
qemu-img create -f qcow2 /virtual/rh436-control-vdb.qcow2 10G
sudo virsh attach-disk rh436-control /virtual/rh436-control-vdb.qcow2 vdb --driver qemu --subdriver qcow2 --persistent

# control
nmcli con mod enp1s0 ipv4.add 172.24.1.100/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.1.15/24
nmcli con mod enp1s0 +ipv4.add 192.168.2.15/24
nmcli con up enp1s0

yum install targetcli -y
systemctl enable targetclid.service --now
firewall-cmd --add-service=iscsi-target
firewall-cmd --add-service=iscsi-target --per
pvcreate /dev/vdb
vgcreate iscsi /dev/vdb
lvcreate -n rh436 -L 5G iscsi

targetcli
/backstores/block create iscsi-rh436 /dev/iscsi/rh436
/iscsi create iqn.2021-04.com.exampl:iscsi-rh436
/iscsi/iqn.2021-04.com.exampl:iscsi-rh436/tpg1/portals/ delete 0.0.0.0 ip_port=3260
/iscsi/iqn.2021-04.com.exampl:iscsi-rh436/tpg1/portals/ create ip_address=192.168.1.15 ip_port=3260
/iscsi/iqn.2021-04.com.exampl:iscsi-rh436/tpg1/portals/ create ip_address=192.168.2.15 ip_port=3260
/iscsi/iqn.2021-04.com.exampl:iscsi-rh436/tpg1/luns create /backstores/block/iscsi-rh436
/iscsi/iqn.2021-04.com.exampl:iscsi-rh436/tpg1/acls create iqn.2021-04.com.example:nodea
/iscsi/iqn.2021-04.com.exampl:iscsi-rh436/tpg1/acls create iqn.2021-04.com.example:nodeb
/iscsi/iqn.2021-04.com.exampl:iscsi-rh436/tpg1/acls create iqn.2021-04.com.example:nodec
saveconfig

# node1
nmcli con mod enp1s0 ipv4.add 172.24.1.101/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.0.101/24
nmcli con mod enp1s0 +ipv4.add 192.168.1.101/24
nmcli con mod enp1s0 +ipv4.add 192.168.2.101/24
nmcli con up enp1s0

# node2
nmcli con mod enp1s0 ipv4.add 172.24.1.102/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.0.102/24
nmcli con mod enp1s0 +ipv4.add 192.168.1.102/24
nmcli con mod enp1s0 +ipv4.add 192.168.2.102/24
nmcli con up enp1s0

# node3
nmcli con mod enp1s0 ipv4.add 172.24.1.103/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.0.103/24
nmcli con mod enp1s0 +ipv4.add 192.168.1.103/24
nmcli con mod enp1s0 +ipv4.add 192.168.2.103/24
nmcli con up enp1s0

# node1, node2, node3, control
cat > /etc/hosts << END
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.24.1.101 node1.domain1.example.com node1 nodea.private.example.com
172.24.1.102 node2.domain1.example.com node2 nodeb.private.example.com
172.24.1.103 node3.domain1.example.com node3 nodec.private.example.com
172.24.1.100 control.domain1.example.com control
END
```

## 1. 创建高可用集群

```bash
# 使用以下节点创建高可用集群
nodea.private.example.com
nodeb.private.example.com
nodec.private.example.com
# 满足以下要求：
# 集群名字设置为 cluster1
# 对于仲裁计算，每个群集成员都有相同的投票数

# 在三个node中执行
firewall-cmd --add-service=high-availability 
firewall-cmd --add-service=high-availability --per
yum install pcs fence-agents-ipmilan -y
systemctl enable pcsd --now
echo redhat | passwd --stdin hacluster

# nodea
[root@nodea ~]# pcs host auth nodea.private.example.com nodeb.private.example.com nodec.private.example.com
Username: hacluster
Password: 
[root@nodea ~]# pcs cluster setup cluster1 --start nodea.private.example.com nodeb.private.example.com nodec.private.example.com
[root@nodea ~]# pcs cluster enable --all
[root@nodea ~]# pcs cluster start --all
```

## 2. 配置集群隔离

```bash
# 集群使用 fence_ipmilan 隔离方法来隔离失败的集群节点，fence_ipmilan 的配置信息：
```

| Machine                   | BMC IP addr   | Login | Password | Cipher | Lanplus |
| ------------------------- | ------------- | ----- | -------- | ------ | ------- |
| nodea.private.example.com | 192.168.0.101 | admin | password | 3      | 1       |
| nodeb.private.example.com | 192.168.0.102 | admin | password | 3      | 1       |
| nodec.private.example.com | 192.168.0.103 | admin | password | 3      | 1       |

```bash
pcs stonith create fence_nodea fence_ipmilan pcmk_host_list=nodea.private.example.com ip=192.168.0.101 username=admin password=password lanplus=1 cipher=3 power_timeout=180
pcs stonith create fence_nodeb fence_ipmilan pcmk_host_list=nodeb.private.example.com ip=192.168.0.102 username=admin password=password lanplus=1 cipher=3 power_timeout=180
pcs stonith create fence_nodec fence_ipmilan pcmk_host_list=nodec.private.example.com ip=192.168.0.103 username=admin password=password lanplus=1 cipher=3 power_timeout=180

# 查看fence状态
pcs stonith

# 查看配置信息:
pcs stonith config

# 测试
ipmitool -I lanplus -H 192.168.0.101 -U admin -P password chassis power status
```

![image-20250301135115038](https://typora.fanyops.com/documents/2025/0301/image-20250301135115038.png)

## 3. 配置集群日志

```bash
# 按一下要求配置集群日志:
# nodea.private.example.com 中所有群集活动都记录在一个名为 /var/log/rh436-cluster-nodea.log 自定义日志文件中
# nodeb.private.example.com 中所有群集活动都记录在一个名为 /var/log/rh436-cluster-nodeb.log 自定义日志文件中
# nodec.private.example.com 中所有群集活动都记录在一个名为 /var/log/rh436-cluster-nodec.log 自定义日志文件中

# 先停止集群
# nodea
pcs cluster stop --all
vi /etc/corosync/corosync.conf
[root@nodea ~]# grep logfile /etc/corosync/corosync.conf
    to_logfile: yes
    logfile: /var/log/rh436-cluster-nodea.log
[root@nodea ~]# pcs cluster sync

# nodeb
[root@nodeb ~]# vi /etc/corosync/corosync.conf
[root@nodeb ~]# grep logfile /etc/corosync/corosync.conf
    to_logfile: yes
    logfile: /var/log/rh436-cluster-nodeb.log
    
# nodec
[root@nodec ~]# vi /etc/corosync/corosync.conf
[root@nodec ~]# grep logfile /etc/corosync/corosync.conf
    to_logfile: yes
    logfile: /var/log/rh436-cluster-nodec.log
    
# 重新启动集群
[root@nodea ~]# pcs cluster start --all
[root@nodea ~]# ll /var/log/rh436-cluster-nodea.log
[root@nodea ~]# ll /var/log/rh436-cluster-nodeb.log
[root@nodea ~]# ll /var/log/rh436-cluster-nodec.log
```

## 4. 配置集群监控

```bash
# 在 cluster436 资源组上配置监控，满足以下条件：
#  一个集群监控资源 clustermonitor 正在运行，并且发送能事件和通知的邮件
#  事件通知的标题是 ClusterAlert
#  邮件将发送给 student@workstation.lab.example.com

# 在三个node中执行
yum install postfix mailx -y
systemctl enable --now postfix

# nodea
pcs resource create clustermonitor MailTo email=student@workstation.lab.example.com subject="ClusterAlert" --group=cluster436
```

## 5. 配置一个集群故障转移服务

```bash
# 配置一个运行在集群上的 HTTP 服务，满足以下要求：
#  服务名为 webserver
#  服务使用的DocumentRoot 在后面配置的共享存储中
#  访问该web服务，显示：RH436 Training
#  通过 ip 172.25.250.80/24 来访问该服务
#  webserver 和 ip 都属于 cluster436 资源组

# 在三个node中执行
yum install httpd -y
firewall-cmd --add-service=http --permanent
firewall-cmd --reload

ssh root@nodea
mkdir /var/www/html
restorecon -Rv /var/www/html
echo "RH436 Training" > /var/www/html/index.html
pcs resource create webip IPaddr2 ip=172.25.250.80 cidr_netmask=24 --group=cluster436
pcs resource create webserver apache --group=cluster436
pcs constraint order start LVMshared-clone then cluster436
pcs constraint
```

## 6. 配置集群服务行为

```bash
# 配置资源组 cluster436 满足以下要求：
#  如果 nodeb 可用，资源组要在 nodeb 上运行
#  如果 nodeb 不可用，可以在 nodec 上运行，如果 nodeb 可用，不在 nodec 上运行
#  资源组永远不会在 nodea 上运行

pcs constraint location cluster436 prefers nodeb.private.example.com
pcs constraint location cluster436 avoids nodec.private.example.com
pcs constraint list

#删除规则，如果有需要
pcs constraint --full #查看规则id
pcs constraint delete location-cluster436-nodeb.private.example.com-INFINITY
```

## 7. 配置存储

```bash
# 先在 workstation 上执行 lab start multipath-review 配置以下节点：
#    nodea.private.example.com
#    nodeb.private.example.com
#    nodec.private.example.com
# 以访问 workstation 上的 iSCSI 存储 iqn.2021-04.com.example:store1 要求： 启动器名字分别设置为：
#    iqn.2021-04.com.example:nodea
#    iqn.2021-04.com.example:nodeb
#    iqn.2021-04.com.example:nodec
# 所有节点都能够通过 192.168.1.15 和 192.168.2.15 网络来登录 iSCSI 存储访问设备

lab start multipath-review

# 在三个node中执行：
yum install iscsi-initiator-utils -y
vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2021-04.com.example:nodeX（X分别是 a b c）
systemctl restart iscsid
iscsiadm -m discovery -t st -p 192.168.1.15
iscsiadm -m discovery -t st -p 192.168.2.15 
iscsiadm -m node -l

#分开登录
iscsiadm -m node -T iqn.2021-04.com.example:store1 -p 192.168.1.15 -l
iscsiadm -m node -T iqn.2021-04.com.example:store1 -p 192.168.2.15 -l

#登出：
iscsiadm -m node -T iqn.2021-04.com.example:store1 -u #登出
iscsiadm -m node -T iqn.2021-04.com.example:store1 -o delete #删除链路
```

## 8. 配置多路径访问共享存储

```bash
# 在以下节点配置存储多路径，多路径设备名称为 /dev/mapper/mpatha :
# nodea.private.example.com
# nodeb.private.example.com
# nodec.private.example.com

# 在三个node中执行：
yum install -y device-mapper-multipath
mpathconf --enable
ll /etc/multipath.conf #验证
systemctl enable --now multipathd
#查看/dev/mapper/mpatha设备是否生成
fdisk -l
```

## 9. 配置共享 LVM 设备

### 题目要求

基于 /dev/mapper/mpatha 配置共享逻辑卷满足以下要求:

1. 逻辑卷属于 rh436vg 卷组
2. 逻辑卷名字为 webdata 且大小为 1024MiB
3. webdata 存在一个文件系统，需要在后续的任务中配置
4. 在集群中的一个节点上创建的文件应该可以从集群中的另一个节点可读

### 解法步骤

#### 1. 安装必要的软件包

在三个节点上安装 `lvm2-lockd` 和 `dlm` 软件包：

```bash
yum install lvm2-lockd dlm -y
```

- **`lvm2-lockd`**：提供 LVM 的锁管理功能，支持共享卷组。
- **`dlm`**：分布式锁管理器，用于在集群中协调资源的访问。

#### 2. 配置集群资源

在 `nodea` 上创建 `dlm` 和 `lvmlockd` 资源，并将其放入 `locking` 资源组:

```bash
pcs resource create dlm ocf:pacemaker:controld op monitor interval=30s on-fail=fence --group=locking
pcs resource create lvmlockd ocf:heartbeat:lvmlockd op monitor interval=30s on-fail=fence --group=locking
pcs resource clone locking interleave=true
```

- **`dlm`**：用于管理分布式锁。
- **`lvmlockd`**：用于管理 LVM 的锁。
- **`clone`**：将资源组克隆到所有节点，确保每个节点都可以访问锁。

#### 3. 创建共享卷组和逻辑卷

在 `nodea` 上创建物理卷和共享卷组:

```bash
pvcreate /dev/mapper/mpatha
vgcreate --shared rh436vg /dev/mapper/mpatha
```

- **`pvcreate`**：初始化物理卷。
- **`vgcreate --shared`**：创建共享卷组，支持多节点访问。

在 `nodeb` 和 `nodec` 上启动卷组的锁管理：

```bash
vgchange --lockstart rh436vg
```

- **`vgchange --lock-start`**：启动卷组的锁管理功能，确保卷组可以在集群中共享。

在 `nodea` 上创建逻辑卷:

```bash
lvcreate --activate sy -L 1024M -n webdata rh436vg
```

- **`lvcreate`**：创建逻辑卷 `webdata`，大小为 1024MiB。
- **`--activate sy`**：激活逻辑卷，并确保其在共享模式下可用。

#### 4 创建逻辑卷资源

在 `nodea` 上创建逻辑卷资源，并将其与集群资源关联:

```bash
pcs resource create sharedlvm LVM-activate vgname=rh436vg lvname=webdata activation_mode=shared vg_access_mode=lvmlockd --group=LVMshared
pcs resource clone LVMshared interleave=true
pcs constraint colocation add LVMshared-clone with locking-clone
pcs constraint order start locking-clone then LVMshared-clone
```

- **`LVM-activate`**：用于激活逻辑卷的 Pacemaker 资源代理。
- **`activation_mode=shared`**：指定逻辑卷在共享模式下激活。
- **`vg_access_mode=lvmlockd`**：指定卷组使用 `lvmlockd` 进行锁管理。
- **`clone`**：将逻辑卷资源克隆到所有节点。
- **`colocation`**：确保逻辑卷资源与锁资源在同一节点上运行。
- **`order`**：确保锁资源先启动，然后启动逻辑卷资源。

#### 5. 验证配置

```bash
pcs constraint list
pcs status --full
```

- **`pcs constraint list`**：列出所有约束，确保逻辑卷资源与锁资源的依赖关系正确。
- **`pcs status --full`**：查看集群资源的详细状态，确保所有资源正常运行。

## 10. 配置 GFS2 文件系统

### 题目要求

在 webdata 上创建一个 GFS2 文件系统，满足以下条件：

1. GFS2 文件系统应当能够在集群启动时挂载在 nodea nodeb nodec 上
2. 文件系统应当挂载在 /var/www
3. GFS2 文件系统的日志数恰好比所需的最低日志个数多1

### 解法步骤

#### 1. 安装 GFS2 工具

安装 GFS2 工具:

```bash
yum install gfs2-utils -y
```

#### 2. 配置 Pacemaker 属性

在 `nodea` 上执行:

```bash
pcs property set no-quorum-policy=freeze
```

#### 3. 查找帮助文档

```bash
man 7 pacemaker-schedulerd
```

#### 4. 验证属性配置

```bash
pcs property list --all | grep -i no
```

#### 5. 创建 GFS2 文件系统

```bash
mkfs.gfs2 -j 4 -t cluster1:gfsdata /dev/rh436vg/webdata
```

#### 6. 创建文件系统资源

```bash
pcs resource create clusterfs Filesystem device=/dev/rh436vg/webdata directory=/var/www fstype=gfs2 op monitor interval=10s on-fail=fence --group=LVMshared
```

#### 7. 验证配置

```bash
pcs status --full
```

## 11. 利用 Alert 来监控集群

### 题目要求

配置 Alert，将任何集群活动的电子邮件发送到 `student@workstation.lab.example.com`。

### 解法配置

#### 1. 复制 Alert 脚本

在三个节点上执行：

```bash
cp /usr/share/pacemaker/alerts/alert_smtp.sh.sample /var/lib/pacemaker/alert_smtp.sh
chown hacluster:haclient /var/lib/pacemaker/alert_smtp.sh
chmod 755 /var/lib/pacemaker/alert_smtp.sh
```

#### 2. 验证脚本权限

```bash
ll /var/lib/pacemaker/alert_smtp.sh
```

#### 3. 创建 Alert

在 `nodea` 上执行：

```bash
pcs alert create id=mailme path=/var/lib/pacemaker/alert_smtp.sh
```

#### 4.  配置 Alert 接收者

```bash
pcs alert recipient add mailme value=student@workstation.lab.example.com
```

