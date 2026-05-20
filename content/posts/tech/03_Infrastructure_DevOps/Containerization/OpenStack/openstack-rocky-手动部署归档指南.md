+++
date = '2026-05-20T08:48:33+08:00'
draft = false
title = 'OpenStack Rocky 手动部署归档指南'
slug = "q2cztexzp1"
description = "OpenStack Rocky 手动部署归档指南"
summary = "OpenStack Rocky 手动部署归档指南"
categories = ["📒虚拟化"]
tags = ["OpenStack Rocky"]
comments = true
+++

# OpenStack Rocky 手动部署归档指南 (历史参考)

> ❗ **重要：版本过时警告** ❗
>
> **OpenStack Rocky 版本已于 2020 年 5 月达到其生命周期终点 (End of Life)，不再有任何安全更新或官方支持。**
>
> 本指南详细记录了其手动部署过程，**仅应用于学习 OpenStack 核心组件交互原理或作为历史技术归档查阅**。
>
> **强烈不推荐在任何新环境（包括测试和生产）中部署此版本。** 对于新的部署，请考虑使用如 Kolla-Ansible, OpenStack-Ansible 或 MicroStack 等自动化工具来部署当前受支持的 OpenStack 版本。

## 1. OpenStack 架构与节点角色

OpenStack 是一个开源的云计算平台，由多个独立但相互协作的服务（称为“组件”）组成。本次部署将采用多节点架构，各节点角色如下：

- **Controller (控制节点)**: 整个云平台的“大脑”，运行所有核心管理服务，如身份认证(Keystone)、镜像服务(Glance)、API 服务(Nova-API, Neutron-Server)以及数据库和消息队列。
- **Compute (计算节点)**: 运行虚拟机 (VM) 的地方，核心组件是 Hypervisor (KVM) 和 Nova-Compute 服务。
- **Network (网络节点)**: (可选，可与控制节点合并) 负责处理虚拟机的东西向和南北向网络流量，运行 Neutron 的 L3 agent, DHCP agent 等。
- **Storage (存储节点)**: (可选) 提供块存储 (Cinder) 或对象存储 (Swift) 服务。

## 2. 环境准备与系统初始化

此阶段将在所有节点上执行基础配置，为 OpenStack 的安装做准备。

### 2.1 节点资源与网络规划

确保您的虚拟机或物理机满足官方最低资源需求，并已规划好网络。

| 主机名                 | 操作系统 | CPU  | 内存   | IP 地址      |
| ---------------------- | -------- | ---- | ------ | ------------ |
| `openstack-controller` | CentOS 7 | 4+   | ≥ 4 GB | `10.0.0.100` |
| `openstack-compute-1`  | CentOS 7 | 4+   | ≥ 2 GB | `10.0.0.99`  |
| `openstack-compute-2`  | CentOS 7 | 4+   | ≥ 2 GB | `10.0.0.98`  |
| `openstack-network`    | CentOS 7 | 4+   | ≥ 2 GB | `10.0.0.97`  |
| `openstack-storage`    | CentOS 7 | 4+   | ≥ 2 GB | `10.0.0.96`  |

### 2.2 定义变量与辅助脚本 (在管理节点执行)

为了简化后续的批量操作，我们首先定义变量和一系列辅助脚本。

```bash
# 定义变量
cat > vars << 'END'
username=root
all_address=(
    10.0.0.100
    10.0.0.99
    10.0.0.98
    10.0.0.97
    10.0.0.96
)
all_fqdn=(
    10.0.0.100 openstack-controller
    10.0.0.99  openstack-compute-1
    10.0.0.98  openstack-compute-2
    10.0.0.97  openstack-network
    10.0.0.96  openstack-storage
);
all_hostname=(
    openstack-controller
    openstack-compute-1
    openstack-compute-2
    openstack-network
    openstack-storage
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

### 2.3 配置系统基础环境 (所有节点)

```bash
# 1. 配置 /etc/hosts 实现节点间的主机名解析
cat > /etc/hosts << END
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.0.0.100 openstack-controller
10.0.0.99  openstack-compute-1
10.0.0.98  openstack-compute-2
10.0.0.97  openstack-network
10.0.0.96  openstack-storage
END

./scp-all-node /etc/hosts /etc/hosts

# 2. 配置 SSH，禁用 DNS 反向解析以加快连接速度
./ssh-all-node "sed -i 's/^#UseDNS.*/UseDNS no/' /etc/ssh/sshd_config && grep UseDNS /etc/ssh/sshd_config"
./ssh-all-node "systemctl restart sshd.service"

# 3. 设置各节点主机名 (需逐一执行)
ssh openstack-controller "hostnamectl set-hostname openstack-controller"
ssh openstack-compute-1 "hostnamectl set-hostname openstack-compute-1"
ssh openstack-compute-2 "hostnamectl set-hostname openstack-compute-2"
ssh openstack-network "hostnamectl set-hostname openstack-network"
ssh openstack-storage "hostnamectl set-hostname openstack-stotage"

# 4. 关闭防火墙和 SELinux (在测试环境中)
# 警告：生产环境应配置详细的防火墙规则
./ssh-all-node "systemctl disable firewalld.service && systemctl stop firewalld.service"
./ssh-all-node "setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config"

# 5. 配置 YUM 软件源
./ssh-all-node "rm -rf /etc/yum.repos.d/*"

# ali
cat > /etc/yum.repos.d/ali.repo << 'END'
[os]
baseurl = https://mirrors.aliyun.com/centos/7.9.2009/os/x86_64
enabled = 1
gpgcheck = 0
name = os

[updates]
baseurl = https://mirrors.aliyun.com/centos/7.9.2009/updates/x86_64
enabled = 1
gpgcheck = 0
name = updates

[extras]
baseurl = https://mirrors.aliyun.com/centos/7.9.2009/extras/x86_64
enabled = 1
gpgcheck = 0
name = extras

[virt]
baseurl = https://mirrors.aliyun.com/centos/7.9.2009/virt/x86_64/kvm-common
enabled = 1
gpgcheck = 0
name = virt

[cloud]
baseurl = https://mirrors.aliyun.com/centos/7.9.2009/cloud/x86_64/openstack-rocky
enabled = 1
gpgcheck = 0
name = cloud

[epel]
baseurl = https://mirrors.aliyun.com/epel/7/x86_64
enabled = 1
gpgcheck = 0
name = epel
END

./scp-all-node /etc/yum.repos.d/*.repo /etc/yum.repos.d
./ssh-all-node "yum makecache"
```

### 2.4 配置时间同步 (NTP)

确保所有节点时间一致，这对于分布式系统至关重要。

```bash
# all node
./ssh-all-node "yum install chrony -y"
./ssh-all-node "sed -i '/^server/d' /etc/chrony.conf && sed -i '/server/a server openstack-controller iburst' /etc/chrony.conf"

# controller node
sed -i "s|^#allow.*|allow 0.0.0.0/0|" /etc/chrony.conf
grep allow /etc/chrony.conf

sed -i '/^server/d' /etc/chrony.conf
sed -i '/server/a \
server 0.centos.pool.ntp.org iburst \
server 1.centos.pool.ntp.org iburst \
server 2.centos.pool.ntp.org iburst \
server 3.centos.pool.ntp.org iburst' /etc/chrony.conf

# check ntp sync
./ssh-all-node "systemctl enable chronyd.service && systemctl restart chronyd.service"
./ssh-all-node "chronyc sources"
```

## 3. 在控制节点上安装核心服务

这些是 OpenStack 各组件共同依赖的基础服务，全部安装在控制节点上。

### 3.1. OpenStack 客户端与 SELinux 模块

```bash
./ssh-all-node "yum install python-openstackclient -y"
./ssh-all-node "yum install openstack-selinux -y"
```

### 3.2. 数据库服务 (MariaDB)

MariaDB 用于存储 OpenStack 各组件的状态和配置信息。

```bash
# control node
yum install mariadb mariadb-server python2-PyMySQL -y

# get default ip address
default_net_card=$(ip route show default | awk 'NR==1' | awk '/default/ {print $5}')
local_address=$(ip a show ${default_net_card} | grep -m 1 inet | awk '{print $2}' | awk -F"/" '{print $1}')

# set mysql
cat > /etc/my.cnf.d/openstack.cnf << END
[mysqld]
bind-address = ${local_address}

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
END

# 启动 mariadb
systemctl enable mariadb.service
systemctl restart mariadb.service
```

### 3.3. 消息队列 (RabbitMQ)

RabbitMQ 是 OpenStack 各组件之间进行通信的“消息总线”。

```bash
# 安装 rabbitmq
yum install rabbitmq-server -y

# 启动 rabbitmq
systemctl enable rabbitmq-server.service
systemctl restart rabbitmq-server.service

# 创建用户
rabbitmqctl add_user openstack RABBIT_PASS

# 设置权限
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

### 3.4. 缓存服务 (Memcached)

Memcached 用于缓存 Keystone 的令牌等数据，以减轻数据库压力。

```bash
# 安装 memcached
yum install memcached python-memcached -y

# edit /etc/sysconfig/memcached
modifyConf "/etc/sysconfig/memcached" "OPTIONS" "-l 127.0.0.1,::1,openstack-controller"

# 启动 memcached
systemctl enable memcached.service
systemctl restart memcached.service
```

### 3.5. 分布式键值存储 (Etcd)

Etcd 用于服务发现和状态存储，在 Neutron 等组件中会使用。

```bash
# 安装 etcd
yum install etcd -y

default_net_card=$(ip route show default | awk 'NR==1' | awk '/default/ {print $5}')
local_address=$(ip a show ${default_net_card} | grep -m 1 inet | awk '{print $2}' | awk -F"/" '{print $1}')

# edit /etc/etcd/etcd.conf
#[Member]
modifyConf "/etc/etcd/etcd.conf" "ETCD_LISTEN_PEER_URLS" "http://${local_address}:2380"
modifyConf "/etc/etcd/etcd.conf" "ETCD_LISTEN_CLIENT_URLS" "http://${local_address}:2379"
modifyConf "/etc/etcd/etcd.conf" "ETCD_NAME" "openstack-controller"
#[Clustering]
modifyConf "/etc/etcd/etcd.conf" "ETCD_INITIAL_ADVERTISE_PEER_URLS" "http://${local_address}:2380"
modifyConf "/etc/etcd/etcd.conf" "ETCD_ADVERTISE_CLIENT_URLS" "http://${local_address}:2379"
modifyConf "/etc/etcd/etcd.conf" "ETCD_INITIAL_CLUSTER" "openstack-controller=http://${local_address}:2380"
modifyConf "/etc/etcd/etcd.conf" "ETCD_INITIAL_CLUSTER_TOKEN" "etcd-cluster-01"
modifyConf "/etc/etcd/etcd.conf" "ETCD_INITIAL_CLUSTER_STATE" "new"

# 启动 etcd
systemctl enable etcd
systemctl restart etcd
```

## 4. 部署身份服务 (Keystone)

> **组件作用**：Keystone 是 OpenStack 的身份认证服务，负责管理用户、项目(租户)、角色，并提供服务目录 (Service Catalog) 和身份验证令牌 (Token)。

### 4.1 数据库与用户准备

```bash
# 1. 创建 Keystone 数据库
mysql << END
create database keystone;
grant all privileges on keystone.* to 'keystone'@'localhost' identified by 'KEYSTONE_DBPASS';
grant all privileges on keystone.* to 'keystone'@'%' identified by 'KEYSTONE_DBPASS';
grant all privileges on keystone.* to 'keystone'@'openstack-controller' identified by 'KEYSTONE_DBPASS';
END

# 数据库连接测试
mysql -ukeystone -pKEYSTONE_DBPASS keystone -e 'show databases;'

# 2. 安装 Keystone 软件包
yum install python2-qpid-proton openstack-keystone httpd mod_wsgi -y
```

### 4.2 配置 `keystone.conf`

```bash
clearConf "/etc/keystone/keystone.conf"
appendConf "/etc/keystone/keystone.conf" "database" "connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@openstack-controller/keystone"
appendConf "/etc/keystone/keystone.conf" "token" "provider = fernet"
```

### 4.3 初始化与引导

```bash
# 同步数据库
# 让 keystone 接管 mysql 进行数据库同步
# 出现的错误: 1. 连接 mysql 数据库失败导致
su -s /bin/sh -c "keystone-manage db_sync" keystone

# 初始化 Fernet 密钥
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

# 引导身份服务
# 1. --bootstrap-admin-url  管理网络
# 2. --bootstrap-public-url 组件之间通讯网络
# 3. --bootstrap-public-url 带外管理网络
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://openstack-controller:5000/v3/ \
  --bootstrap-internal-url http://openstack-controller:5000/v3/ \
  --bootstrap-public-url http://openstack-controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

### 4.4 配置 Apache HTTPD 并启动服务

```bash
# 配置 Apache 服务器名
sed -i 's/^#ServerName.*/ServerName openstack-controller/' /etc/httpd/conf/httpd.conf
grep ^ServerName /etc/httpd/conf/httpd.conf

# 链接 Keystone 的 WSGI 应用
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/

# 启动服务
systemctl enable --now httpd.service
```

### 4.5 验证

```bash
# 生成凭据文件
# OS_USERNAME表示的是用户名
# OS_PASSWORD表示用户的密码
# OS_PROJECT_NAME表示的操作的项目名称
# OS_USER_DOMAIN_NAME表示用户所在的域
# OS_PROJECT_DOMAIN_NAME表示项目的域名
# OS_AUTH_URL表示的是keystone的endpoint
# OS_IDENTITY_API_VERSION表示是的keystone的版本
cat > openrc << END
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://openstack-controller:5000/v3
export OS_IDENTITY_API_VERSION=3
END

source openrc

# 执行命令验证 Keystone 是否正常工作
openstack user list
openstack project list
```

> 后续每个组件的部署流程都与 Keystone 类似，遵循 **“创建数据库 -> 创建服务用户 -> 安装软件包 -> 修改配置文件 -> 同步数据库 -> 启动服务 -> 验证”** 的模式。由于篇幅原因，后续组件将简化描述，但您的原始脚本中的每一步都至关重要。

## 5. 部署镜像服务 (Glance)

> **组件作用**：Glance 负责存储和管理虚拟机镜像，为创建虚拟机提供模板。

```bash
# https://docs.openstack.org/glance/rocky/install/

# 数据库准备
mysql << END
create database glance;
grant all privileges on glance.* to 'glance'@'localhost' identified by 'GLANCE_DBPASS';
grant all privileges on glance.* to 'glance'@'%' identified by 'GLANCE_DBPASS';
grant all privileges on glance.* to 'glance'@'openstack-controller' identified by 'GLANCE_DBPASS';
END

# 数据库连接测试
mysql -uglance -pGLANCE_DBPASS glance -e "show databases;"

# 加载凭据文件
source openrc

# 创建 glance 用户
openstack user create --domain default --password glance glance

# 创建 service project
openstack project create --domain default --description "Service Project" service

# 为 glance 授权
openstack role add --project service --user glance admin

# 创建 glance service
openstack service create --name glance --description "OpenStack Image" image

# 创建 endpoints
openstack endpoint create --region RegionOne image public http://openstack-controller:9292
openstack endpoint create --region RegionOne image internal http://openstack-controller:9292
openstack endpoint create --region RegionOne image admin http://openstack-controller:9292

# 安装 glance
yum install openstack-glance -y

# 配置 glance
clearConf /etc/glance/glance-api.conf

appendConf "/etc/glance/glance-api.conf" "database" "connection = mysql+pymysql://glance:GLANCE_DBPASS@openstack-controller/glance"

appendConf "/etc/glance/glance-api.conf" "keystone_authtoken" "www_authenticate_uri = http://openstack-controller:5000
auth_url = http://openstack-controller:5000
memcached_servers = openstack-controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = glance"

appendConf "/etc/glance/glance-api.conf" "paste_deploy" "flavor = keystone"

appendConf "/etc/glance/glance-api.conf" "glance_store" "stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/"

# 配置 glance-registry
clearConf "/etc/glance/glance-registry.conf"

appendConf "/etc/glance/glance-registry.conf" "database" "connection = mysql+pymysql://glance:GLANCE_DBPASS@openstack-controller/glance"
appendConf "/etc/glance/glance-registry.conf" "keystone_authtoken" "www_authenticate_uri  = http://openstack-controller:5000
auth_url = http://openstack-controller:5000
memcached_servers = openstack-controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = glance"
appendConf "/etc/glance/glance-registry.conf" "paste_deploy" "flavor = keystone"

# 同步 glance 数据库
su -s /bin/sh -c "glance-manage db_sync" glance

# 启动 glance 服务
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl restart openstack-glance-api.service openstack-glance-registry.service

# 上传镜像到 glance
openstack image create --file /iso/ubuntu-16.04.7-server-amd64.iso ubuntu-16.04.7-server-amd64
openstack image show ubuntu-16.04.7-server-amd64
```

## 6. 部署计算服务 (Nova)

> **组件作用**：Nova 是 OpenStack 的核心，负责管理虚拟机的整个生命周期（创建、调度、销毁等）。

### 6.1 Nova Controller

```bash
# https://docs.openstack.org/nova/rocky/install/

# 数据库准备
mysql << END
create database nova_api;
create database nova;
create database nova_cell0;
create database placement;

grant all privileges on nova_api.* to 'nova'@'localhost' identified by 'NOVA_DBPASS';
grant all privileges on nova_api.* to 'nova'@'%' identified by 'NOVA_DBPASS';
grant all privileges on nova_api.* to 'nova'@'openstack-controller' identified by 'NOVA_DBPASS';

grant all privileges on nova.* to 'nova'@'localhost' identified by 'NOVA_DBPASS';
grant all privileges on nova.* to 'nova'@'%' identified by 'NOVA_DBPASS';
grant all privileges on nova.* to 'nova'@'openstack-controller' identified by 'NOVA_DBPASS';

grant all privileges on nova_cell0.* to 'nova'@'localhost' identified by 'NOVA_DBPASS';
grant all privileges on nova_cell0.* to 'nova'@'%' identified by 'NOVA_DBPASS';
grant all privileges on nova_cell0.* to 'nova'@'openstack-controller' identified by 'NOVA_DBPASS';

grant all privileges on placement.* to 'placement'@'localhost' identified by 'PLACEMENT_DBPASS';
grant all privileges on placement.* to 'placement'@'%' identified by 'PLACEMENT_DBPASS';
grant all privileges on placement.* to 'placement'@'openstack-controller' identified by 'PLACEMENT_DBPASS';
END

# 数据库连接测试
mysql -unova -pNOVA_DBPASS nova_api -e "show databases;"
mysql -unova -pNOVA_DBPASS nova -e "show databases;"
mysql -unova -pNOVA_DBPASS nova_cell0 -e "show databases;"
mysql -uplacement -pPLACEMENT_DBPASS placement -e "show databases;"

# 创建 nova 用户
openstack user create --domain default --password nova nova
openstack user create --domain default --password placement placement

# 为 nova 授权
openstack role add --project service --user nova admin
openstack role add --project service --user placement admin

# 创建 nova service
openstack service create --name nova --description "OpenStack Compute" compute
openstack service create --name placement --description "Placement API" placement

# 创建 endpoints
openstack endpoint create --region RegionOne compute public http://openstack-controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://openstack-controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://openstack-controller:8774/v2.1

openstack endpoint create --region RegionOne placement public http://openstack-controller:8778
openstack endpoint create --region RegionOne placement internal http://openstack-controller:8778
openstack endpoint create --region RegionOne placement admin http://openstack-controller:8778

# 查看 catalog
openstack catalog list

# 安装 nova
yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api -y

# 获取默认 ip
default_net_card=$(ip route show default | awk 'NR==1' | awk '/default/ {print $5}')
local_address=$(ip a show ${default_net_card} | grep -m 1 inet | awk '{print $2}' | awk -F"/" '{print $1}')

# 编辑配置文件
clearConf "/etc/nova/nova.conf"

appendConf "/etc/nova/nova.conf" "DEFAULT" "enabled_apis = osapi_compute,metadata"

appendConf "/etc/nova/nova.conf" "api_database" "connection = mysql+pymysql://nova:NOVA_DBPASS@openstack-controller/nova_api"
appendConf "/etc/nova/nova.conf" "database" "connection = mysql+pymysql://nova:NOVA_DBPASS@openstack-controller/nova"
appendConf "/etc/nova/nova.conf" "placement_database" "connection = mysql+pymysql://placement:PLACEMENT_DBPASS@openstack-controller/placement"

appendConf "/etc/nova/nova.conf" "DEFAULT" "transport_url = rabbit://openstack:RABBIT_PASS@openstack-controller"

appendConf "/etc/nova/nova.conf" "api" "auth_strategy = keystone"
appendConf "/etc/nova/nova.conf" "keystone_authtoken" "auth_url = http://openstack-controller:5000/v3
memcached_servers = openstack-controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = nova"

appendConf "/etc/nova/nova.conf" "DEFAULT" "my_ip = ${local_address}"
appendConf "/etc/nova/nova.conf" "DEFAULT" "use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver"

appendConf "/etc/nova/nova.conf" "vnc" "enabled = true
server_listen = \$my_ip
server_proxyclient_address = \$my_ip"

appendConf "/etc/nova/nova.conf" "glance" "api_servers = http://openstack-controller:9292"
appendConf "/etc/nova/nova.conf" "oslo_concurrency" "lock_path = /var/lib/nova/tmp"

appendConf "/etc/nova/nova.conf" "placement" "egion_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://openstack-controller:5000/v3
username = placement
password = placement"

# 编辑 apache 配置文件，解决打包时 bug
sed -i '/<\/VirtualHost>/i <Directory /usr/bin> \
   <IfVersion >= 2.4> \
      Require all granted \
   </IfVersion> \
   <IfVersion < 2.4> \
      Order allow,deny \
      Allow from all \
   </IfVersion> \
</Directory>' /etc/httpd/conf.d/00-nova-placement-api.conf

# 重启 apache
systemctl restart httpd

# 同步数据库
su -s /bin/sh -c "nova-manage api_db sync" nova

# 注册 cell0 数据库
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova

# 创建一个名为 cell1 的 nova-cell
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova

# 同步 nova_cell 数据库
su -s /bin/sh -c "nova-manage db sync" nova

# 验证 nova cell0 和 cell1 是否已经正确注册
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova

# 重启
systemctl enable openstack-nova-api.service openstack-nova-consoleauth openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl restart openstack-nova-api.service openstack-nova-consoleauth openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

### 6.2 Nova Compute

```bash
# https://docs.openstack.org/nova/rocky/install/compute-install-rdo.html

# 安装 openstack-nova-compute
yum install python2-qpid-proton openstack-nova-compute -y

# 获取默认 ip
manage_address=$(grep -E "^([^#].*openstack-controller)" /etc/hosts | awk '{print $1}')

# 编辑配置文件
clearConf "/etc/nova/nova.conf"

appendConf "/etc/nova/nova.conf" "DEFAULT" "enabled_apis = osapi_compute,metadata"
appendConf "/etc/nova/nova.conf" "DEFAULT" "rabbit://openstack:RABBIT_PASS@openstack-controller"

appendConf "/etc/nova/nova.conf" "api" "auth_strategy = keystone"
appendConf "/etc/nova/nova.conf" "keystone_authtoken" "auth_url = http://openstack-controller:5000/v3
memcached_servers = openstack-controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = nova"

appendConf "/etc/nova/nova.conf" "DEFAULT" "my_ip = ${manage_address}"

appendConf "/etc/nova/nova.conf" "DEFAULT" "use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver"

appendConf "/etc/nova/nova.conf" "vnc" "enabled = true
server_listen = 0.0.0.0
server_proxyclient_address = \$my_ip
novncproxy_base_url = http://openstack-controller:6080/vnc_auto.html"

appendConf "/etc/nova/nova.conf" "glance" "api_servers = http://openstack-controller:9292"

appendConf "/etc/nova/nova.conf" "oslo_concurrency" "lock_path = /var/lib/nova/tmp"

appendConf "/etc/nova/nova.conf" "placement" "region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://openstack-controller:5000/v3
username = placement
password = placement"

appendConf "/etc/nova/nova.conf" "libvirt" "virt_type = qemu"

# 检查是否开启虚拟化
egrep -c '(vmx|svm)' /proc/cpuinfo

# 重启服务
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl restart libvirtd.service openstack-nova-compute.service

# 在控制节点查看计算节点
openstack compute service list --service nova-compute

# 让 nova-cell 发现 nova-computer 服务, 将计算节点自动的加入 cell 中
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

# 测试
openstack flavor create --vcpus 1 --ram 64 --disk 1 small
openstack flavor list
```

## 7. 部署网络服务 (Neutron)

> **组件作用**：Neutron 负责提供“网络即服务”，管理虚拟网络、子网、路由器、浮动IP等所有网络资源。

### 7.1 Neutron Controller

```bash
# https://docs.openstack.org/neutron/rocky/install/

# 数据库准备
mysql << END
create database neutron;
grant all privileges on neutron.* to 'neutron'@'localhost' identified by 'NEUTRON_DBPASS';
grant all privileges on neutron.* to 'neutron'@'%' identified by 'NEUTRON_DBPASS';
grant all privileges on neutron.* to 'neutron'@'openstack-controller' identified by 'NEUTRON_DBPASS';
END

# 数据库连接测试
mysql -uneutron -pNEUTRON_DBPASS neutron -e "show databases;"

# 创建 nova 用户
openstack user create --domain default --password neutron neutron

# 为 nova 授权
openstack role add --project service --user neutron admin

# 创建 nova service
openstack service create --name neutron --description "OpenStack Networking" network

# 创建 endpoints
openstack endpoint create --region RegionOne network public http://openstack-controller:9696
openstack endpoint create --region RegionOne network internal http://openstack-controller:9696
openstack endpoint create --region RegionOne network admin http://openstack-controller:9696

# docs: https://docs.openstack.org/neutron/rocky/install/controller-install-option2-rdo.html
# 安装 neutron
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y

# 编辑配置文件
clearConf "/etc/neutron/neutron.conf"

appendConf "/etc/neutron/neutron.conf" "database" "connection = mysql+pymysql://neutron:NEUTRON_DBPASS@openstack-controller/neutron"

appendConf "/etc/neutron/neutron.conf" "DEFAULT" "core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true"
appendConf "/etc/neutron/neutron.conf" "DEFAULT" "transport_url = rabbit://openstack:RABBIT_PASS@openstack-controller"
appendConf "/etc/neutron/neutron.conf" "DEFAULT" "auth_strategy = keystone"

appendConf "/etc/neutron/neutron.conf" "keystone_authtoken" "www_authenticate_uri = http://openstack-controller:5000
auth_url = http://openstack-controller:5000
memcached_servers = openstack-controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron"

appendConf "/etc/neutron/neutron.conf" "DEFAULT" "notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true"

appendConf "/etc/neutron/neutron.conf" "nova" "auth_url = http://openstack-controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = nova"

appendConf "/etc/neutron/neutron.conf" "oslo_concurrency" "lock_path = /var/lib/neutron/tmp"

# 编辑 ml2_conf
clearConf "/etc/neutron/plugins/ml2/ml2_conf.ini"

appendConf "/etc/neutron/plugins/ml2/ml2_conf.ini" "ml2" "type_drivers = flat,vlan,vxlan"
appendConf "/etc/neutron/plugins/ml2/ml2_conf.ini" "ml2" "tenant_network_types = vxlan"
appendConf "/etc/neutron/plugins/ml2/ml2_conf.ini" "ml2" "mechanism_drivers = linuxbridge,l2population"
appendConf "/etc/neutron/plugins/ml2/ml2_conf.ini" "ml2" "extension_drivers = port_security"

appendConf "/etc/neutron/plugins/ml2/ml2_conf.ini" "ml2_type_flat" "flat_networks = provider"

appendConf "/etc/neutron/plugins/ml2/ml2_conf.ini" "ml2_type_vxlan" "vni_ranges = 1:1000"

appendConf "/etc/neutron/plugins/ml2/ml2_conf.ini" "securitygroup" "enable_ipset = true"

# 编辑 linuxbridge_agent
clearConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini"

appendConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini" "linux_bridge" "physical_interface_mappings = provider:eth1"

# 配置 eth1 网卡
nmcli con mod 'Wired connection 1' ipv4.add 172.16.100.11/24 ipv4.meth manual ifname eth1 con-name eth1

appendConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini" "vxlan" "enable_vxlan = true
local_ip = 172.16.100.11
l2_population = true"

appendConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini" "securitygroup" "nable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver"

# 加载内核模块
modprobe br_netfilter
sysctl -a | grep -E 'net.bridge.bridge-nf-call-iptables|net.bridge.bridge-nf-call-ip6tables'

# 编辑 l3_agent
clearConf "/etc/neutron/l3_agent.ini"

appendConf "/etc/neutron/l3_agent.ini" "DEFAULT" "interface_driver = linuxbridge"

# 编辑 dhcp_agent
clearConf "/etc/neutron/dhcp_agent.ini"

appendConf "/etc/neutron/dhcp_agent.ini" "DEFAULT" "interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true"

# 编辑 metadata_agent
clearConf "/etc/neutron/metadata_agent.ini"

appendConf "/etc/neutron/metadata_agent.ini" "DEFAULT" "nova_metadata_host = openstack-controller
metadata_proxy_shared_secret = METADATA_SECRET"

# 配置 nova
appendConf "/etc/nova/nova.conf" "neutron" "url = http://openstack-controller:9696
auth_url = http://openstack-controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
service_metadata_proxy = true
metadata_proxy_shared_secret = METADATA_SECRET"

# 创建软连接
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini

# 同步数据库
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

# 重启计算接口服务
systemctl restart openstack-nova-api.service

# 启动网络服务
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service

# 查看网络
openstack network agent list
```

### 7.2 Neutron Computer

```bash
# https://docs.openstack.org/neutron/rocky/install/compute-install-rdo.html#install-the-components

# 安装
yum install openstack-neutron-linuxbridge ebtables ipset -y

# 配置 neutron
clearConf "/etc/neutron/neutron.conf"

appendConf "/etc/neutron/neutron.conf" "DEFAULT" "transport_url = rabbit://openstack:RABBIT_PASS@openstack-controller"
appendConf "/etc/neutron/neutron.conf" "DEFAULT" "auth_strategy = keystone"

appendConf "/etc/neutron/neutron.conf" "keystone_authtoken" "www_authenticate_uri = http://openstack-controller:5000
auth_url = http://openstack-controller:5000
memcached_servers = openstack-controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron"

appendConf "/etc/neutron/neutron.conf" "oslo_concurrency" "lock_path = /var/lib/neutron/tmp"

# 配置 linuxbridge_agent
clearConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini"
appendConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini" "linux_bridge" "physical_interface_mappings = provider:eth1"

# 配置 eth1 网卡
nmcli con mod 'Wired connection 1' ipv4.add 172.16.100.12/24 ipv4.meth manual ifname eth1 con-name eth1

appendConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini" "vxlan" "enable_vxlan = true
local_ip = 172.16.100.12
l2_population = true"

appendConf "/etc/neutron/plugins/ml2/linuxbridge_agent.ini" "securitygroup" "nable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver"

# 加载内核模块
modprobe br_netfilter
sysctl -a | grep -E 'net.bridge.bridge-nf-call-iptables|net.bridge.bridge-nf-call-ip6tables'

# 配置 nova
appendConf "/etc/nova/nova.conf" "neutron" "url = http://openstack-controller:9696
auth_url = http://openstack-controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = neutron"

# 重启计算节点 nova 服务
systemctl restart openstack-nova-compute.service

# 重启计算节点 neutron-linuxbridge-agent 服务
systemctl enable neutron-linuxbridge-agent.service
systemctl start neutron-linuxbridge-agent.service
```


