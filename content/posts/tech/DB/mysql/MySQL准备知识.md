+++
date = '2025-07-03T01:06:47+08:00'
draft = false
title = 'MySQL准备知识'
slug = "8q8nmfnyld"
description = "MySQL准备知识"
categories = ["📒数据库"]
tags = ["mysql"]
summary = "MySQL准备知识"
+++
# 番外篇

# 1. MySQL目录结构

### windows版本

- bin：存放可执行文件，比如MySQL.exe
- data：存储的是MySQL默认的数据库
- include：存放的C语言的头文件
- lib：存放的C++的动态链接库
- my.ini：数据库的配置文件

# 2. 启动 MySQL 服务器程序的相关可执行文件

启动 MySOL 服务器程序的可执行文件有很多，大多在 MySQL 安装目录的 bin 目录下。

## mysqld（不常用）

- `mysqld` 这个可执行文件就代表着 MySOL 服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。

## mysqld_safe

- `mysqld_safe`是一个启动脚本，它会间接的调用`mysqld`，而且还顺便启动了另外一个监控进程，这个监控进程在服务器进程挂了的时候，可以帮助重启它。
- 除外，使用`mysqld_safe`启动服务器程序时，它会将服务器程序的出错信息和其他诊断信息重定向到某个文件中，产生出错日志，方便找出发生错误的原因。

## mysql.server

`mysql.server`也是一个启动脚本，它会间接的调用`mysqld_safe`。需要注意的是，这个`mysql.server`文件其实是一个链接文件，它的实际文件位置是`support-files/mysql.server`，所以如果在 bin 目录找不到，则到`support-files`去找，也可以自行用 `ln` 命令在 bin 创建一个链接。

在调用`mysql.server`时在后边指定`start`参数就可以启动服务器程序；如指定`stop`参数则关闭正在运行的服务器程序。

```bash
# 开启MySQL服务
mysql.server start
# 关闭MySQL服务
mysql.server stop
```

## mysqld_multi

`mysqld_multi`可以一台计算机上也可以运行多个服务器实例（*即运行了多个MySQL服务器进程*）。此可执行文件可以对每一个服务器进程的启动或停止进行监控。

# 3. Linux 系统安装MySQL

### 3.1. 下载 Linux 安装包

下载地址：https://dev.mysql.com/downloads/mysql/

### 3.2. 安装 MySQL

1. 卸载 centos 中预安装的 mysql

```bash
rpm -qa | grep -i mysql
rpm -e mysql-libs-5.1.71-1.el6.x86_64 --nodeps
```

2. 上传 mysql 的安装包

```
alt + p --> put  E:/test/MySQL-5.6.22-1.el6.i686.rpm-bundle.tar
```

3. 解压 mysql 的安装包

```bash
mkdir mysql
tar -xvf MySQL-5.6.22-1.el6.i686.rpm-bundle.tar -C /root/mysql
```

4. 安装依赖包

```bash
yum -y install libaio.so.1 libgcc_s.so.1 libstdc++.so.6 libncurses.so.5 --setopt=protected_multilib=false
yum  update libstdc++-4.4.7-4.el6.x86_64
```

5. 安装 mysql-client

```bash
rpm -ivh MySQL-client-5.6.22-1.el6.i686.rpm
```

6. 安装 mysql-server

```bash
rpm -ivh MySQL-server-5.6.22-1.el6.i686.rpm
```

### 3.3. 启动/停止 MySQL 服务

```bash
service mysql start

service mysql stop

service mysql status

service mysql restart
```

### 3.4. 登录 MySQL

mysql 安装完成之后, 会自动生成一个随机的密码, 并且保存在一个密码文件中：`/root/.mysql_secret`

```bash
mysql -u root -p
```

登录之后，修改密码：

```bash
set password = password('123456');
```

进入mysql，授权远程访问：

```bash
grant all privileges on *.* to 'root' @'%' identified by '123456';
flush privileges;
```

如果此时使用SSH软件还是无法远程连接mysql数据库，可能就是linux系统的防火墙导致的，所以需要设置关闭防火墙

```bash
# 查询看防火墙状态
service iptables status

# 关闭防火墙（全部、暴力）
service iptables stop
```

## 4. CentOS 7.4 安装与配置MySql 5.7.21（项目B安装）

### 4.1. 环境

- **系统环境**：centos-7.4 64位
- **安装方式**：rpm安装
- **软件**：mysql-5.7.21-1.el7.x86_64.rpm-bundle.tar
- **描述**：上述的tar包中已经包含需要安装的rpm，所以只需要将其放置到系统中使用tar命令解包即可。
- **Mysql的下载地址**：http://dev.mysql.com/downloads/mysql/

### 4.2. 系统原mariadb版本

```bash
# 查看MySql与mariadb安装情况
# grep -i是不分大小写字符查询，只要含有mysql就显示
rpm -qa | grep -i mysql
rpm -qa | grep mariadb

# 卸载mariadb(会与mysql冲突)
rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64
```

### 4.3. 安装新MySQL

使用winSCP将下载的“`mysql-5.7.21-1.el7.x86_64.rpm-bundle.tar`”传到虚拟机系统的/root目录下：

在终端上进入`/root`目录；解包`.tar`包

```bash
# 对” mysql-5.7.21-1.el7.x86_64.rpm-bundle.tar”解包，不是压缩文件不需要解压缩
tar -xvf mysql-5.7.21-1.el7.x86_64.rpm-bundle.tar
```

执行如下安装命令：

```bash
# 1、安装 mysql-community-common
rpm -ivh mysql-community-common-5.7.21-1.el7.x86_64.rpm

# 2、安装 mysql-community-libs
rpm -ivh mysql-community-libs-5.7.21-1.el7.x86_64.rpm

# 3、安装 mysql-community-client
rpm -ivh mysql-community-client-5.7.21-1.el7.x86_64.rpm

# 4、安装 mysql-community-server
yum -y install perl
rpm -ivh mysql-community-server-5.7.21-1.el7.x86_64.rpm

# 5、安装 mysql-community-devel
rpm -ivh mysql-community-devel-5.7.21-1.el7.x86_64.rpm
```

安装完成。MySql默认安装文件位置：

```
/var/lib/mysql/    # 数据库目录
/usr/share/mysql   # 配置文件目录
/usr/bin           # 相关命令目录
/etc/my.cnf        # 核心配置文件
```

### 4.4. 配置MySQL

#### 4.4.1. 启动mysql

```bash
#启动mysql
service mysqld start
#重启mysql
service mysqld restart
#停止mysql
service mysqld stop
#查看mysql状态
service mysqld status

# 设置开机启动Mysql
systemctl enable mysqld

# 设置开机不启动Mysql
systemctl disable mysqld
```

#### 4.4.2. 修改root密码

MySQL安装成功后，会生成一个临时密码，第一次登录需要输入这个密码，所以查看该临时密码，然后修改密码。

```bash
# 查看临时密码(/var/log/mysqld.log)
grep password /var/log/mysqld.log

# 使用root登录
mysql –uroot –p

# 然后输入/var/log/mysqld.log文件中的临时密码。登录后；修改密码为Root_123
set password = password('Root_123');
```

<font color=red>**注意：密码必须包含大小写字母、数字、特殊符号**</font>

在 mysql 内置密码校验器，用于校验用户设置的密码复杂度。使用以下命令可以查看密码校验的规则：

```sql
mysql> SHOW VARIABLES LIKE 'validate_password.%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
```

通过以下命令，降低密码的校验规则

```bash
set global validate_password.policy = 0;
set global validate_password.length = 4;
```

设置简单的密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

> Notes: 官方文档地址：https://dev.mysql.com/doc/refman/8.0/en/validate-password-options-variables.html

#### 4.4.3. 设置允许远程访问

因为 root 用户默认的主机是 localhost，为了可以远程访问 mysql 服务，需要将 root 用户的主机地址修改为 `*`。（*或者创建主机地址为 `*` 的其他用户*）

```bash
#登录，密码为新修改的密码Root_123
mysql -uroot –p

#设置远程访问（使用root密码）：
mysql> grant all privileges on  *.*  to  'root' @'%'  identified by 'Root_123'; 
mysql> flush privileges;
```

#### 4.4.4. 设置3306端口可以被访问

```bash
# 退出mysql，防火墙中打开3306端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

参数说明：

- `–zone`：作用域
- `-–add-port=3306/tcp`：添加端口，格式为：端口/通讯协议
- `-–permanent`：永久生效，没有此参数重启后失效

```bash
# 重启防火墙
firewall-cmd --reload
# 查看已经开放的端口
firewall-cmd --list-ports

# 停止防火墙
systemctl stop firewalld.service
# 启动防火墙
systemctl start firewalld.service
# 禁止防火墙开机启动
systemctl disable firewalld.service
```

## 5. MySQl 安装（Windows 免安装版）

### 5.1. 解压与创建配置文件

- 将MySQL软件包解压在没有中文和空格的目录下
- 在解压目录创建my.ini文件并添加内容如下：

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152622334.webp)

### 5.2. 配置环境变量

- 在【我的电脑】右键 -> 选择【高级系统设置】 -> 选择【高级】 -> 【环境变量】。设置环境变量将【`MYSQL_HOME`】添加到`PATH`环境变量

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152628201.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152635264.webp)

### 5.3. 配置系统开启服务

使用管理员权限进入DOS，在cmd中，进入解压目录下的bin目录依次执行以下命令：

1. 对mysql进行初始化。*请注意，这里会生成一个临时密码，后边要使用这个临时密码*

```bash
mysqld --initialize --user=mysql --console
```

2. 安装mysql服务

```bash
mysqld --install
```

3. 启动mysql服务

```bash
net start mysql
```

4. 登录mysql，*这里需要使用之前生成的临时密码。如果窗口关了没有记录临时密码，可以将mysql目录下的data目录删除，然后再进行初始化*

```bash
mysql -uroot –p
```

5. 修改root用户密码

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```

6. 修改root用户权限

```bash
create user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
```

### 5.4. mysql 服务启动时报“某些服务在未由其他服务或程序使用时将自动停止”

在配置文件中有 `secure-file-priv` 的配置，如果设置了此目录，本来解压后是没有此目录的，如果不手动创建，启动时会报错。

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152644198.webp)

当时排查很久都不知道是什么原因，后来查询资料，发现 `mysqld --console` 命令可以将错误信息输出到控制台上，然后就看到具体的无法启动的报错日志

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152649183.webp)

然后在配置的位置手动创建相应的目录，问题就解决了。

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152657229.webp)

## 6. 安装 MySQL 8.0（Windows版本）

### 6.1. 安装包下载

下载地址：https://downloads.mysql.com/archives/installer/

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152701320.webp)

### 6.2. 安装步骤（截图，不完整，日后优化）

> 此安装示例使用 mysql-installer-community-8.0.27.1.msi

#### 6.2.1. 自定义安装

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152711818.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152717746.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152724660.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152739498.webp)

> 注：以上可以不选择相关文档与示例的组件

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152804929.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152807624.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152817263.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152811027.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152833175.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152838631.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152843831.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152847444.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152852623.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152857498.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152905640.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152913808.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306154425400.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152918846.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152923854.webp)

#### 6.2.2. 默认选择安装

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152931771.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152939773.webp)

当所有的状态都变成Complete之后，点击 Next

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152936043.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152950497.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152958162.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306152955362.webp)

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153005740.webp)

> 设置密码，用于登陆数据，建议使用学习的设置简单的密码

### 6.3. 配置环境变量

配置环境变量与windows免安装版一样

## 7. 开启/停止 MySQL 服务（Windows版本）

### 7.1. 启动MySQL服务方式1

MySQL会以windows服务的方式为我们提供数据存储功能。开启和关闭服务的操作：

1. 右键点击我的电脑 --> 管理 --> 服务与应用程序 --> 服务 --> 找到 MySQL 服务开启或停止。
2. 或者：开始 --> 搜索 --> services.msc --> 服务 --> 可以找到 MySQL 服务开启或停止。

<font color="purple">（如果不需要开机时就启动MySQL，右键MySQL --> 属性 --> 启动类型选“手动”）</font>

![mysql服务](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153011724.webp)

### 7.2. 启动MySQL服务方式2

在DOS窗口，通过命令完成MySQL服务的启动和停止（必须以管理员身份运行cmd命令窗口）

- MySQL 启动: `net start mysql`
- MySQL 关闭: `net stop mysql`

![mysql服务](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153020469.webp)

> Tips: 上述命令的 `mysql` 是在安装 MySQL 时，默认指定的 mysql 的系统服务名，不是固定的，如果未改动，5.7版本默认是`mysql`，8.0版本默认是`mysql80`。

## 8. 客户端连接 MySQL

MySQL 是一个需要账户名密码登录的数据库，登陆后使用，它提供了一个默认的root账号，使用安装时设置的密码即可登录。

### 8.1. 方式一：使用 MySQL 提供的客户端命令行工具

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153017059.webp)

### 8.2. 方式二：使用系统自带的命令行工具执行指令

使用 CMD 命令行窗口，可以操作登陆与退出 MySQL，命令语法如下：

```bash
mysql [-h 127.0.0.1] [-P 3306] -u root -p
```

参数说明：

- `-h`：MySQL 服务所在的主机 IP
- `-P`：MySQL 服务端口号，默认3306。<font color=red>**注意：参数是大写**</font>
- `-u`：MySQL 数据库用户名
- `-p`：MySQL 数据库用户名对应的密码。<font color=red>**注意：参数是小写**</font>

`[]` 内为可选参数，如果需要连接远程的 MySQL，需要加上这两个参数来指定远程主机 IP、端口，如果连接本地的 MySQL，则无需指定这两个参数。

> **Tips: 使用这种方式进行连接时，需要安装完毕后配置PATH环境变量。**

示例1：使用 cmd 命令行输入 `mysql –u用户名 –p密码`

![命令行操作登陆库1](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153027259.webp)

示例2：使用 cmd 命令行输入 `mysql --user=用户名 --password=密码 --host=ip地址 --port=端口号`（这种方式一般用来登陆别人的数据库）

![命令行操作登陆库2](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153030599.webp)

### 8.3. 退出连接 MySQL

直接使用 `exit` 命令即可退出 MySQL 连接

### 8.4. 其他操作数据库方式

- 通过第三方图形化界面操作
- 通过Java代码操作

## 9. 卸载MySQL（Windows版本）

1. 先停止MySQL服务
   - 方式1：输入命令行`net stop mysql`。*注：命令中的`mysql`是服务名称，需要输入根据实际的名称*
   - 方式2：【win+r】打开“运行”面板，输入`services.msc`，进行服务窗口中关闭mysql服务
2. 卸载程序。到【控制面板】中的【程序和功能】中卸载，或者使用第三方的卸载工具，如：Geek Uninstaller
3. 删除根文件夹。进入sql安装位置，手动删除mysql的解压（安装）的文件夹
4. 删除C盘隐藏的数据文件夹（可选）：”C:\ProgramData\MySQL“。*注：如果安装或者改了配置文件，此数据文件夹目录可能不一样*
5. 删除注册表信息。【win+r】打开“运行”面板，输入`regedit`命令打开注册表窗口，删除以下文件：

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153325805.webp)

6. 删除环境变量的配置。进行【高级系统设置】中的【环境变量】，删除安装后配置`MYSQL_HOME`变量和删除path变量中的mysql路径
7. 删除MySQL服务。使用管理员方式打开cmd命令行窗口，输入命令`sc delete Mysql服务名称`

## 10. MySQL 常用图形管理工具

### 10.1. Navicat

Navicat是一套快速、可靠的数据库管理工具，Navicat 是以直觉化的图形用户界面而建的，可以兼容多种数据库，支持多种操作系统。

官网：http://www.navicat.com.cn/download/navicat-premium

### 10.2. SQLyog

SQLyog 是一个快速而简洁的图形化管理MySQL数据库的工具，它能够在任何地点有效地管理你的数据库，由业界著名的Webyog公司出品。

官网：https://sqlyog.en.softonic.com/

安装：提供的SQLyog软件为免安装版，可直接使用

使用：输入用户名、密码，点击连接按钮，进行访问MySQL数据库进行操作

![SQLyog工具1](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153052258.webp)

在Query窗口中，输入SQL代码，选中要执行的SQL代码，按F8键运行，或按执行按钮运行。

![SQLyog工具2](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153055382.webp)

### 10.3. MySQL Workbench

MySQL Workbench MySQL 是官方提供的图形化管理工具，分为社区版和商业版，社区版完全免费，而商业版则是按年收费。支持数据库的创建、设计、迁移、备份、导出和导入等功能，并且支持 Windows、Linux 和 mac 等主流操作系统。

### 10.4. DataGrip

DataGrip 是一款由JetBrains公司出品的数据库管理客户端工具，方便连接到数据库服务器，执行sql、创建表、创建索引以及导出数据等

#### 使用说明

1. 添加数据源

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153101414.webp)

配置以及驱动 jar 包下载完毕之后，就可以点击 "Test Connection" 就可以测试，是否可以连接 MySQL，如果出现 "Successed"，就表示连接成功了。

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153104429.webp)

2. 展示所有数据库。连接上了 MySQL 服务之后，并未展示出所有的数据库，此时，需要设置展示所有的数据库。

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153113195.webp)

3. 创建数据库

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153115979.webp)

> 以下两种方式都可以创建数据库：
>
> - `create database db01;`
> - `create schema db01;`

4. 创建表。在指定的数据库上面右键，选择 new -> Table

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153122283.webp) ![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153129074.webp)

5. 修改表结构。在需要修改的表上，右键选择 "Modify Table..."

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153141325.webp) ![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153208369.webp)

- 如果想增加字段，直接点击+号，录入字段信息，然后点击Execute即可。
- 如果想删除字段，直接点击-号，就可以删除字段，然后点击Execute即可。
- 如果想修改字段，双击对应的字段，修改字段信息，然后点击Execute即可。
- 如果要修改表名，或表的注释，直接在输入框修改，然后点击Execute即可。

6. 在 DataGrip 中执行 SQL 语句。在指定的数据库上，右键，选择 New -> Query Console

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153146564.webp)

然后就可以在打开的 Query Console 控制台，并在控制台中编写 SQL，执行 SQL。

![](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306153215855.webp)

### 10.5. HeidiSQL

HeidiSQL 是免费软件，其目标是易于学习。“Heidi”让您可以从运行 MariaDB、MySQL、Microsoft SQL、PostgreSQL 和 SQLite 数据库系统之一的计算机上查看和编辑数据和结构。HeidiSQL 由 Ansgar 于 2002 年发明，属于全球最流行的 MariaDB 和 MySQL 工具。

官网：https://www.heidisql.com/

### 10.6. DBeaver

DBeaver是一款世界有名的数据库管理软件。DBeaver64位(数据库管理软件)给大家提供的是dbeaver中文版64位版本，支持MySQL、PostgreSQL和任何数据库，其中有一个JDBC驱动程序，可以查看数据库的结构、导出数据库或者执行相应的脚本等操作。

下载地址：[DBeaver Community | Free Universal Database Tool](https://dbeaver.io/)

### 10.7. 其他工具

- phpMyAdmin
- MySQLDumper
- MySQL GUI Tools
- MySQL ODBC Connector



