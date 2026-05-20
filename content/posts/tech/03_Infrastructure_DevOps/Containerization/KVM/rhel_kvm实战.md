+++
date = '2026-05-13T17:18:27+08:00'
draft = false
title = 'RHEL_KVM实战'
slug = "xbp2tpf45z"
description = "RHEL_KVM实战"
summary = "RHEL_KVM实战"
categories = ["📒虚拟化"]
tags = ["KVM"]
comments = true
+++

# RHEL KVM 虚拟化深度实践指南

> 本指南旨在为系统管理员和运维工程师提供一份在 RHEL 及其衍生版系统上部署和管理 KVM 虚拟化的全面、深入的实践手册。内容涵盖了从环境安装、网络配置、虚拟机的创建与生命周期管理，到磁盘镜像和串口控制台等高级操作。

**核心概念简介**

- **KVM (Kernel-based Virtual Machine)**：是内建于 Linux 内核中的一种开源虚拟化技术，它能将 Linux 内核本身转变为一个高效的 Hypervisor (虚拟机监控器)。
- **QEMU (Quick EMUlator)**：一个通用的开源机器模拟器和虚拟器。在 KVM 场景中，QEMU 负责模拟具体的硬件设备（如磁盘、网卡），而 KVM 则负责提供对 CPU 和内存的硬件加速虚拟化。
- **Libvirt**: 一套用于管理虚拟化平台的开源 API、守护进程 (`libvirtd`) 和管理工具 (`virsh`, `virt-manager`)。它为各种虚拟化技术（包括 KVM/QEMU）提供了一个统一、便捷的管理接口。

## 1. KVM 环境安装与准备

### 1.1 检查硬件虚拟化支持

在安装前，请务必确认您的服务器 CPU 支持硬件虚拟化技术 (Intel VT-x 或 AMD-V)。

```bash
# 如果有输出，则表示支持
lscpu | grep -E "Virtualization|Hypervisor"
```

### 1.2 安装核心虚拟化软件包

安装 KVM/QEMU 平台、管理工具以及用于自定义镜像的工具。

```bash
yum install -y qemu-kvm libvirt virt-manager virt-install libguestfs-tools-c
```

### 1.3 启动并验证 Libvirt 服务

`libvirtd` 是管理虚拟机和相关资源的核心后台服务。

```bash
systemctl enable --now libvirtd.service

# 验证服务状态，并查看当前是否有虚拟机（此时应为空）
systemctl status libvirtd.service
virsh list --all
```

## 2. 虚拟网络配置

KVM 提供多种网络模式，最常用的是 **NAT 网络**（由 `libvirt` 管理）和**桥接网络**（由 `NetworkManager` 管理）。

### 2.1 使用 `virsh` 管理 Libvirt NAT 网络

这是 KVM 安装后的默认网络模式，虚拟机可以访问外网，但外网无法直接访问虚拟机。

**查看虚拟网络:**

```bash
virsh net-list --all
```

**创建自定义 NAT 网络:**

```bash
# 1. 准备 XML 配置文件
net_prefix='172.16'
cat > /tmp/net-${net_prefix}.0.0.xml << END
<network>
  <name>net-${net_prefix}.0.0</name>
  <bridge name="net-${net_prefix}.0.0"/>
  <forward mode='nat'/>
  <ip address="${net_prefix}.0.1" netmask="255.255.255.0">
    <dhcp>
      <range start="${net_prefix}.0.50" end="${net_prefix}.0.99"/>
    </dhcp>
  </ip>
</network>
END

# 2. 定义、启动并设置开机自启
virsh net-define /tmp/net-${net_prefix}.0.0.xml
virsh net-start net-${net_prefix}.0.0
virsh net-autostart net-${net_prefix}.0.0
```

**删除虚拟网络:**

```bash
virsh net-destroy default
virsh net-undefine default
```

### 2.2 使用 `nmcli` 创建桥接网络 (推荐)

桥接模式将虚拟机直接连接到物理网络，使其拥有与宿主机同网段的 IP 地址，可以被外部网络直接访问。

#### 场景一: 无 VLAN 的桥接网络

```bash
# 1. 创建名为 vm-bridge 的网桥，并配置静态 IP
nmcli con add con-name vm-bridge ifname vm-bridge type bridge \
  ipv4.addresses 172.24.100.2/24 ipv4.gateway 172.24.100.1 \
  ipv4.dns 114.114.114.114 ipv4.method manual ipv6.method disabled

# 2. 将物理网卡 (enp2s0) 桥接到 vm-bridge
# 注意：此操作会清空物理网卡的原有 IP 配置
nmcli con add con-name enp2s0-slave type ethernet ifname enp2s0 master vm-bridge

# 3. 激活连接
nmcli con up vm-bridge
```

#### 场景二: 带 VLAN 的桥接网络

```bash
# 1. 创建一个不带 IP 的网桥
nmcli con add con-name br-vlan-100 ifname br-vlan-100 type bridge stp no

# 2. 在物理网卡上创建 VLAN 子接口，并将其桥接到网桥
nmcli con add con-name enp2s0-vlan-100 ifname enp2s0.100 type vlan id 100 dev enp2s0 master br-vlan-100

# 3. 激活连接
nmcli con up br-vlan-100
```

> 创建虚拟机时，网络类型选择 `--network bridge=br-vlan-100` 即可接入 VLAN 100。

## 3. 虚拟机生命周期管理

### 3.1 图形化管理 (`virt-manager`)

对于桌面环境，`virt-manager` 提供了一个直观的图形界面来创建和管理虚拟机。

```bash
# 启动图形化虚拟机管理器
virt-manager
```

### 3.2 命令行创建 (`virt-install`)

`virt-install` 是一个功能强大的命令行工具，用于创建虚拟机，非常适合脚本化和自动化部署。

**常用参数详解:**

| 参数              | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| `--name`          | 虚拟机唯一名称。                                             |
| `--memory`        | 分配的内存大小 (MB)。                                        |
| `--vcpus`         | 分配的虚拟 CPU 核心数。                                      |
| `--disk`          | 定义磁盘。`path=` 指定镜像路径，`bus=virtio` 使用高性能 VirtIO 驱动。 |
| `--os-variant`    | 操作系统类型，用于优化性能。`osinfo-query os` 可查看所有支持类型。 |
| `--import`        | 导入现有磁盘镜像，跳过全新安装过程。                         |
| `--cdrom`         | 指定 ISO 镜像路径用于安装。                                  |
| `--location`      | 指定一个包含安装内核的网络位置（用于自动化安装）。           |
| `--extra-args`    | 向安装程序传递额外的内核参数，常用于指定 Kickstart 文件。    |
| `--network`       | 配置网络。`bridge=` 指定要连接的网桥。                       |
| `--graphics`      | 指定图形显示协议，如 `vnc`, `spice`。                        |
| `--noautoconsole` | 创建后不自动连接到虚拟机控制台。                             |

**示例一：从 ISO 本地安装**

```bash
virt-install \
--name test-vm-1 \
--memory 2048 \
--vcpus 2 \
--os-variant rhel9 \
--disk path=/var/lib/libvirt/images/test-vm-1.qcow2,size=20,bus=virtio \
--cdrom /path/to/rhel-9-x86_64-dvd.iso \
--network bridge=vm-bridge,model=virtio \
--graphics vnc
```

**示例二：使用 Kickstart 自动化安装**

> **注意**：此方法需要您预先准备好 Kickstart (`ks.cfg`) 自动应答文件，并通过一个 Web 服务器提供访问。

```bash
virt-install \
--name test-vm-2 \
--memory 1024 \
--vcpus 1 \
--os-variant centos-stream8 \
--disk path=/var/lib/libvirt/images/test-vm-2.qcow2,size=10,bus=virtio \
--location http://172.24.0.248/web/cdrom/CentOS-Stream/8/ \
--network bridge=default,model=virtio \
--extra-args "ks=http://172.24.0.248/ks8.cfg" \
--graphics vnc \
--noautoconsole
```

### 3.3 命令行管理 (`virsh`)

`virsh` 是管理虚拟机的主要命令行接口。

```bash
# 列出所有虚拟机 (运行中和已关闭)
virsh list --all

# 启动/关闭/重启虚拟机
virsh start test-vm-1
virsh shutdown test-vm-1
virsh reboot test-vm-1

# 强制关闭虚拟机 ("拔电源")
virsh destroy test-vm-1

# 删除虚拟机 (会删除配置文件，但不会删除磁盘镜像)
virsh undefine test-vm-1

# 连接到虚拟机的串口控制台 (需要虚拟机内部配置，见第 5 章)
virsh console test-vm-1
```

### 3.4 虚拟机 XML 配置文件

每个虚拟机都有一个 XML 配置文件（位于 `/etc/libvirt/qemu/`），它定义了该虚拟机的所有硬件和配置。

> **重要**：如果您对 XML 文件进行了高级的手动定制（`virsh edit`），请务必备份此文件。`virt-install` 无法覆盖所有 XML 中的高级选项。当您 `undefine` 虚拟机后，此文件将被删除。

**从备份的 XML 文件恢复虚拟机定义：**

```bash
virsh define /path/to/your_backup.xml
```

## 4. 磁盘镜像 (qcow2) 管理

### 4.1. 查看镜像信息

```bash
# 查看 qcow2 镜像的元数据，如虚拟大小、实际大小、格式等
qemu-img info your_vm_disk.qcow2

# 查看 qcow2 镜像内部的文件系统和分区情况
virt-filesystems --long -h --all -a your_vm_disk.qcow2
```

### 4.2. 扩容镜像与根分区

```bash
# 1. 扩大 qcow2 镜像文件的总容量 (例如扩容到 100G)
qemu-img resize source_disk.qcow2 100G

# 2. 使用 virt-resize 工具将镜像内的根分区 (/dev/sda1) 扩展到新容量
# 注意：此操作通常会创建一个新的、已扩容的磁盘文件
virt-resize --expand /dev/sda1 source_disk.qcow2 new_resized_disk.qcow2
```

### 4.3. 自定义镜像内容 (`virt-customize`)

`libguestfs-tools` 提供的 `virt-customize` 工具可以在不启动虚拟机的情况下，对其磁盘镜像进行修改。

```bash
# 示例：修改 GenericCloud 镜像的 root 密码，并重新标记 SELinux 上下文
virt-customize -a CentOS-Stream-GenericCloud-8.qcow2 --root-password password:YourSecurePassword --selinux-relabel
```

## 5. 高级主题: 配置串口控制台

当虚拟机没有网络或图形界面时，串口控制台是唯一的管理入口。

### 步骤 5.1: 在虚拟机内部配置 Grub

**对于 RHEL / CentOS / AlmaLinux:**

```bash
# 在虚拟机内部执行
sed -i 's/GRUB_CMDLINE_LINUX="[^"]*/& console=tty0 console=ttyS0,115200n8/' /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg
```

**对于 Ubuntu / Debian:**

```bash
# 在虚拟机内部执行
sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="[^"]*/& console=ttyS0,115200n8/' /etc/default/grub
update-grub
```

> 配置完成后重启虚拟机。

### 步骤 5.2：从宿主机连接

```bash
virsh console your_vm_name
```

> 按 `Ctrl + ]` 组合键可退出串口控制台会话。


