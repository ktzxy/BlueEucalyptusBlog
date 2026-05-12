+++
date = '2024-07-11T03:15:02+08:00'
draft = false
title = 'Oracle安装'
slug = "bk1y3q3yqo"
description = "Oracle安装"
categories = ["📒数据库"]
tags = ["Oracle"]
summary = "Oracle安装"

+++
# 部署脚本

```shell
#!/bin/sh
yum clean all
yum makecache
find . -type f -exec touch {} \; 
echo 删除现有的用户和组
userdel oracle
groupdel dba
groupdel oinstall
echo 创建orale用户
groupadd dba
groupadd oinstall
useradd -g oinstall -G dba oracle
echo "1234" | passwd --stdin "oracle"
echo orale用户创建完成

echo 创建oracle安装目录
mkdir -p /opt/oracle/product/11.2/db_1
mkdir -p /opt/oraInventory
mkdir -p /opt/oracle/oradata
mkdir -p /var/opt/oracle
echo oracle安装目录完成

echo 给目录赋予权限
chown -R oracle.oinstall /opt/oracle
chown -R oracle.oinstall /opt/oracle/oradata
chown -R oracle.oinstall /opt/oracle/product/11.2/db_1
chown -R oracle.dba /opt/oraInventory
chown oracle.dba /var/opt/oracle
chmod -R 775 /opt/oracle
chmod -R 755 /var/opt/oracle
echo 目录所有组赋予完成

echo 设置oracle环境变量
echo  'export ORACLE_BASE=/opt/oracle
   export ORACLE_HOME=$ORACLE_BASE/product/11.2/db_1
   export ORACLE_SID=orcl
   export ORACLE_OWNER=oracle
   export ORACLE_TERM=vt100
   export PATH=$PATH:$ORACLE_HOME/bin:$HOME/bin
   export PATH=$ORACLE_HOME/bin:$PATH
   LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:/usr/local/lib
   export LD_LIBRARY_PATH
   CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
   CLASSPATH=$CLASSPATH:$ORACLE_HOME/network/jlib
   export CLASSPATH
   NLS_LANG="AMERICAN_AMERICA.AL32UTF8"
   PATH=$PATH:/usr/sbin; export PATH
   PATH=$PATH:/usr/bin; export PATH
   export ORA_NLS33=$ORACLE_HOME/nls/admin/data' >>/home/oracle/.bash_profile
source  /home/oracle/.bash_profile
echo 设置oracle环境变量完成

echo 安装依赖包，可能提示没有成功安装，最后跳过即可
yum install -y libaio-*
yum install -y gcc-*
yum install -y glibc-*
yum install -y compat-libstdc*
yum install -y elfutils-libelf-devel*
yum install -y libstdc++*
yum install -y unixODBC-*
yum install -y unixODBC-devel-*
echo 依赖包安装完成

echo 开始安装jdk 7.0
rpm -e --nodeps jdk-1.7.0_80-fcs.x86_64
rpm -ivh jdk-7u80-linux-x64.rpm
echo jdk安装完成 路径为:/usr/java

echo 设置jdk环境变量

echo '
export JAVA_HOME=/usr/java/jdk1.7.0_80
export PATH=$PATH:$JAVA_HOME/bin
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib'>>/etc/profile
echo jdk环境变量设置完成


echo 设置软限制和硬限制
echo 'oracle soft nproc 2047
  oracle hard nproc 16384
  oracle soft nofile 1024
  oracle hard nofile 65536
  oracle hard stack 10240' >>/etc/security/limits.conf
echo 设置限制完成

echo 修改内核参数
echo '
 fs.aio-max-nr = 1048576
 fs.file-max = 6815744
 kernel.shmall = 2097152
 kernel.shmmax = 8405194752
 kernel.shmmni = 4096
 kernel.sem = 250 32000 100 128
 net.ipv4.ip_local_port_range = 9000 65500
 net.core.rmem_default = 262144
 net.core.rmem_max = 4194304
 net.core.wmem_default = 262144
 net.core.wmem_max = 1048586' >>/etc/sysctl.conf
cd /etc
sysctl -p
echo 修改内核完成

echo -先设置开启启动装完自己修改即可
echo "su - oracle -lc 'dbstart \$ORACLE_HOME'" >>/etc/rc.local
chmod -R 777 /home/database
echo -oracle自启完成
echo -重启电脑
reboot
```

# Oracle 19c Server 部署文档

## 1. 部署 Oracle 19C

### 1.1  前置准备

#### 1.1.1 设置系统资源限制

```bash
cat > /etc/security/limits.d/30-oracle.conf << 'END'
oracle   soft   nofile    1024
oracle   hard   nofile    65536
oracle   soft   nproc     16384
oracle   hard   nproc     16384
oracle   soft   stack     10240
oracle   hard   stack     32768
oracle   hard   memlock   134217728
oracle   soft   memlock   134217728
END
```

#### 1.1.2 配置内核参数

```bash
cat > /etc/sysctl.d/98-oracle.conf << 'END'
fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 1073741824
kernel.shmmax = 4398046511104
kernel.panic_on_oops = 1
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
END
```

#### 1.1.3 安装依赖包

```bash
yum install xauth xorg-x11-utils binutils.x86_64 gcc.x86_64 gcc-c++.x86_64 glibc.x86_64 glibc-devel.x86_64 ksh libaio.x86_64 libaio-devel.x86_64 libgcc.x86_64 libnsl zip wget -y
```

#### 1.1.4 设置主机名

```bash
hostnamectl set-hostname db-oracle
```

#### 1.1.5 创建用户

```bash
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
groupadd -g 54324 backupdba
groupadd -g 54325 dgdba
groupadd -g 54326 kmdba
groupadd -g 54330 racdba

useradd -u 54321 -g oinstall -G dba,oper,backupdba,dgdba,kmdba,racdba oracle
echo "oracle" | passwd --stdin oracle
```

#### 1.1.6 创建目录和 oraInst.loc

```bash
mkdir /u01
mkdir -p /u01/app/oracle/product/19.3.0/dbhome_1
chown oracle:oinstall -R /u01/

cat > /etc/oraInst.loc << 'END'
inventory_loc=/u01/app/oraInventory
inst_group=oinstall
END
chown oracle:oinstall /etc/oraInst.loc
```

### 1.2 开始安装

#### 1.2.1 使用 Oracle 登录

```bash
ssh oracle@localhost
```

#### 1.2.2 设置环境变量

```bash
cat >> ~/.bashrc << 'END'
export ORACLE_HOSTNAME=db-oracle
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.3.0/dbhome_1
export ORA_INVENTORY=/u01/app/oraInventory
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_SID=test
export PDB_NAME=pdb1
export DATA_DIR=/u01/oradata
END

sed -i "s/db-oracle/$(hostname -f)/" ~/.bashrc
. ~/.bashrc
```

#### 1.2.3 下载并解压安装包

[Oracle Database 19c Download for Linux x86-64](https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html)

```bash
LINUX.X64_193000_db_home.zip

unzip -q LINUX.X64_193000_db_home.zip  -d /u01/app/oracle/product/19.3.0/dbhome_1/
cd /u01/app/oracle/product/19.3.0/dbhome_1/
```

#### 1.2.4 安装 Oracle (静默)

```bash
export CV_ASSUME_DISTID=OEL7.6
export LC_ALL="en_US.UTF-8"
export LANG="en_US.UTF-8"

./runInstaller -ignorePrereq -waitforcompletion -silent \
  oracle.install.option=INSTALL_DB_SWONLY \
  ORACLE_HOSTNAME=${ORACLE_HOSTNAME} \
  UNIX_GROUP_NAME=oinstall \
  INVENTORY_LOCATION=${ORA_INVENTORY} \
  ORACLE_HOME=${ORACLE_HOME} \
  ORACLE_BASE=${ORACLE_BASE} \
  oracle.install.db.InstallEdition=EE \
  oracle.install.db.OSDBA_GROUP=dba \
  oracle.install.db.OSBACKUPDBA_GROUP=backupdba \
  oracle.install.db.OSDGDBA_GROUP=dgdba \
  oracle.install.db.OSKMDBA_GROUP=kmdba \
  oracle.install.db.OSRACDBA_GROUP=racdba \
  SECURITY_UPDATES_VIA_MYORACLESUPPORT=false \
  DECLINE_SECURITY_UPDATES=true
```

#### 1.2.5 以 root 用户实行脚本

```bash
su - root -c /u01/app/oracle/product/19.3.0/dbhome_1/root.sh

ssh root@localhost /u01/app/oracle/product/19.3.0/dbhome_1/root.sh
```

#### 1.2.6 初始化数据库

```bash
dbca \
-silent -createDatabase \
-templateName General_Purpose.dbc \
-gdbname ${ORACLE_SID} \
-sid  ${ORACLE_SID}   \
-responseFile NO_VALUE   \
-characterSet AL32UTF8   \
-sysPassword Fhl3y_Jsdwj   \
-systemPassword Fhl3y_Jsdwj   \
-createAsContainerDatabase true   \
-numberOfPDBs 1   -pdbName ${PDB_NAME}   \
-pdbAdminPassword Fhl3y_Jsdwj   \
-databaseType MULTIPURPOSE   \
-automaticMemoryManagement false   \
-totalMemory 800   \
-storageType FS   \
-datafileDestination "${DATA_DIR}"   \
-redoLogFileSize 50   \
-emConfiguration NONE   \
-ignorePreReqs
```

### 1.3  配置系统服务

#### 1.3.1 使用 root 登录

```bash
ssh root@localhost
```

#### 1.3.2 创建 Oracle 启动脚本

```bash
mkdir /opt/script
cat > /opt/script/start_oracle.sh << 'END'
#!/bin/bash
export ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1
export ORACLE_SID=TEST
$ORACLE_HOME/bin/dbstart $ORACLE_HOME
END

chown oracle:oinstall /opt/script/start_oracle.sh
chmod +x /opt/script/start_oracle.sh
```

#### 1.3.3 创建 systemd 服务

```bash
cat > /etc/systemd/system/oracle.service << END
[Unit]
Description=Oracle Database Service
After=network.target

[Service]
Type=forking
User=oracle
Group=oinstall
ExecStart=/opt/script/start_oracle.sh
Restart=no

[Install]
WantedBy=multi-user.target
END

systemctl enable oracle.service --now
```

#### 1.3.4 修改 oratab 以启用自启动

```sql
sed -i 's|^test:/u01/app/oracle/product/19.3.0/dbhome_1:N|test:/u01/app/oracle/product/19.3.0/dbhome_1:Y|' /etc/oratab
```

### 1.4 图形化安装

```dbca```

![image-20260512112501922](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112501922.png)

![image-20260512112549667](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112549667.png)

![image-20260512112610517](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112610517.png)

![image-20260512112631865](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112631865.png)

![image-20260512112645899](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112645899.png)

![image-20260512112700215](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112700215.png)

![image-20260512112713224](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112713224.png)

![image-20260512112733347](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112733347.png)

![image-20260512112747399](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112747399.png)

![image-20260512112810941](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112810941.png)

![image-20260512112827160](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112827160.png)

![image-20260512112843650](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112843650.png)

![image-20260512112857109](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260512112857109.png)

>   执行完成之后，继续 1.3.4 操作

## 2. Oracle 常用命令

### 1. 登陆数据库

```bash
su - oracle -c "sqlplus / as sysdba"
startup
SELECT instance_name, status FROM v$instance;
```

### 2. 创建用户及连接 PDB

```sql
# 1. 确认当前连接到 PDB
SELECT name, open_mode FROM v$pdbs;

# 2. 打开 PDB1
# 将 PDB1 状态从 MOUNTED 改为 READ WRITE 模式：
ALTER PLUGGABLE DATABASE PDB1 OPEN;
# 或者，如果希望直接打开 PDB 并设置为可读写模式：
ALTER PLUGGABLE DATABASE PDB1 OPEN READ WRITE;

# 3. 确认 PDB1 状态
SELECT name, open_mode FROM v$pdbs;

# 4. 切换到 PDB1 并创建用户
# -- 切换到 PDB1
ALTER SESSION SET CONTAINER = PDB1;

# -- 创建用户
CREATE USER my_user IDENTIFIED BY my_password;

# -- 授予权限
GRANT CREATE SESSION TO my_user;
GRANT CONNECT, RESOURCE TO my_user;

# -- 设置表空间配额
ALTER USER my_user QUOTA UNLIMITED ON USERS;
```