+++
date = '2026-05-20T10:37:59+08:00'
draft = false
title = 'zabbix部署'
slug = "1e6maoodof"
description = "zabbix部署"
summary = "zabbix部署"
categories = ["📒运维"]
tags = ["zabbix"]
comments = true
+++

# zabbix 6.x 企业级开源解决方案

## Zabbix 组件分离部署

### 1. 前置准备

#### 1.1 文档地址

[https://www.zabbix.com/cn/download?zabbix=6.0&os_distribution=red_hat_enterprise_linux&os_version=8&components=server_frontend_agent&db=mysql&ws=nginx](https://www.zabbix.com/cn/download?zabbix=6.0&os_distribution=red_hat_enterprise_linux&os_version=8&components=server_frontend_agent&db=mysql&ws=nginx)

#### 1.2 服务列表

| Address       | 描述          |
| ------------- | ------------- |
| 10.173.25.101 | zabbix-db     |
| 10.173.25.102 | zabbix-server |
| 10.173.25.103 | zabbix-web    |

### 2开始部署

#### 2.1 变量定义

```bash
# 分别登录节点 10.173.25.101, 10.173.25.102, 10.173.25.103
ZABBIX_DB=10.173.25.101
ZABBIX_SERVER=10.173.25.102
ZABBIX_WEB=10.173.25.103
```

#### 2.2 DB 部署

```bash
# 修改主机名
hostnamectl set-hostname zabbix-db

# 配置软件仓库
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-latest-6.0.el8.noarch.rpm
yum module disable mariadb -y
cat > /etc/yum.repos.d/mariadb.repo << 'END'
[MariaDB]
baseurl = https://downloads.mariadb.com/MariaDB/mariadb-10.5.28/yum/rhel/$releasever/$basearch
enabled = 1
gpgcheck = 0
name = MariaDB
module_hotfixes=1
END
yum clean all
yum makecache

# 安装数据库
yum -y install mariadb-server zabbix-sql-scripts zabbix-get
rpm -qal zabbix-sql-scripts

# 放行防火墙
firewall-cmd --add-service=mysql
firewall-cmd --add-service=mysql --per

# 启动并设置开机自启
systemctl enable mariadb --now

# 初始化数据库
mysql -e "create database zabbix character set utf8 collate utf8_bin;"
mysql -e "create user zabbix@'%' identified by 'zabbix'; grant all privileges on zabbix.* to zabbix@'%';"
mysql -e "create user zabbix@'localhost' identified by 'zabbix'; grant all privileges on zabbix.* to zabbix@'localhost';"
mysql -e "create user zabbix@'zabbix-server' identified by 'zabbix'; grant all privileges on zabbix.* to zabbix@'zabbix-server';"
mysql -e "flush privileges;"

# 导入初始架构和数据
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -pzabbix zabbix

# 查看数据表
mysql -uzabbix -pzabbix -e "use zabbix;" -e "show tables;"
```

#### 2.3 Server 部署

```bash
# 修改主机名
hostnamectl set-hostname zabbix-server

# 设置 selinux
setsebool -P zabbix_can_network on

# 配置软件仓库
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-latest-6.0.el8.noarch.rpm
dnf clean all

# 安装服务
yum -y install zabbix-server-mysql zabbix-selinux-policy zabbix-agent2 zabbix-get

# 规整 zabbix server 配置文件
ls /etc/zabbix/zabbix_server.conf.bak &>/dev/null || cp -r /etc/zabbix/zabbix_server.conf{,.bak}
egrep -v '^#|^$' /etc/zabbix/zabbix_server.conf.bak > /etc/zabbix/zabbix_server.conf

# 配置 zabbix server 数据库信息
grep '^DBHost' /etc/zabbix/zabbix_server.conf || echo 'DBHost=' >> /etc/zabbix/zabbix_server.conf
grep '^DBPassword' /etc/zabbix/zabbix_server.conf || echo 'DBPassword=' >> /etc/zabbix/zabbix_server.conf
sed -i "s/^DBHost.*/DBHost=${ZABBIX_DB}/" /etc/zabbix/zabbix_server.conf && grep '^DBHost' /etc/zabbix/zabbix_server.conf
sed -i 's/^DBPassword.*/DBPassword=zabbix/' /etc/zabbix/zabbix_server.conf && grep '^DBPassword' /etc/zabbix/zabbix_server.conf

# 规整 zabbix agent 配置文件
ls /etc/zabbix/zabbix_agent2.conf.bak &>/dev/null || cp -r /etc/zabbix/zabbix_agent2.conf{,.bak}
egrep -v '^#|^$' /etc/zabbix/zabbix_agent2.conf.bak > /etc/zabbix/zabbix_agent2.conf

# 配置 zabbix agent2 数据库信息
sed -i "s/^Server=.*/Server=${ZABBIX_DB},${ZABBIX_SERVER},${ZABBIX_WEB}/" /etc/zabbix/zabbix_agent2.conf && grep '^Server=' /etc/zabbix/zabbix_agent2.conf
sed -i "s/^ServerActive=.*/ServerActive=${ZABBIX_SERVER}/" /etc/zabbix/zabbix_agent2.conf && grep '^ServerActive=' /etc/zabbix/zabbix_agent2.conf
sed -i "s/^Hostname=.*/Hostname=$(hostname)/" /etc/zabbix/zabbix_agent2.conf && grep '^Hostname=' /etc/zabbix/zabbix_agent2.conf

# 启动并设置开机自启
systemctl enable zabbix-server zabbix-agent2 --now

# 防火墙放行
firewall-cmd --add-service=zabbix-server 
firewall-cmd --add-service=zabbix-server --per
firewall-cmd --add-service=zabbix-agent
firewall-cmd --add-service=zabbix-agent --per
```

#### 2.4 Web 部署

```bash
# 修改主机名
hostnamectl set-hostname zabbix-web

# 设置 selinux
setsebool -P httpd_can_network_connect_db on

# 配置软件仓库
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-latest-6.0.el8.noarch.rpm
dnf clean all

# 安装服务
yum -y install zabbix-web-mysql zabbix-nginx-conf zabbix-selinux-policy zabbix-agent2 zabbix-get

# 配置 WEB 服务
default_ip=$(ip route get 1.1.1.1 | awk '/src/ {for (i=1; i<=NF; i++) if ($i=="src") print $(i+1)}')
grep '^php_value\[date.timezone\]' /etc/php-fpm.d/zabbix.conf &>/dev/null || echo 'php_value[date.timezone] = Asia/Shanghai' >> /etc/php-fpm.d/zabbix.conf && grep '^php_value\[date.timezone\]' /etc/php-fpm.d/zabbix.conf 
sed -i 's/^#.*listen.*/        listen          80;/' /etc/nginx/conf.d/zabbix.conf && grep listen /etc/nginx/conf.d/zabbix.conf
sed -i "s/^.*server_name.*/        server_name     ${default_ip};/" /etc/nginx/conf.d/zabbix.conf && grep server_name /etc/nginx/conf.d/zabbix.conf

# 规整 zabbix agent 配置文件
ls /etc/zabbix/zabbix_agent2.conf.bak &>/dev/null || cp -r /etc/zabbix/zabbix_agent2.conf{,.bak}
egrep -v '^#|^$' /etc/zabbix/zabbix_agent2.conf.bak > /etc/zabbix/zabbix_agent2.conf

# 配置 zabbix agent2 数据库信息
sed -i "s/^Server=.*/Server=${ZABBIX_DB},${ZABBIX_SERVER},${ZABBIX_WEB}/" /etc/zabbix/zabbix_agent2.conf && grep '^Server=' /etc/zabbix/zabbix_agent2.conf
sed -i "s/^ServerActive=.*/ServerActive=${ZABBIX_SERVER}/" /etc/zabbix/zabbix_agent2.conf && grep '^ServerActive=' /etc/zabbix/zabbix_agent2.conf
sed -i "s/^Hostname=.*/Hostname=$(hostname)/" /etc/zabbix/zabbix_agent2.conf && grep '^Hostname=' /etc/zabbix/zabbix_agent2.conf

# 启动并设置开机自启
systemctl enable nginx php-fpm zabbix-agent2 --now

# 防火墙放行
firewall-cmd --add-service=http
firewall-cmd --add-service=http  --per
firewall-cmd --add-service=zabbix-agent
firewall-cmd --add-service=zabbix-agent  --per
```

#### 2.5 Agent 部署

```bash
# 配置软件仓库
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-latest-6.0.el8.noarch.rpm
dnf clean all

# 安装服务
yum -y install zabbix-agent2 zabbix-get

# 规整 zabbix agent 配置文件
ls /etc/zabbix/zabbix_agent2.conf.bak &>/dev/null || cp -r /etc/zabbix/zabbix_agent2.conf{,.bak}
egrep -v '^#|^$' /etc/zabbix/zabbix_agent2.conf.bak > /etc/zabbix/zabbix_agent2.conf

# 配置 zabbix agent2 数据库信息
sed -i "s/^Server=.*/Server=${ZABBIX_DB},${ZABBIX_SERVER},${ZABBIX_WEB}/" /etc/zabbix/zabbix_agent2.conf && grep '^Server=' /etc/zabbix/zabbix_agent2.conf
sed -i "s/^ServerActive=.*/ServerActive=${ZABBIX_SERVER}/" /etc/zabbix/zabbix_agent2.conf && grep '^ServerActive=' /etc/zabbix/zabbix_agent2.conf
sed -i "s/^Hostname=.*/Hostname=$(hostname)/" /etc/zabbix/zabbix_agent2.conf && grep '^Hostname=' /etc/zabbix/zabbix_agent2.conf

# 启动并设置开机自启
systemctl enable zabbix-agent2 --now

# 防火墙放行
firewall-cmd --add-service=zabbix-agent
firewall-cmd --add-service=zabbix-agent  --per
```

#### 2.6 Proxy 部署

```bash
# 修改主机名
hostnamectl set-hostname zabbix-proxy1

# 设置 selinux
setsebool -P zabbix_can_network on

# 配置软件仓库
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-latest-6.0.el8.noarch.rpm
dnf clean all

# 安装服务
yum -y install zabbix-proxy-mysql zabbix-sql-scripts zabbix-selinux-policy

# Make sure you have database server up and running.

# 初始化数据库
mysql -e "create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;"
mysql -e "create user zabbix@'%' identified by 'zabbix'; grant all privileges on zabbix_proxy.* to zabbix@'%';"
mysql -e "create user zabbix@localhost identified by 'zabbix'; grant all privileges on zabbix_proxy.* to zabbix@localhost;"
mysql -e "create user zabbix@'zabbix-proxy1' identified by 'zabbix'; grant all privileges on zabbix_proxy.* to zabbix@'zabbix-proxy1';"
mysql -e "flush privileges;"

# 导入初始架构和数据
cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql --default-character-set=utf8mb4 -uzabbix -pzabbix zabbix_proxy

# 规整 zabbix proxy 配置文件
ls /etc/zabbix/zabbix_proxy.conf.bak &>/dev/null || cp -r /etc/zabbix/zabbix_proxy.conf{,.bak}
egrep -v '^#|^$' /etc/zabbix/zabbix_proxy.conf.bak > /etc/zabbix/zabbix_proxy.conf

# 配置 zabbix proxy 配置文件
default_ip=$(ip route get 1.1.1.1 | awk '/src/ {for (i=1; i<=NF; i++) if ($i=="src") print $(i+1)}')
grep '^DBHost' /etc/zabbix/zabbix_proxy.conf || echo 'DBHost=' >> /etc/zabbix/zabbix_proxy.conf
grep '^DBPassword' /etc/zabbix/zabbix_proxy.conf || echo 'DBPassword=' >> /etc/zabbix/zabbix_proxy.conf
sed -i "s/^Server=.*/Server=${ZABBIX_SERVER}/" /etc/zabbix/zabbix_proxy.conf && grep '^Server' /etc/zabbix/zabbix_proxy.conf
sed -i "s/^Hostname=.*/Hostname=$(hostname)/" /etc/zabbix/zabbix_proxy.conf && grep '^Hostname' /etc/zabbix/zabbix_proxy.conf
sed -i "s/^DBHost=.*/DBHost=${default_ip}/" /etc/zabbix/zabbix_proxy.conf && grep '^DBHost' /etc/zabbix/zabbix_proxy.conf
sed -i "s/^DBPassword=.*/DBPassword=zabbix/" /etc/zabbix/zabbix_proxy.conf && grep '^DBPassword' /etc/zabbix/zabbix_proxy.conf

# 防火墙放行
firewall-cmd --add-service=zabbix-server
firewall-cmd --add-service=zabbix-server --per
firewall-cmd --add-service=zabbix-agent
firewall-cmd --add-service=zabbix-agent --per

# WEB 节点添加代理
# 登录 Zabbix Web 界面 → 配置 → 代理（Proxies）

# 启动并设置开机自启
systemctl enable --now zabbix-proxy
```

### 3. 测试服务

```bash
zabbix_get -s ${ZABBIX_DB} -p 10050 -k 'system.hostname'
zabbix_get -s ${ZABBIX_SERVER} -p 10050 -k 'system.hostname'
zabbix_get -s ${ZABBIX_WEB} -p 10050 -k 'system.hostname'
```

### 4. 网页初始化

#### 4.1 脚本方式

```bash
yum install langpacks-zh_CN -y

cat > /etc/zabbix/web/zabbix.conf.php << 'END'
<?php
$DB['TYPE']             = 'MYSQL';
$DB['SERVER']           = 'ZABBIX_DB';
$DB['PORT']             = '0';
$DB['DATABASE']         = 'zabbix';
$DB['USER']             = 'zabbix';
$DB['PASSWORD']         = 'zabbix';
$DB['SCHEMA']           = '';
$DB['ENCRYPTION']       = false;
$DB['KEY_FILE']         = '';
$DB['CERT_FILE']        = '';
$DB['CA_FILE']          = '';
$DB['VERIFY_HOST']      = false;
$DB['CIPHER_LIST']      = '';
$DB['VAULT_URL']        = '';
$DB['VAULT_DB_PATH']    = '';
$DB['VAULT_TOKEN']      = '';
$DB['DOUBLE_IEEE754']   = true;
$ZBX_SERVER_NAME        = 'zabbix-server';
$ZBX_SERVER             = 'ZABBIX_SERVER';
$IMAGE_FORMAT_DEFAULT   = IMAGE_FORMAT_PNG;
END

sed -i "s/ZABBIX_DB/${ZABBIX_DB}/" /etc/zabbix/web/zabbix.conf.php
sed -i "s/ZABBIX_SERVER/${ZABBIX_SERVER}/" /etc/zabbix/web/zabbix.conf.php
```

#### 4.2 浏览器方式

```bash
打开网页: http://localhost/setup.php
```

![image-20230927162351412](https://typora.fanyops.com/documents/2023/0927/image-20230927162351412.png)

```bash
# 安装语言包
yum install langpacks-zh_CN glibc-langpack-zh -y
```

![image-20230927162509615](https://typora.fanyops.com/documents/2023/0927/image-20230927162509615.png)

![image-20230927162538951](https://typora.fanyops.com/documents/2023/0927/image-20230927162538951.png)

![image-20230927162609499](https://typora.fanyops.com/documents/2023/0927/image-20230927162609499.png)

![image-20230927162623815](https://typora.fanyops.com/documents/2023/0927/image-20230927162623815.png)

![image-20230927162635861](https://typora.fanyops.com/documents/2023/0927/image-20230927162635861.png)

### 5. 登陆系统

```bash
# 默认用户名和密码
Admin/zabbix
```

![image-20230927162653855](https://typora.fanyops.com/documents/2023/0927/image-20230927162653855.png)

![image-20230927162815108](https://typora.fanyops.com/documents/2023/0927/image-20230927162815108.png)

## zabbix 模板配置

### 1. 主机群组

```bash
# 配置 > 主机群组 > 创建主机群组
```

![image-20231024100031325](https://typora.fanyops.com/documents/2023/1024/image-20231024100031325.png)

```bash
# 添加物理机群组
# PhysicalMachine
```

![image-20231024102924191](https://typora.fanyops.com/documents/2023/1024/image-20231024102924191.png)

### 2. 模板

#### 2.1 创建模板

```bash
# 配置 > 模板 > 创建模板
```

![image-20231024101234624](https://typora.fanyops.com/documents/2023/1024/image-20231024101234624.png)

```bash
# 模板名称: PhysicalMachine
# 群组: PhysicalMachine
# 描述: 安装了 zabbix-agent 的物理主机
```

![image-20231024103000149](https://typora.fanyops.com/documents/2023/1024/image-20231024103000149.png)

#### 2.2 模板 > 检测项

```bash
# 配置 > 模板 > 检测项 > 创建检测项
```

![image-20231024104850365](https://typora.fanyops.com/documents/2023/1024/image-20231024104850365.png)

![image-20231024105011031](https://typora.fanyops.com/documents/2023/1024/image-20231024105011031.png)

```bash
# 名称: 节点过去1分钟cpu的平均负载 (名称尽可能的见字而知其意)
# 键值: system.cpu.load[all,avg1]
# 信息类型: 浮点类型
# 更新间隔: 1m (单位有s,m,h,d,表示秒,分钟,小时,天)
# 其他保持默认
```

![image-20231024105754076](https://typora.fanyops.com/documents/2023/1024/image-20231024105754076.png)

#### 2.3 模板 > 图形

```bash
# 配置 > 模板 > 图形 > 创建图形
```

![image-20231024105938395](https://typora.fanyops.com/documents/2023/1024/image-20231024105938395.png)

```bash
# 配置 -> 模板 -> 图形 -> 创建图形
# 名称: 节点过去1分钟cpu的平均负载展示图 (名称尽可能的见字而知其意)
# 纵轴Y最小值MIN, 纵轴最大值: 
#     一般在生产环境都会设置固定的
#     固定值都在能容忍的最大值之上一点点
#     比如cpu负载最大容忍为2,那么纵轴的固定值应该比2高一点点
#     然后我们会设置两个触发器
#     比如值超过1了，我们会通过邮件报警
#     比如值超过1.5了，我们会设置微信或在钉钉报警
#     比如值超过2了，我们会设置短信或者电话报警
```

![image-20231024110212265](https://typora.fanyops.com/documents/2023/1024/image-20231024110212265.png)

```bash
# 图形乱码问题解决
# 在 windowns 找一个中文字体
mv /usr/share/fonts/dejavu/DejaVuSans.ttf{,.bak}
wget https://typora.fanyops.com/software/fonts/simhei.ttf
mv simhei.ttf /usr/share/fonts/dejavu/DejaVuSans.ttf
```

#### 2.4 模板 > 触发器

```bash
# 配置 -> 模板 -> 触发器 -> 创建触发器
# 名称: 主机节点cpu过去一分钟的平均负载超过0.8 (名称尽可能的见字而知其意)
# 表达式: {zabbix-server:system.cpu.load[all,avg1].last(#1,20)}>0.8
#        zabbix-server 表示主机
#        system.cpu.load[all,avg1]表示键值
#        last(#1,20)表示last函数,最后的1个值,持续20s
#        >0.8表示last函数的取值如果大于0.8,那么触发器就生效.
```

### 3. 主机

```bash
# 配置 > 主机 > 创建主机
```

![image-20231024103344914](https://typora.fanyops.com/documents/2023/1024/image-20231024103344914.png)

## zabbix 监控 snmp

### 1. 交换机或者系统配置 snmp 服务

```bash
# 安装软件
yum install net-snmp net-snmp-utils -y

# 备份配置文件
ls /etc/snmp/snmpd.conf.bak &>/dev/null || cp -r /etc/snmp/snmpd.conf{,.bak}

# 启动服务
systemctl enable snmpd.service --now

#  测试 snmp 服务
snmpwalk -v2c -c public localhost system

# 防火墙放行
firewall-cmd --add-port=161/udp
firewall-cmd --add-port=161/udp --per
```

### 2. zabbix 添加 snmp 设备

```bash
# 配置 > 主机 > 创建主机
# 主机名称: switch-10.173.26.1
# 模板: Cisco IOS by SNMP
# 群组: switch
# 接口: 10.173.26.1
#       SNMP v3
#       allbest
```

![image-20231026095338623](https://typora.fanyops.com/documents/2023/1026/image-20231026095338623.png)

## zabbix 监控 ipmi

### 1. 开启华为服务器的 ipmi

```bash
https://172.24.99.254/index.php
```

![image-20231025173626167](https://typora.fanyops.com/documents/2023/1025/image-20231025173626167.png)

### 2. zabbix server 开启 ipmi

```bash
grep '^StartIPMIPollers' /etc/zabbix/zabbix_server.conf || sed -i '$a StartIPMIPollers=5' /etc/zabbix/zabbix_server.conf
systemctl restart zabbix-server.service
```

### 3. zabbix web 添加 ipmi

```bash
# 主机 > 创建主机 > 模板 > 接口
```

![image-20231025174318843](https://typora.fanyops.com/documents/2023/1025/image-20231025174318843.png)

```bash
# 主机 > 创建主机 > IPMI
```

![image-20231025174506834](https://typora.fanyops.com/documents/2023/1025/image-20231025174506834.png)

## zabbix 监控 esxi

### 1. esxi 配置

```bash
# 选择 ESXi 主机 > 配置 > 高级系统配置 > Config.HostAgent.plugins.solo.enableMob
# 将 false 改为 true
```

![image-20231024142134993](https://typora.fanyops.com/documents/2023/1024/image-20231024142134993.png)

### 2. 修改 zabbix server 配置文件

```bash
StartVMwareCollectors=6     # 监控esxi的实例，一般写5就好
VMwareCacheSize=50M         # 从单个VMware服务收集数据之间的延迟(秒)
VMwareFrequency=10          # 用于存储VMware数据的共享内存大小
VMwarePerfFrequency=60      # 从单个VMware服务检索性能计数器统计信息之间的延迟(秒)
VMwareTimeout=300           # 响应超时时间

grep '^StartVMwareCollectors' /etc/zabbix/zabbix_server.conf || sed -i '$a StartVMwareCollectors=6' /etc/zabbix/zabbix_server.conf
grep '^VMwareCacheSize' /etc/zabbix/zabbix_server.conf || sed -i '$a VMwareCacheSize=50M' /etc/zabbix/zabbix_server.conf
grep '^VMwareFrequency' /etc/zabbix/zabbix_server.conf || sed -i '$a VMwareFrequency=10' /etc/zabbix/zabbix_server.conf
grep '^VMwarePerfFrequency' /etc/zabbix/zabbix_server.conf || sed -i '$a VMwarePerfFrequency=60' /etc/zabbix/zabbix_server.conf
grep '^VMwareTimeout' /etc/zabbix/zabbix_server.conf || sed -i '$a VMwareTimeout=300' /etc/zabbix/zabbix_server.conf
```

### 3. 重启 zabbix 服务并查看日志

```bash
systemctl restart zabbix-server.service
tail -f -n500 /var/log/zabbix/zabbix_server.log | grep 'VMware monitoring'
```

![image-20231024144134829](https://typora.fanyops.com/documents/2023/1024/image-20231024144134829.png)

### 4. zabbix web 添加 esxi 主机

```bash
# 配置 > 主机 > 创建主机
https://10.173.28.254/mob/?moid=ha-host&doPath=hardware.systemInfo

# vSphere 查看 UUID
https://10.172.23.52/mob/?moid=ServiceInstance&doPath=content.about
```

![image-20231024144935083](https://typora.fanyops.com/documents/2023/1024/image-20231024144935083.png)

![image-20231024151626832](https://typora.fanyops.com/documents/2023/1024/image-20231024151626832.png)

```bash
# 设置宏
{$VMWARE.HV.UUID}
{$VMWARE.URL}
{$VMWARE.USERNAME}
{$VMWARE.PASSWORD}
```

![image-20231024151439903](https://typora.fanyops.com/documents/2023/1024/image-20231024151439903.png)

## zabbix 监控 vcenter

### 1. 启用 VMware Collectors

```bash
vi /etc/zabbix/zabbix_server.conf

# 取消注释并设置一个大于0的值。这个值代表 Zabbix 会启动多少个进程来专门处理 VMware 数据采集。
# 初始值建议从 2 或 3 开始，根据你环境的大小（ESXi主机和虚拟机的数量）来调整。
# 如果你的 vCenter 环境很大，后续需要增加这个值。
StartVMwareCollectors=3

# (可选) 调整 VMware 缓存大小，对于大型环境有帮助。
# 默认是 8M，可以适当增加到 64M 或 128M。
VMwareCacheSize=64M

# (可选) 调整 API 请求的超时时间。如果你的 vCenter 响应慢，可以适当增加。
# 默认是 10 秒，可以增加到 20 或 30。
Timeout=20

# 修改配置后，必须重启 Zabbix Server 服务才能生效。
systemctl restart zabbix-server
```

### 2. 创建专用的 vCenter 只读用户

```bash
# 永远不要使用 administrator@vsphere.local 这样的管理员账户来做监控，这既不安全也不合规。
1. 登录 vCenter vSphere Client。
2. 进入 “管理” > “角色”。
3. 选择 “只读” 角色，确认它拥有 “System.View” 和 “System.Read” 等权限（默认就有）。你不需要创建新角色，使用内置的“只读”角色即可。
4. 进入 “管理” > “用户和组”，创建一个新用户，例如 zabbix-monitor。
5. 进入 “主机和集群” 视图，在 vCenter 对象的根级别（最顶层）上，右键点击选择 “添加权限”。
# 这样，zabbix-monitor 用户就拥有了对整个 vCenter 环境的只读权限，足以满足 Zabbix 的监控需求。
```

### 3. 在 Zabbix Web 界面中配置 vCenter 主机

```bash
1. 进入 **“配置” > “主机”**，点击 **“创建主机”**。
2. “主机” 选项卡:
    主机名称: 填写你的 vCenter Server 的 FQDN 或 IP 地址，例如 vcenter.lab.local。
    模板: 这是核心。点击“选择”，然后链接到官方的 VMware FQDN 或 VMware 模板。
        VMware FQDN: 推荐使用，它会自动发现 ESXi 主机和虚拟机，并以它们的 FQDN 命名。
        VMware: 旧版模板，以 UUID 命名，可读性较差。
    主机群组: 选择一个合适的群组，例如 Virtualization。
    接口: 不需要添加 Agent 接口。API 监控不依赖 Agent。可以留空或添加一个 JMX 接口作为占位符。
```

![image-20250918092004943](https://typora.fanyops.com/documents/2023/1024/image-20250918092004943.png)

```bash
3. “宏” 选项卡:
    这是配置认证信息的关键部分。选择“继承的宏和主机宏”。
    {$VMWARE.URL}: 填入 vCenter 的 SDK URL。格式为 https://<vcenter_fqdn_or_ip>/sdk。例如: https://vcenter.lab.local/sdk
    {$VMWARE.USERNAME}: 填入你刚刚创建的只读用户名，例如 zabbix-monitor。
    {$VMWARE.PASSWORD}: 填入该用户的密码。
4. 点击 “添加” 保存。
```

![image-20250918092410795](https://typora.fanyops.com/documents/2023/1024/image-20250918092410795.png)

## zabbix api 接口

### 1. shell 前置操作

```bash
# 配置 hosts 解析
grep zabbix.local.com /etc/hosts || echo "10.173.28.224 zabbix.local.com" >> /etc/hosts

# 安装 jq
yum install jq -y

# 生成 post 函数
function curlPost() {
    local url=http://zabbix.local.com/zabbix/api_jsonrpc.php
    local data=${1}

    resturl=$(curl -sS -X POST "${url}" -H "Content-Type: application/json-rpc" -d "${data}")
    echo ${resturl} | jq '.'
}
```

### 2. 版本

```json
curlPost '{
  "jsonrpc": "2.0",
  "method": "apiinfo.version",
  "id": 1,
  "auth": null,
  "params": {}
}'
```

### 3. 用户

#### 3.1 单用户认证

```json
curlPost '{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
        "user": "Admin",
        "password": "zabbix"
    },
    "id": 1,
    "auth": null
}'
```

### 4. 主机组

#### 4.1 获取

```bash
curlPost '{
    "jsonrpc": "2.0",
    "method": "hostgroup.get",
    "params": {
        "output": "extend",
        "filter": {
            "name": [
                "虚拟主机群组"
            ]
        }
    },
    "auth": "e022f9c13969f93a50b0909531765f9b",
    "id": 1
}'
```

### 5. 主机

#### 5.1 创建

```bash
curlPost '{
    "jsonrpc": "2.0",
    "method": "host.create",
    "params": {
        "host": "dhcp-server",
        "name": "virtual-ops-dhcp-server",
        "interfaces": [
            {
                "type": 1,
                "main": 1,
                "useip": 1,
                "ip": "10.173.28.229",
                "dns": "",
                "port": "10050"
            }
        ],
        "groups": [
            {
                "groupid": "29"
            }
        ]
    },
    "auth": "e022f9c13969f93a50b0909531765f9b",
    "id": 1
}'
```

#### 5.2 删除

```bash
curlPost '{
    "jsonrpc": "2.0",
    "method": "host.delete",
    "params": [
        "10684"
    ],
    "auth": "e022f9c13969f93a50b0909531765f9b",
    "id": 1
}'
```

