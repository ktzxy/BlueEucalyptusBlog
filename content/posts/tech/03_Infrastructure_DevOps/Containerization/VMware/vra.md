+++
date = '2026-05-20T09:39:24+08:00'
draft = false
title = 'vRA'
slug = "at1e2sxo3x"
description = "vRA"
summary = "vRA"
categories = ["📒虚拟化"]
tags = ["VMware"]
comments = true
+++

# vRA (VMware vRealize Automation)

## 部署要求

### 1. 官方文档

```bash
https://docs.vmware.com/cn/vRealize-Automation/index.html
```

### 2. 节点配置

| 用途                        | 磁盘大小                                    | CPU        | 内存           |
| :-------------------------- | ------------------------------------------- | ---------- | -------------- |
| VMware Aria Suite Lifecycle | 78G                                         | 2          | 6GB            |
| VMware Identity Manager     | 100G                                        | 8          | 16GB           |
| VMware Aria Automation      | 246G (仅适用单节点)<br /246G (仅适用单节点) | 12<br />24 | 48GB<br />96GB |

## 安装准备

### 1. ntp 准备

```bash
yum install chrony -y

sed -i "s|^#allow.*|allow 0.0.0.0/0|" /etc/chrony.conf
grep allow /etc/chrony.conf

sed -i '/^server/d' /etc/chrony.conf
sed -i '/server/a \
ntp.aliyun.com iburst' /etc/chrony.conf

systemctl enable chronyd.service && systemctl restart chronyd.service
chronyc sources
```

### 2. dns 准备

```bash
yum install unbound -y

sed -i 's/# interface: 0.0.0.0$/interface: 0.0.0.0/' /etc/unbound/unbound.conf
sed -i 's|# access-control: 0.0.0.0/0 refuse$|access-control: 0.0.0.0/0 allow|' /etc/unbound/unbound.conf

cat << "END" > /etc/unbound/local.d/moonpac.com.conf
local-zone: "moonpac.com." static
local-data: "moonpac.com. IN SOA ns.moonpac.com. root.moonpac.com. 1 1h 1h 1h 1h"
local-data: "moonpac.com. IN NS ns.moonpac.com."
local-data: "ns.moonpac.com. IN A 10.173.28.219"

local-data: "vcenter.moonpac.com. IN A 10.173.28.241"
local-data: "vrlcm.moonpac.com. IN A 172.24.98.192"
local-data: "vidm.moonpac.com. IN A 172.24.98.191"
local-data: "vra.moonpac.com. IN A 172.24.98.190"

local-data-ptr: "10.173.28.241 vcenter.moonpac.com"
local-data-ptr: "172.24.98.192 vrlcm.moonpac.com"
local-data-ptr: "172.24.98.191 vidm.moonpac.com"
local-data-ptr: "172.24.98.190 vra.moonpac.com"
END

cat << "END" > /etc/unbound/conf.d/forward.com.conf
server:
    domain-insecure: "com."
    domain-insecure: "net."
    domain-insecure: "org."
forward-zone:
    name: "."
    forward-addr: 114.114.114.114
END

unbound-control-setup

systemctl enable unbound.service --now
systemctl disable firewalld.service --now

ss -tunlp | grep :53
```

## 开始安装

### 1. 挂载拷贝

```bash
sudo mount /iso/VMware-Aria-Automation-Lifecycle-Installer-21628956.iso /cdrom/
cp -r /cdrom/* ~/Templates/
chmod -R 777 ~/Templates/{ova,vrlcm,vrlcm-ui-installer}
```

![image-20240325153855928](https://typora.fanyops.com/documents/2024/0325/image-20240325153855928.png)

### 2. 接受协议

![image-20240325153950280](https://typora.fanyops.com/documents/2024/0325/image-20240325153950280.png)

### 3. 部署目标

![image-20240325154100463](https://typora.fanyops.com/documents/2024/0325/image-20240325154100463.png)

### 4. 安装位置

![image-20240325154148350](https://typora.fanyops.com/documents/2024/0325/image-20240325154148350.png)

### 5. 选择计算资源

![image-20240325154208635](https://typora.fanyops.com/documents/2024/0325/image-20240325154208635.png)

### 6. 选择存储位置

![image-20240325154438236](https://typora.fanyops.com/documents/2024/0325/image-20240325154438236.png)

### 7. 网络配置

![image-20240325154515257](https://typora.fanyops.com/documents/2024/0325/image-20240325154515257.png)

### 8. 密码配置

![image-20240325154607666](https://typora.fanyops.com/documents/2024/0325/image-20240325154607666.png)

### 9. vRLCM 配置

![image-20240325154656161](https://typora.fanyops.com/documents/2024/0325/image-20240325154656161.png)

### 10. vIDM 配置

![image-20240325154831314](https://typora.fanyops.com/documents/2024/0325/image-20240325154831314.png)

### 11. vRA 配置

![image-20240325154939400](https://typora.fanyops.com/documents/2024/0325/image-20240325154939400.png)

### 12. 开始安装

![image-20240325153050468](https://typora.fanyops.com/documents/2024/0325/image-20240325153050468.png)

### 13. 完成安装

![image-20240326100119084](https://typora.fanyops.com/documents/2024/0325/image-20240326100119084.png)

