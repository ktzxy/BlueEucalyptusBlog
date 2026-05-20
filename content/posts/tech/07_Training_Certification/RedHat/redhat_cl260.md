+++
date = '2026-05-20T09:50:04+08:00'
draft = false
title = 'RedHat_CL260'
slug = "qv79f4b4d2"
description = "RedHat_CL260"
summary = "RedHat_CL260"
categories = ["📒考试"]
tags = ["RedHat"]
comments = true
series = ["RedHat 认证考试"]
showSeries = true
+++

# CL260 考试环境练习

## 1. 环境准备

### 1.1 kvm 虚拟机准备

```bash
# 准备系统盘
qemu-img create -f qcow2 /virtual/ceph-serverc-vda.qcow2 40G
qemu-img create -f qcow2 /virtual/ceph-serverd-vda.qcow2 40G
qemu-img create -f qcow2 /virtual/ceph-servere-vda.qcow2 40G
qemu-img create -f qcow2 /virtual/ceph-clienta-vda.qcow2 40G

# 使用 ks 安装 almalinux8 系统
virt-install --name ceph-serverc --memory 4096 --vcpus 4 --import --os-variant almalinux8 --disk path=/virtual/ceph-serverc-vda.qcow2,bus=virtio --location http://192.168.122.248/web/cdrom/rocky/8.10/ --network bridge=virbr0,model=virtio --extra-args ks=http://192.168.122.248/web/ks8.10.cfg --graphics vnc --noautoconsole

virt-install --name ceph-serverd --memory 4096 --vcpus 4 --import --os-variant almalinux8 --disk path=/virtual/ceph-serverd-vda.qcow2,bus=virtio --location http://192.168.122.248/web/cdrom/rocky/8.10/ --network bridge=virbr0,model=virtio --extra-args ks=http://192.168.122.248/web/ks8.10.cfg --graphics vnc --noautoconsole

virt-install --name ceph-servere --memory 4096 --vcpus 4 --import --os-variant almalinux8 --disk path=/virtual/ceph-servere-vda.qcow2,bus=virtio --location http://192.168.122.248/web/cdrom/rocky/8.10/ --network bridge=virbr0,model=virtio --extra-args ks=http://192.168.122.248/web/ks8.10.cfg --graphics vnc --noautoconsole

virt-install --name ceph-clienta --memory 4096 --vcpus 4 --import --os-variant almalinux8 --disk path=/virtual/ceph-clienta-vda.qcow2,bus=virtio --location http://192.168.122.248/web/cdrom/rocky/8.10/ --network bridge=virbr0,model=virtio --extra-args ks=http://192.168.122.248/web/ks8.10.cfg --graphics vnc --noautoconsole

# 删除附加盘
rm -rf /virtual/ceph-serverc-vdb.qcow2
rm -rf /virtual/ceph-serverc-vdc.qcow2
rm -rf /virtual/ceph-serverc-vdd.qcow2

rm -rf /virtual/ceph-serverd-vdb.qcow2
rm -rf /virtual/ceph-serverd-vdc.qcow2
rm -rf /virtual/ceph-serverd-vdd.qcow2

rm -rf /virtual/ceph-servere-vdb.qcow2
rm -rf /virtual/ceph-servere-vdc.qcow2
rm -rf /virtual/ceph-servere-vdd.qcow2

rm -rf /virtual/ceph-clienta-vdb.qcow2
rm -rf /virtual/ceph-clienta-vdc.qcow2
rm -rf /virtual/ceph-clienta-vdd.qcow2

# 添加附加盘
qemu-img create -f qcow2 /virtual/ceph-serverc-vdb.qcow2 10G
qemu-img create -f qcow2 /virtual/ceph-serverc-vdc.qcow2 20G
qemu-img create -f qcow2 /virtual/ceph-serverc-vdd.qcow2 50G
virsh attach-disk ceph-serverc /virtual/ceph-serverc-vdb.qcow2 vdb --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-serverc /virtual/ceph-serverc-vdc.qcow2 vdc --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-serverc /virtual/ceph-serverc-vdd.qcow2 vdd --driver qemu --subdriver qcow2 --persistent

qemu-img create -f qcow2 /virtual/ceph-serverd-vdb.qcow2 10G
qemu-img create -f qcow2 /virtual/ceph-serverd-vdc.qcow2 20G
qemu-img create -f qcow2 /virtual/ceph-serverd-vdd.qcow2 50G
virsh attach-disk ceph-serverd /virtual/ceph-serverd-vdb.qcow2 vdb --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-serverd /virtual/ceph-serverd-vdc.qcow2 vdc --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-serverd /virtual/ceph-serverd-vdd.qcow2 vdd --driver qemu --subdriver qcow2 --persistent

qemu-img create -f qcow2 /virtual/ceph-servere-vdb.qcow2 10G
qemu-img create -f qcow2 /virtual/ceph-servere-vdc.qcow2 20G
qemu-img create -f qcow2 /virtual/ceph-servere-vdd.qcow2 50G
virsh attach-disk ceph-servere /virtual/ceph-servere-vdb.qcow2 vdb --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-servere /virtual/ceph-servere-vdc.qcow2 vdc --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-servere /virtual/ceph-servere-vdd.qcow2 vdd --driver qemu --subdriver qcow2 --persistent

qemu-img create -f qcow2 /virtual/ceph-clienta-vdb.qcow2 10G
qemu-img create -f qcow2 /virtual/ceph-clienta-vdc.qcow2 20G
qemu-img create -f qcow2 /virtual/ceph-clienta-vdd.qcow2 50G
virsh attach-disk ceph-clienta /virtual/ceph-clienta-vdb.qcow2 vdb --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-clienta /virtual/ceph-clienta-vdc.qcow2 vdc --driver qemu --subdriver qcow2 --persistent
virsh attach-disk ceph-clienta /virtual/ceph-clienta-vdd.qcow2 vdd --driver qemu --subdriver qcow2 --persistent
```

### 1.2 系统配置

```bash
# 登录 ceph-serverc
virsh console ceph-serverc
## 配置网络
nmcli con modify enp1s0 ipv4.add 192.168.122.51/24 ipv4.gate 192.168.122.1 ipv4.dns 192.168.122.1 ipv4.meth man ipv6.meth disab
nmcli con up enp1s0
## 设置主机名
hostnamectl set-hostname ceph-serverc
cat << 'END' >> /etc/hosts
192.168.122.51 ceph-serverc
192.168.122.52 ceph-serverd
192.168.122.53 ceph-servere
192.168.122.50 ceph-clienta
END
## 生成 ssh 密钥对
ssh-keygen -f ~/.ssh/id_rsa -P '' -q
## 安装 sshpass
yum install sshpass -y


# 登录 ceph-serverd
virsh console ceph-serverd
nmcli con modify enp1s0 ipv4.add 192.168.122.52/24 ipv4.gate 192.168.122.1 ipv4.dns 192.168.122.1 ipv4.meth man ipv6.meth disab
nmcli con up enp1s0
hostnamectl set-hostname ceph-serverd
cat << 'END' >> /etc/hosts
192.168.122.51 ceph-serverc
192.168.122.52 ceph-serverd
192.168.122.53 ceph-servere
192.168.122.50 ceph-clienta
END
ssh-keygen -f ~/.ssh/id_rsa -P '' -q
yum install sshpass -y


# 登录 ceph-servere
virsh console ceph-servere
nmcli con modify enp1s0 ipv4.add 192.168.122.53/24 ipv4.gate 192.168.122.1 ipv4.dns 192.168.122.1 ipv4.meth man ipv6.meth disab
nmcli con up enp1s0
hostnamectl set-hostname ceph-servere
cat << 'END' >> /etc/hosts
192.168.122.51 ceph-serverc
192.168.122.52 ceph-serverd
192.168.122.53 ceph-servere
192.168.122.50 ceph-clienta
END
ssh-keygen -f ~/.ssh/id_rsa -P '' -q
yum install sshpass -y


# 登录 ceph-clienta
virsh console ceph-clienta
nmcli con modify enp1s0 ipv4.add 192.168.122.50/24 ipv4.gate 192.168.122.1 ipv4.dns 192.168.122.1 ipv4.meth man ipv6.meth disab
nmcli con up enp1s0
hostnamectl set-hostname ceph-clienta
cat << 'END' >> /etc/hosts
192.168.122.51 ceph-serverc
192.168.122.52 ceph-serverd
192.168.122.53 ceph-servere
192.168.122.50 ceph-clienta
END
ssh-keygen -f ~/.ssh/id_rsa -P '' -q
yum install sshpass -y


# 分别在 ceph-serverc， ceph-serverd， ceph-servere, ceph-clienta 设置免密
sshpass -p'3wsxzaq1`' ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa root@ceph-serverc
sshpass -p'3wsxzaq1`' ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa root@ceph-serverd
sshpass -p'3wsxzaq1`' ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa root@ceph-servere
sshpass -p'3wsxzaq1`' ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa root@ceph-clienta
```

### 1.3 常用脚本及变量定义

```bash
# 定义变量
cat > vars << 'END'
username=root
all_address=(
    192.168.122.50
    192.168.122.51
    192.168.122.52
    192.168.122.53
)
all_fqdn=(
    192.168.122.50 ceph-clienta
    192.168.122.51 ceph-serverc
    192.168.122.52 ceph-serverd
    192.168.122.53 ceph-servere
);
all_hostname=(
    ceph-clienta
    ceph-serverc
    ceph-serverd
    ceph-servere
);
END

# 定义函数
cat > func << 'END'
function line() {
    echo -e "\033[33m#################### ${1} ##################\033[0m"
}
function check() {
    if [[ $? == 0 ]]; then
        echo -e "\033[32m${1} --------> success\033[0m";
    else
        echo -e "\033[31m${1} --------> fail\033[0m";
    fi
}
function modifyConf() {
    local file_path="${1}"
    local pattern="${2}"
    local replacement="${3}"
    
    sed -i "/${pattern}/s/^#//; s|^${pattern}=.*|${pattern}=\"${replacement}\"|" "${file_path}"
    grep "^${pattern}=" "${file_path}"
}
function clearConf() {
    local conf_file="${1}"
    local backup_file="${conf_file}.bak"

    ls "${backup_file}" &>/dev/null || cp "${conf_file}"{,.bak}

    sed -i '/^#/d; /^$/d' "${conf_file}" && ls "${backup_file}"
}
function appendConf() {
    local file_path="${1}"
    local section="${2}"
    local config_item="${3}"

    reverse_lines=$(echo -e "$config_item" | tac)

    count=0; tmpfile=$(mktemp)

    echo "$reverse_lines" | while IFS= read -r line; do
        ((count++))
        echo "${count}" > "$tmpfile"
        grep "\[${section}\]" -A${count} "${file_path}" | grep -Fxq "${line}"
        if [ $? -ne 0 ]; then
            sed -i "/\[${section}\]/a ${line}" "${file_path}"
        fi
    done

    count=$(cat "${tmpfile}") && rm -rf "${tmpfile}"

    grep "\[${section}\]" -A${count} "${file_path}"
}
END
. func

# ssh 批量操作
cat > ssh-all-node << 'END'
#!/bin/bash
source ./vars
source ./func

for (( i = 0; i < ${#all_address[*]}; i++ )); do
    line "${all_address[i]}"
    ssh $username@${all_address[i]} ${1}
    check "${1}"
done
END

cat > scp-all-node << 'END'
#!/bin/bash
source ./vars
source ./func

for (( i = 0; i < ${#all_address[*]}; i++ )); do
    line "${all_address[i]}"
    scp -r ${1} ${username}@${all_address[i]}:${2} &>/dev/null
    check "scp -r ${1} $username@${all_address[i]}:${2}"
done
END

chmod +x ssh-all-node scp-all-node
```

### 1.4 时间同步

```bash
# all node
./ssh-all-node "yum install chrony -y"
./ssh-all-node "sed -i '/^pool/d' /etc/chrony.conf && sed -i '/server/a pool ceph-clienta iburst' /etc/chrony.conf"

# ntp server
sed -i "s|^#allow.*|allow 0.0.0.0/0|" /etc/chrony.conf
grep allow /etc/chrony.conf

sed -i '/^pool/d' /etc/chrony.conf

sed -i '/^pool/d' /etc/chrony.conf
sed -i '/server/a \
pool 2.almalinux.pool.ntp.org iburst' /etc/chrony.conf

# 放行防火墙
firewall-cmd --add-service=ntp
firewall-cmd --add-service=ntp --per

# check ntp sync
./ssh-all-node "systemctl enable chronyd.service && systemctl restart chronyd.service"
./ssh-all-node "chronyc -n sources"
```

### 1.5 快照备份

```bash
virsh destroy ceph-clienta
virsh destroy ceph-serverc
virsh destroy ceph-serverd
virsh destroy ceph-servere
virsh snapshot-create-as ceph-clienta init
virsh snapshot-create-as ceph-serverc init
virsh snapshot-create-as ceph-serverd init
virsh snapshot-create-as ceph-servere init
virsh start ceph-clienta
virsh start ceph-serverc
virsh start ceph-serverd
virsh start ceph-servere
```

### 1.6 初始化环境

```bash
virsh snapshot-revert ceph-clienta ceph
virsh snapshot-revert ceph-serverc ceph
virsh snapshot-revert ceph-serverd ceph
virsh snapshot-revert ceph-servere ceph

virsh start ceph-clienta
virsh start ceph-serverc
virsh start ceph-serverd
virsh start ceph-servere

ssh clienta 'hostnamectl set-hostname clienta.lab.example.com'
ssh serverc 'hostnamectl set-hostname serverc.lab.example.com'
ssh serverd 'hostnamectl set-hostname serverd.lab.example.com'
ssh servere 'hostnamectl set-hostname servere.lab.example.com'
```

## 2. Ceph 部署

### 2.0 repo 仓库配置

```bash
# 删除默认仓库
./ssh-all-node "rm -rf /etc/yum.repos.d/*.repo"

# 互联网地址
cat << 'END' >> /etc/yum.repos.d/almalimux.repo
[baseos]
name=AlmaLinux $releasever - BaseOS
baseurl=https://repo.almalinux.org/almalinux/$releasever/BaseOS/$basearch/os/
enabled=1
gpgcheck=0

[appstream]
name=AlmaLinux $releasever - AppStream
baseurl=https://repo.almalinux.org/almalinux/$releasever/AppStream/$basearch/os/
enabled=1
gpgcheck=0

[extras]
name=AlmaLinux $releasever - Extras
baseurl=https://repo.almalinux.org/almalinux/$releasever/extras/$basearch/os/
enabled=1
gpgcheck=0
END

# 内网地址
./ssh-all-node 'echo "172.24.0.248 mirror.hflxhn.com" >> /etc/hosts'

cat << 'END' >> /etc/yum.repos.d/iso.repo
[baseos]
name=AlmaLinux $releasever - BaseOS
baseurl=http://mirror.hflxhn.com/web/cdrom/AlmaLinux/8/BaseOS/
enabled=1
gpgcheck=0

[appstream]
name=AlmaLinux $releasever - AppStream
baseurl=http://mirror.hflxhn.com/web/cdrom/AlmaLinux/8/AppStream/
enabled=1
gpgcheck=0
END

cat << 'END' >> /etc/yum.repos.d/ceph.repo
[Ceph]
name=ceph
baseurl=http://mirror.hflxhn.com/web/mirror/AlmaLinux/8.8/ceph/
enabled=1
gpgcheck=0

[Ceph-noarch]
name=ceph-noarch
baseurl=http://mirror.hflxhn.com/web/mirror/AlmaLinux/8.8/ceph-noarch/
enabled=1
gpgcheck=0
END

cat << 'END' >> /etc/yum.repos.d/epel.repo
[epel]
name=ceph
baseurl=http://mirror.hflxhn.com/web/mirror/AlmaLinux/8.8/epel/
enabled=1
gpgcheck=0

[extras]
name=extras
baseurl=http://mirror.hflxhn.com/web/mirror/AlmaLinux/8.8/extras/
enabled=1
gpgcheck=0
END

# 同步到其他节点
./scp-all-node /etc/yum.repos.d/iso.repo  /etc/yum.repos.d
./scp-all-node /etc/yum.repos.d/ceph.repo /etc/yum.repos.d
./scp-all-node /etc/yum.repos.d/epel.repo /etc/yum.repos.d
```

### 2.1 安装 cephadm

```bash
./ssh-all-node 'yum install python3 -y'
curl --silent --remote-name --location https://download.ceph.com/rpm-17.2.7/el8/noarch/cephadm
chmod +x cephadm
./cephadm add-repo --release quincy
./cephadm install
yum install -y ceph-common
which cephadm
```

### 2.2 安装 Podman

```bash
./ssh-all-node 'yum install podman -y'

ls /etc/containers/registries.conf.bak &>/dev/null || cp -r /etc/containers/registries.conf{,.bak}
cat > /etc/containers/registries.conf << 'END'
unqualified-search-registries = ["registry.hflxhn.com:50000", "docker.io"]

[[registry]]
prefix = "docker.io"
location = "fiyc0dbc.mirror.aliyuncs.com"

[[registry]]
prefix = "registry.hflxhn.com:50000"
location = "registry.hflxhn.com:50000"
insecure = true
END

./scp-all-node /etc/containers/registries.conf /etc/containers/registries.conf
./ssh-all-node 'echo "172.24.0.248 registry.hflxhn.com" >> /etc/hosts'

podman login --username fanyang --password Fhl3y_Jsdwj registry.hflxhn.com:50000

# 内网镜像需要操作
## ceph-node-1
podman pull registry.hflxhn.com:50000/ceph/ceph-grafana:9.4.7
podman pull registry.hflxhn.com:50000/prometheus/prometheus:v2.43.0
podman pull registry.hflxhn.com:50000/prometheus/alertmanager:v0.25.0
podman pull registry.hflxhn.com:50000/prometheus/node-exporter:v1.5.0

podman tag registry.hflxhn.com:50000/ceph/ceph-grafana:9.4.7 quay.io/ceph/ceph-grafana:9.4.7
podman tag registry.hflxhn.com:50000/prometheus/prometheus:v2.43.0 quay.io/prometheus/prometheus:v2.43.0
podman tag registry.hflxhn.com:50000/prometheus/alertmanager:v0.25.0 quay.io/prometheus/alertmanager:v0.25.0
podman tag registry.hflxhn.com:50000/prometheus/node-exporter:v1.5.0 quay.io/prometheus/node-exporter:v1.5.0

podman image rm -f registry.hflxhn.com:50000/ceph/ceph-grafana:9.4.7
podman image rm -f registry.hflxhn.com:50000/prometheus/prometheus:v2.43.0
podman image rm -f registry.hflxhn.com:50000/prometheus/alertmanager:v0.25.0
podman image rm -f registry.hflxhn.com:50000/prometheus/node-exporter:v1.5.0

## ceph-node-2, ceph-node-3
podman pull registry.hflxhn.com:50000/prometheus/node-exporter:v1.5.0
podman tag registry.hflxhn.com:50000/prometheus/node-exporter:v1.5.0 quay.io/prometheus/node-exporter:v1.5.0
podman image rm -f registry.hflxhn.com:50000/prometheus/node-exporter:v1.5.0

sed -i 's/quay.io/registry.hflxhn.com:50000/' /usr/sbin/cephadm
```

### 2.3 引导 Ceph 集群

```bash
cephadm bootstrap --mon-ip 192.168.122.50 --allow-fqdn-hostname --initial-dashboard-user admin --initial-dashboard-password redhat --dashboard-password-noupdate --registry-url registry.hflxhn.com:50000 --registry-username fanyang --registry-password Fhl3y_Jsdwj

# 配置免密
ssh-copy-id -f -i /etc/ceph/ceph.pub clienta
ssh-copy-id -f -i /etc/ceph/ceph.pub serverc
ssh-copy-id -f -i /etc/ceph/ceph.pub serverd
ssh-copy-id -f -i /etc/ceph/ceph.pub servere

# 添加新节点到 ceph 集群
ceph orch host add clienta.lab.example.com 192.168.122.50
ceph orch host add serverc.lab.example.com 192.168.122.51
ceph orch host add serverd.lab.example.com 192.168.122.52
ceph orch host add servere.lab.example.com 192.168.122.53

# 设置管理节点
ceph orch host label add clienta.lab.example.com _admin
ceph orch host label add serverc.lab.example.com _admin

ceph orch apply mon serverc.lab.example.com,serverd.lab.example.com,servere.lab.example.com
ceph orch apply mgr --placement="serverc.lab.example.com serverd.lab.example.com servere.lab.example.com"

# 添加 OSD
for i in b c d; do 
 ceph orch daemon add osd serverc.lab.example.com:/dev/vd$i; 
 ceph orch daemon add osd serverd.lab.example.com:/dev/vd$i; 
 ceph orch daemon add osd servere.lab.example.com:/dev/vd$i; 
done
```

## 3. Ceph 260 Exam

### 1. 安装ceph

```bash
1. 容器 image 存储在: registry.lab.example.com 账号: registry/redhat
2. 在 serverc、serverd、servere 和 clienta 节点上部署 Ceph 集群
3. serverc.lab.example.com 和 clienta.lab.example.com 为 Ceph 管理节点
4. 3 个存储节点使用 /dev/vdb、/dev/vdc、/dev/vdd 作为 OSD 硬盘
5. Dashboard 的管理员密码是 redhat
6. 安装并配置其它题目所要求的服务
7. cephadm 软件包已经提前安装到了 serverc 节点


# 登录 podman 容器
podman login registry.lab.example.com -u registry -p redhat

# 安装 cephadm
yum install -y ceph-common

# 引导集群
cephadm bootstrap --mon-ip 172.25.250.12 --registry-url registry.lab.example.com --registry-username registry --registry-password redhat --allow-fqdn-hostname --initial-dashboard-user admin --initial-dashboard-password redhat --dashboard-password-noupdate

# 分发 SSH 公钥到其他节点
ssh-copy-id -f -i /etc/ceph/ceph.pub root@serverd
ssh-copy-id -f -i /etc/ceph/ceph.pub root@servere
ssh-copy-id -f -i /etc/ceph/ceph.pub root@clienta

# 添加其他的 Ceph 节点
ceph orch host add serverd.lab.example.com 172.25.250.13
ceph orch host add servere.lab.example.com 172.25.250.14
ceph orch host add clienta.lab.example.com 172.25.250.10

# 标记 Ceph 管理节点
ceph orch host label add clienta.lab.example.com _admin
ceph orch host label add serverc.lab.example.com _admin

# 查看 Ceph 主机列表
ceph orch host ls

# 部署 Mon (Monitor) 节点
ceph orch apply mon serverc.lab.example.com,serverd.lab.example.com,servere.lab.example.com

# 部署 Mgr (Manager) 节点
ceph orch apply mgr --placement "serverc.lab.example.com serverd.lab.example.com servere.lab.example.com"

# 配置 serverc 的 OSD 硬盘
ceph orch daemon add osd serverc.lab.example.com:/dev/vdb
ceph orch daemon add osd serverc.lab.example.com:/dev/vdc
ceph orch daemon add osd serverc.lab.example.com:/dev/vdd

# 配置 serverd 的 OSD 硬盘
ceph orch daemon add osd serverd.lab.example.com:/dev/vdb
ceph orch daemon add osd serverd.lab.example.com:/dev/vdc
ceph orch daemon add osd serverd.lab.example.com:/dev/vdd

# 配置 servere 的 OSD 硬盘
ceph orch daemon add osd servere.lab.example.com:/dev/vdb
ceph orch daemon add osd servere.lab.example.com:/dev/vdc
ceph orch daemon add osd servere.lab.example.com:/dev/vdd

# 在 serverc 上配置 Ceph 客户端
ssh root@clienta
yum install ceph-common -y

# 复制 Ceph 集群配置文件和密钥环到 clienta
scp root@serverc:/etc/ceph/*.keyring /etc/ceph/
scp root@serverc:/etc/ceph/*.conf /etc/ceph/

# 检查集群状态
ceph -s
```

### 2. 健康状态

```bash
Ceph 的健康状态应该为: HEALTH_OK

ceph -s
```

### 3. 配置 Ceph

```bash
1. Ceph 中的 pool 允许被删除

ceph config set mon mon_allow_pool_delete true

# ceph config set 修改 Ceph 配置项的命令
# mon 指定配置更改是针对 Ceph Monitor（监视器）组件的
# mon_allow_pool_delete 控制是否允许删除 Ceph 存储池
```

### 4. 配置 Ceph dashboard

```bash
1. 配置 Ceph dashboard 支持 SSL

https://serverc.lab.example.com:8443/
```

### 5. 配置纠错码 profile 和 pool

```bash
1. 创建名为 ceph260-ecprofile 的纠删代码 profile, 包含 3 个 data chunks, 2 个 coding chunks, crush-failure-domain 参数为 osd
2. 创建使用这个 profile 的名为 ecpool 的存储池，池包含 32 个 PGs，用于 rgw 类似的应用

ssh root@serverc

ceph osd erasure-code-profile set ceph260-ecprofile k=3 m=2 crush-failure-domain=osd
# ceph osd erasure-code-profile set 定义一个新的纠删编码 Profile 或修改现有的 Profile
# k=3 原始数据被分割成多少个数据块
# m=2 纠删码生成的冗余块的数量
# crush-failure-domain=osd 在何种层级上数据丢失是可以接受的; 
#   osd 作为故障域，意味着系统可以在某些 OSD 失败的情况下恢复数据。
#   如果设置为 host，则意味着数据会分布在不同的主机上，主机级别的故障也能被容忍

ceph osd erasure-code-profile get ceph260-ecprofile
# 查看创建的纠删编码 Profile 的详细信息

ceph osd pool create ecpool 32 erasure ceph260-ecprofile
# ceph osd pool create 创建 OSD 存储池
# 32 存储池的 PG（Placement Groups）数量
# erasure 表示存储池使用纠删编码进行数据存储
# ceph260-ecprofile 创建存储池时要使用的纠删编码 Profile

ceph osd pool application enable ecpool rgw
# 将 ecpool 设置为支持 RADOS Gateway (RGW) 的应用类型. RGW 是 Ceph 的对象存储网关，提供兼容的 S3 和 Swift 接口。通过启用 rgw 应用，我们将 ecpool 设置为可以用作 RGW 的存储池。

ceph osd pool ls detail
# 列出了 Ceph 集群中所有存储池的详细信息

ceph osd pool get ecpool all
# 获取 ecpool 的所有配置细节
```

### 6. 配置 Ceph 身份认证

```bash
1. 创建 Ceph 用户 thomas, 可以读写 ecpool 存储池中的 dev namespace 中的对象
2. thomas 用户可从 clienta.lab.example.com 访问 ecpool,但该用户不可以访问其它任何存储池
3. 创建 Ceph 用户 rbd, 可以对 ceph260-pool 存储池执行读、写和执行扩展的对象类
4. rbd 用户可以从 clienta.lab.example.com 访问 ceph260-pool, 但该用户不可以访问其它任何存储池

ssh root@serverc

ceph auth get-or-create client.thomas mon 'allow r' osd 'allow rw pool=ecpool namespace=dev' -o /etc/ceph/ceph.client.thomas.keyring
# ceph auth get-or-create 获取现有用户的认证密钥或创建一个新的用户并生成其密钥
# client.thomas lient. 是 Ceph 系统中用户标识符的前缀，thomas 是具体的用户名
# mon 'allow r'：为 client.thomas 用户分配在 Monitor（监视器）上的权限
# osd 'allow rw pool=ecpool namespace=dev'：为 client.thomas 用户分配在 OSD（对象存储守护进程）上的权限
# -o /etc/ceph/ceph.client.thomas.keyring 将生成的密钥环文件输出到指定路径

scp /etc/ceph/ceph.client.thomas.keyring root@clienta:/etc/ceph/
# 将密钥环文件复制到目标客户端

ceph auth get-or-create client.rbd mon 'profile rbd' osd 'profile rbd pool=ceph260-pool' -o /etc/ceph/ceph.client.rbd.keyring
# mon 'profile rbd' 为 rbd 用户提供与 RBD（RADOS Block Device）相关的监视器权限
# osd 'profile rbd pool=ceph260-pool' 允许用户在 ceph260-pool 存储池上执行与 RBD 相关的操作，包括读、写和扩展对象类

scp /etc/ceph/ceph.client.rbd.keyring root@clienta:/etc/ceph/
# 将 rbd 用户的密钥环文件从 serverc 复制到 clienta

ceph auth ls
# 列出 Ceph 集群中所有已创建的用户及其权限

ssh root@clienta

ceph -s --id thomas
# 使用 rbd 用户检查 Ceph 集群状态，确保配置正确
```

### 7. 配置Ceph RBD

```bash
1. 以 rbd 用户身份创建复制类型的存储池 ceph260-pool，含有 32 个 PGs
2. 在存储池中创建一个 512MB 的 image: ceph260-rbd
3. 使用 ext4 文件系统格式化这个 image
4. 在 clienta.lab.example.com 上将这个 image 永久挂载到 /mnt/rbd

ssh root@serverc

ceph osd pool create ceph260-pool 32
# 创建名为 ceph260-pool 的存储池，并指定其包含 32 个 PGs（Placement Groups）

ceph osd pool application enable ceph260-pool rbd
# 启用存储池 ceph260-pool 作为 RBD（RADOS Block Device）应用程序的支持池

rbd create ceph260-pool/ceph260-rbd --size 512M
# 在 ceph260-pool 中创建一个名为 ceph260-rbd 的 RBD image，其大小为 512MB

rbd -p ceph260-pool ls
# 列出 ceph260-pool 中的所有 RBD images，以确认 ceph260-rbd 已成功创建

rbd info ceph260-pool/ceph260-rbd
# 显示 ceph260-rbd image 的详细信息，包括大小、状态、和特性

rbd map ceph260-pool/ceph260-rbd
# 将 ceph260-pool 中的 ceph260-rbd RBD image 映射到本地系统，使其作为块设备出现（通常为 /dev/rbd0 或类似的设备文件）

mkfs.ext4 /dev/rbd0
rbd unmap /dev/rbd0
# 取消对 ceph260-rbd image 的块设备映射

ssh root@clienta
mkdir /mnt/rbd
echo "/dev/rbd/ceph260-pool/ceph260-rbd /mnt/rbd ext4 defaults,_netdev 0 0" >> /etc/fstab

echo "ceph260-pool/ceph260-rbd id=rbd,keyring=/etc/ceph/ceph.client.rbd.keyring" >> /etc/ceph/rbdmap

systemctl enable --now rbdmap
# 启用并立即启动 rbdmap 服务

mount -a && df -h ##TEST##
reboot ##TEST##
```

### 8. 导出RBD image为文件

```bash
1. 在 serverc.lab.example.com 上导出 image: ceph260-rbd 到 student 用户的家目录，名为: ceph260-export.img

ssh student@serverc
cd ~
rbd export ceph260-pool/ceph260-rbd ceph260-export.img --id rbd
# rbd export：这是 RBD 的导出命令，用于将 RBD image 导出为文件
# ceph260-pool/ceph260-rbd：指定要导出的 RBD image 的全名，包括存储池名称和 image 名称
# eph260-export.img：输出文件的名称
# --id rbd：指定使用 rbd 用户进行操作
```

### 9. 导入RBD image文件

```bash
1. 将 files/ceph260_import.img 导入 ceph260-pool 中，名为: ceph260-image

rbd import /root/files/ceph260_import.img ceph260-pool/ceph260-image
# rbd import：导入 RBD image 的命令。
# /root/files/ceph260_import.img：要导入的文件路径。
# ceph260-pool/ceph260-image：目标 RBD image 的全名，包括存储池名称 ceph260-pool 和 image 名称 ceph260-image
```

### 10. 创建RBD快照

```bash
1. 为 ceph260-image 创建名为 rbd-snap 的快照

ssh root@serverc
rbd snap create ceph260-pool/ceph260-image@rbd-snap
# rbd snap create：创建 RBD image 的快照。
# ceph260-pool/ceph260-image@rbd-snap：指定快照的路径和名称，其中 ceph260-pool 是存储池名称，ceph260-image 是 RBD image 名称，@rbd-snap 是快照名称。

rbd snap ls ceph260-pool/ceph260-image
# rbd snap ls：列出指定 RBD image 的所有快照。
# ceph260-pool/ceph260-image：指定要查看快照的 RBD image
```

### 11. 创建RBD克隆

```bash
1. 对 rbd-snap 快照创建一个名为 rbd-clone 的克隆
2. 使用rbd用户身份在 clienta.lab.example.com 上将这个克隆永久挂载到 /mnt/clone

ssh root@serverc
rbd snap protect ceph260-pool/ceph260-image@rbd-snap
# rbd snap protect：将指定的快照标记为"受保护"，防止删除操作
# ceph260-pool/ceph260-image@rbd-snap：指定需要保护的快照

rbd clone ceph260-pool/ceph260-image@rbd-snap ceph260-pool/rbd-clone
# rbd clone：基于指定的快照创建一个新的 RBD image
# ceph260-pool/ceph260-image@rbd-snap：源快照，克隆将基于此快照创建
# ceph260-pool/rbd-clone：目标克隆的全名，包括存储池名称和新 image 的名称

rbd children ceph260-pool/ceph260-image
# rbd children：列出指定 RBD image 快照的所有克隆子项
# ceph260-pool/ceph260-image：指定要查看克隆子项的 RBD image

ssh root@clienta

mkdir /mnt/clone

echo "ceph260-pool/rbd-clone id=rbd,keyring=/etc/ceph/ceph.client.rbd.keyring" >> /etc/ceph/rbdmap
# ceph260-pool/rbd-clone：指定需要映射的 RBD image
# id=rbd：指定使用 rbd 用户进行映射
# keyring=/etc/ceph/ceph.client.rbd.keyring：指定 rbd 用户的 keyring 文件位置

systemctl restart rbdmap

rbd showmapped
# rbd showmapped：显示当前所有映射的 RBD 设备

blkid /dev/rbd1
# blkid /dev/rbd1：显示 /dev/rbd1 设备的文件系统类型

echo "/dev/rbd/ceph260-pool/rbd-clone /mnt/clone ext4 defaults,_netdev 0 0" >> /etc/fstab

mount -a
```

### 12. 部署RADOS Gateway

```bash
1. 在 serverc.lab.example.com 上部署 RADOS Gateway
2. Gateway 使用 80 和 81 端口
3. Gateway 的名字是 ceph260_radosgw

ssh root@serverc
ceph orch apply rgw ceph260_radosgw --port 80 --placement="2 serverc.lab.example.com"
# ceph orch apply rgw：部署或更新一个 RADOS Gateway 服务。
# ceph260_radosgw：指定 RADOS Gateway 服务的名称。
# --port 80 --port 81：指定 RADOS Gateway 监听的端口，可以同时监听多个端口。
# --placement="2 serverc.lab.example.com"：指定 RADOS Gateway 的部署位置。2 表示在 serverc 上运行两个实例

ss -tunpla |grep :80 && ss -tunpla |grep :81
ceph orch ls
# ceph orch ls：列出 Ceph orchestrator 中所有被管理的服务

ceph orch ps
# ceph orch ps：显示当前所有 Ceph 服务实例的状态和信息
```

### 13. 部署MDS和CephFS

```bash
1. 在 serverc.lab.example.com 上部署 Metadata Server
2. 部署名为 cephfs_ceph260 的 CephFS volume
3. cephfs_ceph260:/ 可以被 client id 为 rw 的用户进行读写访问和创建快照
4. cephfs_ceph260:/archive 可以被 client id 为 ro 的用户只读访问

ssh root@serverc
ceph fs volume create cephfs_ceph260 --placement="serverc.lab.example.com"
# ceph fs volume create：创建一个新的 Ceph 文件系统 volume
# cephfs_ceph260：指定新创建的 CephFS volume 的名称
# --placement="serverc.lab.example.com"：指定在哪个节点上放置 volume

# # 检查 Ceph 集群的状态和 CephFS volume 列表
ceph -s && ceph fs volume ls

ceph fs authorize cephfs_ceph260 client.rw / rws
# ceph fs authorize：为指定的 CephFS volume 配置客户端访问权限。
# cephfs_ceph260：CephFS volume 的名称。
# client.rw：客户端 ID。
# /：授权路径，即根路径。
# rws：权限标志，r 代表读权限，w 代表写权限，s 代表快照权限

ceph fs authorize cephfs_ceph260 client.ro /archive r
# cephfs_ceph260：CephFS volume 的名称。
# client.ro：客户端 ID。
# /archive：授权路径。
# r：权限标志，只读权限

ceph auth get client.rw
ceph auth get client.ro
# ceph auth get：获取指定客户端的认证信息。
# client.rw 和 client.ro：客户端 ID
```

### 14. 配置Ceph全局参数

```bash
1. full_ratio 参数设为 97 %
2. nearfull_ratio 参数设为 80 %

ssh root@serverc
ceph osd set-full-ratio 0.97
# 设定 OSD 的 full ratio。当 OSD 使用达到这个比率（0.97，即 97%）时，Ceph 将停止所有写入操作，确保不会超出可用存储空间

ceph osd set-nearfull-ratio 0.80
# 定 OSD 的 near full ratio。当 OSD 使用达到这个比率（0.80，即 80%）时，Ceph 会发出警告，并可能开始数据的重新平衡以减轻负载，但仍允许写入操作

ceph osd dump
# 显示当前 OSD 的配置信息，包括存储池、OSD 的状态、full 和 near full 比率等详细数据
```

### 15. 基于S3配置RGW对象存储（第九章练习）

```bash
在 RADOS Gateway 主机中，按以下要求创建一个 S3 用户和一个 S3 bucket:

1. S3 用户的 UID: Sunny, Access Key: 12345, secret: 67890
2. S3 bucket名为: ceph260bucket
3. 将 files/S3_file 以 my_S3_file 的文件名上传到 ceph260bucket
4. DNS wildcard: ceph260bucket.serverc 已经配置好, ceph260bucket 可以 http://serverc/ceph260bucket 或 http://ceph260bucket.serverc/ 方式访问
5. 设置 Sunny 用户的最大上传 quota 为 20971520 字节（全局）
6. 设置 Sunny 用户最大单 bucket quota 为 10485760 字节，并且最多能上传 100 个文件

root@clienta
radosgw-admin user create --uid="Sunny" --display-name="Sunny" --access-key="12345" --secret="67890"
aws configure --profile=ceph
AWS Access Key ID : 12345
AWS Secret Access Key : 67890
Default region name [None]:
Default output format [None]:

aws --profile=ceph --endpoint=http://serverc:80 s3 mb s3://ceph260bucket
aws --profile=ceph --endpoint=http://serverc:80 s3 ls
aws --profile=ceph --endpoint=http://serverc:80 --acl=public-read-write s3 cp ~/files/S3_file s3://ceph260bucket/my_S3_file
ceph config set client.rgw rgw_dns_name serverc
ceph orch ls
ceph orch restart rgw.ceph260_radosgw
wget -O /dev/null http://ceph260bucket.serverc/my_S3_file
wget -O /dev/null http://serverc/ceph260bucket/my_S3_file

# 设置配额
radosgw-admin quota set --quota-scope=user --uid=Sunny --max-size=20971520
radosgw-admin quota enable --quota-scope=user --uid=Sunny
radosgw-admin quota set --quota-scope=bucket --uid=Sunny --max-size=10485760 --max-objects=100
radosgw-admin quota enable --quota-scope=bucket --uid=Sunny
radosgw-admin user info --uid=Sunny
```
