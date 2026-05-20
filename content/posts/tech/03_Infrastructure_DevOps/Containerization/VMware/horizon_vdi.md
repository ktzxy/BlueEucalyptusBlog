+++
date = '2026-05-20T09:38:50+08:00'
draft = false
title = 'Horizon_VDI'
slug = "5c0uejahn3"
description = "Horizon_VDI"
summary = "Horizon_VDI"
categories = ["📒虚拟化"]
tags = ["VMware"]
comments = true
+++

# Virtual Desktop Infrastructure

## 节点规划

| 服务       | 主机名                    | OS               | IP地址        | 用户名                      | 密码       |
| ---------- | ------------------------- | ---------------- | ------------- | --------------------------- | ---------- |
| DNS Server | dns.moonpac.com           | AlmaLinux 8      | 10.173.28.219 | root                        | Iop[]=-09* |
| NTP Server | ntp.moonpac.com           | AlmaLinux 8      | 10.173.28.219 | root                        | Iop[]=-09* |
| vCenter    | vcenter.moonpac.com       | VMware Photon OS | 172.24.98.200 | <administrator@vsphere.local> | Iop[]=-09* |
| ESXi       | dev1.esxi.moonpac.com     | VMware ESXi      | 172.24.98.250 | root                        | Iop[]=-09* |
| AD Server  | ad.vdi.moonpac.com        | Win Server 2019  | 10.173.21.100 | administrator               | Iop[]=-09* |
| Horizon    | horizon.vdi.moonpac.com   | Win Server 2019  | 10.173.21.101 | administrator               | Iop[]=-09* |
| NSX        | nsx.vdi.moonpac.com       |                  | 10.173.21.102 | root                        | Iop[]=-09* |
| vRops      | vrops.vdi.moonpac.com     | VMware Photon OS | 10.173.21.103 | root                        | Iop[]=-09* |
| LogInsight | vli.vdi.moonpac.com       | VMware Photon OS | 10.173.21.104 | root                        | Iop[]=-09* |
| SqlServer  | sqlserver.vdi.moonpac.com | Win Server 2019  | 10.173.21.105 | administrator               | Iop[]=-09* |
| Win10-VDI  |                           | Win 10           |               | administrator               | Iop[]=-09* |

## Win 2019 Template

### 新建虚拟机

#### 1. 选择创建类型

![image-20240828090508615](https://typora.fanyops.com/documents/2024/0829/image-20240828090508615.png)

#### 2. 选择名称和文件夹

![image-20240828090541235](https://typora.fanyops.com/documents/2024/0829/image-20240828090541235.png)

#### 3. 选择计算资源

![image-20240828090606040](https://typora.fanyops.com/documents/2024/0829/image-20240828090606040.png)

#### 4. 选择存储

![image-20240828090727271](https://typora.fanyops.com/documents/2024/0829/image-20240828090727271.png)

#### 5. 选择兼容性

![image-20240828090752112](https://typora.fanyops.com/documents/2024/0829/image-20240828090752112.png)

#### 6. 选择客户机操作系统

![image-20240828090808589](https://typora.fanyops.com/documents/2024/0829/image-20240828090808589.png)

#### 7. 自定义硬件

##### 7.1 CPU

![image-20240828090909169](https://typora.fanyops.com/documents/2024/0829/image-20240828090909169.png)

##### 7.2 内存

![image-20240828091015416](https://typora.fanyops.com/documents/2024/0829/image-20240828091015416.png)

##### 7.3 硬盘

![image-20240828091044817](https://typora.fanyops.com/documents/2024/0829/image-20240828091044817.png)

##### 7.4 网络

![image-20240828091124743](https://typora.fanyops.com/documents/2024/0829/image-20240828091124743.png)

##### 7.5 CD/DVD 驱动器

![image-20240828091203264](https://typora.fanyops.com/documents/2024/0829/image-20240828091203264.png)

#### 8. 即将完成

![image-20240828091233375](https://typora.fanyops.com/documents/2024/0829/image-20240828091233375.png)

### 系统安装

#### 1. 选择创建的虚拟机

![image-20240828092155134](https://typora.fanyops.com/documents/2024/0829/image-20240828092155134.png)

#### 2. 打开 WEB 控制台

> 在如下界面需要按 “ESC” 键进入安装界面

![image-20240828092335396](https://typora.fanyops.com/documents/2024/0829/image-20240828092335396.png)

#### 3. 选择语言

![image-20240828092429509](https://typora.fanyops.com/documents/2024/0829/image-20240828092429509.png)

#### 4. 现在安装

![image-20240828092516983](https://typora.fanyops.com/documents/2024/0829/image-20240828092516983.png)

#### 5. 没有产品密钥

![image-20240828092551503](https://typora.fanyops.com/documents/2024/0829/image-20240828092551503.png)

#### 6. 选择桌面体验

![image-20240828092613579](https://typora.fanyops.com/documents/2024/0829/image-20240828092613579.png)

#### 7. 接受许可

![image-20240828092640624](https://typora.fanyops.com/documents/2024/0829/image-20240828092640624.png)

#### 8. 自定义

![image-20240828092709053](https://typora.fanyops.com/documents/2024/0829/image-20240828092709053.png)

#### 9. 选择硬盘

![image-20240828092725406](https://typora.fanyops.com/documents/2024/0829/image-20240828092725406.png)

#### 10. 开始安装

![image-20240828092754784](https://typora.fanyops.com/documents/2024/0829/image-20240828092754784.png)

#### 11. 设置管理员密码

![image-20240828093233732](https://typora.fanyops.com/documents/2024/0829/image-20240828093233732.png)

#### 12. 登录系统

![image-20240828093347207](https://typora.fanyops.com/documents/2024/0829/image-20240828093347207.png)

#### 13. 安装 VM Tools

![image-20240828093523371](https://typora.fanyops.com/documents/2024/0829/image-20240828093523371.png)

![image-20240828093540781](https://typora.fanyops.com/documents/2024/0829/image-20240828093540781.png)

![image-20240828093735068](https://typora.fanyops.com/documents/2024/0829/image-20240828093735068.png)

### 系统配置

#### 1. 设置分辨率

![image-20240828093942668](https://typora.fanyops.com/documents/2024/0829/image-20240828093942668.png)

#### 2. 设置远程登录

![image-20240828094113079](https://typora.fanyops.com/documents/2024/0829/image-20240828094113079.png)

#### 3. 激活系统

```bash
slmgr /ipk N69G4-B89J2-4G8F4-WWYCC-J464C
slmgr /skms kms.03k.org
slmgr /ato
```

### 转换为模板

![image-20240828110044621](https://typora.fanyops.com/documents/2024/0829/image-20240828110044621.png)

## Win 10 Template

### 新建虚拟机

#### 1. CPU

![image-20240828100235754](https://typora.fanyops.com/documents/2024/0829/image-20240828100235754.png)

#### 2. 内存

![image-20240828100257170](https://typora.fanyops.com/documents/2024/0829/image-20240828100257170.png)

#### 3. 硬盘

![image-20240828100321970](https://typora.fanyops.com/documents/2024/0829/image-20240828100321970.png)

#### 4. 网络

![image-20240828100347165](https://typora.fanyops.com/documents/2024/0829/image-20240828100347165.png)

#### 5. CD/DVD

![image-20240828100412365](https://typora.fanyops.com/documents/2024/0829/image-20240828100412365.png)

#### 6. 即将完成

![image-20240828100427973](https://typora.fanyops.com/documents/2024/0829/image-20240828100427973.png)

### 系统安装

#### 1. 选择磁盘

![image-20240828101701349](https://typora.fanyops.com/documents/2024/0829/image-20240828101701349.png)

#### 2. 选择区域

![image-20240828104202549](https://typora.fanyops.com/documents/2024/0829/image-20240828104202549.png)

#### 3. 创建用户

![image-20240828104311085](https://typora.fanyops.com/documents/2024/0829/image-20240828104311085.png)

#### 4. 设置密码

![image-20240828104327330](https://typora.fanyops.com/documents/2024/0829/image-20240828104327330.png)

#### 5. 设置隐私

![image-20240828104417473](https://typora.fanyops.com/documents/2024/0829/image-20240828104417473.png)

#### 6. 安装 VM Tools

![image-20240828104630368](https://typora.fanyops.com/documents/2024/0829/image-20240828104630368.png)

### 系统配置

#### 1. 设置分辨率

![image-20240828105518243](https://typora.fanyops.com/documents/2024/0829/image-20240828105518243.png)

#### 2. 设置远程登录

![image-20240828105629302](https://typora.fanyops.com/documents/2024/0829/image-20240828105629302.png)

## 虚拟机自定义规范

### Win 2019

![image-20240828142816674](https://typora.fanyops.com/documents/2024/0829/image-20240828142816674.png)

![image-20240828142905802](https://typora.fanyops.com/documents/2024/0829/image-20240828142905802.png)

![image-20240828142918766](https://typora.fanyops.com/documents/2024/0829/image-20240828142918766.png)

![image-20240828142946904](https://typora.fanyops.com/documents/2024/0829/image-20240828142946904.png)

![image-20240828143027787](https://typora.fanyops.com/documents/2024/0829/image-20240828143027787.png)

![image-20240828143144383](https://typora.fanyops.com/documents/2024/0829/image-20240828143144383.png)

![image-20240828143206379](https://typora.fanyops.com/documents/2024/0829/image-20240828143206379.png)

![image-20240828143242112](https://typora.fanyops.com/documents/2024/0829/image-20240828143242112.png)

![image-20240828143353639](https://typora.fanyops.com/documents/2024/0829/image-20240828143353639.png)

![image-20240828143422047](https://typora.fanyops.com/documents/2024/0829/image-20240828143422047.png)

![image-20240828143443141](https://typora.fanyops.com/documents/2024/0829/image-20240828143443141.png)

### Win 10

![image-20240829115436366](https://typora.fanyops.com/documents/2024/0829/image-20240829115436366.png)

![image-20240829115455862](https://typora.fanyops.com/documents/2024/0829/image-20240829115455862.png)

![image-20240829115520749](https://typora.fanyops.com/documents/2024/0829/image-20240829115520749.png)

![image-20240829115606280](https://typora.fanyops.com/documents/2024/0829/image-20240829115606280.png)

![image-20240829115631236](https://typora.fanyops.com/documents/2024/0829/image-20240829115631236.png)

![image-20240829115645849](https://typora.fanyops.com/documents/2024/0829/image-20240829115645849.png)

![image-20240829115655662](https://typora.fanyops.com/documents/2024/0829/image-20240829115655662.png)

![image-20240829115922251](https://typora.fanyops.com/documents/2024/0829/image-20240829115922251.png)

![image-20240829115818984](https://typora.fanyops.com/documents/2024/0829/image-20240829115818984.png)

![image-20240829115842159](https://typora.fanyops.com/documents/2024/0829/image-20240829115842159.png)

![image-20240829134259275](https://typora.fanyops.com/documents/2024/0829/image-20240829134259275.png)

![image-20240829134309476](https://typora.fanyops.com/documents/2024/0829/image-20240829134309476.png)

## AD 域配置

### 创建 AD

![image-20240828105909691](https://typora.fanyops.com/documents/2024/0829/image-20240828105909691.png)

![image-20240828110127714](https://typora.fanyops.com/documents/2024/0829/image-20240828110127714.png)

![image-20240828110140719](https://typora.fanyops.com/documents/2024/0829/image-20240828110140719.png)

![image-20240828110153691](https://typora.fanyops.com/documents/2024/0829/image-20240828110153691.png)

![image-20240828110203818](https://typora.fanyops.com/documents/2024/0829/image-20240828110203818.png)

### 设置 IP

![image-20240828111519236](https://typora.fanyops.com/documents/2024/0829/image-20240828111519236.png)

### 修改计算机名

![image-20240828111955480](https://typora.fanyops.com/documents/2024/0829/image-20240828111955480.png)

### 安装 AD 域

![image-20240828112951953](https://typora.fanyops.com/documents/2024/0829/image-20240828112951953.png)

![image-20240828113718646](https://typora.fanyops.com/documents/2024/0829/image-20240828113718646.png)

### 提升域控制器

![image-20240828113941137](https://typora.fanyops.com/documents/2024/0829/image-20240828113941137.png)

![image-20240828114038814](https://typora.fanyops.com/documents/2024/0829/image-20240828114038814.png)

![image-20240828114220220](https://typora.fanyops.com/documents/2024/0829/image-20240828114220220.png)

![image-20240828114338888](https://typora.fanyops.com/documents/2024/0829/image-20240828114338888.png)

![image-20240828114438746](https://typora.fanyops.com/documents/2024/0829/image-20240828114438746.png)

### 创建用户

![image-20240828134416880](https://typora.fanyops.com/documents/2024/0829/image-20240828134416880.png)

![image-20240828134551081](https://typora.fanyops.com/documents/2024/0829/image-20240828134551081.png)

![image-20240828134715255](https://typora.fanyops.com/documents/2024/0829/image-20240828134715255.png)

![image-20240828134748686](https://typora.fanyops.com/documents/2024/0829/image-20240828134748686.png)

![image-20240828134822240](https://typora.fanyops.com/documents/2024/0829/image-20240828134822240.png)

![image-20240828135246679](https://typora.fanyops.com/documents/2024/0829/image-20240828135246679.png)

![image-20240828135656047](https://typora.fanyops.com/documents/2024/0829/image-20240828135656047.png)

## SqlServer 部署

### 系统安装

![image-20240829092438125](https://typora.fanyops.com/documents/2024/0829/image-20240829092438125.png)

![image-20240829092458389](https://typora.fanyops.com/documents/2024/0829/image-20240829092458389.png)

![image-20240829092511607](https://typora.fanyops.com/documents/2024/0829/image-20240829092511607.png)

![image-20240829092528333](https://typora.fanyops.com/documents/2024/0829/image-20240829092528333.png)

![image-20240829092545055](https://typora.fanyops.com/documents/2024/0829/image-20240829092545055.png)

![image-20240829092609880](https://typora.fanyops.com/documents/2024/0829/image-20240829092609880.png)

![image-20240829092622161](https://typora.fanyops.com/documents/2024/0829/image-20240829092622161.png)

### SQLServer 安装

#### 1. 获取安装包

![image-20240829093637197](https://typora.fanyops.com/documents/2024/0829/image-20240829093637197.png)

#### 2. 挂载 ISO

![image-20240829093734019](https://typora.fanyops.com/documents/2024/0829/image-20240829093734019.png)

#### 3. 运行 setup

![image-20240829093830155](https://typora.fanyops.com/documents/2024/0829/image-20240829093830155.png)

#### 4. 安装全新的 SQL Server

![image-20240829094039586](https://typora.fanyops.com/documents/2024/0829/image-20240829094039586.png)

#### 5. 产品密钥

![image-20240829094216783](https://typora.fanyops.com/documents/2024/0829/image-20240829094216783.png)

#### 6. 许可条款

![image-20240829094246467](https://typora.fanyops.com/documents/2024/0829/image-20240829094246467.png)

#### 7. Microsoft 更新

![image-20240829094453592](https://typora.fanyops.com/documents/2024/0829/image-20240829094453592.png)

#### 8. 产品更新

> 取掉对钩

![image-20240829095254007](https://typora.fanyops.com/documents/2024/0829/image-20240829095254007.png)

#### 9. 安装规则

![image-20240829095412467](https://typora.fanyops.com/documents/2024/0829/image-20240829095412467.png)

#### 10. 关闭防火墙

![image-20240829095551673](https://typora.fanyops.com/documents/2024/0829/image-20240829095551673.png)

#### 11. 功能选择

![image-20240829095817446](https://typora.fanyops.com/documents/2024/0829/image-20240829095817446.png)

#### 12. 实例配置

![image-20240829095940769](https://typora.fanyops.com/documents/2024/0829/image-20240829095940769.png)

#### 13. 服务器配置

![image-20240829100156412](https://typora.fanyops.com/documents/2024/0829/image-20240829100156412.png)

#### 14. 数据库引擎配置

![image-20240829103955059](https://typora.fanyops.com/documents/2024/0829/image-20240829103955059.png)

#### 15. 准备安装

![image-20240829104028939](https://typora.fanyops.com/documents/2024/0829/image-20240829104028939.png)

#### 16. 完成安装

![image-20240829104944364](https://typora.fanyops.com/documents/2024/0829/image-20240829104944364.png)

### SSMS 安装

![image-20240829113256340](https://typora.fanyops.com/documents/2024/0829/image-20240829113256340.png)

![image-20240829113652393](https://typora.fanyops.com/documents/2024/0829/image-20240829113652393.png)

### 创建数据库

![image-20240829114246523](https://typora.fanyops.com/documents/2024/0829/image-20240829114246523.png)

![image-20240829114354341](https://typora.fanyops.com/documents/2024/0829/image-20240829114354341.png)

![image-20240829114455400](https://typora.fanyops.com/documents/2024/0829/image-20240829114455400.png)

## Horizon 安装

### 虚拟机创建

![image-20240828143655130](https://typora.fanyops.com/documents/2024/0829/image-20240828143655130.png)

![image-20240828143724498](https://typora.fanyops.com/documents/2024/0829/image-20240828143724498.png)

![image-20240828143736480](https://typora.fanyops.com/documents/2024/0829/image-20240828143736480.png)

![image-20240828143749538](https://typora.fanyops.com/documents/2024/0829/image-20240828143749538.png)

![image-20240828143800010](https://typora.fanyops.com/documents/2024/0829/image-20240828143800010.png)

![image-20240828143928383](https://typora.fanyops.com/documents/2024/0829/image-20240828143928383.png)

![image-20240828143936902](https://typora.fanyops.com/documents/2024/0829/image-20240828143936902.png)

### 加入域

![image-20240828145146499](https://typora.fanyops.com/documents/2024/0829/image-20240828145146499.png)

![image-20240828145238196](https://typora.fanyops.com/documents/2024/0829/image-20240828145238196.png)

![image-20240828145303012](https://typora.fanyops.com/documents/2024/0829/image-20240828145303012.png)

### 安装 Horizon

#### 准备安装包

![image-20240828150426398](https://typora.fanyops.com/documents/2024/0829/image-20240828150426398.png)

#### 获取 Keys

![image-20240828151517028](https://typora.fanyops.com/documents/2024/0829/image-20240828151517028.png)

#### 运行安装程序

![image-20240828151701114](https://typora.fanyops.com/documents/2024/0829/image-20240828151701114.png)

![image-20240828151721053](https://typora.fanyops.com/documents/2024/0829/image-20240828151721053.png)

![image-20240828152247509](https://typora.fanyops.com/documents/2024/0829/image-20240828152247509.png)

![image-20240828152303918](https://typora.fanyops.com/documents/2024/0829/image-20240828152303918.png)

![image-20240828152332797](https://typora.fanyops.com/documents/2024/0829/image-20240828152332797.png)

![image-20240828152408974](https://typora.fanyops.com/documents/2024/0829/image-20240828152408974.png)

![image-20240828152649545](https://typora.fanyops.com/documents/2024/0829/image-20240828152649545.png)

![image-20240828152701324](https://typora.fanyops.com/documents/2024/0829/image-20240828152701324.png)

![image-20240828153426982](https://typora.fanyops.com/documents/2024/0829/image-20240828153426982.png)

### Horizon 配置

#### 登录 Horizon

[https://horizon.vdi.moonpac.com/admin/#/login](https://horizon.vdi.moonpac.com/admin/#/login)

![image-20240828154809352](https://typora.fanyops.com/documents/2024/0829/image-20240828154809352.png)

#### 配置许可

![image-20240828154851450](https://typora.fanyops.com/documents/2024/0829/image-20240828154851450.png)

#### 添加 vCenter

![image-20240828155650053](https://typora.fanyops.com/documents/2024/0829/image-20240828155650053.png)

![image-20240828155113638](https://typora.fanyops.com/documents/2024/0829/image-20240828155113638.png)

![image-20240828155310586](https://typora.fanyops.com/documents/2024/0829/image-20240828155310586.png)

![image-20240828155336206](https://typora.fanyops.com/documents/2024/0829/image-20240828155336206.png)

![image-20240828155428630](https://typora.fanyops.com/documents/2024/0829/image-20240828155428630.png)

![image-20240828155439178](https://typora.fanyops.com/documents/2024/0829/image-20240828155439178.png)

#### 添加事件数据库

![image-20240829115013099](https://typora.fanyops.com/documents/2024/0829/image-20240829115013099.png)

![image-20240829115156375](https://typora.fanyops.com/documents/2024/0829/image-20240829115156375.png)

## vRops 部署

### 系统安装

#### 1. OVF 导入

![image-20240828172900920](https://typora.fanyops.com/documents/2024/0829/image-20240828172900920.png)

#### 2. 虚拟机名称

![image-20240828173034091](https://typora.fanyops.com/documents/2024/0829/image-20240828173034091.png)

#### 3. 选择计算资源

![image-20240828173104314](https://typora.fanyops.com/documents/2024/0829/image-20240828173104314.png)

#### 4. 查看详细信息

![image-20240828173148436](https://typora.fanyops.com/documents/2024/0829/image-20240828173148436.png)

#### 5. 许可协议

![image-20240828173243816](https://typora.fanyops.com/documents/2024/0829/image-20240828173243816.png)

#### 6. 配置

![image-20240828173325662](https://typora.fanyops.com/documents/2024/0829/image-20240828173325662.png)

#### 7. 选择存储

![image-20240828173342157](https://typora.fanyops.com/documents/2024/0829/image-20240828173342157.png)

#### 8. 选择网络

![image-20240828173357287](https://typora.fanyops.com/documents/2024/0829/image-20240828173357287.png)

#### 9. 自定义模板

![image-20240828173659709](https://typora.fanyops.com/documents/2024/0829/image-20240828173659709.png)

#### 10. 即将完成

![image-20240828173711081](https://typora.fanyops.com/documents/2024/0829/image-20240828173711081.png)

### 初始服务配置

[https://vrops.vdi.moonpac.com/](https://vrops.vdi.moonpac.com/)

#### 1. 开始

![image-20240828180750327](https://typora.fanyops.com/documents/2024/0829/image-20240828180750327.png)

#### 2. 新建集群

![image-20240828180803352](https://typora.fanyops.com/documents/2024/0829/image-20240828180803352.png)

#### 3. 设置管理员凭据

![image-20240828180821646](https://typora.fanyops.com/documents/2024/0829/image-20240828180821646.png)

#### 4. 选择证书

![image-20240828180916286](https://typora.fanyops.com/documents/2024/0829/image-20240828180916286.png)

#### 5. 部署设置

![image-20240828180949181](https://typora.fanyops.com/documents/2024/0829/image-20240828180949181.png)

#### 6. 配置可用性

![image-20240828181114813](https://typora.fanyops.com/documents/2024/0829/image-20240828181114813.png)

#### 7. 节点

![image-20240828181125126](https://typora.fanyops.com/documents/2024/0829/image-20240828181125126.png)

#### 8. 后续部署

![image-20240828181133679](https://typora.fanyops.com/documents/2024/0829/image-20240828181133679.png)

#### 9. 启动集群

![image-20240829110137384](https://typora.fanyops.com/documents/2024/0829/image-20240829110137384.png)

### vRops 使用

[https://vrops.vdi.moonpac.com/ui/login.action](https://vrops.vdi.moonpac.com/ui/login.action)

#### 1. 登录

![image-20240829110327314](https://typora.fanyops.com/documents/2024/0829/image-20240829110327314.png)

#### 2. 欢迎使用

![image-20240829110414987](https://typora.fanyops.com/documents/2024/0829/image-20240829110414987.png)

#### 3. 许可协议

![image-20240829110425646](https://typora.fanyops.com/documents/2024/0829/image-20240829110425646.png)

#### 4. 许可密钥

> JF69H-4DJ1K-480H8-N90QP-2AK64

![image-20240829110954966](https://typora.fanyops.com/documents/2024/0829/image-20240829110954966.png)

#### 5. 改善计划

![image-20240829111009048](https://typora.fanyops.com/documents/2024/0829/image-20240829111009048.png)

#### 6. 即将完成

![image-20240829111024234](https://typora.fanyops.com/documents/2024/0829/image-20240829111024234.png)

#### 7. 集成新的 vCenter

![image-20240829111537771](https://typora.fanyops.com/documents/2024/0829/image-20240829111537771.png)

![image-20240829111904431](https://typora.fanyops.com/documents/2024/0829/image-20240829111904431.png)

## LogInsight 部署

### 系统安装

#### 1. OVF 导入

![image-20240828174414138](https://typora.fanyops.com/documents/2024/0829/image-20240828174414138.png)

#### 2. 选择名称和文件夹

![image-20240828174517243](https://typora.fanyops.com/documents/2024/0829/image-20240828174517243.png)

#### 3. 选择计算资源

![image-20240828174538722](https://typora.fanyops.com/documents/2024/0829/image-20240828174538722.png)

#### 4. 查看详细信息

![image-20240828174616458](https://typora.fanyops.com/documents/2024/0829/image-20240828174616458.png)

#### 5. 许可协议

![image-20240828174628526](https://typora.fanyops.com/documents/2024/0829/image-20240828174628526.png)

#### 6. 配置

![image-20240828174638342](https://typora.fanyops.com/documents/2024/0829/image-20240828174638342.png)

#### 7. 选择存储

![image-20240828174652206](https://typora.fanyops.com/documents/2024/0829/image-20240828174652206.png)

#### 8. 选择网络

![image-20240828174703174](https://typora.fanyops.com/documents/2024/0829/image-20240828174703174.png)

#### 9. 自定义模板

![image-20240828174858878](https://typora.fanyops.com/documents/2024/0829/image-20240828174858878.png)

![image-20240828175019123](https://typora.fanyops.com/documents/2024/0829/image-20240828175019123.png)

#### 10. 即将完成

![image-20240828175033161](https://typora.fanyops.com/documents/2024/0829/image-20240828175033161.png)

### 服务配置

[https://vli.vdi.moonpac.com/](https://vli.vdi.moonpac.com/)

#### 1. 安装程序

![image-20240828181252172](https://typora.fanyops.com/documents/2024/0829/image-20240828181252172.png)

#### 2. 选择部署类型

![image-20240828181306729](https://typora.fanyops.com/documents/2024/0829/image-20240828181306729.png)

#### 3. 管理员凭证

![image-20240829104733354](https://typora.fanyops.com/documents/2024/0829/image-20240829104733354.png)

#### 4. 许可证

> JC002-0F110-48960-CC3Q6-1CAK4

![image-20240829105535353](https://typora.fanyops.com/documents/2024/0829/image-20240829105535353.png)

#### 5. 常规配置

![image-20240829105547002](https://typora.fanyops.com/documents/2024/0829/image-20240829105547002.png)

#### 6. 时间配置

![image-20240829105919812](https://typora.fanyops.com/documents/2024/0829/image-20240829105919812.png)

#### 7. SMTP 配置

![image-20240829105950983](https://typora.fanyops.com/documents/2024/0829/image-20240829105950983.png)

#### 8. SSL 配置

![image-20240829110010203](https://typora.fanyops.com/documents/2024/0829/image-20240829110010203.png)

#### 9. 设置完成

![image-20240829110019792](https://typora.fanyops.com/documents/2024/0829/image-20240829110019792.png)

#### 10. 添加 vCenter

![image-20240829112409185](https://typora.fanyops.com/documents/2024/0829/image-20240829112409185.png)

