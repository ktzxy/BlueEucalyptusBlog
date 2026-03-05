+++
date = '2025-10-02T19:40:35+08:00'
draft = false
title = 'mysql完整备份和恢复'
description = "mysql完整备份和恢复"
categories = ["📒数据库"]
tags = ["mysql","备份"]
summary = "mysql完整备份和恢复"
+++

mysql 完整备份和恢复
=============

 

**一、MySQL 完整备份操作**

**1、直接打包数据库文件夹**

**创建数据库 auth：**

```	shell
MariaDB [(none)]> create database auth;  
Query OK, 1 row affected (0.00 sec)
```

**进入数据库：**

```shell
MariaDB [(none)]> use auth  
Database changed  
```

**创建数据表：**

```shell
MariaDB [auth]> create table user(name char(10)not null,ID int(48));  
Query OK, 0 rows affected (0.01 sec)
```



**插入数据信息：**

```shell
MariaDB [auth]> insert into user values('crushlinux','123');  
Query OK, 1 row affected (0.01 sec)
```



**查看数据信息：**

```shell
MariaDB [auth]> select * from user;  
+------------+------+  
| name | ID |  
+------------+------+  
| crushlinux | 123 |  
+------------+------+  
1 row in set (0.00 sec)
```



**对它进行备份**

**先退出 MySQL 停库**

```shell
[root@localhost ~]# systemctl stop mariadb
```



**直接对它进行打包压缩（新引入一个小命令）**

```shell
[root@localhost ~]# rpm -q xz  
xz-5.1.2-9alpha.el7.x86_64

[root@localhost ~]# mkdir backup                // 创建一个文件，把压缩包放进去  
[root@localhost ~]# tar Jcf backup/mysql_all-$(date +%F).tar.xz /var/lib/mysql/  
tar: 从成员名中删除开头的 “/”
```



**模拟数据丢失：**

```shell
[root@localhost ~]# rm -rf /var/lib/mysql/auth/
```



**起服务：**

```shell
[root@localhost ~]# systemctl start mariadb
```



**恢复数据：**

```shell
[root@localhost ~]# mkdir restore              // 虽已创建一个文件  
[root@localhost ~]# tar xf backup/mysql_all-2019-10-13.tar.xz -C restore/    将那个压缩包解压到这个文件里  
[root@localhost ~]# cd restore/                    // 切换到这个文件里查看  
[root@localhost restore]# ls  
var  
[root@localhost restore]# cd var/lib/mysql/       // 继续查看  
[root@localhost mysql]# ls  
aria_log.00000001 auth ibdata1 ib_logfile1 performance_schema  
aria_log_control crushlinux ib_logfile0 mysql test  
[root@localhost mysql]# mv auth/ /var/lib/mysql/          // 发现有 auth，将它移动到 / var/lib/mysql / 下
```



**登录 MySQL 查看：**

```shell
MariaDB [(none)]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| auth |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
6 rows in set (0.01 sec)

MariaDB [auth]> select * from user;  
+------------+------+  
| name | ID |  
+------------+------+  
| crushlinux | 123 |  
+------------+------+  
1 row in set (0.00 sec)
```



**2、使用专用备份工具 mysqldump**

MySQL 自带的备份工具 mysqldump，可以很方便的对 MySQL 进行备份。通过该命令工具可以将数据库、数据表或全部的库导出为 SQL 脚本，便于该命令在不同版本的 MySQL 务器上使用。例如， 当需要升级 MySQL 服务器时，可以先使用 mysqldump 命令将原有库信息到导出，然后直接在升级后的 MySQL 服务器中导入即可。

**（1）对单个库进行完全备份**

```shell
[root@localhost ~]# mysqldump -uroot -p123 --databases auth > backup/auth-$(date +%Y%m%d).sql

[root@localhost ~]# ls backup/  
auth-20191013.sql mysql_all-2019-10-13.tar.xz  
[root@localhost ~]# grep -Ev "^/|^$|^-" backup/auth-20191013.sql   
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `auth` /*!40100 DEFAULT CHARACTER SET latin1 */;  
USE `auth`;  
DROP TABLE IF EXISTS `user`;  
CREATE TABLE `user` (  
`name` char(10) NOT NULL,  
`ID` int(48) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=latin1;  
LOCK TABLES `user` WRITE;  
INSERT INTO `user` VALUES ('crushlinux',123);  
UNLOCK TABLES;
```



**（2）对多个库进行完全备份：**

```shell
[root@localhost ~]# mysqldump -uroot -p123 --databases auth mysql> backup/auth+mysql-$(date +%Y%m%d).sql

[root@localhost ~]# ls backup/  
auth-20191013.sql auth+mysql-20191013.sql mysql_all-2019-10-13.tar.xz
```



（3）对所有库进行完全备份：

```shell
[root@localhost ~]# mysqldump -uroot -p123 --events --opt --all-databases > backup/mysql_all.$(date +%Y%m%d).sql

[root@localhost ~]# ls backup/  
auth-20191013.sql mysql_all.20191013.sql  
auth+mysql-20191013.sql mysql_all-2019-10-13.tar.xz
```



**（4）对表进行完全备份：**

```shell
[root@localhost ~]# mysqldump -uroot -p123 auth user > backup/auth_user-$(date +%Y%m%d).sql  
[root@localhost ~]# ls backup/  
auth-20191013.sql auth_user-20191013.sql mysql_all-2019-10-13.tar.xz  
auth+mysql-20191013.sql mysql_all.20191013.sql

[root@localhost ~]# grep -Ev "^/|^$|^-" backup/auth_user-20191013.sql   
DROP TABLE IF EXISTS `user`;  
CREATE TABLE `user` (  
`name` char(10) NOT NULL,  
`ID` int(48) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=latin1;  
LOCK TABLES `user` WRITE;  
INSERT INTO `user` VALUES ('crushlinux',123);  
UNLOCK TABLES;
```



**（5）对表结构的备份**

```shell
[root@localhost ~]# **mysqldump -uroot -p123 -d auth user > backup/desc_auth_user-$(date +%Y%m%d).sql**

[root@localhost ~]# grep -Ev "^/|^$|^-" backup/desc_auth_user-20191013.sql   
DROP TABLE IF EXISTS `user`;  
CREATE TABLE `user` (  
`name` char(10) NOT NULL,  
`ID` int(48) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```



**二、恢复数据**

**1、使用 source 命令**

```shell
[root@localhost ~]# mysql -uroot -p123  
Welcome to the MariaDB monitor. Commands end with ; or \g.  
Your MariaDB connection id is 8  
Server version: 5.5.41-MariaDB MariaDB Server

Copyright (c) 2000, 2014, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> drop database auth;  
Query OK, 1 row affected (0.02 sec)

MariaDB [(none)]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
5 rows in set (0.00 sec)
```



**先确定 auth 的备份的路径：**backup/auth-20191013.sql

```shell
MariaDB [(none)]> source backup/auth-20191013.sql

MariaDB [auth]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| auth |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
6 rows in set (0.00 sec)
```



**2、用 mysql 命令**

```shell
MariaDB [auth]> drop database auth;  
Query OK, 1 row affected (0.01 sec)

[root@localhost ~]# mysql -uroot -p123 < backup/auth-20191013.sql 

MariaDB [(none)]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| auth |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
6 rows in set (0.00 sec)
```



**-e 后面可以加执行的语句：**

```shell
[root@localhost ~]# mysql -uroot -p123 -e 'select * from auth.user;'  
+------------+------+  
| name | ID |  
+------------+------+  
| crushlinux | 123 |  
+------------+------+
```



**三、MySQL 备份思路**  
1、定期实施备份，指定备份计划或策略，并严格遵守.  
2、除了进行完全备份，开启 MySQL 服务器的 binlog_日志功能是很重要的（完全备份加上日志，可以对 MySQL 进行最大化还原）。

3、使用统一和易理解的备份名称，推荐使用库名或者表名加上时间的命名规则，如 mysql_user-20181214.sql，不要使用 backup1 或者 abc 之类没有意义的名字。

4、定期抽查备份的可靠性，做还原测试或者检查文件大小等方式。

5、通过异地或者跨机房等方式来存放备份数据，防止源数据和备份文件一起损坏。

**一、MySQL 完整备份操作**

**1、直接打包数据库文件夹**

**创建数据库 auth：**

```shell
MariaDB [(none)]> create database auth;  
Query OK, 1 row affected (0.00 sec)
```



**进入数据库：**

```shell
MariaDB [(none)]> use auth  
Database changed  
```



**创建数据表：**

```shell
MariaDB [auth]> create table user(name char(10)not null,ID int(48));  
Query OK, 0 rows affected (0.01 sec)
```



**插入数据信息：**

```shell
MariaDB [auth]> insert into user values('crushlinux','123');  
Query OK, 1 row affected (0.01 sec)
```



**查看数据信息：**

```shell
MariaDB [auth]> select * from user;  
+------------+------+  
| name | ID |  
+------------+------+  
| crushlinux | 123 |  
+------------+------+  
1 row in set (0.00 sec)
```



**对它进行备份**

**先退出 MySQL 停库**

```shell
[root@localhost ~]# systemctl stop mariadb
```



**直接对它进行打包压缩（新引入一个小命令）**

```shell
[root@localhost ~]# rpm -q xz  
xz-5.1.2-9alpha.el7.x86_64

[root@localhost ~]# mkdir backup                // 创建一个文件，把压缩包放进去  
[root@localhost ~]# tar Jcf backup/mysql_all-$(date +%F).tar.xz /var/lib/mysql/  
tar: 从成员名中删除开头的 “/”
```

**模拟数据丢失：**

```shell
[root@localhost ~]# rm -rf /var/lib/mysql/auth/
```

**起服务：**

```shell
[root@localhost ~]# systemctl start mariadb
```



**恢复数据：**

```shell
[root@localhost ~]# mkdir restore              // 虽已创建一个文件  
[root@localhost ~]# tar xf backup/mysql_all-2019-10-13.tar.xz -C restore/    将那个压缩包解压到这个文件里  
[root@localhost ~]# cd restore/                    // 切换到这个文件里查看  
[root@localhost restore]# ls  
var  
[root@localhost restore]# cd var/lib/mysql/       // 继续查看  
[root@localhost mysql]# ls  
aria_log.00000001 auth ibdata1 ib_logfile1 performance_schema  
aria_log_control crushlinux ib_logfile0 mysql test  
[root@localhost mysql]# mv auth/ /var/lib/mysql/          // 发现有 auth，将它移动到 / var/lib/mysql / 下
```



**登录 MySQL 查看：**

```shell
MariaDB [(none)]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| auth |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
6 rows in set (0.01 sec)

MariaDB [auth]> select * from user;  
+------------+------+  
| name | ID |  
+------------+------+  
| crushlinux | 123 |  
+------------+------+  
1 row in set (0.00 sec)
```



**2、使用专用备份工具 mysqldump**

MySQL 自带的备份工具 mysqldump，可以很方便的对 MySQL 进行备份。通过该命令工具可以将数据库、数据表或全部的库导出为 SQL 脚本，便于该命令在不同版本的 MySQL 务器上使用。例如， 当需要升级 MySQL 服务器时，可以先使用 mysqldump 命令将原有库信息到导出，然后直接在升级后的 MySQL 服务器中导入即可。

**（1）对单个库进行完全备份**

```shell
[root@localhost ~]# mysqldump -uroot -p123 --databases auth > backup/auth-$(date +%Y%m%d).sql

[root@localhost ~]# ls backup/  
auth-20191013.sql mysql_all-2019-10-13.tar.xz  
[root@localhost ~]# grep -Ev "^/|^$|^-" backup/auth-20191013.sql   
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `auth` /*!40100 DEFAULT CHARACTER SET latin1 */;  
USE `auth`;  
DROP TABLE IF EXISTS `user`;  
CREATE TABLE `user` (  
`name` char(10) NOT NULL,  
`ID` int(48) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=latin1;  
LOCK TABLES `user` WRITE;  
INSERT INTO `user` VALUES ('crushlinux',123);  
UNLOCK TABLES;
```



**（2）对多个库进行完全备份：**

```shell
[root@localhost ~]# mysqldump -uroot -p123 --databases auth mysql> backup/auth+mysql-$(date +%Y%m%d).sql

[root@localhost ~]# ls backup/  
auth-20191013.sql auth+mysql-20191013.sql mysql_all-2019-10-13.tar.xz
```



（3）对所有库进行完全备份：

```shell
[root@localhost ~]# mysqldump -uroot -p123 --events --opt --all-databases > backup/mysql_all.$(date +%Y%m%d).sql

[root@localhost ~]# ls backup/  
auth-20191013.sql mysql_all.20191013.sql  
auth+mysql-20191013.sql mysql_all-2019-10-13.tar.xz
```



**（4）对表进行完全备份：**

```shell
[root@localhost ~]# mysqldump -uroot -p123 auth user > backup/auth_user-$(date +%Y%m%d).sql  
[root@localhost ~]# ls backup/  
auth-20191013.sql auth_user-20191013.sql mysql_all-2019-10-13.tar.xz  
auth+mysql-20191013.sql mysql_all.20191013.sql

[root@localhost ~]# grep -Ev "^/|^$|^-" backup/auth_user-20191013.sql   
DROP TABLE IF EXISTS `user`;  
CREATE TABLE `user` (  
`name` char(10) NOT NULL,  
`ID` int(48) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=latin1;  
LOCK TABLES `user` WRITE;  
INSERT INTO `user` VALUES ('crushlinux',123);  
UNLOCK TABLES;
```



**（5）对表结构的备份**

```shell
[root@localhost ~]# **mysqldump -uroot -p123 -d auth user > backup/desc_auth_user-$(date +%Y%m%d).sql**

[root@localhost ~]# grep -Ev "^/|^$|^-" backup/desc_auth_user-20191013.sql   
DROP TABLE IF EXISTS `user`;  
CREATE TABLE `user` (  
`name` char(10) NOT NULL,  
`ID` int(48) DEFAULT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```



**二、恢复数据**

**1、使用 source 命令**

```shell
[root@localhost ~]# mysql -uroot -p123  
Welcome to the MariaDB monitor. Commands end with ; or \g.  
Your MariaDB connection id is 8  
Server version: 5.5.41-MariaDB MariaDB Server

Copyright (c) 2000, 2014, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> drop database auth;  
Query OK, 1 row affected (0.02 sec)

MariaDB [(none)]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
5 rows in set (0.00 sec)
```



**先确定 auth 的备份的路径：**backup/auth-20191013.sql

```shell
MariaDB [(none)]> source backup/auth-20191013.sql

MariaDB [auth]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| auth |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
6 rows in set (0.00 sec)
```



**2、用 mysql 命令**

```shell
MariaDB [auth]> drop database auth;  
Query OK, 1 row affected (0.01 sec)

[root@localhost ~]# mysql -uroot -p123 < backup/auth-20191013.sql 

MariaDB [(none)]> show databases;  
+--------------------+  
| Database |  
+--------------------+  
| information_schema |  
| auth |  
| crushlinux |  
| mysql |  
| performance_schema |  
| test |  
+--------------------+  
6 rows in set (0.00 sec)
```



**-e 后面可以加执行的语句：**

```shell
[root@localhost ~]# mysql -uroot -p123 -e 'select * from auth.user;'  
+------------+------+  
| name | ID |  
+------------+------+  
| crushlinux | 123 |  
+------------+------+
```



**三、MySQL 备份思路**  
1、定期实施备份，指定备份计划或策略，并严格遵守.  
2、除了进行完全备份，开启 MySQL 服务器的 binlog_日志功能是很重要的（完全备份加上日志，可以对 MySQL 进行最大化还原）。

3、使用统一和易理解的备份名称，推荐使用库名或者表名加上时间的命名规则，如 mysql_user-20181214.sql，不要使用 backup1 或者 abc 之类没有意义的名字。

4、定期抽查备份的可靠性，做还原测试或者检查文件大小等方式。

5、通过异地或者跨机房等方式来存放备份数据，防止源数据和备份文件一起损坏。