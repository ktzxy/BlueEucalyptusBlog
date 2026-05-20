+++
date = '2026-05-20T09:39:49+08:00'
draft = false
title = 'vCenter'
slug = "ym9ea048ox"
description = "vCenter"
summary = "vCenter"
categories = ["📒虚拟化"]
tags = ["VMware"]
comments = true
+++

# VMware vCenter Server 7.x 部署与核心功能指南

## 1. 核心概念：为什么需要 vCenter Server？

```bash
我们之前学习的 ESXi 管理是“单机版”模式。当环境中只有少数几台 ESXi 主机时，通过 Web 界面逐一登录管理尚可接受。但当主机数量增长到数十甚至数百台时，这种方式将成为一场管理灾难。

vCenter Server Appliance (VCSA) 正是为解决这一挑战而生。它是 vSphere 套件中功能最强大的核心组件之一，扮演着整个虚拟化平台的“大脑”角色。
```

### 1.1 VCSA 带来的核心价值

vCenter Server 的引入，为您的虚拟化环境带来了质的飞跃：

- **集中化管理 (Centralized Management)：** 将数据中心内所有的 ESXi 主机、虚拟机、存储和网络资源纳入统一的 Web 管理界面，实现对整个虚拟化环境的全局监控和配置。
- **解锁企业级高级功能：** 许多 vSphere 的标志性功能**必须依赖 vCenter Server** 才能实现，单台 ESXi 无法提供。这些功能包括：
  - **vMotion：** 在不中断服务的情况下，将正在运行的虚拟机从一台物理主机实时迁移到另一台。
  - **High Availability (HA)：** 在物理主机发生故障时，自动在集群内的其他健康主机上重启受影响的虚拟机，最大程度减少停机时间。
  - **Distributed Resource Scheduler (DRS)：** 持续监控集群内各主机的资源利用率，通过 vMotion 自动进行负载均衡，确保虚拟机获得最佳性能。
  - **Distributed Virtual Switch (DVS)：** 创建跨多台主机的统一虚拟交换机，简化网络配置和管理，增强网络功能。
  - **统一的权限管理 (RBAC)：** 基于角色的精细化访问控制，可以为不同用户或团队分配不同的管理权限。
  - **虚拟机克隆与模板：** 实现虚拟机的快速复制和标准化部署，极大提升交付效率。

### 1.2 VCSA 部署模式

- **嵌套部署 (Nested Deployment)**：**最常见、最推荐**的方式。将 VCSA 本身作为一个虚拟机，安装在某一台 ESXi 主机上。这种方式充分利用了虚拟化特性，VCSA 自身也可以被 HA 等功能保护，具备高可用性。
- **物理部署 (Physical Deployment)**：将 VCSA 直接安装在物理服务器上。这种方式仅在超大规模（如超过500台ESXi主机）的环境下才可能被考虑，极为罕见。

## 2. VCSA 7.x 命令行部署实践

VCSA 的部署可以通过图形化界面 (GUI) 或命令行 (CLI) 完成。CLI 方式尤其适合自动化、标准化和重复性部署。本节详细介绍如何使用 CLI 在一台 Linux 主机上完成 VCSA 的部署。

### 2.1 准备工作

- **准备一台 Linux 操作机：** 用于执行部署命令。可以是 CentOS, Ubuntu, RHEL 等。
- 下载 VCSA ISO 镜像：
  - 官网地址: [VMware vSphere 7.0 Documentation](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vcenter.install.doc/GUID-C17AFF44-22DE-41F4-B85D-19B7A995E144.html)
  - 互联网镜像: `https://www.tssn.services/public/vmware/vcenter/VMware-VCSA-all-7.0.3-20395099.iso`
- **规划网络信息：** 准备好目标 ESXi 主机、VCSA 虚拟机以及 DNS、NTP 等服务的 IP 地址和凭据信息。

### 2.2 上传并挂载 ISO

将下载好的 ISO 文件上传到 Linux 操作机。

![image-20240606173545490](https://typora.fanyops.com/documents/2024/0607/image-20240606173545490.png)

```bash
# (可选) 校验 ISO 文件完整性，确保下载过程中没有损坏
sha512sum VMware-VCSA-all-7.0.3-20395099.iso
```

![image-20240606173650818](https://typora.fanyops.com/documents/2024/0607/image-20240606173650818.png)

```bash
# 挂载 ISO 文件
sudo mount VMware-VCSA-all-7.0.3-20395099.iso /mnt
```

![image-20240606173949463](https://typora.fanyops.com/documents/2024/0607/image-20240606173949463.png)

### 2.3 准备部署配置文件

从挂载的 ISO 中拷贝部署模板文件到用户主目录，这是部署的核心配置文件。

```bash
# 拷贝安装模板文件到用户主目录
cp /mnt/vcsa-cli-installer/templates/install/embedded_vCSA_on_ESXi.json ~
```

![image-20240606174029149](https://typora.fanyops.com/documents/2024/0607/image-20240606174029149.png)

### 2.4 编辑部署配置文件

这是部署过程中最关键的一步。你需要根据你的环境信息，详细配置  `embedded_vCSA_on_ESXi.json` JSON 文件。

```json
{
    "__version": "2.13.0",
    "__comments": "部署内嵌式 PSC 的 VCSA 到一台 ESXi 主机上的模板",
    "new_vcsa": {
        "esxi": {
            "__comments": "目标 ESXi 主机信息，VCSA 将作为虚拟机部署在这台主机上",
            "hostname": "10.1.1.10",               // 目标 ESXi 主机的 IP 地址
            "username": "root",                   // ESXi 主机的 root 用户名
            "password": "Your_ESXi_Password",     // ESXi 主机的 root 密码
            "deployment_network": "VM Network",     // VCSA 虚拟机管理网卡所连接的端口组名称
            "datastore": "datastore1"             // VCSA 虚拟机文件要存放的数据存储名称
        },
        "appliance": {
            "__comments": "VCSA 虚拟机本身的配置",
            "thin_disk_mode": true,               // 使用精简置备模式创建虚拟磁盘
            "deployment_option": "small",         // 部署规模: tiny, small, medium, large。决定 VCSA 的 vCPU、内存和存储大小
            "name": "vcsa-prod-01"                // VCSA 虚拟机的名称
        },
        "network": {
            "__comments": "VCSA 虚拟机的网络配置",
            "ip_family": "ipv4",
            "mode": "static",
            "ip": "10.1.1.11",                    // 分配给 VCSA 的静态 IP 地址
            "prefix": "24",                       // 子网掩码位数
            "gateway": "10.1.1.254",              // 网关地址
            "dns_servers": [ "223.5.5.5", "114.114.114.114" ] // DNS 服务器地址
        },
        "os": {
            "__comments": "VCSA 操作系统 (Photon OS) 的配置",
            "password": "Your_VCSA_Root_Password", // VCSA 虚拟机的 root 用户密码 (用于SSH登录)
            "ntp_servers": "ntp.aliyun.com",      // NTP 时间服务器
            "ssh_enable": true                    // 启用 SSH 服务
        },
        "sso": {
            "__comments": "vCenter Single Sign-On (SSO) 配置",
            "password": "Your_SSO_Admin_Password",// SSO 管理员 administrator 的密码
            "domain_name": "vsphere.local"        // SSO 域名，通常保持默认
        }
    },
    "ceip": {
        "__comments": "客户体验提升计划，建议禁用",
        "settings": {
            "ceip_enabled": false
        }
    }
}
```

![image-20230815102312941](https://typora.fanyops.com/documents/2024/0607/image-20230815102312941.png)

![image-20230815102437962](https://typora.fanyops.com/documents/2024/0607/image-20230815102437962.png)

### 2.5 执行安装命令

```bash
# (某些系统可能需要) 安装依赖库
sudo yum install libnsl -y

# 执行部署命令
/mnt/vcsa-cli-installer/lin64/vcsa-deploy install ~/embedded_vCSA_on_ESXi.json --accept-eula --no-ssl-certificate-verification -v --log-dir ~/vcsa-install-log
```

- `--accept-eula`: 接受最终用户许可协议。
- `--no-ssl-certificate-verification`: 忽略目标 ESXi 主机的 SSL 证书验证。
- `-v`: 显示详细输出。
- `--log-dir`: 指定日志文件存放目录。

部署过程分为两个阶段，耗时较长（通常为20-40分钟），请耐心等待命令行输出 `Installation complete successfully`。

![image-20240606175819404](https://typora.fanyops.com/documents/2024/0607/image-20240606175819404.png)

### 2.6 登录 vCenter Server

- 部署成功后，在浏览器中访问 `https://<你为VCSA配置的IP地址>`。
- 使用用户 `administrator@vsphere.local` 和你设置的 SSO 密码登录。

![image-20240607093756861](https://typora.fanyops.com/documents/2024/0607/image-20240607093756861.png)

## 3. vCenter 核心架构与清单管理

登录 VCSA 后，你将看到一个层级化的**清单 (Inventory)** 结构，这是 vCenter 管理的基础。

### 3.1 清单对象层级

- **数据中心 (Datacenter)**：是清单中的最高级别容器，是一个逻辑概念。它可以用来组织不同地理位置或不同用途的资源。例如，你可以创建 "北京-数据中心" 和 "上海-数据中心"。
- **集群 (Cluster)**：创建在数据中心下的对象，是 **ESXi 主机的集合**。**集群是实现 HA 和 DRS 等高级功能的边界**。
- **主机 (Host)**：即被 vCenter 纳管的 ESXi 物理服务器。

![image-20240607094745981](https://typora.fanyops.com/documents/2024/0607/image-20240607094745981.png)

![image-20240607094855284](https://typora.fanyops.com/documents/2024/0607/image-20240607094855284.png)

### 3.2 创建集群并添加主机

1. 在**数据中心**对象上右键，选择“新建集群”。
2. 为集群命名，并在此处**启用 vSphere HA 和 vSphere DRS** 等关键服务。
3. 在创建好的**集群**对象上右键，选择“添加主机”。
4. 输入 ESXi 主机的 IP 地址和 root 凭据，即可将其加入集群。

> **最佳实践**：强烈建议一个集群内的所有 ESXi 主机硬件配置（特别是 CPU 型号和代数）保持高度一致。这确保了 vMotion 的兼容性，并保证了 HA 故障切换后，虚拟机能够在任何一台主机上无缝运行。

![image-20240607104939814](https://typora.fanyops.com/documents/2024/0607/image-20240607104939814.png)

![image-20240607104836343](https://typora.fanyops.com/documents/2024/0607/image-20240607104836343.png)

![image-20240607104914627](https://typora.fanyops.com/documents/2024/0607/image-20240607104914627.png)

![image-20240607105837712](https://typora.fanyops.com/documents/2024/0607/image-20240607105837712.png)

![image-20240607105919717](https://typora.fanyops.com/documents/2024/0607/image-20240607105919717.png)

![image-20240607105933949](https://typora.fanyops.com/documents/2024/0607/image-20240607105933949.png)

![image-20240607105951102](https://typora.fanyops.com/documents/2024/0607/image-20240607105951102.png)

![image-20240607110008060](https://typora.fanyops.com/documents/2024/0607/image-20240607110008060.png)

![image-20240607110143034](https://typora.fanyops.com/documents/2024/0607/image-20240607110143034.png)

### 3.3 直接添加主机到数据中心

```bash
我们也可以直接将主机添加到数据中心下，不必非得将 ESXi 加入到某个集群。但这将使该主机无法使用 HA、DRS 等依赖于集群的功能。根据你的实际情况选择将 ESXi 加入到哪里。
```

> 当你添加完 ESXi 主机之后，这个主机中的存储和网络资源包括原来运行在 ESXi 上的虚拟机都会被 VCSA 检测到。

![image-20240607110253342](https://typora.fanyops.com/documents/2024/0607/image-20240607110253342.png)

![image-20240607110311665](https://typora.fanyops.com/documents/2024/0607/image-20240607110311665.png)

![image-20240607110325595](https://typora.fanyops.com/documents/2024/0607/image-20240607110325595.png)

## 4.  虚拟机生命周期管理

### 4.1 创建虚拟机

流程与单机版 ESXi 类似，但在 vCenter 中，你可以选择将虚拟机部署到集群或特定主机，并选择集群内所有主机均可访问的存储和网络。

![image-20240607110406843](https://typora.fanyops.com/documents/2024/0607/image-20240607110406843.png)

![image-20240607110448704](https://typora.fanyops.com/documents/2024/0607/image-20240607110448704.png)

![image-20240607110707611](https://typora.fanyops.com/documents/2024/0607/image-20240607110707611.png)

![image-20240607110720074](https://typora.fanyops.com/documents/2024/0607/image-20240607110720074.png)

![image-20240607110748935](https://typora.fanyops.com/documents/2024/0607/image-20240607110748935.png)

![image-20240607110800637](https://typora.fanyops.com/documents/2024/0607/image-20240607110800637.png)

![image-20240607110815157](https://typora.fanyops.com/documents/2024/0607/image-20240607110815157.png)

![image-20240607110850340](https://typora.fanyops.com/documents/2024/0607/image-20240607110850340.png)

![image-20240607110912306](https://typora.fanyops.com/documents/2024/0607/image-20240607110912306.png)

![image-20240607110928166](https://typora.fanyops.com/documents/2024/0607/image-20240607110928166.png)

![image-20240607111653439](https://typora.fanyops.com/documents/2024/0607/image-20240607111653439.png)

### 4.2 克隆虚拟机

这是 ESXi 单机模式不具备的核心功能。你可以对一台虚拟机（无论是开机还是关机状态）进行完整复制，快速创建副本。

**注意事项**：

- **存储位置**：如果克隆的目标数据存储与源虚拟机位于同一个**共享存储**上，克隆过程是存储内部的数据复制，速度很快。如果跨存储（尤其是跨主机的本地存储），则数据需要通过管理网络传输，会消耗网络带宽且速度较慢。
- **磁盘格式**：克隆时可以选择虚拟磁盘的置备模式，**精简置备 (Thin Provisioning)** 是最常用、最节省空间的模式。

![image-20240607112024141](https://typora.fanyops.com/documents/2024/0607/image-20240607112024141.png)

![image-20240607112046718](https://typora.fanyops.com/documents/2024/0607/image-20240607112046718.png)

![image-20240607112058088](https://typora.fanyops.com/documents/2024/0607/image-20240607112058088.png)

![image-20240607112115141](https://typora.fanyops.com/documents/2024/0607/image-20240607112115141.png)

![image-20240607112204726](https://typora.fanyops.com/documents/2024/0607/image-20240607112204726.png)

![image-20240607112215825](https://typora.fanyops.com/documents/2024/0607/image-20240607112215825.png)

![image-20240607112232097](https://typora.fanyops.com/documents/2024/0607/image-20240607112232097.png)

![image-20240607112250928](https://typora.fanyops.com/documents/2024/0607/image-20240607112250928.png)

### 4.3 使用模板高效部署

模板是标准化的虚拟机“母版”，它不可编辑、不可开机，专门用于快速、一致地部署新的虚拟机。

**获取模板的两种方式**：

#### 4.3.1 转换为模板

将一台已配置好的虚拟机（需处于关机状态）直接转换成模板。

![image-20240607112504009](https://typora.fanyops.com/documents/2024/0607/image-20240607112504009.png)

![image-20240607112807320](https://typora.fanyops.com/documents/2024/0607/image-20240607112807320.png)

#### 4.3.2 克隆一个模板

在克隆虚拟机的向导中，选择“克隆为模板”。

![image-20240607112439736](https://typora.fanyops.com/documents/2024/0607/image-20240607112439736.png)

![image-20240607112549928](https://typora.fanyops.com/documents/2024/0607/image-20240607112549928.png)

![image-20240607112651518](https://typora.fanyops.com/documents/2024/0607/image-20240607112651518.png)

![image-20240607112703305](https://typora.fanyops.com/documents/2024/0607/image-20240607112703305.png)

![image-20240607112717925](https://typora.fanyops.com/documents/2024/0607/image-20240607112717925.png)

![image-20240607112726776](https://typora.fanyops.com/documents/2024/0607/image-20240607112726776.png)

### 4.4 从模板部署虚拟机

**从模板部署虚拟机**：

- 在模板上右键，选择“从该模板新建虚拟机”。
- 在向导中，可以对新虚拟机进行自定义，如修改名称、IP 地址等（需要 Guest OS Customization Specification）。这对于批量部署服务器极为高效。

![image-20240607112940520](https://typora.fanyops.com/documents/2024/0607/image-20240607112940520.png)

![image-20240607113131858](https://typora.fanyops.com/documents/2024/0607/image-20240607113131858.png)

![image-20240607113029777](https://typora.fanyops.com/documents/2024/0607/image-20240607113029777.png)

![image-20240607113040455](https://typora.fanyops.com/documents/2024/0607/image-20240607113040455.png)

![image-20240607113050992](https://typora.fanyops.com/documents/2024/0607/image-20240607113050992.png)

![image-20240607113149347](https://typora.fanyops.com/documents/2024/0607/image-20240607113149347.png)

![image-20240607113208026](https://typora.fanyops.com/documents/2024/0607/image-20240607113208026.png)

## 5. vSphere 网络核心: 标准交换机 vs. 分布式交换机

```bash
当 VCSA 纳管多个 ESXi 主机后，资源整合便成为核心议题。计算资源相对容易整合（通过集群），但网络和存储资源则更为复杂。特别是网络，每个 ESXi 主机都有自己独立的 vSwitch 和 PortGroup，配置起来既繁琐又容易出错。

问题： 如果你有 100 台 ESXi 主机，需要创建一个名为 "VLAN-100-PROD" 的端口组，你将需要手动登录 100 次，或者编写复杂的脚本来完成，这极易导致配置不一致（如 VLAN ID 写错），造成虚拟机网络故障。
```

![image-20240607113830792](https://typora.fanyops.com/documents/2024/0607/image-20240607113830792.png)

![image-20240607115528872](https://typora.fanyops.com/documents/2024/0607/image-20240607115528872.png)

## 6. DVS（Distributed Virtual Switch）

为解决上述问题，vCenter 提供了 **分布式虚拟交换机 (DVS)** 这一高级功能。

- **DVS** 是一个逻辑上的、跨所有纳管主机的统一虚拟交换机。
- 你只需在 vCenter层面**创建一次 DVS**，并创建相应的**分布式端口组 (Distributed Port Group)**。
- 随后将集群内的所有 ESXi 主机添加到这个 DVS 中，这些主机上就会自动出现这个统一的交换机和端口组。
- **优势：** 实现了网络策略的**集中管理和配置下发**，确保了所有主机网络配置的一致性，极大简化了运维工作。

> **注意：** 当 ESXi 主机被 vCenter 纳管后，你将不能再通过 vCenter 界面在该主机上创建或修改“标准虚拟交换机 (Standard vSwitch)”，所有网络管理操作都推荐通过 DVS 进行。

![image-20240607115755589](https://typora.fanyops.com/documents/2024/0607/image-20240607115755589.png)

## 7. 自动化管理：PowerCLI 与 govc

vCenter 提供了强大的 API，可以通过命令行工具进行自动化管理，极大提升运维效率。

### 7.1 使用 PowerCLI 管理 vCenter

PowerCLI 是 VMware 官方提供的基于 PowerShell 的命令行工具集。

#### 7.1.1 在 Ubuntu 上安装 PowerCLI

```bash
# https://learn.microsoft.com/en-us/powershell/scripting/install/install-ubuntu?view=powershell-7.4

# 安装 PowerShell 依赖
sudo apt-get update
sudo apt-get install -y wget apt-transport-https software-properties-common

# 导入微软 GPG 密钥
source /etc/os-release
wget -q https://packages.microsoft.com/config/ubuntu/$VERSION_ID/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
sudo apt-get update

# 安装 PowerShell
sudo apt-get install -y powershell

# 启动 PowerShell
pwsh

# 在 PowerShell 环境中安装 PowerCLI 模块
Install-Module -Name VMware.PowerCLI -Scope CurrentUser
```

#### 7.1.2 PowerCLI 常用命令示例

```bash
# 忽略 SSL 证书警告 (仅限测试环境)
Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore -Confirm:$false

# 连接到 vCenter
Connect-VIServer -Server <vCenter-IP> -User <用户名> -Password <密码>
```

![image-20240126095523544](https://typora.fanyops.com/documents/2024/0126/image-20240126095523544.png)

```bash
# 获取所有虚拟机列表
Get-VM
```

![image-20240126095636871](https://typora.fanyops.com/documents/2024/0126/image-20240126095636871.png)

```bash
# 从虚拟机 Guest OS 复制文件到本地
Copy-VMGuestFile -Source "/etc/hosts" -Destination "C:\temp\" -VM "My-Linux-VM" -GuestToLocal -GuestUser root -GuestPassword 'YourGuestPassword'

# 在虚拟机 Guest OS 中执行命令
Invoke-VMScript -VM "My-Linux-VM" -ScriptText "ls /tmp" -GuestUser root -GuestPassword 'YourGuestPassword'
```

![image-20240126095711989](https://typora.fanyops.com/documents/2024/0126/image-20240126095711989.png)

```bash
# 断开连接
Disconnect-VIServer -Server <vCenter-IP> -Confirm:$false
```

### 7.2 使用 govc 创建虚拟机

govc 是一个轻量级的 vSphere CLI，基于 Go 语言开发，非常适合在 Linux/macOS 环境下进行脚本化操作。

```bash
# 第一步：设置环境变量进行认证
export GOVC_URL="https://10.xxx.xxx.xxx/sdk"
export GOVC_USERNAME="administrator@vsphere.local"
export GOVC_PASSWORD="xxxxxxxx"
export GOVC_INSECURE=1 # 忽略 SSL 证书验证

# 第二步：验证连接
govc about

# 第三步：批量克隆、自定义并启动虚拟机的示例脚本
for i in {141..150}; do
    VM_NAME="VMMPU22${i}"
    echo "Processing VM: $VM_NAME"

    # 从模板克隆虚拟机，指定网络、文件夹和资源池
    govc vm.clone -vm Win-10-Template -on=false -ds datastore-main \
        -net "Production-Network-PG" -folder "/shanghai-datacenter/vm/VDI" \
        -pool "/shanghai-datacenter/host/Cluster-01/Resources" \
        "$VM_NAME"

    # 应用客户机操作系统自定义规范，实现 IP 等配置自动化
    govc vm.customize -vm "$VM_NAME" -name Win-10-Custom-Spec \
        -ip 10.172.24.${i} -netmask 255.255.255.0 -gateway 10.172.24.1 \
        -dns-server 10.172.24.100,10.172.98.219 -dns-suffix vdi.sh.moonpac.com

    # 启动虚拟机
    govc vm.power -on "$VM_NAME"
done
```

## 8. 常见问题与实用技巧

### 8.1 EVC 特性报错与 BIOS 设置

- **问题：** 在集群中启用 EVC (Enhanced vMotion Compatibility) 特性时报错。
- **原因：** VMware EVC 要求物理主机的 BIOS 中必须启用 `Monitor/Mwait` 特性。
- **解决方案：** 重启 ESXi 主机，进入 BIOS 设置，路径通常在 `Advanced -> CPU Configuration` 或类似菜单下，找到并启用 `Monitor/Mwait` 选项。

### 8.2 vCenter 忘记 SSH root 密码

如果您忘记了 VCSA 的 root 用户密码，但仍能通过 `administrator@vsphere.local` 登录 Web 界面，可以按以下步骤重置：

```bash
# 使用 administrator@vsphere.local 账号通过 SSH 登录到 VCSA。
ssh administrator@vsphere.local@<VCSA-IP-Address>

# 进入 shell 环境。

# 使用 sudo 权限重置 root 密码。
sudo passwd root

# 重置 root 用户的登录失败计数。
sudo pam_tally2 --user=root --reset

# (可选) 设置 root 密码永不过期。
sudo chage -M -1 -E -1 root
```

