+++
date = '2025-01-11T10:38:44+08:00'
draft = false
title = 'Linux学习笔记'
slug = "9ot8187agx"
description = "Linux学习笔记"
categories = ["os"]
tags = ["Linux"]
summary = "Linux学习笔记"
+++
# 概述

Linux是一种自由和开放源码的类UNIX操作系统。该操作系统的内核由林纳斯·托瓦兹在1991年10月5日首次发布在加上用户空间的应用程序之后，成为Linux操作系统。

# 安装与下载

[VMware虚拟机下载安装Linux(CentOS7)系统_虚拟机linux系统下载_](https://blog.csdn.net/da_ge_de_nv_ren/article/details/128408962)

# 目录结构

Linux 的一切资源都挂载在 / 节点下。  
/bin： Binary的缩写。存放系统命令，普通用户和 root 都可以执行。放在 /bin 下的命令在单用户模式下也可以执行。  
/boot： 这里存放的是启动 Linux 时使用的一些核心文件，包括一些连接文件以及镜像文件。  
/dev： Device的缩写。该目录下存放的是 Linux 的外部设备，在 Linux 中访问设备的方式和访问文件的方式是相同的。  
/etc： Etcetera的缩写。这个目录用来存放所有的系统管理所需要的配置文件和子目录。  
/home： 用户的主目录，在 Linux 中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。  
/lib： Library的缩写。这个目录里存放着系统最基本的动态连接共享库，其作用类似于 Windows 里的 DLL 文件。几乎所有的应用程序都需要用到这些共享库。  
/lib64： 64位相关的库会放在这。  
/media： linux 系统会自动识别一些设备，例如U盘、光驱等等，当识别后，Linux 会把识别的设备挂载到这个目录下。  
/mnt： 系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在 /mnt/ 上，然后进入该目录就可以查看光驱里的内容了。  
/opt： optional的缩写。这是给主机额外安装软件所摆放的目录。  
/proc： Processes的缩写。/proc 是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。这个目录的内容不在硬盘上而是在内存里  
/root： 该目录为系统管理员，也称作超级权限者的用户主目录。   
/run： 运行目录  
/sbin： s 就是 Super User 的意思，是 Superuser Binaries (超级用户的二进制文件) 的缩写，这里存放的是系统管理员使用的系统管理程序。  
/srv： 该目录存放一些服务启动之后需要提取的数据。  
/sys： 虚拟文件系统。和 /proc/ 目录相似，该目录中的数据都保存在内存中，主要保存与内核相关的信息  
/tmp： temporary的缩写这个目录是用来存放一些临时文件的。  
/usr： unix system resources缩写。用于存储系统软件资源。  
/var： 用于存储动态数据，例如缓存、日志文件、软件运行过程中产生的文件等  

# 常用命令

[Linux命令大全(手册) – 真正好用的Linux命令在线查询网站 (linuxcool.com)](https://www.linuxcool.com/)

```
# 查看cpu信息
lscpu
# 查看内存大小
free
# 查看硬盘和分区情况
lsblk
# 查看内核版本
uname -r
# 查看操作系统发行版本
cat /etc/redhat-release 
cat /etc/os-release 
# 显示日历
cal –y 
# 关机
halt
poweroff
# 重启
reboot
# 用户登录信息查看命令
whoami: 显示当前登录有效用户
who: 系统当前所有的登录会话
w: 系统当前所有的登录会话及所做的操作
# 显示当前工作目录
pwd
# 列出当前目录的内容或指定目录
ls
	常用选项
	-a 包含隐藏文件
    -l 显示额外的信息
    -R 目录递归
    -ld 目录和符号链接信息
    -1 文件分行显示
    -S 按从大到小排序
    -t 按mtime排序
    -u 配合-t选项，显示并按atime从新到旧排序
    -U 按目录存放顺序显示
    -X 按文件后缀排序
```

# 路径

- 绝对路径
  以正斜杠/ 即根目录开始
  完整的文件的位置路径
  可用于任何想指定一个文件名的时候
- 相对路径名
  不以斜线开始
  一般情况下，是指相对于当前工作目录的路径，特殊场景下，是相对于某目录的位置
  可以作为一个简短的形式指定一个文件名

# 学习网址

http://www.yunweipai.com/

# 文件管理

## 创建文件

```
touch 文件名
```

## 创建目录

```
mkdir 路径和目录名
```

## 复制

```
cp 源文件路径 目标文件夹
```

## 移动

```
mv 源文件路径 目标文件路径
```

## 删除

```
rm -rf 文件或目录的路径
```

## 查看

```
cat全部
more翻页
head头部
tail尾部
grep过滤关键字
语法：grep 关键字   文件名
# grep 'abc' /root/file1.txt
```

## 修改

图像文件编辑器 gedit

文本编辑器 vi vim

```
# 光标定位 
hjkL              //上下左右
0 $               //行首行尾
gg G 			//页首页尾
3G 进入第三行  
/string (n N 可以循环的)     //查找字符，按n键选下一个（重要）

#文本编辑
文本编辑
yy 复制
dd 删除
p 粘贴
u undo撤销

# 进入其他模式
进入其它模式
a 进入插入模式
i 进入插入模式
o 进入插入模式
A 进入插入模式

: 进入末行模式（扩展命令模式）
v 进入可视模式
ESC 返回命令模式

# 扩展命令模式
保存退出

:w 保存 
:q 退出 
:wq 保存并退出 
查找替换

:范围 s/原内容/新内容/全局 
:1,5 s/root/qianfeng/g          从1－5行的root 替换为qianfeng 
另存为

:w file9.txt 另存为 file9.txt
:set nu 设置行号 
:set nonu 取消设置行号 

:set list 显示控制字符
```

## 文件类型

```
- 普通文件（文本文件，二进制文件，压缩文件，电影，图片。。。）
d 目录文件（蓝色）
b 设备文件（块设备）存储设备硬盘，U盘 /dev/sda, /dev/sda1
c 设备文件（字符设备）打印机，终端 /dev/tty1
l 链接文件（淡蓝色）
s 套接字文件
p 管道文件
```

# 用户与用户组

```
[root@localhost ~]# head -1 /etc/passwd
root:x:0:0:root:/root:/bin/bash
用户名：x：uid：gid：描述：HOME：shell

系统约定：RHEL7
uid：0 特权用户
uid：1~499 系统用户
uid：1000+ 普通用户

用户基本信息文件	/etc/passwd
用户密码信息文件	/etc/shadow
组信息文件		  /etc/group
```

## 用户管理

1、创建用户 `useradd [选项] 用户名`

```
选项：
    -d	指定用户登录时的目录
    -e	指定用户的有效期限
    -f	缓冲天数，密码过期时在指定天数后关闭该账号
    -g	指定用户所属组
    -G	指定用户的附加组
    -m	自动创建用户的登录目录
    -r	创建系统账号
    -s	指定用户的登录shell
    -u	指定用户的用户ID
```

2、查看用户的属性信息`cat /etc/passwd`

3、设置用户密码 `passwd [选项] 用户名`

修改密码也使用同样的语法格式

```
选项：
-l	锁定密码，锁定后密码失效，无法登录
-d	删除密码，仅系统管理员可用
-S	列出密码的相关信息，仅管理员可使用
-f	强制执行
```

查看用户的密码信息（已加密）`cat /etc/shadow`

4、删除用户 `userdel [选项] 用户名`

```
-f	强制删除，即使是当前用户
-r	删除用户的同时，删除与用户相关的所有文件
```

5、修改用户信息 `usermod [选项] 参数`

```
usermod命令用与修改用户属性信息，包括用户ID、主目录、用户组、账号有效信息
使用usermod命令时，必须先明确该用户没有在电脑上执行任何程序
-d	修改用户的的登录目录
-e	修改账号的有效期限
-f	修改缓冲天数
-g	修改指定用户所属组
-G	修改指定用户的附加组
-l	修改用户账号名称
-L	锁定密码，锁定后密码失效，无法登录
-U	解除密码锁定
-s	修改指定用户的登录shell
-u	修改指定用户的用户ID

用法：
# usermod -u 678 user1
```

## 用户组管理

1、新增用户 `groupadd [选项] 参数`

```
-g	指定新建用户组的组ID
-r	创建系统用户组，组ID的取值范围为1~499
-o	允许创建组ID已存在的用户

用法：
# groupadd -g 550 group1
```

Linux系统将用户组信息储存在`/etc/group`文件中，新用户组创建成功后，该文件将多一条与该用户组相关的记录。

2、删除用户组 `groupdel 组名`

3、修改用户组属性 `groupmod [选项] 参数`

```
选项：
-g	为用户组指定新的组ID
-n	修改用户组的组名
-o	允许用户创建组ID已存在的用户组

用法：
# groupmod -n <旧组名> <新组名>
# groupmod -o <组名> -g <已经存在的组ID>
```

4、用户组切换 `newgrp 用户组`

```
用户组分为基本组和附加组，将用户添加附加组后，用户可拥有对应组的权限。用户的基本组唯一，但附加组可以不唯一。用户可从附加组中移除，但不能从基本组中移除。
```

5、用户组管理

gpasswd命令用于管理用户组 `gpasswd [选项] 参数`

```
-a	添加用户到用户组
-d	从用户组中删除用户
-r	删除密码
-R	限制用户登入组，只有组中的成员才可以用newgrp加入用户组

用法：
# gpasswd -a user1 group1	将用户user1添加到group1
```

## 用户切换

1、使用su命令切换用户，可以任意用户切换 `su [选项] 用户名`

若选项和用户名缺省（默认），则表示切换到root用户，但此时保留原来用户的工作环境，若使用“su -”，则表示从当前用户切换到root用户，并切换到root用户的工作目录。

```
选项：
-c	执行完指定的指令后，切换到原来的用户
-l	切换用户的同时，切换到对应用户的工作目录，环境变量也会随之改变。
-m,-p	切换用户，不改变环境目录
-s	指定要执行的shell
```

2、sudo命令的格式`sudo [选项] [参数]`

sudo可使当前用户以其他身份来执行命令，若不指定用户名，则默认以root身份执行。在使用sudo命令时，用户需要输入自己的密码，密码验证在之后的5分钟内有效，若超过则需要重新验证。

使用sudo命令之前，需要先在etc目录下的sudoers文件中对可执行sudo指令的用户进行设置。sudoers文件内容须遵循语法规范，visudo命令可防止其他用户同时修改sudoers文件。

```
# visudo
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
# 第二条语句是对root用户权限设置，作用：使root用户能够在任何情景下执行任何命令，
格式：
账户名	主机名称=(可切换的身份)	可用的命令

说明：
账户名：只有账户名被写入sudoers文件时，该用户才能使用sudo命令
主机名称：该参数决定此条语句中账户名对应的用户可以从哪些网络主机连接当前Linux主机，root用户默认可以			来自任何一台网络主机。
可用的命令：该参数指数此条语句中的用户可以执行哪些命令。注意，命令的路径为绝对路径。

以上的语句ALL分别代表任何主机，任何身份和任何命令。以用户user1为例，若要使用户user1能以root用户执行/bin/more命令，则应在sudoers文件中添加如下内容。
user1	ALL=(root)	/bin/more
保存退出后，切换到用户user1，使用命令`sudo -l`可查看该用户可以使用的命令。
```

当需要操作的用户较多时，如此操作显然相对麻烦，Linux系统支持为用户组内的整组用户统一设置权限。

```
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
"%"声明之后的字符串是一个用户组，该语句表示任何加入用户组wheel的用户，都能通过任意主机连接、任何身份执行全部命令。若想提升某些用户的权限为ALL，将它们添加到用户组wheel即可。

例子
%group1 ALL=(root)	/bin/more
禁用
%group1 ALL=(root)	!/bin/more
```

# 用户的权限

## 基本权限UGO

| 权限 | 对应字符 | 文件           | 目录                       |
| ---- | -------- | -------------- | -------------------------- |
| 读   | r 4      | 可查看文件内容 | 可以列出目录中的内容       |
| 写   | w 2      | 可修改文件内容 | 可以在目录中创建、删除文件 |
| 执行 | x 1      | 可执行该文件   | 可以进入目录               |

常用的权限管理命令有chmod、chown、chgrp等，默认情况下，普通用户不能使用权限。

设置权限：

1、chmod更改权限，其功能为变更文件或目录权限（使用字符，使用数字）
语法：

```
chmod 	对象(u/g/o)赋值符(+/-/=)权限类型(rwx) 	文件/目录
选项:
-f	不显示错误信息
-v	显示指令执行过程
-R	递归处理，处理指定目录及其中所有的文件与子目录
```

2、chown其功能为更改文件或目录的所有者。默认情况下文件的所有者为创建该文件的用户，或在文件被创建时通过命令指定用户，需要时可使用chown对文件的所有者进行修改。

命令格式 `chown [选项] [用户] [文件或目录]`

```
选项:
-f	不显示错误信息
-v	显示指令执行过程
-R	递归处理，处理指定目录及其中所有的文件与子目录

chown:设置文件属于谁，属组
	语法：	chown	用户名.组名	文件
					chown	user01.hr	/tmp/file1 
```

3、chgrp用于更改文件或目录的所属组，一般情况下，文件或目录与创建该文件的用户属于同一组，或在被创建时通过选项指定所属组，但在需要时，可通过chgrp命令更改文件的所属组。

```
# chgrp 组名 文件或目录
```

小结：chmod、chown、chgrp都可以对目录进行改权、改主、改组，但底下文件权限不会变，若要改变，使用-R递归操作

## 基本权限ACL

access contral list 限制用户对文件的访问
ACL是UGO的补充，或者说是加强版
命令：`setfacl -m g:hr:rwx /home/file1`

设置文件 -设置 对象：对象名：权限

```
[root@localhost tmp]# useradd alice
[root@localhost tmp]# useradd jack

[root@localhost tmp]# setfacl -m u:alice:rw /home/test.txt
[root@localhost tmp]# setfacl -m u:jack:- /home/test.txt
[root@localhost tmp]# getfacl /home/test.txt
getfacl: Removing leading '/' from absolute path names
# file: home/test.txt
# owner: root
# group: root
user::rw-
user:alice:rw-
user:jack:---
group::r--
mask::rw-
other::r--
#删除alice对test.txt文件的权限
[root@localhost tmp]# setfacl -x u:alice /home/test.txt
[root@localhost tmp]# getfacl /home/test.txt
getfacl: Removing leading '/' from absolute path names
# file: home/test.txt
# owner: root
# group: root
user::rw-
user:jack:---
group::r--
mask::r--
other::r--
```

## 特殊权限（了解）

1、特殊位suid：高级权限类型（suid,(sgid)针对文件/程序时，具备临时获得属主权限）
suid是针对文件所设置的一个特别的权限。
功能：使调用文件的用户临时具备属主的能力
普通用户不能查看root目录下的文件，但赋予/usr/bin/cat一个s权限后，普通用户可以cat，root文件

```
[root@localhost ~]#  cat /root/file1.txt
cat: /root/file1.txt: 权限不够
[root@localhost ~]# chmod u+s /usr/bin/cat
[root@localhost ~]# ll /usr/bin/cat
-rwsr-xr-x. 1 root root 54080 8月  20 2019 /usr/bin/cat
[root@localhost ~]# cat /root/file1.txt
123
```

2、文件属性chattr
i:在文件上启用这个属性时，我们不能更改、重命名或者删除这文件
a:允许在文件中追加操作

```
#chattr改变文件属性
[root@localhost ~]# chattr +i ./file1.txt
#lsattr列出文件属性
[root@localhost ~]# lsattr ./file1.txt
----i----------- ./file1.txt
#不允许修改、删除文件
[root@localhost ~]# rm -rf file1.txt
rm: 无法删除"file1.txt": 不允许的操作
#去除i属性
[root@localhost ~]# chattr -i ./file1.txt
```

3、进程掩码umask

# 进程管理

进程存在于计算机内存中，计算机内存中可同时存在多个进程，每个CPU上同时只会执行一个进程，但计算机上似乎能够同时运行多个进程。实际上这是计算机采用了“多道程序设计”技术，即计算机允许多个相互独立的程序同时进入内存，在内核的管理控制下，相互之间穿插运行。

## 进程状态

初始态、就绪态、运行态、睡眠态和终止态。

## 进程管理

1、ps 在命令行输入ps后按回车键就能查看当前系统中正在运行的进程。

```
选项：
a	显示当前终端机下的所有进程，包括其他用户启动的进程
u	以用户的形式，显示系统中的进程
x	忽略终端机，显示所有进程
e	显示每个进程使用的环境变量
r	只列出当前终端机中正在执行的进程
```

```
-a	显示所有终端机中除阶段作业领导进程(拥有子进程的进程)之外的进程
-e	显示所有进程
-f	除默认项外，显示UUID、PPID、C、STIME项
-o	指定显示哪些字段，字段名可以使用长格式，也可以使用“%字符”的短格式指定，多个字段名使用逗号分割
-l	使用详细的格式显示进程信息
```

进程是已启动的可执行程序的运行实例，进程有一下组成部分：
1、一个文件：
2、被分配内存空间：
3、有权限控制；
4、程序代码的一个或多个副本（也叫执行线程）；
5、拥有状态

ps -ef 显示含义（查看进程的父子关系）

列的含义

| 字段名 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|  UID   |                      该进程执行的用户id                      |
|  PID   |                            进程id                            |
|  PPID  | 该进程的父进程id，如果一个程序的父级进程找不到，该程序的进程被称之为僵尸进程 |
|   C    |                 cpu的占用率，其形式是百分数                  |
| STIME  |                        进行的启动时间                        |
|  TTY   | 终端设备，发起该进程的设备识别符号，如果显示“?”则表示该进程不是由终端设备发起的 |
|  TIME  |                        进程的执行时间                        |
|  CMD   |                  该进程的名称或者对应的路径                  |

案例（100%使用的命令）在ps的结果中过滤出想要查看的进程状态

```
#ps -ef |grep 进程名称
自定义显示内容
ps  axo  user,pid,ppid,%mem | head -3
```

**ps aux命令展示的进程信息**

| user    | 用户                         |
| ------- | ---------------------------- |
| PID     | 进程的编号                   |
| %CPU    | 占用CPU时间的百分比          |
| %MEM    | 占用内存空间的百分           |
| VSZ RSS | 虚拟内存和实际内存占用大小   |
| TTY     | 终端类型                     |
| STAT    | 运行，睡眠，停止，退出，僵死 |
| START   | 进程启动时间                 |
| TIME    | 占用cpu的时间                |
| COMMAND | 程序的路径和名称             |

2、top命令可以实时观察系统的整体运行情况，默认时间间隔为3s，即每3秒更新一次界面，是一个很实用的系统性能检测工具。

`top [选项]`

```
top -d 1 //每1秒刷新
top -d 1 -p 进程号	//查看指定进程的动态信息
```

3、pstree

一个新进程由已存在的进程创建，创建新进程的进程与新创建的进程为父子进程，一个父进程可以创建多个子进程，由同一个进程创建的多个子进程又称为兄弟进程。

pstree命令可以以树状的形式显示系统中的进程，直接观察进程之间的派生关系

`pstree [选项]`

```
选项：
-a	显示每个进程的完整命令
-c	不使用精简标识法
-h	列出树状图，特别标明当前正在执行的进程
-u	显示用户名称
-n	使用程序识别码排序
```

4、pgrep

命令根据进程名从进程队列中查找进程，查找成功后默认显示进程的pid

5、nice

进程的优先级会影响进程执行的顺序，在linux系统中，可通过改变进程的nice值来更改进程的优先级

`nice [选项] [参数]`

如修改bash的优先级为5 `nice -n 5 bash`

6、jobs

使用jobs命令可以查看当前内存中的作业列表

7、bg和fg

使用快捷键Ctrl+Z也能将进程调入后台，但调入后台的进程会被暂时停止。若要将后台的命令调回前台继续执行，可以使用fg命令

`fg 作业号`

8、kill

kill命令一般用于管理进程，它的工作原理是发送某个信号给指定进程，以改变进程的状态，命令格式

`kill 选项 [参数]`

```
kill  序号(一般为9)  进程ID
killall 服务名
```

```
1) SIGHUP	重新加载配置       
2) SIGINT	键盘中断Ctrl+C       
3) SIGQUIT	键盘退出Ctrl+\，类似SIGINT   
9) SIGKILL	强制终止，无条件     
15) SIGTERM	终止（正常结束），缺省信号
18) SIGCONT	继续
19) SIGSTOP	暂停
20) SIGTSTP	键盘暂停Ctrl+C   
```

# 管道和重定向

## 重定向和管道符

> FD,文件描述符，文件句柄进程使用文件描述符来管理打开的文件
> 作用：FD给文件一个描述符（软链接，数字范围0-255），进程调用文件时就不用使用长路径，直接使用文件描述符就可以调用文件进程了。
> 0号，标准输入（一个文件代表键盘，按键盘数字先进入文件中，然后通过0号FD进入程序中去），文件结果通过1号描述符输出在显示器文件，然后在显示器/终端显示出来，2号描述符，标准错误输出，输出在显示器文件，然后在显示器/终端显示出来。

## 输出重定向

可分为两种：正确输出和错误输出

正确输出：1> 等价于 > 1>> 等价于 >>
错误输出：2> 没有简写 2>> 没有简写
将正确输出和错误输出重定向到一个文件里

## 输入重定向发送邮件

默认邮件发送的过程：
向系统其他用户发送邮件

```
[root@localhost tmp]# mail -s 'ssss' xulei			'ssss'  为标题
你好呀！hello,world								   内容
.												按键 . 表示输入完成
EOT												
[root@localhost ~]# su - xulei						查看mail消息
上一次登录：三 6月 23 13:23:16 CST 2021pts/0 上
[xulei@localhost ~]$ mail				回车后按键1，再回车
#使用重定向快速创建邮件
[xulei@localhost ~]$ vim test01.txt
[xulei@localhost ~]$ mail -s '标题' admin < test01.txt	将内容以文本编辑发送
[xulei@localhost ~]$ su - admin
密码：
上一次登录：五 6月 25 17:46:20 CST 2021:0 上
[admin@localhost ~]$ mail
```

## 管道符

### 进程管道

1、管道命令可以将多条命令组合起来，一次性完成复杂的处理任务

```
[root@localhost ~]# cat /etc/passwd | grep 'root'
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@localhost ~]# cat /etc/passwd | grep 'root' |tail -1
operator:x:11:0:operator:/root:/sbin/nologin
[root@localhost tmp]# cat /etc/passwd |grep ntp
ntp:x:38:38::/etc/ntp:/sbin/nologin
[root@localhost tmp]# cat /etc/passwd |grep ntp |cut -d: -f3  #cut命令，-d：（以冒号作为分隔符）-f3（切割第三列）
38
```

### tee管道

三通管道，既交给另一个程序处理，又保存一份副本。

```
passwd内容放到file1.txt，并且显示第一行在终端（类似水管的三通管）
[root@localhost tmp]# cat /etc/passwd |tee file1.txt |head -1
```

### 参数传递Xargs

cp、rm一些特殊命令就是不服其他程序

```
[root@localhost tmp]# touch file{1..5}	创建1-5的文件
[root@localhost tmp]# cat files.txt		以下是文本内容
/tmp/file1
/tmp/file2
[root@localhost tmp]# cat files.txt |rm -rvf		将文本的内容传到rm执行，结果失败
[root@localhost tmp]# cat files.txt |xargs rm -rvf	 使用xargs参数，执行成功，v参数：显示信息
已删除"/tmp/file1"
已删除"/tmp/file2"
```

使用<<EOF EOF指令

```
#写一个多行的代码文本
[root@localhost dir1]# vim js.sh   			以下是脚本的内容
cat > /tmp/dir/tt.txt <<EOF					<<EOF  EOF用法（cat内容，将内容定向到tt.txt文中
11111111111--第1行										
22222222222--第2行							 具体内容  	
33333333333--第3行					
EOF

[root@localhost dir1]# chmod +x js.sh
[root@localhost dir1]# ./js.sh
[root@localhost dir1]# cat tt.txt
11111111111--第1行
22222222222--第2行
33333333333--第3行
```

# 磁盘管理

## 存储管理

需要掌握的知识点：
1、磁盘长什么样？
2、磁盘有哪些？
3、什么是好磁盘，什么是次磁盘
4、把磁盘的软件操作（分区格式化）
5、写入输入数据到磁盘

磁盘类型：

​		机械硬盘 由：盘片、马达、磁头、磁臂等组成的机器结构存储器

​		固态硬盘：由芯片和集成电路组成。尺寸：2.5英寸、3.5英寸

转速：每分钟旋转的速度
5400转、7200转、15000转

接口：早期使用IDE，现在使用SATA

厂商：西部数据、希捷、三星、日立、金士顿

磁盘命名方式
kernel对不同接口硬盘的命名方式：如RHEL7/CentOs

（略）IDE（并口）

SATA（串口）：/dev/sda sda是一个文件，s代表sata就是串口，d代表磁盘，a代表第一块

磁盘分区方式
MBR：主引导记录（MBR，Master Boot Record）是 硬盘
支持最大磁盘容量是<2TB，设计时分配四个分区。如果超过4个分区，需放弃主分区，改为扩展分区和逻辑分区。

GPT： 全局唯一标识分区表（GUID Partition Table，缩写：GPT）是一个实体磁盘的分区表的结构布局的标准。
支持最大磁盘容量是>2TB，分配128个分区

使用MBR方式创建分区，可通过fdisk命令进行管理 fdisk [选项] [磁盘]

```
选项：
-l	详细显示磁盘及其信息
-s	显示磁盘分区容量（单位为block）
-b	设置扇区大小（扇区大小取值为512、1024、2048或4096，单位为MB）			
```

## 管理磁盘

详细解析 [CentOS 7 磁盘挂载](https://blog.csdn.net/weixin_46860149/article/details/109718894)

三部曲：分区（隔间）、格式化（放家具/打造柜子格）、挂载（加个门/目录，才能访问）

1、创建和查看磁盘

```
# 查看
# fdisk -l /dev/sdb	
# lsblk
# ll /dev/sd* 	

# fdisk /dev/sdb	创建
# partprobe /dev/sdb		刷新
```

```
[root@localhost ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x4dc36548.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p		//请选择主分区，或扩展分区
Partition number (1-4, default 1):		//选择分区号
First sector (2048-2097151, default 2048):	//选择磁盘开始的扇区
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-2097151, default 2097151): +200M
Partition 1 of type Linux and of size 200 MiB is set		//选择磁盘分区结束的扇区，即分区大小，实际环境根据磁盘划分

Command (m for help): w		//输入w保存分区信息，自动退出
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

2、创建挂载信息

```
[root@localhost ~]# mkfs.ext4 /dev/sdb1			#使用ext4格式，格式化sdb1的磁盘 创建一个文件系统
[root@localhost ~]# mkdir /mnt/disk1				#创建一个挂载目录
[root@localhost ~]# mount -t ext4 /dev/sdb1 /mnt/disk1	#将磁盘挂载在目录上
[root@localhost ~]# df -hT						#查看文件系统信息
```

补充：系统目录结构是统一的（根下面有/mnt目录，在mnt/disk1里写入数据，并不是使用sda的内存空间，而是使用sdb1的空间），实际的内存存储在硬盘上。磁盘一旦被挂载，数据写入到新挂载的磁盘，而不是sda的内存。

3、MBR主引导记录是一个文件，共64个字节，每16个字节记录一个主分区，所以一块硬盘只有4个分区，如果想要超过4个分区，则需要一个主分区分出来成为扩展分区，扩展分区可以继续分为N个逻辑分区

## 交换分区管理Swap

作用：“提升”内存的容量，防止OOM（out of memory）
实际生产：大于4GB而小于16GB，最小需要4GB交换空间
大于16GB而小于64GB，最小需要8GB交换空间
大于64GB而小于256GB，最小需要16GB交换空间

```
[root@localhost ~]# free -m					查看挂载信息
             total    used     free   shared  buff/cache   available
Mem:         972      428       201      10       342         391
Swap:        3271      0        3271
----------------------------------------------------------------------------------
增加交换分区内存空间（以磁盘sde为例）
[root@localhost ~]# fdisk /dev/sde	  #加一个扩展分区为1号  e  在扩展分区上加一个100M的逻辑分区l  分区为2号
[root@localhost ~]# mkswap /dev/sde2
[root@localhost ~]# swapon /dev/sde2
[root@localhost ~]# free -m
              total    used     free   shared  buff/cache   available
Mem:            972    428       201      10       342         391
Swap:          3371      0      3371
由上可见，swap交换分区多了100M
```

补充：在挂载的磁盘上/mnt/disk1写入数据(创建1-5的文件夹，数据指向/dev/sdb1)，然后卸载磁盘/dev/sdb1,进入/mnt/disk1并没有1-5的文件夹(此时数据指向sda)，此时再创建6-10的文件夹，挂载/dev/sdb1后，/mnt/disk1并没有6-10的文件夹，除非将挂载点转到/mnt/disk2。

逻辑卷LVM
目的：管理磁盘的一种方式，性质与基本磁盘无异
特点：随意扩张大小
PV：物理卷
VG：卷组
LV：逻辑卷

1、pvcreat命令用于将磁盘分区初始化为物理卷，命令格式 pvcreate [选项] 参数

```
选项：
    -f	强制创建物理卷，不需要用户确认
    -u	指定设备的UUID（通用唯一识别码）
    -y	所有的问题都用yes回答
    -z	是否使用前四个扇区（y/n）
```

```
[root@localhost ~]# pvcreate /dev/sdd		//将物理磁盘转换成物理卷-PV
  Physical volume "/dev/sdd" successfully created.
[root@localhost ~]# vgcreate vg1 /dev/sdd	//创建卷组VG
  Volume group "vg1" successfully created
[root@localhost ~]# lvcreate -L 800M -n lv1 vg1		//指定大小，创建逻辑卷 -L大小 -n卷名 vg1组名
  Logical volume "lv1" created.
[root@localhost ~]# mkfs.ext4 /dev/vg1/lv1	//创建文件系统并挂载  /dev/卷组名/逻辑卷名
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
51296 inodes, 204800 blocks
10240 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=209715200
7 block groups
32768 blocks per group, 32768 fragments per group
7328 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@localhost ~]# mkdir /mnt/lv1
[root@localhost ~]# mount /dev/vg1/lv1 /mnt/lv1
[root@localhost ~]# df -hT
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        17G  1.3G   16G   8% /
devtmpfs                devtmpfs  478M     0  478M   0% /dev
tmpfs                   tmpfs     489M     0  489M   0% /dev/shm
tmpfs                   tmpfs     489M  6.8M  482M   2% /run
tmpfs                   tmpfs     489M     0  489M   0% /sys/fs/cgroup
/dev/sda1               xfs      1014M  125M  890M  13% /boot
tmpfs                   tmpfs      98M     0   98M   0% /run/user/0
/dev/mapper/vg1-lv1     ext4      772M  1.6M  714M   1% /mnt/lv1
[root@localhost ~]# dd if=/dev/zero of=/mnt/lv1/1.txt bs=1M count=800	//向目标分区写入大量数据，填满
dd: error writing ‘/mnt/lv1/1.txt’: No space left on device
754+0 records in
753+0 records out
790433792 bytes (790 MB) copied, 6.09358 s, 130 MB/s
[root@localhost ~]# df -hT
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        17G  1.3G   16G   8% /
devtmpfs                devtmpfs  478M     0  478M   0% /dev
tmpfs                   tmpfs     489M     0  489M   0% /dev/shm
tmpfs                   tmpfs     489M  6.8M  482M   2% /run
tmpfs                   tmpfs     489M     0  489M   0% /sys/fs/cgroup
/dev/sda1               xfs      1014M  125M  890M  13% /boot
tmpfs                   tmpfs      98M     0   98M   0% /run/user/0
/dev/mapper/vg1-lv1     ext4      772M  756M     0 100% /mnt/lv1
[root@localhost ~]# pvcreate /dev/sde  //创建PV。将PV增加到VG中
  Physical volume "/dev/sde" successfully created.
[root@localhost ~]# pvs
  PV         VG     Fmt  Attr PSize    PFree
  /dev/sda2  centos lvm2 a--   <19.00g      0
  /dev/sdd   vg1    lvm2 a--  1020.00m 220.00m
  /dev/sde          lvm2 ---     1.00g   1.00g
[root@localhost ~]# vgextend vg1 /dev/sde		//扩容VG
  Volume group "vg1" successfully extended
[root@localhost ~]# pvs
  PV         VG     Fmt  Attr PSize    PFree
  /dev/sda2  centos lvm2 a--   <19.00g       0
  /dev/sdd   vg1    lvm2 a--  1020.00m  220.00m
  /dev/sde   vg1    lvm2 a--  1020.00m 1020.00m
[root@localhost ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree
  centos   1   2   0 wz--n- <19.00g    0
  vg1      2   1   0 wz--n-   1.99g 1.21g
  [root@localhost ~]# lvextend -L +200M /dev/vg1/lv1		//扩容LV
  Size of logical volume vg1/lv1 changed from 800.00 MiB (200 extents) to 1000.00 MiB (250 extents).
  Logical volume vg1/lv1 successfully resized.
[root@localhost ~]# df -hT
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        17G  1.3G   16G   8% /
devtmpfs                devtmpfs  478M     0  478M   0% /dev
tmpfs                   tmpfs     489M     0  489M   0% /dev/shm
tmpfs                   tmpfs     489M  6.8M  482M   2% /run
tmpfs                   tmpfs     489M     0  489M   0% /sys/fs/cgroup
/dev/sda1               xfs      1014M  125M  890M  13% /boot
tmpfs                   tmpfs      98M     0   98M   0% /run/user/0
/dev/mapper/vg1-lv1     ext4      772M  756M     0 100% /mnt/lv1
[root@localhost ~]# resize2fs /dev/vg1/lv1
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vg1/lv1 is mounted on /mnt/lv1; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg1/lv1 is now 256000 blocks long.

[root@localhost ~]# df -hT
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        17G  1.3G   16G   8% /
devtmpfs                devtmpfs  478M     0  478M   0% /dev
tmpfs                   tmpfs     489M     0  489M   0% /dev/shm
tmpfs                   tmpfs     489M  6.8M  482M   2% /run
tmpfs                   tmpfs     489M     0  489M   0% /sys/fs/cgroup
/dev/sda1               xfs      1014M  125M  890M  13% /boot
tmpfs                   tmpfs      98M     0   98M   0% /run/user/0
/dev/mapper/vg1-lv1     ext4      970M  756M  154M  84% /mnt/lv1

```

2、vgcreat命令用于将物理卷整合为卷组，语法格式 `vgcreate [选项] 卷组名称 物理卷路径1 物理卷路径2 ……`

```
选项：
-l	设置卷组上允许创建的最大逻辑卷数
-p	设置卷组上允许添加的最大物理卷数
-s	设置卷组上的最小存储单元（PE）

用法：
# 将物理卷/dev/sdc2、/dev/sdc4整合为卷组，并命名为vg1
vgcreate vg1 /dev/sdc2 /dev/sdc4
或者vgcreate vg1 /dev/sdc{2,4}

# 将物理卷/dev/sdc2、/dev/sdc4整合为卷组，设置最小存储单位为8MB，并命名为vg2
vgcreate -s 8M vg2 /dev/sdc2 /dev/sdc4
```

3、lvcreate命令的功能是在已经存在的卷组中创建逻辑卷，命令格式

`lvcreate [选项] 卷组名/路径 物理卷路径`

```
-n	逻辑卷名称
```

4、vgdisplay用于显示LVM卷组信息，命令格式 vgdisplay [选项] 卷组

```
选项：
-s	使用短格式输出信息
-A	仅显示活动卷组的属性
```

5、lvextend

当逻辑卷的可用空间不足时，需要为其拓展存储空间。使用LVM机制管理磁盘时可通过lvextend命令动态地调整分区大小，命令格式 

`lvextend [选项] 逻辑卷`

```
选项：
-l	以PE为单位指定逻辑卷容量
-L	指定逻辑卷的容量，单位B/S/K/M/G/T/P/E
```

6、lvremove命令用于删除指定的LVM逻辑卷，若逻辑卷已经被挂载到系统中，则应先使用unmount命令将其卸载，再进行删除，命令格式 `lvremove [选项] 逻辑卷`

lvremove命令的常用选项为-f，其功能为强制删除指定逻辑卷

删除卷组、物理卷分别使用vgremove、pvremove，删除是应逆向删除，即先删除逻辑分区、卷组，最后删除物理卷。

## 文件系统

蓝色的小方块在文件系统中称为块（block），每一块可以存放4096个字节（4K），如果一个文件有5K,它会占用2个块。如果file1占用1,2,5，file2占用3,4，产生文件碎片，使用inode文件记录file文件的碎片的存放位置。之所以一个500G的硬盘，实际没用500G显示，是因为它inode和每个块之间有

block：存储文件的实际数据，实际存储文件的内容，若文件较大，会占用多个 block，block大小默认为4K。

inode（索引节点）：每产生一个小片，就是一个inode，每个小片对应的就是块的位置，我们想调用块的时候就找索引。

记录文件的属性（文件的元数据metadata，元数据，文件的属性，大小，权限，属主，属组，连接数，块数量，块的编号），一个文件占用一个inode，同时记录此文件所在的block number。inode大小为128bytes。

（略）superblock：通俗易懂说，就是一堆inode和block的统称。

每使用一次mkfs.ext4，就是创建一个文件系统.

使用df -i 查看sdb2可以存放多少个文件

```
[root@localhost disk1]# df -i |grep sdb2
/dev/sdb2                 51200      11     51189       1% /mnt/disk2
						总创建量  已用量	剩余
[root@localhost disk2]# ls
lost+found
[root@localhost disk2]# touch file{1..51189}		#创建51189个文件
[root@localhost disk2]# touch aaa.txt			大于51189则无法创建
touch: 无法创建"aaa.txt": 设备上没有空间
[root@localhost disk2]# df -i |grep sdb2
/dev/sdb2                 51200   51200       0     100% /mnt/disk2
#表示inode已经用完了，不能继续创建文件夹，但可以在其中的文件夹中写入数据。
```

结论：磁盘空间的限制根据inode和block两方面，请清理掉填满的分区，避免不必要的报错。

## 文件链接

### 符号链接

1、创建一个文件，并输入内容

```
[root@localhost disk4]# echo 123 > /file00
[root@localhost disk4]# cat /file00
123								硬		软
[root@localhost disk4]# ln -s /file00 /tmp/file11   #创建软连接
[root@localhost disk4]# cat /tmp/file11
123
[root@localhost disk4]# rm -rf /file00 删除硬连接，软连接不复存在  
[root@localhost disk4]# cat /tmp/file11
cat: /tmp/file11: 没有那个文件或目录
#可以对源文件进行追加，软链接文件也有追加内容
```

### 硬链接(可以理解为文件备份)

```
[root@localhost disk4]# echo 222 >/file2
[root@localhost disk4]# cat /file2
222
[root@localhost disk4]# ln /file2 /tmp/file-h1
[root@localhost disk4]# cat /tmp/file-h1
222
[root@localhost disk4]# rm -rf /file2
[root@localhost disk4]# cat /tmp/file-h1
222
#把源文件删除，硬链接文件还存在原有数据
```

## RAID磁盘列阵(了解)

RAID 0
将N块硬盘上选择合理的带区来创建带区集。其原理是将类似于显示器隔行扫描，将数据分割成不同条带(Stripe)分散写入到所有的硬盘中同时进行读写。多块硬盘的并行操作使同一时间内磁盘读写的速度提升N倍。

RAID 1
称为磁盘镜像，原理是把一个磁盘的数据镜像到另一个磁盘上，也就是说数据在写入一块磁盘的同时，会在另一块闲置的磁盘上生成镜像文件，在不影响性能情况下最大限度的保证系统的可靠性和可修复性上，只要系统中任何一对镜像盘中至少有一块磁盘可以使用，甚至可以在一半数量的硬盘出现问题时系统都可以正常运行，当一块硬盘失效时，系统会忽略该硬盘，转而使用剩余的镜像盘读写数据，具备很好的磁盘冗余能力。虽然这样对数据来讲绝对安全，但是成本也会明显增加，磁盘利用率为50%。

RAID5（分布式奇偶校验的独立磁盘结构）。
从它的示意图上可以看到，它的奇偶校验码存在于所有磁盘上，其中的p0代表第0带区的奇偶校验值，其它的意思也相同。RAID5的读出效率很高，写入效率一般，块式的集体访问效率不错，RAID5至少需要3块磁盘。

缺点：

1、RAID0没有冗余功能，如果一个磁盘（物理）损坏，则所有的数据都无法使用。 2、RAID1磁盘的利用率最高只能达到50%(使用两块盘的情况下)，是所有RAID级别 中最低的。
3、RAID0+1以理解为是RAID 0和RAID 1的折中方案。RAID 0+1可以为系统提供数据安全保障，但保障程度要比 Mirror低而磁盘空间利用率要比Mirror高。

创建RAID

Linux系统中使用mdadm命令创建和管理RAID，语法格式 mdadm [模式] <RAID设备名> [选项] <组件设备名>

```
模式：
-A/--accemble			组合，组装一个预先存在的阵列
-B/--build				构建，构建一个不需要超级块的阵列，阵列中的每个设备都没有超级块
-C/--create				创建，创建一个新阵列
-F/--follow/--monitor	 监视，监视一个或多个阵列的状态
-G/--grow				增长，更改RAID的容量或阵列中的设备数目
--auto-detect			自动侦测，请求内核启动任何自动检测的阵列
-I/--incremental		增加，向阵列中添加单个设备，或从阵列中删除单个设备
```

在-C模式下选项

```
-l	指定RAID级别
-n	指定设备数量
-a{yes/no}	是否自动为其创建设备文件
-c	指定数据块大小
-x	指定空闲盘个数，空闲盘可自动顶替损坏的工作盘
```

实验：准备四块硬盘

```
[root@localhost ~]#ls /dev/sd* -l	//查看
#创建RAID
[root@localhost ~]#yum -y install mdadm 	//确保mdadm命令可用
[root@localhost ~]# mdadm -C /dev/md0   -l5  -n3 -x1 /dev/sd{h,i,j,k}
			-C创建RAID    RAID的名称	-l5 RAID5 数据盘3  热备盘数量1    磁盘
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
# 格式化，挂载
[root@localhost ~]# mkfs.ext4 /dev/md0		格式化
[root@localhost ~]# mkdir /mnt/raid5		创建挂载文件
[root@localhost ~]# mount /dev/md0 /mnt/raid5	挂载
[root@localhost ~]# cp -rf /etc/ /mnt/raid5/etc		使用raid5
[root@localhost ~]# df -hT |tail -1
/dev/md0                ext4      9.8G  126M  9.1G    2% /mnt/raid5
#添加了4块5G的盘，显示了10G。因为两块盘是数据盘，第三块的校验盘，第四块的热备盘。
[root@localhost ~]# mdadm -D /dev/md0		-D 查看raid5详细信息
```

破坏一块磁盘

```
[root@localhost ~]# watch -n0.5 'mdadm -D /dev/md0 |tail -10'
#实时查看信息
[root@localhost ~]# mdadm /dev/md0 -f /dev/sde -r /dev/sde
#模拟损坏的数据盘
```

# 查找和压缩

## 文件查找

which ：命令查找

alias：起别名(可以输入别名就可以执行对应的命令)，语法：alias 别名=‘ls -l’

locate：文件查找，语法：locate 文件名

find：任意文件查找，语法：find 路径 选项 文件名（文件名记不全可用代替，如hos 就会查找以hos字样的文件）

```
选项：
    -name ：按照文件名区分大小写
    -iname：不区分大小写 
    -size：按文件大小查找（如后接 +5M）
    -maxdepth：按照目录级查找（如find  / maxdepth 4  -a  -name  "文件字眼" ）
            从根开始向下查找，包括根目录，总共4级
    -user/group:  根据用户/组查找（如 find  /home -user  alice）
    -type：按照文件类型查找（find  /dev/  -type b）(b:块设备，f：普通文件)
    -perm：按照文件权限查找
```

find后接命令：
find ………… -delete （表示将查到的内容删除）
find ………… -ok cp -rvf {} 目标目录  \; （-ok表示连接符，{}表示引用查找的内容后的文件，\;表示结束符）

```
[root@localhost ~]# find /etc/ -name 'ifcfg*' -ok cp -rvf {} /tmp \;
```

# 文件打包及压缩

## 打包

tar命令本是用于备份文件的命令，该命令可以打包多个文件或目录，亦可将被打包的文件与目录中包中还原，

命令格式 `tar 选项 包名 [参数]`

```
选项：
-c	创建新的备份文件
-x	从备份文件中还原文件
-v	显示命令执行过程
-f	指定备份文件
-z	通打包完成后使用gzip命令将包压缩
-j	打包完成后使用bzip2命令将包压缩
-p	保留包中文件原来的属性

用法：
# 将目录test下的文件打包
$ tar -cvf test.tar ./test

# 将目录test下的文件打包,并以gzip命令将包压缩
$ tar -zcvf test.tar.gz ./test

# 将目录test下的文件打包,并以bzip2命令将包压缩
$ tar -jcvf test.tar.bz2 ./test

#从包test.tar.bz2中还原文件
$ tar -xvf test.tar.bz2
```

## 压缩和解压

1、zip/unzip命令

使用zip命令压缩文件时压缩包一般命名为"文件名.zip" `zip [选项] 压缩包名 参数`

```
选项：
-j	只保留文件名称及其内容，不存放任何目录名称
-m	文件压缩完成后，删除原始文件
-o	以压缩文件内拥有最新更改时间的文件为准，更新压缩文件的更改时间
-r	当参数为目录时，递归处理目录下的所有文件和子目录

$ zip -r dir1.zip dir1
注意：不能在当前目录压缩该目录
```

2、使用tar命令压缩 `tar [选项] [压缩包名] [源文件]`

```
压缩类型分析（了解）
[root@localhost ~]# tar -cf etc.tar /etc
[root@localhost ~]# tar -czf etc.gz /etc
[root@localhost ~]# tar -cjf etc.bz /etc
[root@localhost ~]# tar -cJf etc.xz /etc
压缩时间由上到下越来越长，压缩程度越来越狠
[root@localhost ~]# ls -lh |grep etc
-rw-r--r--. 1 root root  11M 6月  28 00:18 etc.bz
-rw-r--r--. 1 root root  12M 6月  28 00:17 etc.gz
-rw-r--r--. 1 root root  38M 6月  28 00:17 etc.tar
-rw-r--r--. 1 root root 8.3M 6月  28 00:19 etc.xz
```

# 软件包管理

Linux提供了软件包的集中管理机制，该机制将软件以包的形式存储在仓库中，方便用户搜索、安装和管理软件包。

红帽系操作系统软件管理分类：yum、rpm、source、bin。

## RPM软件包管理

> 软件名称 版本号(主版本、次版本、修订号) 操作系统 cpu平台 
>
> 操作系统:el6 el5 fedora suse debin ubuntu 
>
> cpu平台:i386 486 586 686 表示32位软件 
>
> ​	x86_64 表示64为软件 
>
> ​	noarch 表示32,64通用

一、RPM软件包分为两种：二进制包和源码包。

1、二进制包中封装的是编译后生成的可执行文件，类似于Windows系统的.exe文件，可使用rpm命令直接安装。

2、源码包中封装的是源代码，在安装之前需先安装源码包以生成源码，再对源码进行编译生成后缀名为.rpm的RPM包，之后才能安装软件本身。

3、后缀.rpm表示二进制包，后缀.src.rpm表示源码包
**RPM工具**

```
[root@localhost Packages]#rpm -ivh  wget-1.14-18.el7_6.1.x86_64.rpm
#使用-i:install  -v:显示安装信息  -h:百分比
```

使用rpm安装软件包：
安装软件包：rpm -ivh 软件包
查询是否安装完成：rpm -q 软件包
查询软件安装路径：rpm -ql 软件名称
查询软件名称：rpm -qa |grep ftp
查询软件信息：rpm -qi 软件名称

删除软件包：rpm -evh wget-1.14-18.el7_6.1.x86_64（卸载的是软件，不用加.rpm后缀）
强制删除软件包（不安装依赖）：rpm -ivh 软件包 --force
删除软件包（不检查依赖）：rpm -ivh 软件包 --nodeps

查询某一个文件是哪个软件产生的：rpm -qf /etc/passwd

软件卸载：rpm -e 软件名称

查询软件的配置文件：rpm -qc 软件名称

​	--force 在安装的时候用(强制安装) 

​	--nodeps 在卸载的时候用(卸载的时候不检查依赖关系)

无法自动安装依赖性软件包

出现下载错误信息：可能出现的原因，/etc/yum.repos.d 文件配置错误

## YUM工具

基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且依次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

```
1、安装	install
2、查询	list和info。
yum list用于列出一个或一组软件包；yum info用于显示关于软件包或组的详细信息
```

使用yum安装软件包

```
语法：yum  -y  install   软件包
重新安装：yum  -y  reinstall  软件包
更新：yum  -y  update  软件包
查询软件包：yum  list   软件包
卸载软件包：yum  -y  remove   软件包
查看文件属于哪个软件：yum	provides vim	
使用install把对应的软件包下载下来，即可以使用vim
```

```
清理Yum缓存:
[root@localhost ~]# yum clean all
缓存软件包信息:
提高搜索/安装软件的速度
[root@localhost ~]# yum makecache
查询yum源信息:
[root@localhost ~]# yum repolist
查找软件:
[root@localhost ~]# yum search mysql
此命令会搜索到系统已经安装和yum源里没有安装的软件信息,可以用他简单测试yum是否好用
查看软件依赖性关系:
[root@localhost ~]# yum deplist
查看文件属于哪个软件
[root@localhost ~]# yum provides 想要安装的服务名
查看系统已经安装好的软件和没有安装的软件:
[root@localhost ~]# yum list
查看系统已经安装好的软件组和没有安装的软件组:
[root@localhost ~]# yum grouplist
查看软件组包含的具体软件：
[root@localhost ~]# yum groupinfo
安装软件组:
[root@localhost ~]# yum groupinstall ‘软件组名称’
如果软件或者软件组名称内有空格，要给空格转义或者加引号
安装软件:
[root@localhost ~]# yum install 软件名称
[root@localhost ~]# yum install mysql mysql-server -y
-y跳过确认提示直接安装
重装：
[root@localhost ~]# yum reinstall 软件名
卸载软件:
[root@localhost ~]# yum erase mysql-server
[root@localhost ~]# yum remove mysql-server
```

### 配置本地YUM源

介绍：基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动依赖处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐的一次次下载，安装。

**yum 源的配置与更新**

（1）在etc/yum/repos.d目录下，找到CentOS6-Base.repo文件替换（即替换CentOS6-Base.repo文件中的所有内容）。

或者直接下载替换即可：

```
#配置阿里云源
[root@localhost ~]#wget -O /etc/yum.repos.d/CentOS-Base.repo
http://mirrors.aliyun.com/repo/Centos-7.repo
```

[centos-vault安装包下载_开源镜像站-阿里云 (aliyun.com)](https://mirrors.aliyun.com/centos-vault/)

[清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/)

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=https://mirrorlist.centos.org/centos/$releasever/os/$basearch/
baseurl=https://mirrors.aliyun.com/centos-vault/7.4.1708/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
#released updates 
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos-vault/$releasever/updates/$basearch/
baseurl=https://mirrors.aliyun.com/centos-vault/7.4.1708/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos-vault/$releasever/extras/$basearch/
baseurl=https://mirrors.aliyun.com/centos-vault/7.4.1708/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos-vault/$releasever/centosplus/$basearch/
baseurl=https://mirrors.aliyun.com/centos-vault/7.4.1708/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib&infra=$infra
#baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos-vault/$releasever/contrib/$basearch/
baseurl=https://mirrors.aliyun.com/centos-vault/7.4.1708/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

连续执行以下命令

```
yum install all
yum makecache
yum -y update
```

## 源码包管理

缺点：配置复杂

```
1.解压缩 tar -xf
2.配置	./configure 用户	组	功能模块
3.编译	make
4.安装	make install
```

#### 获取源码包

来源：官方网站（如：Apache:www.apache.org Nginx:www.nginx.org
Tengine:[The Tengine Web Server (taobao.org)](https://tengine.taobao.org/)）

在宿主机的源码包上传到VMare机器

```
#查找rz命令需要什么软件支持
#rz：将window的文件拷贝到Linux			sz：将Linux的文件拷贝到Windows
[root@localhost ~]# yum provides rz		
[root@localhost ~]# yum install -y lrzsz-0.12.20-36.el7.x86_64
#使用rz命令，打开在弹窗的文件包，即上传到当下目录
[root@localhost ~]# rz
[root@localhost ~]# yum clean all
 [root@localhost ~]# yum makecache
[root@localhost ~]# yum -y install gcc make zlib-devel pcre pcre-devel openssl-devel
[root@localhost ~]# useradd www
[root@localhost ~]# tar xvf tengine-3.0.0.tar.gz
[root@localhost ~]# cd tengine-3.0.0/
[root@localhost tengine-3.0.0]#
./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_sub_module --with-http_ssl_module --with-pcre
[root@localhost tengine-3.0.0]# make
[root@localhost tengine-3.0.0]# make install
[root@localhost tengine-3.0.0]# lsof -i:80
[root@localhost tengine-3.0.0]# yum provides lsof
[root@localhost tengine-3.0.0]# yum install lsof-4.87-6.el7.x86_64
[root@localhost tengine-3.0.0]# lsof -i:80
[root@localhost tengine-3.0.0]# /usr/local/nginx/sbin/nginx
[root@localhost tengine-3.0.0]# lsof -i:80
```

# 任务计划

## 一次性调度执行 at

语法：at

案例：now +5min
teatime tomorrow(teatime is 16:00)
noon +4 days
5pm august 3 2029
4:00 2021-11-27

```
[root@localhost ~]# at now+1min
at> useradd aaa
at> <EOT>
job 3 at Sun Jul  4 11:41:00 2021
[root@localhost ~]# id aaa
uid=1006(aaa) gid=1008(aaa) 组=1008(aaa)
```

## 循环调度执行cron

用于设置周期性被执行的指令，该命令从标准输入设备读取指令，并将其存放于“crontab”文件中，以供以后读取和执行。想让计算机干什么，就往里面写指令，不执行的就删掉。

1、程序不执行问题，查看进程状态

```
[root@localhost ~]# systemctl status crond.service 
[root@localhost ~]# ps aux |grep crond
```

计划任务存储位置 /var/spool/cron

```
安装软件
[root@localhost ~]# yum -y install crontabs
启动服务
rhel5/6:
[root@localhost ~]# /etc/init.d/crond status
[root@localhost ~]# /etc/init.d/crond start
rhel7:
[root@localhost ~]# systemctl start crond.service
[root@localhost ~]# systemctl status crond.service
[root@localhost ~]# systemctl enable crond.service
开机启动(rhel5/6)
[root@localhost ~]# chkconfig crond on
创建计划任务：用户级别的计划任务
[root@localhost ~]# crontab -u 用户 -e
-u 指定用户 默认不写就是root
[root@localhost ~]# crontab -e
配置分两部分 拿空格分开
第一部分:时间
	分钟  小时  日   月    周
范围 0-59 0-23 1-31 1-12 0-7
上面的时间范围可以查看man手册: 
[root@localhost ~]# man 5 crontab
各种时间写法：
5 10 * * *
5 10 8 * *
1 5 7 * 5
1,5,9 * * * *
8-12 * * * *
5-20,40 * * * *
8-12,20-25 * * * *
*/5 * * * *

ps: * 表示每...
    , 取不同的时间点
    - 表示范围
    */5 每5分钟
第二部分:动作
把上面规定的时间要执行的命令写在这里，当然包括脚本(最常用)，命令最好要写绝对路径
查看计划任务:两种方法
    1)[root@localhost ~]# crontab -l
    -u 用户名 查看某一个账户的计划任务
    2)[root@localhost ~]# cat /var/spool/cron/root
    计划任务删除:两种方法
    1)[root@localhost ~]# crontab -r -u wing
    -r 删除
    -u 指定用户
    [root@localhost ~]# crontab -e -u tom
    2)[root@localhost ~]# rm -f /var/spool/cron/root
计划任务的权限控制
    [root@localhost ~]# cat /etc/cron.deny
    如果这个文件存在，凡是写到这个文件里面的账户不允许执行crontab命令
    [root@localhost ~]# cat /etc/cron.allow
    如果这个文件存在，没有写到这个文件里面的账户不允许执行crontab命令
    如果有allow文件，那不管deny是否存在，都是只允许allow文件里面的用户
```

# 日志管理

## 路由

```
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.101.2   0.0.0.0         UG    100    0        0 ens33
192.168.101.0   0.0.0.0         255.255.255.0   U     100    0        0 ens33
[root@localhost ~]# ip r	//查看路由表
default via 192.168.101.2 dev ens33 proto dhcp metric 100
192.168.101.0/24 dev ens33 proto kernel scope link src 192.168.101.128 metric 100
[root@localhost ~]# ip r d default	//删除默认网关
[root@localhost ~]# ip r del 192.168.101.0/24	//删除静态路由
[root@localhost ~]# ip r add default via 192.168.101.2 dev ens33	//添加默认网关
[root@localhost ~]# ip r add 192.168.101.0/24 via 192.168.101.2 dev ens33	//添加静态路由
```

## 常见系统日志

```
/var/log/message:记录Linux操作系统常见的系统和服务错误信息
/var/log/boot.log:记录了系统在引导过程中发生的事件，就是Linux系统开机自检过程显示的信息
/var/log/lastlog :记录最后一次用户成功登陆的时间、登陆IP等信息(一般通过命令lastlog查看)
/var/log/secure : Linux系统安全日志,记录用户和工作组变坏情况、用户登陆认证情况
/var/log/btmp :记录Linux登陆失败的用户、时间以及远程IP地址
/var/log/wtmp:该日志文件永久记录每个用户登录、注销及系统的启动、停机的事件，使用1ast命令查看
```

常用是1、4点，遇到错误提示时，可以查看系统日志，上图的第一点使用
在/root目录下，tailf /var/log/messages # 实时查看系统日志

## 日志类型

auth pam 产生的日志 

authpriv ssh，ftp 等登录信息的验证信息 

cron 时间任务相关 

kern 内核 

lpr 打印 

mail 邮件 

mark(syslog)-rsyslog 服务内部的信息，时间标识 

news 新闻组 

user 用户程序产生的相关信息

## 日志的优先级

日志的优先级分为7种，级别代号0~7：
0 debug 有调试信息的，日志信息最多
1 info 一般信息的日志，最常用
2 notice 最具有重要性的普通条件的信息
3 warning 警告级别
4 err 错误级别，阻止某个功能或者模块不能正常工作的信息
5 crit 严重级别，阻止整个系统或者整个软件不能工作的信息
6 alert 需要立即修改的信息
7 emerg 内核崩溃等严重信息
none 什么都不记录

## 自定义日志

```
[root@localhost ~]#vim /etc/rsyslog.conf
日志对象(设备):你要对什么东东做日志
日志级别:级别越低，信息越多
日志文件:存储日志的文件
日志对象.日志级别 日志文件
. 大于或者等于后面指定的日志级别
.= 等于后面指定的日志级别
.! 非
例：
*.* /var/log/mylog
kern.err /var/log/kernel.log
*.info;mail.none /var/log/big.log
mail.info /var/log/mail.log
cron.info;cron.!err /var/log/newcron
cron.info /var/log/newcron
重启日志服务：
systemctl restart rsyslog
```

## 日志进程rsyslog

系统专职日志程序，处理大部分日志记录，操作系统有关的信息，如登录信息，程序启动关闭信息，错误信息。

任务一：rsyslog系统日志管理 关心的问题：哪类程序，产生什么的日志，放在什么地方

查看日志进程

```
[root@localhost ~]# ps aux | grep rsyslogd
root      20730  0.0  0.4 214460  4644 ?        Ssl  15:48   0:00 /usr/sbin/rsyslogd -n
root      69210  0.0  0.0 112812   976 pts/1    R+   17:02   0:00 grep --color=auto rsyslogd
```

动态查看日志文件的尾部：tail -f /var/log/messages 开启两个终端窗口

查看各种日志 /var/log

## rsyslog配置文件

```
[root@localhost log]# rpm -qc rsyslog      #查看配置文件
/etc/logrotate.d/syslog			#rsyslogd相关文件，定义级别（了解）
/etc/rsyslog.conf				#rsyslogd的主配置文件(告诉rsyslog进程什么日志，应该存放在哪)
/etc/sysconfig/rsyslog			#和日志办轮转（切割）相关
```

## 日志轮转logrotate

任务二：将大量的日志，分割管理，删除旧日志

简介
记录程序运行时的各种信息，通过日志可以分析用户的行为，记录运行轨迹，查找程序问题，但磁盘空间有限，记录的信息再重要也只能记录最后一段时间发生的事。为了节省空间和整理方便，日志文件需要经常按时间或大小等维度分成多份，删除时间久远

工作原理
按照配置进行轮转，配置文件有主配置文件和子配置文件夹（自定义，便于管理）
主文件：/etc/ logrotate.conf （决定每个日志文件如何轮转）
子文件夹：/etc/logrotate.d/*

```
[root@localhost log]# vi /etc/logrotate.conf
=================全局设置======================
weekly		//轮转的周期，一周轮转
rotate 4		//保留4份，当产生第五份的时候，自动删除最旧的一份
create		//轮转后创建新文件
dateext		//使用日期作为后缀
# compress		//是否压缩
include /etc/logrotate.d		//包含该目录下的子配置文件，程序运行，在读主										配置文件时，到子配置文件查看，读取
/var/log/wtmp {		//对某日志文件设置轮转的方法，优先级最高
	monthly		//一个月轮转一次
	minsize 1M		//最小达到1M才轮转，monthly and minsize
	create 0664 root utmp		//轮转后创建文件并设置权限
	rotate 1		//保留一份
}
/var/log/btmp {
	missingok		//丢失不提示
	monthly			//每月轮转一次
	create 0600 root utmp		//轮转后创建新文件，并设置权限
    rotate 1		//保留一份
}
```

```
[root@localhost log]# vim /etc/logrotate.d/yum	#子配置文件下的yum
/var/log/yum.log {
    missingok
    #notifempty
    #maxsize 30k
    #yearly
    daily			#自定义每天轮转一次
    rotate 3		#自定义保留三份
    create 0600 root root
}
```

```
#测试
[root@localhost log]# date			#查看当下时间
2021年 07月 09日 星期五 23:20:49 CST
[root@localhost log]# date 07092322		#修改系统时间（月日时分）
2021年 07月 09日 星期五 23:22:00 CST
[root@localhost log]# logrotate /etc/logrotate.conf	#手动轮转日志
[root@localhost log]# ls /var/log/yum.*				#查看日志生成
/var/log/yum.log  /var/log/yum.log-20210709
```

## 日志安全

```
[root@localhost log]# vim /etc/logrotate.d/syslog 
/etc/logrotate.d/messages
建议测试时先把/etc/logrotate.d/syslog中messages删除
#设置messages轮转
/var/log/messages {
 prerotate			#设置轮转前的动作
     chattr -a /var/log/messages     
 endscript			#动作结束

 daily
 create 0777 root root
 missingok
 rotate 3

 postrotate			#轮转之后的动作
    chattr +a /var/log/messages
 endscript
 }

[root@localhost log]# logrotate /etc/logrotate.conf	#手动轮转测试配置文件无误。

#为多个日志文件配置日志轮转
[root@localhost ~]#vim /etc/logrotate.d/syslog
/var/log/cron
/var/log/maillog
/var/log/secure
/var/log/spooler
{
    missingok
    sharedscripts
    postrotate
    	/bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
```

# 网络管理

## ip地址的划分

```
A类地址:范围从0-127，0是保留的并且表示所有IP地址，而127也是保留的地址，并且是用于测试环回用
的。因此A类地址的范围其实是从1-126之间。
如：10.0.0.1，第一段号码为网络号码，剩下的三段号码为本地计算机的号码。转换为2进制来说，一个A
类IP地址由1字节的网络地址和3字节主机地址组成，网络地址的最高位必须是“0”， 地址范围从0.0.0.1
到126.0.0.0。可用的A类网络有126个，每个网络能容纳1亿多个主机（2的24次方的-2主机数目）。
以子网掩码来进行区别：：255.0.0.0
127.0.0.0到127.255.255.255是保留地址，用做循环测试用的
B类地址：范围从128-191，如172.168.1.1，第一和第二段号码为网络号码，剩下的2段号码为本地计算
机的号码。转换为2进制来说，一个B类IP地址由2个字节的网络地址和2个字节的主机地址组成，网络地址
的最高位必须是“10”，地址范围从128.0.0.0到191.255.255.255。可用的B类网络有16382个，每个网
络能容纳6万多个主机 。(2的16次方-2)
以子网掩码来进行区别：255.255.0.0
169.254.0.0到169.254.255.255是保留地址。如果你的IP地址是自动获取IP地址，而你在网络上又没
有找到可用的DHCP服务器，这时你将会从169.254.0.0到169.254.255.255中临时获得一个IP地址。
C类地址：范围从192-223，如192.168.1.1，第一，第二，第三段号码为网络号码，剩下的最后一段号码
为本地计算机的号码。转换为2进制来说，一个C类IP地址由3字节的网络地址和1字节的主机地址组成，网
络地址的最高位必须是“110”。范围从192.0.0.0到223.255.255.255。C类网络可达209万余个，每个
网络能容纳254个主机。(2的8次方-2)
以子网掩码来进行区别： 255.255.255.0
D类地址：范围从224-239，D类IP地址第一个字节以“1110”开始，它是一个专门保留的地址。它并不指向
特定的网络，目前这一类地址被用在多点广播（Multicast）中。多点广播地址用来一次寻址一组计算
机，它标识共享同一协议的一组计算机。
224.0.0.0-239.255.255.255 组播地址
E类地址：范围从240-254，以“11110”开始，为将来使用保留。 全零（“0．0．0．0”）地址对应于当前
主机。全“1”的IP地址（“255．255．255．255”）是当前子网的广播地 址。
240.0.0.0-255.255.255.254 保留地址
```

## 子网掩码

> 就是为了区分ip地址的中的网络号和主机号的

例1：

```
ip地址: 202.197.119.110
若掩码为:255.255.255.0 求网络号和主机号
ip转换为2进制 1100 1010. 1100 0101. 0111 0111. 0110 1110
子网掩码2进制 1111 1111. 1111 1111. 1111 1111. 0000 0000
相与运算 1100 1010. 1100 0101. 0111 0111. 0000 0000 网络号
ip转换为2进制 1100 1010. 1100 0101. 0111 0111. 0110 1110
子网掩码取反 0000 0000. 0000 0000. 0000 0000. 1111 1111
相与运算 0000 0000. 0000 0000. 0000 0000. 0110 1110 主机号
```

```
ip 202.197.118.110 是否与上一个ip在同一网段? 求网络号，相同则同一网段
ip转换为2进制 1100 1010. 1100 0101. 0111 0110.0110 1110
求得网络号    1100 1010.1100 0101.0111 0110.0000 0000
网络号 不同，所以不再同一网络中
```

```
ip地址: 202.197.119.110
若掩码为:255.255.128.0 求网络号和主机号
ip转换为2进制 1100 1010. 1100 0101. 0111 0111. 0110 1110
子网掩码2进制 1111 1111. 1111 1111. 1000 0000. 0000 0000
相与运算 1100 1010. 1100 0101. 0000 0000. 0000 0000 网络号
主机号 0000 0000. 0000 0000. 0111 0111. 0110 1110 主机号
```

```
ip 202.197.118.110 是否与上一个ip再统一网段?
ip转换为2进制 1100 1010. 1100 0101. 0111 0110. 0110 1110
求得网络号    1100 1010.1100 0101.0000 0000. 0000 0000
同上一个ip在同一个网络中
所以判断两个ip是否在同一网络要看子网掩码的设置
```

## 网络接口名称规则

```
ifconfig eth0 //单独查看eth0
ifconfig //查看所有网卡
ip a //查看所有网卡
ip a s ens33	//单独查看ens33
ifconfig ens33 192.168.2.250/24 //会覆盖旧的IP
ifconfig ens33:0 192.168.2.251/24  //子网掩码可以不写
ifconfig ens33 up //启动网卡
ifup ens33	//启动网卡
ifconfig ens33 down	//关闭网卡
ifdown ens33	//关闭网卡
```

1、认识网卡；2、找到网卡文件；3、学会修改文件；4，多台服务器互通

```
[root@localhost ~]#/etc/sysconfig/network-scripts/	#查看网卡文件
[root@localhost ~]#systemctl status NetworkManager	#查看网络管理程序状态
[root@localhost ~]#systemctl status network			#重启网络
[root@localhost ~]#vi /etc/sysconfig/network-scripts/ifcfg-ens33
[root@localhost ~]# ip neigh   #查路由，网关
[root@localhost ~]# ss -tnl		#查端口服务
```

```
DEVICE=eth0
NAME="System eth0" //名称 可以不存在
BOOTPROTO=none //（none或static 静态获取 ）（dhcp 动态获取IP）
NM_CONTROLLED=no //关闭NetworkManager
ONBOOT=yes //开机启动
TYPE=Ethernet // 以太网类型
HWADDR=00:0c:29:8e:a5:d3 //MAC地址
IPADDR=172.16.80.252 //IP地址
NETMASK=255.255.0.0 //子网掩码
PREFIX=24 //子网掩码
NETWORK=172.16.0.0 //网络
GATEWAY=172.16.0.1 //网关
DNS1=172.16.0.1 //domain name server域名服务器
DNS2=114.114.114.114 //第2台DNS服务器
```

重启网络服务: 配置文件修改后必须重起网络服务

```
systemctl restart network //rhel7
/etc/init.d/network restart //rhel5/6
 service network restart //rhel5/6
```

```
最小化安装
1.为你的服务器配置root密码
2.配置IP地址
3.配置YUM源
4.关防火墙
    systemctl stop firewalld
    systemctl disable firewalld
    systemctl status firewalld
5.selinux
    查看selinux getenforce
    临时关闭	setenforce 0
    永久关闭	vi /etc/sysconfig/selinux
    SELINUX=disabled
6.安装常用程序
上传下载工具     系统状态   字符浏览器  下载工具  网络工具  自动补全
    yum install -y lrzsz sysstat elinks wget net-tools bash-completion vim
7.关机快照
```

## 网络测试工具

使用ping命令

> 用来测试主机之间网络的连通性。执行ping指令会使用ICMP传输协议，发出要求回应的信息，若 远端主机的网络功能没有问题，就会回应该信息，因而得知该主机运作正常。

（linux）-c数字：表示发送多少个包
（windows）-n数字：表示发送多少个包
-t：一直发送数据包

ping www.baidu.com -i 0.01 # ping百度，间隔时间为0.01s

常用：ping -c1000 -i 0.01 www.baidu.com

```
ping 参数 目标主机
参数 详解
-a Audible ping.
-A 自适应ping，根据ping包往返时间确定ping的速度；
-b 允许ping一个广播地址；
-B 不允许ping改变包头的源地址；
-c count ping指定次数后停止ping；
-d 使用Socket的SO_DEBUG功能；
-F flow_label 为ping回显请求分配一个20位的“flow label”，如果未设置，内核会为ping随机分
配；
-f 极限检测，快速连续ping一台主机，ping的速度达到100次每秒；
-i interval 设定间隔几秒发送一个ping包，默认一秒ping一次；
-I interface 指定网卡接口、或指定的本机地址送出数据包；
-l preload 设置在送出要求信息之前，先行发出的数据包；
千锋云计算学院
-L 抑制组播报文回送，只适用于ping的目标为一个组播地址
-n 不要将ip地址转换成主机名；
-p pattern 指定填充ping数据包的十六进制内容，在诊断与数据有关的网络错误时这个选项就非
常有用，如：“-p ff”；
-q 不显示任何传送封包的信息，只显示最后的结果
-Q tos 设置Qos(Quality of Service)，它是ICMP数据报相关位；可以是十进制或十六进制数，详
见rfc1349和rfc2474文档；
-R 记录ping的路由过程(IPv4 only)；
注意：由于IP头的限制，最多只能记录9个路由，其他会被忽略；
-r 忽略正常的路由表，直接将数据包送到远端主机上，通常是查看本机的网络接口是否有问题；
如果主机不直接连接的网络上，则返回一个错误。
-S sndbuf Set socket sndbuf. If not specified, it is selected to buffer not more than one
packet.
-s packetsize 指定每次ping发送的数据字节数，默认为“56字节”+“28字节”的ICMP头，一共是84
字节；
包头+内容不能大于65535，所以最大值为65507（linux:65507, windows:65500）；
-t ttl 设置TTL(Time To Live)为指定的值。该字段指定IP包被路由器丢弃之前允许通过的最大网段
数；
-T timestamp_option 设置IP timestamp选项,可以是下面的任何一个：
    'tsonly' (only timestamps)
    'tsandaddr' (timestamps and addresses)
    'tsprespec host1 [host2 [host3]]' (timestamp prespecified hops).
-M hint 设置MTU（最大传输单元）分片策略。
可设置为：
    'do'：禁止分片，即使包被丢弃；
    'want'：当包过大时分片；
    'dont'：不设置分片标志（DF flag）；
-m mark 设置mark；
-v 使ping处于verbose方式，它要ping命令除了打印ECHO-RESPONSE数据包之外，还打印其它
所有返回的ICMP数据包；
-U Print full user-to-user latency (the old behaviour).
Normally ping prints network round trip time, which can be different f.e. due to DNS
failures.
-W timeout 以毫秒为单位设置ping的超时时间；
-w deadline deadline；
```

使用traceroute 命令

> 通过traceroute我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径。当然每 次数据包由某一同样的出发点（source）到达某一同样的目的地(destination)走的路径可能会不 一样，但基本上来说大部分时候所走的路由是相同的。linux系统中，我们称之为traceroute,在 MS Windows中为tracert。 traceroute通过发送小的数据包到目的设备直到其返回，来测量其需 要多长时间。一条路径上的每个设备traceroute要测3次。输出结果中包括每次测试的时间(ms)和 设备的名称（如有的话）及其IP地址。

```text
traceroute www.baidu.com # 测试ping百度经过多少个网关
traceroute -m 10 www.baidu.com # 设置ping百度10个网关
traceroute -n www.baidu.com # 只显示ip地址 # 探测包使用基本UDP端口设置6888
traceroute -p 6888 www.baidu.com
#对外探测包响应时间为3秒
traceroute -w 3 www.baidu.com
```

```
traceroute [参数] [主机]

traceroute指令让你追踪网络数据包的路由途径，预设数据包大小是40Bytes，用户可另行设置

#traceroute [-dFlnrvx][-f<存活数值>][-g<网关>...][-i<网络界面>][-
m<存活数值>][-p<通信端口>][-s<来源地址>][-t<服务类型>][-w<超时秒数>][主机名称或IP地址]
[数据包大小]

-d 使用Socket层级的排错功能。
-f 设置第一个检测数据包的存活数值TTL的大小。
-F 设置勿离断位。
-g 设置来源路由网关，最多可设置8个。
-i 使用指定的网络界面送出数据包。
-I 使用ICMP回应取代UDP资料信息。
-m 设置检测数据包的最大存活数值TTL的大小。
-n 直接使用IP地址而非主机名称。
-p 设置UDP传输协议的通信端口。
-r 忽略普通的Routing Table，直接将数据包送到远端主机上。
-s 设置本地主机送出数据包的IP地址。
-t 设置检测数据包的TOS数值。
-v 详细显示指令的执行过程。
-w 设置等待远端主机回报的时间。
-x 开启或关闭数据包的正确性检验。
```

Traceroute的工作原理

> Traceroute程序的设计是利用ICMP及IP header的TTL（Time To Live）栏位（field）。首先， traceroute送出一个TTL是1的IP datagram（其实，每次送出的为3个40字节的包，包括源地址， 目的地址和包发出的时间标签）到目的地，当路径上的第一个路由器（router）收到这个 datagram时，它将TTL减1。此时，TTL变为0了，所以该路由器会将此datagram丢掉，并送回一 个「ICMP time exceeded」消息（包括发IP包的源地址，IP包的所有内容及路由器的IP地址）， traceroute 收到这个消息后，便知道这个路由器存在于这个路径上，接着traceroute 再送出另一 个TTL是2 的datagram，发现第2 个路由器...... traceroute 每次将送出的datagram的TTL 加1来 发现另一个路由器，这个重复的动作一直持续到某个datagram 抵达目的地。当datagram到达目 的地后，该主机并不会送回ICMP time exceeded消息，因为它已是目的地了，那么traceroute如 何得知目的地到达了呢？ Traceroute在送出UDP datagrams到目的地时，它所选择送达的port number 是一个一般应用程 序都不会用的号码（30000 以上），所以当此UDP datagram 到达目的地后该主机会送回一个 「ICMP port unreachable」的消息，而当traceroute 收到这个消息时，便知道目的地已经到达 了。所以traceroute 在Server端也是没有所谓的Daemon 程式。 Traceroute提取发 ICMP TTL到期消息设备的IP地址并作域名解析。每次 ，Traceroute都打印出 一系列数据,包括所经过的路由设备的域名及 IP地址,三个包每次来回所花时间。

# 文件服务

## Ftp 介绍 

文件传输协议（File Transfer Protocol，FTP），基于该协议FTP客户端与服务端可以实现共享文 件、上传文件、下载文件。 FTP 基于TCP协议生成一个虚拟的连接，主要用于控制FTP连接信息， 同时再生成一个单独的TCP连接用于FTP数据传输。用户可以通过客户端向FTP服务器端上传、下 载、删除文件，FTP服务器端可以同时提供给多人共享使用。 

FTP服务是Client/Server（简称C/S）模式，基于FTP协议实现FTP文件对外共享及传输的软件称之 为FTP服务器源端，客户端程序基于FTP协议，则称之为FTP客户端，FTP客户端可以向FTP服务器 上传、下载文件。 

目前主流的FTP服务器端软件包括：Vsftpd、ProFTPD、PureFTPd、Wuftpd、Server-U FTP、 FileZilla Server等软件，其中Unix/Linux使用较为广泛的FTP服务器端软件为Vsftpd 。 

## Ftp 传输模式介绍 

FTP基于C/S模式，FTP客户端与服务器端有两种传输模式，分别是FTP主动模式、FTP被动模式， 主被动模式均是以FTP服务器端为参照。 

FTP主动模式：客户端从一个任意的端口N（N>1024）连接到FTP服务器的port 21命令端口，客 户端开始监听端口N+1，并发送FTP命令“port N+1”到FTP服务器，FTP服务器以数据端口（20）连 接到客户端指定的数据端口（N+1）。 

FTP被动模式：客户端从一个任意的端口N（N>1024）连接到FTP服务器的port 21命令端口，客 户端开始监听端口N+1，客户端提交 PASV命令，服务器会开启一个任意的端口（P >1024），并 发送PORT P命令给客户端。客户端发起从本地端口N+1到服务器的端口P的连接用来传送数据。 

企业实际环境中，如果FTP客户端与FTP服务端均开放防火墙，FTP需以主动模式工作，这样只需 要在FTP服务器端防火墙规则中，开放20、21端口即可。 

## Vsftp 服务器简介 

非常安全的FTP服务进程（Very Secure FTP daemon，Vsftpd），Vsftpd在Unix/Linux发行版中 最主流的FTP服务器程序，优点小巧轻快，安全易用、稳定高效、满足企业跨部门、多用户的使用 （1000用户）等。 

Vsftpd基于GPL开源协议发布，在中小企业中得到广泛的应用，Vsftpd可以快速上手，基于Vsftpd 虚拟用户方式，访问验证更加安全。Vsftpd还可以基于MYSQL数据库做安全验证，多重安全防 护。 

## Vsftp 的登陆类型 

VSFTP提供了系统用户、匿名用户、和虚拟用户三种不同的登陆方式。所有的虚拟用户会映射成一个系 统用户，访问时的文件目录是为此系统用户的家目录；匿名用户也是虚拟用户，映射的系统用户为ftp， 详细信息可以通过man vsftpd.conf查看 

## Vsftp 安装配置

```
#安装 epel 源
[root@localhost ~]# yum -y install epel-release
# 安装 vsftpd 及相关依赖
[root@localhost ~]# yum -y install vsftpd* pam* db4*
vsftpd： ftp软件 
pam：认证模块 
DB4：支持文件数据库
```

### vsftpd 配置文件说明

| 配置文件                           | 作用                              |
| ---------------------------------- | --------------------------------- |
| /etc/vsftpd/vsftpd.conf            | vsftpd的核心配置文件              |
| /etc/vsftpd/ftpusers               | 用于指定哪些用户不能访问FTP服务器 |
| /etc/vsftpd/user_list              | 指定允许使用vsftpd的用户列表文件  |
| /etc/vsftpd/vsftpd_conf_migrate.sh | 是vsftpd操作的一些变量和设置脚本  |
| /var/ftp/                          | 默认情况下匿名用户的根目录        |

### vsftpd 修改配置前备份配置文件

```
[root@localhost ~]# cd /etc/vsftpd/
[root@localhost vsftpd]# ls
ftpusers  user_list  vsftpd.conf  vsftpd_conf_migrate.sh
[root@localhost vsftpd]# cp vsftpd.conf{,.bak}
[root@localhost vsftpd]# ls
ftpusers  user_list  vsftpd.conf  vsftpd.conf.bak  vsftpd_conf_migrate.sh
```

### vsftpd 配置匿名用户 

1、编辑配置文件

```
[root@localhost vsftpd]# cat vsftpd.conf | grep -v ^#
[root@localhost vsftpd]# vim vsftpd.conf

#:%d删除所有内容
write_enable=YES
anon_umask=022

anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

2、常用的匿名FTP配置项

```
anonymous_enable=YES 			# 是否允许匿名用户访问
anon_umask=022 					# 匿名用户所上传文件的权限掩码
anon_root=/var/ftp 				# 设置匿名用户的FTP根目录
anon_upload_enable=YES 			# 是否允许匿名用户上传文件
anon_mkdir_write_enable=YES 	# 是否允许匿名用户允许创建目录
anon_other_write_enable=YES 	# 是否允许匿名用户有其他写入权
（改名，删除，覆盖）
anon_max_rate=0 				# 限制最大传输速率（字节/秒）0为
无限制
```

开启 vsftp 服务

```
[root@localhost vsftpd]# systemctl start vsftpd
[root@localhost vsftpd]# netstat -lnpt |grep vsftpd
```

修改权限实现上传

```
[root@localhost vsftpd]# cd /var/ftp/
[root@localhost ftp]# ls
pub
[root@localhost ftp]# chown -R ftp.ftp pub/
```

重点：改变根目录的属主，如果不改变的话，只能访问，其他权限不能生效。因为我们是以ftp用 户的身份访问的，而pub默认的属主属组是root。 

注意： 修改完配置之后需要重启完服务才能生效 还需要从新从客户端登陆，否则修改后的配置看不到效果。

### vsftp 配置本地（系统）用户

```
[root@localhost ftp]# useradd zhangsan
[root@localhost ftp]# useradd lisi
[root@localhost ftp]# echo "123456" | passwd --stdin zhangsan
[root@localhost ftp]# echo "123456" | passwd --stdin lisi
```

修改配置文件

```
[root@localhost ftp]# vi /etc/vsftpd/vsftpd.conf
local_enable=YES
local_umask=077
chroot_local_user=YES
allow_writeable_chroot=YES
userlist_deny=NO

write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

常用的本地用户FTP配置项

```
local_enable=YES # 是否允许本地系统用户访问
local_umask=022 # 本地用户所上传文件的权限掩码
local_root=/var/ftp # 设置本地用户的FTP根目录
chroot_list_enable=YES # 表示是否开启chroot的环境，默认
没有开启
chroot_list_file=/etc/vsftpd/chroot_list # 表示写
在/etc/vsftpd/chroot_list文件里面的用户是不可以出chroot环境的。默认是可以的。
Chroot_local_user=YES # 表示所有写
在/etc/vsftpd/chroot_list文件里面的用户是可以出chroot环境的，和上面的相反。
local_max_rate=0 # 限制最大传输速率（字节/秒）0为
无限制
```

添加用户到白名单

```
[root@localhost ftp]# vi /etc/vsftpd/user_list
zhangsan
lisi
```

重启服务

```
 systemctl restart vsftpd
```

### vsftp 配置虚拟用户

建立虚拟 FTP 用户的帐号

```
useradd -s /sbin/nologin vu
```

创建虚拟用户文件

```
[root@localhost ~]# cd /etc/vsftpd/
[root@localhost vsftpd]# vi user
wangwu
12345
maliu
12345
```

- 基数行代表用户名，偶数行代表密码

创建数据文件

- 通过 db_load 工具创建出 Berkeley DB 格式的数据库文件

```
[root@localhost vsftpd]# db_load -T -t hash -f user user.db
```

- -f 指定数据原文件 
- -T 允许非Berkeley DB的应用程序使用文本格式转换的DB数据文件 
- -t hash 读取文件的基本方法

建立支持虚拟用户的PAM认证文件

```
vi /etc/pam.d/vsftpd.vu
auth required /lib64/security/pam_userdb.so db=/etc/vsftpd/user
account required /lib64/security/pam_userdb.so db=/etc/vsftpd/user
```

- 对应刚才生成 user.db 的文件

修改配置文件

```
[root@localhost vsftpd]# vi vsftpd.conf
write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=YES
userlist_enable=YES
tcp_wrappers=YES
allow_writeable_chroot=YES
local_enable=YES
local_umask=077
chroot_local_user=YES

pam_service_name=vsftpd.vu

guest_enable=YES
guest_username=vu
virtual_use_local_privs=YES
user_config_dir=/etc/vsftpd/user_dir
```

常用的全局配置项

```
listen=YES # 是否以独立运行的方式监听服务
千锋云计算学院
7、为用户建立独立的配置目录及文件
8、创建虚拟用户数据存放目录
listen_address=192.168.4.1 # 设置监听FTP服务的IP地址
listen_port=21 # 设置监听FTP服务的端口号
write_enable=YES # 是否启用写入权限（上传，删除文
件）
download_enable＝YES # 是否允许下载文件
dirmessage_enable=YES # 用户切换进入目录时显
示.message文件
xferlog_enable=YES # 启用日志文件，记录
到/var/log/xferlog
xferlog_std_format=YES # 启用标准的xferlog日志格式，禁
用此项将使用vsftpd自己的格式
connect_from_port_20=YES # 允许服务器主动模式（从20端口建
立数据连接）
pasv_enable=YES # 允许服务器被动模式
pasv_max_port=24600 # 设置被动模式服务器的最大端口号
pasv_min_port=24500 # 设置被动模式服务器的最小端口号
pam_service_name=vsftpd # 用户认证的PAM文件位置
（/etc/pam.d/vsftpd.vu）
userlist_enable=YES # 是否启用user_list列表文件
userlist_deny=YES # 是否禁用user_list中的用户
max_clients=0 # 限制并发客户端连接数
max_per_ip=0 # 限制同一IP地址的并发连接数
tcp_wrappers=YES # 是否启用tcp_wrappers主机访问
控制
chown_username=root # 表示匿名用户上传的文件的拥有人
是root，默认关闭
ascii_upload_enable=YES # 表示是否允许用户可以上传一个二
进制文件，默认是不允许的
ascii_download_enable=YES # 这个是代表是否允许用户可以下载
一个二进制文件，默认是不允许的
nopriv_user=vsftpd # 设置支撑Vsftpd服务的宿主用户为
手动建立的Vsftpd用户
async_abor_enable=YES # 设定支持异步传输功能
ftpd_banner=Welcome to Awei FTP servers # 设定Vsftpd的登陆标语
guest_enable=YES # 设置启用虚拟用户功能
guest_username=ftpuser # 指定虚拟用户的宿主用户
virtual_use_local_privs=YES # 设定虚拟用户的权限符合他们的宿
主用户
user_config_dir=/etc/vsftpd/vconf # 设定虚拟用户个人Vsftp的配置文件存放路径
```

为用户建立独立的配置目录及文件

```
[root@localhost vsftpd]# mkdir /etc/vsftpd/user_dir
[root@localhost vsftpd]# cd /etc/vsftpd/user_dir
[root@localhost user_dir]# vi wangwu
local_root=/etc/vsftpd/data # 虚拟用户数据的存放路径
```

创建虚拟用户数据存放目录

```
[root@localhost user_dir]# cd ..
[root@localhost vsftpd]# mkdir data
[root@localhost vsftpd]# chmod 777 data/
```

重启服务

```
systemctl restart vsftpd
```

## Ftp 客户端 lftp

### lftp 介绍 

lftp命令 是一款优秀的文件客户端程序，它支持ftp、SETP、HTTP和FTPs等多种文件传输协议。 lftp支持tab自动补全，记不得命令双击tab键，就可以看到可能的选项了。

### lftp 语法

```
lftp(选项)(参数)
```

### lftp 选项

```
-f：指定lftp指令要执行的脚本文件；
-c：执行指定的命令后退出；
--help：显示帮助信息；
--version：显示指令的版本号。
```

### lftp 参数

- 站点：要访问的站点的 ip 地址或者域名。

### lftp 使用实例

#### 登录ftp

```
lftp 用户名:密码@ftp地址:传送端口（默认21）
```

- 也可以先不带用户名登录，然后在接口界面下用login命令来用指定账号登录，密码不显示。

#### 查看文件与改变目录

```
ls
cd # 对应ftp目录
```

#### 下载 

- get当然是可以的，还可以：

```
mget -c *.pdf # 把所有的pdf文件以允许断点续传的方式下载。
mirror aaa/ # 将aaa目录整个的下载下来，子目录也会自动复制。
pget -c -n 10 file.dat # 以最多10个线程以允许断点续传的方式下载file.dat，可以通过
设置pget:default-n的值而使用默认值。
```

#### 上传 

- 同样的put、mput都是对文件的操作，和下载类似。

```
mirror -R 本地目录名
```

- 将本地目录以迭代（包括子目录）的方式反向上传到ftp site。

#### 模式设置

```
set ftp:charset gbk
```

- 远程ftp site用gbk编码，对应的要设置为utf8,只要替换gbk为utf8即可。

```
set file:charset utf8
```

- 本地的charset设定为utf8,如果你是gbk，相应改掉。

```
set ftp:passive-mode 1
```

- 使用被动模式登录，有些site要求必须用被动模式或者主动模式才可以登录，这个开关就是设置这 个的。0代表不用被动模式。 

#### 书签 

- 其实命令行也可以有书签，在lftp终端提示符下：

```
bookmark add ustc
```

- 就可以把当前正在浏览的ftp site用ustc作为标签储存起来。以后在shell终端下，直接 lftp ustc 就可以自动填好用户名和密码，进入对应的目录了。

```
bookmark edit
```

- 会调用编辑器手动修改书签。当然，也可以看到，这个书签其实就是个简单的文本文件。密码，用 户名都可以看到。 

#### 配置文件

```
vim /etc/lftp.conf
```

一般添加这几行：

```
set ftp:charset gbk
set file:charset utf8
set pget:default-n 5
```

- 这样，就不用每次进入都要打命令了。其他的 set 可以自己 tab 然后 help 来看

## NAS 存储

### NAS 介绍

- NAS 指 Network Area Storage，网络连接存储(NAS)是连接到TCP/IP网络(通常是以太网)的文件级 数据存储设备。它通常使用网络文件系统(NFS)或CIFS协议，但也可以使用其他选项，如HTTP。它 一般是将本地的存储空间共享给其他主机使用，一般通过 C/S 架构实现通信。它实现的是文件级 别的共享，计算机通常将共享的设别识别为一个文件系统，其文件服务器会管理锁以实现并发访 问。常见的 NAS 有 NFS 和 CIFS。 
- NAS在操作系统中显示为共享文件夹。工作人员像访问网络上的其他文件一样访问NAS中的文件。 NAS依赖于局域网运行，如果局域网出现故障，那么NAS服务将中断。 
- NAS通常不像基于块存储的SAN速度那么快，但高速局域网可以克服大多数性能和延迟问题

### SAN 介绍

- SAN 指 Storage Area Network，它将传输网络模拟成 SCSI 总线来使用，每一个主机的网卡相当 于 SCSI 总线中的 initiator，服务器相当于一个或多个 target，它需要借助客户端和服务端的 SCSI 驱动，通过 FC 或 TCP/IP 协议封装 SCSI 报文。它实现的是块级别的共享，通常被识别为一个块设 备，但是需要借助专门的锁管理软件才能实现多主机并发访问。 
- SAN是用于整合块级存储的专用高性能网络。网络将存储设备、交换机和主机互连。高端企业存储 区域网络(SAN)还可能包括SAN导向器级交换机，以实现更高性能和更高效的容量使用 服务器使用主机总线适配器(HBA)连接到SAN结构。服务器将SAN标识为本地连接存储，因此多台 服务器可以共享一个存储池。
- SAN不依赖局域网，并通过直接从连接的服务器卸载数据来减轻本地 网络的压力。

### NAS 与 SAN 的区别

- Fabric。NAS使用TCP/IP网络，最常见的是以太网。传统SAN通常运行在高速光纤通道网络上，尽 管更多的SAN采用基于IP的光纤架构，这是因为光纤通道的成本和复杂性。高性能仍然是SAN的要 求，基于闪存的光纤协议有助于缩小光纤通道速度和IP速度之间的差距。 
- 数据处理。两种存储体系结构处理数据的方式不同：NAS处理基于文件的数据，而SAN处理块数 据。这个故事并不那么简单：NAS可以使用全局命名空间，SAN可以访问专门的SAN文件系统。全 局命名空间聚合多个NAS文件系统以呈现统一视图。SAN文件系统使服务器能够共享文件。在SAN 架构中，每台服务器都维护一个专用的非共享LAN。 SAN文件系统允许服务器通过对同一LAN上 的服务器提供文件级访问来安全共享数据。 
- 协议。NAS通过电缆直接连接以太网到以太网的交换机。NAS可以使用多种协议与服务器连接，包 括NFS、SMB/CIFS、HTTP。在SAN方面，服务器使用SCSI协议与SAN磁盘驱动器设备进行通信。 网络是使用SAS/SATA结构或将映射层映射到其他协议(例如光纤通道协议(FCP)，通过光纤通道映射 SCSI或通过TCP/IP映射SCSI形成的。 
- 性能。对于需要高速流量的环境，例如高交易数据库和电子商务网站，SAN的性能更高。由于NAS 速度较慢的文件系统层，NAS通常具有较低的吞吐量和较高的延迟，但高速网络可弥补NAS内部的 性能损失。 
- 可扩展性。入门级和NAS设备的可扩展性不高，但高端NAS系统使用集群或横向扩展节点扩展到 PB级。相反，可扩展性是购买SAN的主要驱动因素。其网络架构使管理员能够在扩展或扩展配置 中扩展性能和容量。 
- 价格。虽然高端NAS的价格会高于入门级SAN，但通常NAS的购买和维护成本较低。NAS设备与存 储区域网络相比，硬件和软件管理组件更少。行政费用也计入比较因素中。在复杂堆栈的基础上， 使用FC SAN管理SAN更为复杂。其经验法则是将购买成本的10到20倍的费用作为年度维护计算。 
- 易于管理。在一对一的比较中，NAS赢得了管理竞赛的便利。该设备可轻松插入局域网并提供简化 的管理界面。SAN需要比NAS设备更多的管理时间。部署通常需要对数据中心进行物理更改，而持 续管理通常需要专门的管理人员。对于SAN来说，例外的是更多的NAS设备不共享公共管理控制 台。

### NAS 和 SAN 使用案例

#### NAS：当需要整合、集中和共享时 

- 文件存储和共享。这是NAS在中小企业和企业远程办公室的主要用例。单个NAS设备允许IT整合多 个文件服务器，简化管理，节省空间和能源。 
- 活动档案。长期存档最好存储在成本比较低廉的存储介质上，如磁带或基于云计算的冷存储库。 NAS是搜索和访问活跃存档的理想选择，而高容量NAS可以替代大型磁带库进行存档。 
- 大数据。企业对于大数据有多种选择：横向扩展NAS、分布式JBOD节点、全闪存阵列、基于对象 的存储。横向扩展NAS适用于处理大型文件、ETL(提取，转换，加载)、智能数据服务(如自动分层 和分析)。NAS也是大型非结构化数据(如视频监控和流媒体)以及后期制作存储的理想选择。 
- 虚拟化。并非每个用户都在使用NAS来进行虚拟化网络销售，但用例正在增长，VMware和HyperV都支持NAS上的数据存储。当企业尚未拥有SAN时，这是新型或小型虚拟化环境的流行选择。 
- 虚拟桌面界面(VDI)。中档和高端NAS系统提供支持VDI的本机数据管理功能，如快速桌面克隆和重 复数据删除。 千锋云计算学院 

#### SAN：当需要加速，扩展和保护时

- 数据库和电子商务网站。通用文件服务或NAS可用于较小的数据库，但高速事务环境需要SAN的高 I/O处理速度和非常低的延迟。这使SAN非常适合企业数据库和高流量电子商务网站。 
- 快速备份。服务器操作系统将SAN视为附加存储，从而实现对SAN的快速备份。由于服务器直接备 份到SAN，备份流量不会通过LAN传输。这可以在不增加以太网负载的情况下实现更快速的备份。 
- 虚拟化。NAS支持虚拟化环境，但SAN更适合大规模和/或高性能部署。存储区域网络可以在虚拟 机和虚拟化主机之间快速传输多个I/O流，并且拥有高扩展性支持动态处理。 
- 视频编辑。视频编辑应用程序需要非常低的延迟和非常高的数据传输速率。SAN提供这种高性能， 因为它直接连接到视频编辑桌面客户端，无需额外的服务器层。视频编辑环境需要第三方SAN分布 式文件系统和每节点负载均衡控制。 

在NFS，FTP，SAMBA中，其中下载速度或者说性能最好的是NFS，其次FTP，最后SAMBA 

### NFS 服务器和 samba 服务器介绍 

#### NFS 介绍 

- NFS 全称是 Network FileSystem，NFS 和其他文件系统一样，是在 Linux 内核中实现的，因此 NFS 很难做到与 Windows 兼容。NFS 共享出的文件系统会被客户端识别为一个文件系统，客户端 可以直接挂载并使用。 
- NFS 的实现使用了 RPC（Remote Procedure Call） 的机制，远程过程调用使得客户端可以调用 服务端的函数。由于有 VFS 的存在，客户端可以像使用其他普通文件系统一样使用 NFS 文件系 统，由操作系统内核将 NFS 文件系统的调用请求通过 TCP/IP 发送至服务端的 NFS 服务，执行相 关的操作，之后服务端再讲操作结果返回客户端。

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304154808936.webp)

- NFS 文件系统仅支持基于 IP 的用户访问控制，NFS 是在内核实现的，因此 NFS 服务由内核监听在 TCP 和 UDP 的 2049 端口，对于 NFS 服务的支持需要在内核编译时选择。它同时还使用了几个用 户空间进程用于访问控制，用户映射等服务，这些程序由 nfs-utils 程序包提供。 
- RPC 服务在 CentOS 6.5 之后改名为 portmapper，它监听在 TCP/UDP 的 111 端口，其他基于 RPC 的服务进程需要监听时，先像 RPC 服务注册，RPC 服务为其分配一个随机端口供其使用。客 户端在请求时，先向 RPC 服务请求对应服务监听的端口，然后再向改服务发出调用请求。 

#### Samba 简介  

- Samba是在Linux和UNIX系统上实现SMB协议的一个免费软件，由服务器及客户端程序构成。 SMB（Server Messages Block，信息服务块）是一种在局域网上共享文件和打印机的一种通信协 议，它为局域网内的不同计算机之间提供文件及打印机等资源的共享服务。SMB协议是客户机/服 务器型协议，客户机通过该协议可以访问服务器上的共享文件系统、打印机及其他资源。通过设置 “NetBIOS over TCP/IP”使得Samba不但能与局域网络主机分享资源，还能与全世界的电脑分享资 源。 

- Samba最大的功能就是可以用于Linux与windows系统直接的文件共享和打印共享，Samba既可 以用于windows与Linux之间的文件共享，也可以用于Linux与Linux之间的资源共享。 

- Samba由两个主要程序组成，它们是 smbd 和 nmbd 。这两个守护进程在服务器启动到停止期间持 续运行，功能各异。 Smbd 和 nmbd 使用的全部配置信息全都保存在smb.conf文件中。Smb.conf向 smbd和nmbd两个守护进程说明输出什么以便共享，共享输出给谁及如何进行输出。 

- Samba提供了基于CIFS的四个服务：文件和打印服务、授权与被授权、名称解析、浏览服务。前 两项服务由 smbd 提供，后两项服务则由 nmbd 提供。 简单地说， smbd 进程的作用是处理到来的 SMB软件包，为使用该软件包的资源与Linux进行协商， nmbd 进程使主机(或工作站)能浏览Linux 服务器。 

- SMB是Samba 的核心启动服务，主要负责建立 Linux Samba服务器与Samba客户机之间的对 话， 验证用户身份并提供对文件和打印系统的访问，只有SMB服务启动，才能实现文件的共享， 监听139 TCP端口；而NMB服务是负责解析用的，类似与DNS实现的功能，NMB可以把Linux系统 共享的工作组名称与其IP对应起来，如果NMB服务没有启动，就只能通过IP来访问共享文件，监 听137和138 UDP端口。 

- Samba服务器的IP地址为192.168.122.15，对应的工作组名称为 MYWORKGROUP，那么在 Windows的IE浏览器输入下面两条指令都可以访问共享文件。其实这就是Windows下查看Linux Samba服务器共享文件的方法。 

  \192.168.122.15\共享目录名称

  \MYWORKGROUP\共享目录名称 

- Samba服务器可实现如下功能：WINS和DNS服务； 网络浏览服务； Linux和Windows域之间的 认证和授权； UNICODE字符集和域名映射；满足CIFS协议的UNIX共享等。

### 安装配置 NFS 服务

#### 添加 hosts 解析

```
vim /etc/hosts [可选]
192.168.101.100 zhangsan
192.168.101.102 nas
```

#### 安装 NFS 服务器

```
yum install -y nfs-utils
```

#### 创建 NFS 存储目录

```
mkdir /data
```

#### 配置 NFS 服务

```
vim /etc/exports
/data 192.168.101.0/24(rw,sync,no_root_squash)
```

192.168.101.0/24 一个网络号的主机可以挂载 NFS服务器上的 /data 目录到自己的文件系统中

#### NFS 定制参数说明

- ro：目录只读 
- rw： 这个选项允许 NFS 客户机进行读/写访问。缺省选项是只读的。 
- secure： 这个选项是缺省选项，它使用了 1024 以下的 TCP/IP 端口实现 NFS 的连接。指定 insecure 可以禁用这个选项。 
- async： 这个选项可以改进性能，但是如果没有完全关闭 NFS 守护进程就重新启动了 NFS 服务 器，这也可能会造成数据丢失。 
- no_wdelay： 这个选项关闭写延时。如果设置了 async，那么 NFS 就会忽略这个选项。 
- nohide： 如果将一个目录挂载到另外一个目录之上，那么原来的目录通常就被隐藏起来或看起来 像空的一样。要禁用这种行为，需启用 hide 选项。 
- no_subtree_check： 这个选项关闭子树检查，子树检查会执行一些不想忽略的安全性检查。缺省 选项是启用子树检查。 
- no_auth_nlm： 这个选项也可以作为 insecure_locks 指定，它告诉 NFS 守护进程不要对加锁请求 进行认证。如果关心安全性问题，就要避免使用这个选项。缺省选项是 auth_nlm 或 secure_locks。 
- mp (mountpoint=path)： 通过显式地声明这个选项，NFS 要求挂载所导出的目录。 
- fsid=num： 这个选项通常都在 NFS 故障恢复的情况中使用。如果希望实现 NFS 的故障恢复，请 参考 NFS 文档。

#### NFS 用户映射的选项

- 在使用 NFS 挂载的文件系统上的文件时，用户的访问通常都会受到限制，这就是说用户都是以匿 名用户的身份来对文件进行访问的，这些用户缺省情况下对这些文件只有只读权限。如果用户希望 以 root 用户或锁定义的其他用户身份访问远程文件系统上的文件，NFS 允许指定访问远程文件的 用户——通过用户标识号（UID）和组标识号（GID）进行用户映射。 
- root_squash： 这个选项不允许 root 用户访问挂载上来的 NFS 卷。 
- no_root_squash： 这个选项允许 root 用户访问挂载上来的 NFS 卷。 
- all_squash： 这个选项对于公共访问的 NFS 卷来说非常有用，它会限制所有的 UID 和 GID，只使 用匿名用户。缺省设置是 no_all_squash。 anonuid 和 
- anongid： 这两个选项将匿名 UID 和 GID 修改成特定用户和组帐号。

#### 设置 NFS 服务开机启动

```
systemctl enable rpcbind.service
systemctl enable nfs-server.service
systemctl start rpcbind.service
systemctl start nfs-server.service
```

#### 确认 NFS 服务器启动

```
exportfs -v
```

- rpcinfo -p：检查 NFS 服务器是否挂载我们想共享的目录 /data：
- exportfs -r： 使配置生效

### 挂载端安装 NFS 客户端

#### 安装 NFS 软件包

```
yum -y install nfs-utils
```

#### 设置 rpcbind 开机启动

```
systemctl enable rpcbind.service
```

#### 启动 rpcbind 服务

```
systemctl start rpcbind.service
```

- 注意：客户端不需要启动 nfs 服务

#### 查看 NFS 服务端共享

检查 NFS 服务器端是否有目录共享：showmount -e nfs服务器IP/nfs服务器主机名

```
showmount -e nas
```

#### 挂在使用 NFS 存储

```
mkdir /data	#手动挂载
mount -t nfs nas:/data /data
umount /data
vim /etc/fstab	#开机自动挂载
mount -a
df	#查看挂载
```

- 如果服务器端修改了 NFS 的配置，而又不想重启 NFS 服务（因为有客户端正在使用）可以使用 exportfs 命令重新载入 NFS 配置。 

- export -ar: 重新导出所有的文件系统 
- export -au: 关闭导出的所有文件系统 
- export -u FS: 关闭指定的导出的文件系

```
rpcinfo -p #查看 RPC 服务列表
```

# 网站服务

## Apache 静态站点

```
[root@localhost ~]# yum -y install httpd
[root@localhost ~]# systemctl start httpd
[root@localhost ~]# systemctl status  httpd
[root@localhost ~]# systemctl enable httpd
[root@localhost ~]# systemctl stop  firewalld
[root@localhost ~]# setenforce 0
[root@localhost ~]# systemctl   stop   firewalld
[root@localhost ~]# httpd -v
```

## LAMP 动态站点

### 部署论坛系统discuz

```
#永久关闭selinux
[root@apache ~]# sed -ri '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config 
临时关闭selinux
[root@apache ~]# setenforce 0
[root@apache ~]# systemctl stop firewalld.service 
[root@apache ~]# systemctl disable firewalld.service
#安装LAMP
[root@apache ~]# yum -y install httpd mariadb-server mariadb php php-mysql gd php-gd
[root@apache ~]# systemctl start httpd mariadb
[root@apache ~]# systemctl enable httpd mariadb
#安装discuz
##导入discuz网站源码
wget http://download.comsenz.com/DiscuzX/2.5/Discuz_X2.5_SC_UTF8.zip
[root@apache ~]# mkdir    -p      /webroot/discuz
[root@apache ~]# yum  install  -y   unzip
[root@apache ~]#unzip  Discuz_X2.5_SC_UTF8.zip
[root@apache ~]#cp -rf upload/* /webroot/discuz/
[root@apache ~]#chown -R  apache.apache  /webroot/discuz/
# Apache 配置虚拟主机
[root@apache ~]# vim /etc/httpd/conf.d/discuz.conf
<VirtualHost *:80>
   ServerName www.discuz.com
   DocumentRoot /webroot/discuz
</VirtualHost>

<Directory "/webroot/discuz">
   Require all granted
</Directory>
[root@apache ~]# systemctl restart httpd
#准备数据库
[root@localhost discuz]# mysql
MariaDB [(none)]> create database discuz ;
```

# 域名服务

hosts文件：实现名字解析，主要为本地主机名，集群节点提供快速解析。缺点是不便于查询，更新。

DNS：（Domain Name System，域名系统）实现名字解析（例如将主机名解析为IP），分布式，层次性。

FQDN：(Fully Qualified Domain Name)完全合格域名/全称域名。

主机名.四级域.三级域.二级域.顶级域.（根域）

命名空间：

![image-20231015210632640](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304154819850.webp)

## DNS解析流程

例如客户端解析 www.126.com
1. 客户端查询自己的缓存（包含hosts中的记录），如果没有将查询发送/etc/resolv.conf中的DNS服务器

2. 如果本地DNS服务器对于请求的信息具有权威性，会将（权威答案）发送到客户端。

3. 否则（不具有权威性），如果DNS服务器在其缓存中有请求信息，则将（非权威答案）发送到客户端 

4. 如果缓存中没有该查询信息，DNS服务器将搜索权威DNS服务器以查找信息：
a. 从根区域开始，按照DNS层次结构向下搜索，直至对于信息具有权威的名称服务器，为客户端获答案
DNS服务器将信息传递给客户端 ，并在自己的缓存中保留一个副本，以备以后查找。
b. 转发到其它DNS服务器

正向解析/反向解析：

DNS服务主要起到两个作用：

　　1）可以把相对应的域名解析为对应的IP地址，这叫正向解析。

　　2）可以把相对应的IP地址解析为对应的域名，这叫反向解析。（反垃圾邮件）

# 远程管理

## ssh服务

```
安装软件：
    openssh-server 提供服务
    openssh-clients 客户端
    openssh
    [root@localhost ~]# yum install openssh* -y
ssh 端口22
服务器端：
	启动服务：
	[root@localhost ~]# systemctl start sshd
    查看：
    [root@localhost ~]# lsof -i:22
    关闭防火墙和selinux
客户端：
    远程登陆管理：
    [root@localhost ~]# ssh -X tom@10.18.44.208 -p 2222
    [root@localhost ~]# ssh 10.18.44.208
    如登陆果账户没有密码，默认不能
无密码登陆(ssh密钥认证)
client:
    产生公钥和私钥：
    [root@localhost ~]# ssh-keygen //一路回车
    拷贝公钥给对方：
    [root@localhost ~]# ssh-copy-id -i 10.18.44.208
直接执行远程命令：
	[root@localhost ~]# ssh 10.18.44.208 "reboot"
远程拷贝：
    需要先安装客户端
    [root@localhost ~]# cp 源文件 目标路径
    谁是远程谁加IP
    [root@localhost ~]# scp /a.txt 192.168.2.108:/
    [root@localhost ~]# scp 192.168.2.108:/a.txt ./
    -P端口
    拷贝目录加-r选项
    [root@localhost ~]# scp 192.168.2.108:/a.txt 192.168.2.109:/
修改端口号
    [root@localhost ~]# vim /etc/ssh/sshd_config
    Port 22
    ListenAddress 192.168.2.8
    PermitRootLogin yes
    MaxSessions 10 最大并发量
    PermitEmptyPasswords no
```

## rz sz命令

```
安装
root 账号登陆后执行以下命令：
[root@localhost ~]#yum install -y lrzsz
使用
sz命令发送文件到本地：
[root@localhost ~]# sz filename
rz命令本地上传文件到服务器：
[root@localhost ~]# rz
执行该命令后，在弹出框中选择要上传的文件即可。
```

# 文件服务器

## Ftp 介绍

文件传输协议（File Transfer Protocol，FTP），基于该协议FTP客户端与服务端可以实现共享文件、上传文件、下载文件。 FTP 基于TCP协议生成一个虚拟的连接，主要用于控制FTP连接信息，

同时再生成一个单独的TCP连接用于FTP数据传输。用户可以通过客户端向FTP服务器端上传、下载、删除文件，FTP服务器端可以同时提供给多人共享使用。

FTP服务是Client/Server（简称C/S）模式，基于FTP协议实现FTP文件对外共享及传输的软件称之为FTP服务器源端，客户端程序基于FTP协议，则称之为FTP客户端，FTP客户端可以向FTP服务器上传、下载文件。

目前主流的FTP服务器端软件包括：Vsftpd、ProFTPD、PureFTPd、Wuftpd、Server-U FTP、FileZilla Server等软件，其中Unix/Linux使用较为广泛的FTP服务器端软件为Vsftpd 。

## **Ftp** **传输模式介绍**

FTP基于C/S模式，FTP客户端与服务器端有两种传输模式，分别是FTP主动模式、FTP被动模式，主被动模式均是以FTP服务器端为参照。

FTP主动模式：客户端从一个任意的端口N（N>1024）连接到FTP服务器的port 21命令端口，客户端开始监听端口N+1，并发送FTP命令“port N+1”到FTP服务器，FTP服务器以数据端口（20）连接到客户端指定的数据端口（N+1）。

FTP被动模式：客户端从一个任意的端口N（N>1024）连接到FTP服务器的port 21命令端口，客户端开始监听端口N+1，客户端提交 PASV命令，服务器会开启一个任意的端口（P >1024），并发送PORT P命令给客户端。客户端发起从本地端口N+1到服务器的端口P的连接用来传送数据。

企业实际环境中，如果FTP客户端与FTP服务端均开放防火墙，FTP需以主动模式工作，这样只需要在FTP服务器端防火墙规则中，开放20、21端口即可。

## **Vsftp** **服务器简介**

非常安全的FTP服务进程（Very Secure FTP daemon，Vsftpd），Vsftpd在Unix/Linux发行版中最主流的FTP服务器程序，优点小巧轻快，安全易用、稳定高效、满足企业跨部门、多用户的使用（1000用户）等。

Vsftpd基于GPL开源协议发布，在中小企业中得到广泛的应用，Vsftpd可以快速上手，基于Vsftpd虚拟用户方式，访问验证更加安全。Vsftpd还可以基于MYSQL数据库做安全验证，多重安全防护。

## **Vsftp** **的登陆类型**

VSFTP提供了系统用户、匿名用户、和虚拟用户三种不同的登陆方式。所有的虚拟用户会映射成一个系统用户，访问时的文件目录是为此系统用户的家目录；匿名用户也是虚拟用户，映射的系统用户为ftp，详细信息可以通过man vsftpd.conf查看

## **Vsftp** **安装配置**

### **安装** **epel** **源**

```
yum -y install epel-release.noarch
```

### **安装** **vsftpd** **及相关依赖**

```
yum -y install vsftpd* pam* db4*
```

- vsftpd： ftp软件 
- pam：认证模块 
- DB4：支持文件数据库

### **vsftpd** **配置文件说明**

### **vsftpd** **配置详解**





