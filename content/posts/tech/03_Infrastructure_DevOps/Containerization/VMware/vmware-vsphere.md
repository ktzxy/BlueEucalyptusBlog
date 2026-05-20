+++
date = '2026-05-20T09:36:09+08:00'
draft = false
title = 'VMware vSphere'
slug = "el580wlpwc"
description = "VMware vSphere"
summary = "VMware vSphere"
categories = ["📒虚拟化"]
tags = ["VMware"]
comments = true
+++

# VMware vSphere 企业级虚拟化部署与管理指南

## 1. 前置知识 (Foundational Knowledge)

### 1.1 虚拟化解决了什么问题？

```bash
虚拟化的核心思想是资源整合与提高利用率。

举一个经典的例子：假设一个公司有三台物理服务器，分别承担邮件、Web 和内部应用服务。在传统架构下，每台服务器可能只使用了其计算能力的 30%。即使资源闲置，为了保证服务的独立性和稳定性，这三台服务器仍需独立运行，持续产生着电力、散热和维护成本。

虚拟化技术通过引入一个“Hypervisor”（虚拟机监控器）的抽象层，打破了“一台服务器、一个操作系统、一个应用”的僵化模式。
通过虚拟化，我们可以：
1. 整合负载：将原先运行在三台物理服务器上的邮件、Web 和内部应用，以“虚拟机 (VM)”的形式，整合到一台性能更强的物理服务器上运行
2. 提高资源利用率：这台物理服务器的资源利用率可以从 30% 提升到 80% 甚至更高，从而减少了闲置的硬件数量
3. 降低成本：闲置的服务器可以关停或挪作他用，显著降低了数据中心的总体拥有成本 (TCO)，包括硬件采购、电力、散热和物理空间成本
4. 提升灵活性：虚拟机本质上是文件，可以被轻松地备份、复制和迁移，为灾难恢复和快速部署提供了极大的便利
```

### 1.2 虚拟化的部署模式

```bash
常规的就是你的windows装一个 vmware workstation ，或者你的 linux 装个 qemu-kvm

使用带有 hypervisor 的操作系统，然后直接在上面安装虚拟机，而不是先有操作系统再安装虚拟化软件，然后再安装虚拟机

ESXi 就是带有 hypervisor 的操作系统，我们在安装完这个操作系统之后，直接在这个操作系统上创建虚拟机，这个操作系统就是为创建虚拟机而使用的。
其实任何主流的 Linux 发行版本都是带有 hypervisor 的操作系统， 而这个 hypervisor 是集成在 Linux 内核上的

ESXi/vSphere 是商业软件
```

### 1.3 选择正确的 ESXi ISO 版本

```bash
在安装前，选择与您的服务器硬件兼容的 ISO 镜像至关重要.
通用原则:
    最新硬件：通常完全兼容最新的 ESXi 版本 (如 8.0)
    较旧硬件：可能需要使用较旧的 ESXi 版本 (如 6.7, 7.0)，因为新版 ESXi 可能会移除对旧硬件驱动的支持
    不新不旧的硬件：一般选择主流稳定的版本 (如 7.0)
    
最佳实践:
    查询 VMware 兼容性指南 (HCL)：访问 VMware 官网的 Hardware Compatibility List，输入您的服务器型号，官方会明确告知支持哪个版本的 ESXi
    联系硬件厂商：最可靠的方法是联系您的服务器厂商（如 Dell, HPE, Lenovo）技术支持。他们通常会提供为其硬件定制化 (Customized) 的 ESXi ISO 镜像。这些镜像预集成了服务器特有的驱动程序（如 RAID 卡、网卡），能有效避免安装过程中的硬件识别问题
    厂商支持网站：在厂商的官方支持和驱动下载页面，通常也能找到推荐的 ESXi 定制化镜像
```

### 1.4 如何解决硬件不识别问题？

```bash
在物理服务器上安装 ESXi 时，最常见的问题是安装程序无法识别 RAID 卡或网卡，导致找不到可安装的磁盘或无法配置网络。
原因：官方通用版的 ESXi ISO 镜像仅包含主流和通用的驱动，可能不包含您服务器上特定或最新的硬件驱动
解决方案：
首选：获取厂商定制化 ISO：如上所述，这是最简单、最可靠的方法。
手动集成驱动：如果无法获取定制化 ISO，您需要：
    从硬件厂商网站下载适用于 ESXi 的离线驱动包 (VIB 文件)。
    使用 VMware 官方提供的 PowerCLI 工具或社区工具 ESXi-Customizer-PS 脚本，将下载的驱动包集成到原始的 ESXi ISO 镜像中，生成一个新的、包含所需驱动的 ISO 文件。这是一个相对高级的操作，需要一定的命令行基础。
```

## 2. ESXi 7.x 部署

### 2.1 下载 ESXi 7.x ISO

```bash
# 从官方或可靠来源获取 ISO 镜像，并制作成可引导的 USB 或通过服务器的 iDRAC/iLO 加载。
https://cios.dhitechnical.com/VMware/VMware%20vSphere%207/vSphere%20Hypervisor%20%28ESXi%29%207.0U3n/VMware-VMvisor-Installer-7.0U3n-21930508.x86_64.iso
```

### 2.2 Welcome to the VMware ESXi 7.0.3 Installation

服务器从安装介质引导后，将进入 "Welcome to the VMware ESXi 7.0.3 Installation" 界面。按 `Enter` 继续。

![image-20240606154634100](https://typora.fanyops.com/documents/2024/0607/image-20240606154634100.png)

### 2.3  End User License Agreement (EULA)

阅读协议内容，按 `F11` 接受并继续。

![image-20240606154724949](https://typora.fanyops.com/documents/2024/0607/image-20240606154724949.png)

### 2.4 Select a Disk to Install or Upgrade

安装程序会扫描并列出所有可用的本地存储设备。选择您希望将 ESXi 安装到的磁盘（通常是一块 RAID 1 的系统盘或 SD 卡）。按 `Enter` 继续。

![image-20240606154825831](https://typora.fanyops.com/documents/2024/0607/image-20240606154825831.png)

### 2.5 Please select a keyboard layout

保持默认的 `US Default` 即可。按 `Enter` 继续。

![image-20240606154859077](https://typora.fanyops.com/documents/2024/0607/image-20240606154859077.png)

### 2.6 Enter a root password

输入并确认 ESXi 主机的 `root` 管理员密码。**请务必使用强密码并妥善保管**，这是管理主机的最高权限。按 `Enter` 继续。

![image-20240606154939311](https://typora.fanyops.com/documents/2024/0607/image-20240606154939311.png)

### 2.7 Confirm Install

系统会警告所选磁盘将被重新分区和格式化。按 `F11` 开始安装。

![image-20240606155133721](https://typora.fanyops.com/documents/2024/0607/image-20240606155133721.png)

### 2.8 Installation Complete

安装过程很快。完成后，系统会提示您移除安装介质并按 `Enter` 重启服务器。

![image-20240606155527478](https://typora.fanyops.com/documents/2024/0607/image-20240606155527478.png)

### 2.9 登陆 ESXi

#### 2.9.1 Customize System/View Logs

重启后，您会看到 ESXi 的直接控制台用户界面 (Direct Console User Interface, DCUI)。这是一个基于文本的管理界面，用于进行最核心的配置

![image-20240606160315744](https://typora.fanyops.com/documents/2024/0607/image-20240606160315744.png)

#### 2.9.2 Authentication Required

按 `F2` 进入登录界面，输入之前设置的 `root` 密码

![image-20240606160433730](https://typora.fanyops.com/documents/2024/0607/image-20240606160433730.png)

### 2.10 配置管理网络

这是最关键的步骤，它让您可以通过 Web 浏览器远程管理 ESXi 主机。

#### 2.10.1 Configure Management Network

**选择网卡 (Network Adapters)**：选择一个或多个物理网卡作为管理网络接口。在生产环境中，通常会选择两个网卡做冗余

![image-20240606160815693](https://typora.fanyops.com/documents/2024/0607/image-20240606160815693.png)

#### 2.10.2 VLAN (optional)

**VLAN (optional)**：如果您的管理网络需要划分 VLAN，在此处输入对应的 VLAN ID。

![image-20240606160929461](https://typora.fanyops.com/documents/2024/0607/image-20240606160929461.png)

![image-20240606160951011](https://typora.fanyops.com/documents/2024/0607/image-20240606160951011.png)

#### 2.10.3 IPv4 Configuration

选择 `Set static IPv4 address and network configuration`。

输入为 ESXi 主机规划的静态 IP 地址、子网掩码和默认网关。

![image-20240606161126872](https://typora.fanyops.com/documents/2024/0607/image-20240606161126872.png)

#### 2.10.4 Test Management Network

在 DCUI 主菜单中，选择 `Test Management Network`。

![image-20240606161339953](https://typora.fanyops.com/documents/2024/0607/image-20240606161339953.png)

尝试 `ping` 您的网关地址和 DNS 服务器，确保网络连通性正常

![image-20240606161352070](https://typora.fanyops.com/documents/2024/0607/image-20240606161352070.png)

![image-20240606161405282](https://typora.fanyops.com/documents/2024/0607/image-20240606161405282.png)

#### 2.10.5 ping 测试

![image-20240606161504609](https://typora.fanyops.com/documents/2024/0607/image-20240606161504609.png)

### 2.11 WEB 端管理

![image-20240606161651758](https://typora.fanyops.com/documents/2024/0607/image-20240606161651758.png)

## 3. 网络配置

虚拟化环境的网络比物理环境更复杂，也更灵活。理解其核心组件是关键。

### 3.1 核心网络概念

**虚拟交换机 (vSwitch)**：

- 在 ESXi 内部工作的虚拟设备，功能类似于物理交换机。它负责将虚拟机、VMkernel 端口和物理网卡连接在一起，并转发它们之间的网络流量。
- ESXi 支持两种 vSwitch：**标准交换机 (Standard vSwitch)** 和 **分布式交换机 (Distributed vSwitch)**。本指南聚焦于配置简单的标准交换机。

**端口组 (Port Group)**：

- Switch 上的端口集合，定义了虚拟机如何连接到网络。您可以将端口组想象成物理交换机上一组配置了相同策略（如 VLAN ID）的端口。
- 虚拟机创建时，其虚拟网卡 (vNIC) 会连接到某个特定的端口组。

**VMkernel NIC (vmk)**：

- ESXi 主机自身用于网络通信的特殊虚拟接口。它拥有自己的 IP 地址，用于处理管理流量 (`Management`)、虚拟机实时迁移 (`vMotion`)、存储网络流量 (`iSCSI`, `NFS`) 等非虚拟机业务的流量。

### 3.2 网络规划最佳实践

在企业环境中，通常会将不同类型的流量隔离在不同的网络中，以提高性能和安全性

- **管理网络 (Management Network)**：用于 ESXi 主机和 vCenter 的管理通信。
- **业务网络 (VM Network)**：虚拟机对外提供服务所使用的网络。
- **存储网络 (Storage Network)**：用于连接 iSCSI 或 NFS 等网络存储，对性能和延迟要求极高，通常是独立的物理网络。
- **迁移网络 (vMotion Network)**：用于虚拟机在线热迁移，需要高带宽以减少迁移时间。

### 3.3 实践：配置 VLAN 业务网络

以下场景为例：我们将 ESXi 的一块物理网卡连接到思科交换机的 Trunk 端口，并为 `VLAN 10` 创建一个业务网络。

![image-20250901163012174](https://typora.fanyops.com/documents/2025/0901/image-20250901163012174.png)

#### 3.3.1 思科交换机配置 (示例)

物理交换机端口需要配置为 Trunk 模式，以允许携带多个 VLAN 的流量。

```bash
! 进入特权EXEC模式
enable

! 进入特权EXEC模式
configure terminal

! 配置连接 ESXi 的物理接口 e0/0
interface e0/0
  description "Link to ESXi Host vmnic1"
  ! 将接口模式设置为 Trunk
  switchport mode trunk
  ! (可选) 定义Trunk封装协议，较新设备默认为 dot1q
  switchport trunk encapsulation dot1q
  ! (可选) 定义允许通过的 VLAN，建议按需放行
  switchport trunk allowed vlan 10,20,30
  no shutdown

! (可选) 创建 VLAN 10 的三层接口(SVI) 作为网关
interface Vlan10
  ip address 10.1.10.254 255.255.255.0
  no shutdown

! 保存配置
end
write memory

! 显示所有接口的IP地址和状态摘要，用于验证刚刚的配置是否正确生效
do show ip interface brief
```

#### 3.3.2 创建虚拟交换机 (vSwitch)

```bash
登录到 ESXi 主机 Web UI，我们将创建一个新的 vSwitch 来处理数据流量，并将其绑定到专用的物理网卡。
1. 在左侧导航栏中，选择 “网络”。
2. 选择 “虚拟交换机” 选项卡。
3. 点击 “添加标准虚拟交换机”。
    vSwitch 名称: 输入 data-vswitch。
    上行链路: 选择 vmnic1 (即连接到交换机 e0/0 的那块物理网卡)
4. 点击 “添加”。
```

![image-20250901163720158](https://typora.fanyops.com/documents/2025/0901/image-20250901163720158.png)

![image-20250901165037843](https://typora.fanyops.com/documents/2025/0901/image-20250901165037843.png)

#### 3.3.3 创建端口组 (Port Group)

```bash
端口组是虚拟机和 VMkernel 端口连接到 vSwitch 的入口，我们在这里定义 VLAN ID。
1. 在 “网络” 页面，选择 “端口组” 选项卡。
2. 点击 “添加端口组”。
    名称: 输入 Data-vlan10。
    VLAN ID: 输入 10。这是最关键的一步，它会为所有离开此端口组的流量打上 VLAN 10 的标签。
    虚拟交换机: 从下拉列表中选择我们刚刚创建的 data-vswitch。
3. 点击 “添加”。
```

![image-20250901164649421](https://typora.fanyops.com/documents/2025/0901/image-20250901164649421.png)

![image-20250901165012710](https://typora.fanyops.com/documents/2025/0901/image-20250901165012710.png)

#### 3.3.4 创建 vlan10 端口组

![image-20250901175939851](https://typora.fanyops.com/documents/2025/0901/image-20250901175939851.png)

#### 3.3.5 (可选) 为 VLAN 10 创建 VMkernel 接口

```bash
为了让 ESXi 主机本身能够管理和访问 VLAN 10 网络，我们需要为其创建一个 VMkernel 端口。
1. 在 “网络” 页面，选择 “VMkernel NIC” 选项卡。
2. 点击 “添加 VMkernel NIC”。
    在 “选择目标设备” 步骤中，选择我们创建的端口组 Data-vlan10。
    在 “IPv4 设置” 步骤中:
        选择 “静态”。
        地址: 输入 10.1.10.1。
        子网掩码: 输入 255.255.255.0。
    服务: 您可以根据需要勾选服务（例如 vMotion、置备等），如果只是为了测试连通性，保持默认即可。
3. 点击 “创建”。

如果 ESXi 主机自身也需要在此 VLAN 中通信（例如，从这个网段访问 NFS 存储），可以创建一个 VMkernel 接口。
重要提示: 一个被 VMkernel 使用的端口组，通常不建议再给虚拟机使用，以实现管理流量和业务流量的隔离。最佳实践是为 VMkernel 和虚拟机创建不同的端口组，即使它们使用同一个 VLAN ID 和 vSwitch。
```

![Snipaste_2025-09-01_18-00-11](https://typora.fanyops.com/documents/2025/0901/Snipaste_2025-09-01_18-00-11.png)

![image-20250901180102722](https://typora.fanyops.com/documents/2025/0901/image-20250901180102722.png)

### 3.4 ESXi ping 测试

![image-20250901170458467](https://typora.fanyops.com/documents/2025/0901/image-20250901170458467.png)

### 3.5 抓包测试

![image-20250901170821263](https://typora.fanyops.com/documents/2025/0901/image-20250901170821263.png)

### 3.6 虚拟机配置

```bash
现在，我们可以创建一台新的虚拟机，或者编辑一台现有的虚拟机，将其连接到 VLAN 10。
1. 在 “虚拟机” 列表中，右键点击目标虚拟机，选择 “编辑设置”。
2. 在 “虚拟硬件” 选项卡中，找到 “网络适配器”。
3. 在网络适配器的下拉菜单中，选择我们创建的端口组 Data-vlan10。
4. 点击 “保存”。
5. 启动虚拟机，并进入其操作系统内部，为其配置一个静态 IP 地址。
    IP 地址: 10.1.10.2 (或其他未被占用的地址)
    子网掩码: 255.255.255.0
    默认网关: 10.1.10.254 (即思科交换机的 SVI 地址)
    DNS: (根据您的网络环境配置)
```

![image-20250901220143617](https://typora.fanyops.com/documents/2025/0901/image-20250901220143617.png)

## 4. 存储管理

ESXi 主机需要一个被称为“数据存储 (Datastore)”的逻辑容器来存放虚拟机文件。数据存储可以基于本地磁盘，也可以基于网络存储。

### 4.1 存储类型简介

- **本地存储 (Local Storage)**：直接使用服务器内部的硬盘。配置简单，但数据仅存在于单台主机，无法实现 vMotion 等高级功能。
- **网络附加存储 (NAS)**：通过网络文件共享协议（如 **NFS**）提供存储。配置相对简单，性价比高。
- **存储区域网络 (SAN)**：通过块级协议（如 **iSCSI**, **Fibre Channel**）提供存储。性能最高，是大型企业环境的主流选择。

### 4.2 实践：挂载 NFS 数据存储

以挂载 NFS 存储为例：

**准备 NFS 服务器**：确保您有一台 NFS 服务器，并已创建了一个共享目录，同时授权了 ESXi 主机的管理 IP 地址有权访问。

#### 4.2.1 New datastore

在 ESXi Web UI 中，导航至 **存储 -> 数据存储**。点击 **“新建数据存储”**。

![image-20240606162558197](https://typora.fanyops.com/documents/2024/0607/image-20240606162558197.png)

#### 4.2.2 Mount NFS datastore

选择 **“挂载 NFS 数据存储”**。

![image-20240606162702009](https://typora.fanyops.com/documents/2024/0607/image-20240606162702009.png)

#### 4.2.3 Provide NFs mount details

**提供 NFS 详细信息**

- **名称**: 为此数据存储命名，如 `NFS-Datastore-01`。
- **NFS 服务器**: 输入 NFS 服务器的 IP 地址或主机名。
- **NFS 共享**: 输入 NFS 服务器上共享的目录路径，例如 `/vol/esxi_data`。
- **NFS 版本**: 通常选择 `NFS 3` 或 `NFS 4.1`，根据您的服务器配置。

![image-20240606162759215](https://typora.fanyops.com/documents/2024/0607/image-20240606162759215.png)

#### 4.2.4 Ready to complete

![image-20240606162830091](https://typora.fanyops.com/documents/2024/0607/image-20240606162830091.png)

#### 4.2.5 Datastore browser

![image-20240606162944800](https://typora.fanyops.com/documents/2024/0607/image-20240606162944800.png)

![image-20240606163000494](https://typora.fanyops.com/documents/2024/0607/image-20240606163000494.png)

## 5. 虚拟机安装

所有基础设置就绪后，我们就可以创建虚拟机了。

### 5.1 Create / Register VM

![image-20240606162113188](https://typora.fanyops.com/documents/2024/0607/image-20240606162113188.png)

### 5.2 Create a new virtual machine

- 导航至 **“虚拟机”**，点击 **“创建/注册虚拟机”**。
- 选择 **“创建新虚拟机”**。

![image-20240606162134895](https://typora.fanyops.com/documents/2024/0607/image-20240606162134895.png)

### 5.3 Select a name and huest OS

- **名称**: 为虚拟机命名，如 `CentOS7-Test-VM`。
- **兼容性**: 保持默认即可。
- **客户机操作系统系列**: `Linux`
- **客户机操作系统版本**: `CentOS 7 (64-bit)`。

![image-20240606162237264](https://typora.fanyops.com/documents/2024/0607/image-20240606162237264.png)

### 5.4 Select storage

- 选择一个数据存储来存放此虚拟机的相关文件（如之前创建的 `NFS-Datastore-01`）。

![image-20240606163228719](https://typora.fanyops.com/documents/2024/0607/image-20240606163228719.png)

### 5.5 Customize settings

- **CPU**: 分配虚拟 CPU (vCPU) 的数量。
- **内存**: 分配内存大小。
- **硬盘**: 创建一个新的虚拟硬盘 (VMDK)，并指定其大小和置备类型（**精简置备**-用多少占多少，**厚置备**-立即分配所有空间）。
- **网络适配器**: 选择连接的端口组，例如 `Data-vlan10`。
- **CD/DVD 驱动器**: 选择 **“数据存储 ISO 文件”**，然后浏览并选择您已上传到数据存储的 CentOS 7 安装镜像。**务必勾选“打开电源时连接”**。

![image-20240606162356387](https://typora.fanyops.com/documents/2024/0607/image-20240606162356387.png)

![image-20240606163325871](https://typora.fanyops.com/documents/2024/0607/image-20240606163325871.png)

### 5.6 Ready to complete

- 检查配置摘要，点击 **“完成”**。

![image-20240606163401578](https://typora.fanyops.com/documents/2024/0607/image-20240606163401578.png)

### 5.7 Power on

- 在虚拟机列表中找到新创建的虚拟机，点击 **“打开电源”**。
- 点击虚拟机缩略图或 **“启动控制台”** 打开虚拟机屏幕。
- 此时，虚拟机会从您指定的 ISO 文件启动，进入 CentOS 7 的标准安装流程。按照操作系统提示完成安装即可。

![image-20240606164639470](https://typora.fanyops.com/documents/2024/0607/image-20240606164639470.png)

