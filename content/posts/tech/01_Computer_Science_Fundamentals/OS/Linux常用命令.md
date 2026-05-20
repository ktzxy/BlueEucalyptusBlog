+++
date = '2025-03-20T16:56:40+08:00'
draft = false
title = 'Linux常用命令'
slug = "u7ki415osm"
description = "Linux常用命令"
categories = ["📒os"]
tags = ["Linux"]
summary = "Linux常用命令"

+++
# Linux常用命令

### 一. 文件和目录

cd 命令，用于切换当前目录，它的参数是要切换到的目录的路径，可以是绝对路径，也可以是相对路径。

```sh
cd /home 进入 '/ home' 目录
cd .. 返回上一级目录
cd ../.. 返回上两级目录
cd               进入个人的主目录
cd ~user1 进入个人的主目录
cd - 返回上次所在的目录
```

pwd 命令，显示工作路径

```sh
[root@mailvip ~]# pwd
/root
```

ls 命令，查看文件与目录的命令，list 之意

```sh
ls 查看目录中的文件
ls -l 显示文件和目录的详细资料
ls -a 列出全部文件，包含隐藏文件
ls -R 连同子目录的内容一起列出（递归列出），等于该目录下的所有文件都会显示出来
ls [0-9] 显示包含数字的文件名和目录名
```

cp 命令，用于复制文件，copy 之意，它还可以把多个文件一次性地复制到一个目录下

```sh
-a ：将文件的特性一起复制
-p ：连同文件的属性一起复制，而非使用默认方式，与-a相似，常用于备份
-i ：若目标文件已经存在时，在覆盖时会先询问操作的进行
-r ：递归持续复制，用于目录的复制行为 //经常使用递归复制
-u ：目标文件与源文件有差异时才会复制
```

mv 命令，用于移动文件、目录或更名，move 之意

```sh
-f ：force强制的意思，如果目标文件已经存在，不会询问而直接覆盖
-i ：若目标文件已经存在，就会询问是否覆盖
-u ：若目标文件已经存在，且比目标文件新，才会更新
```

rm 命令，用于删除文件或目录，remove 之意

```sh
-f ：就是force的意思，忽略不存在的文件，不会出现警告消息
-i ：互动模式，在删除前会询问用户是否操作
-r ：递归删除，最常用于目录删除，它是一个非常危险的参数
```

### 二、查看文件内容

cat 命令，用于查看文本文件的内容，后接要查看的文件名，通常可用管道与 more 和 less 一起使用

```
cat file1 从第一个字节开始正向查看文件的内容
tac file1 从最后一行开始反向查看一个文件的内容
cat -n file1 标示文件的行数
more file1 查看一个长文件的内容

head -n 2 file1 查看一个文件的前两行
tail -n 2 file1 查看一个文件的最后两行
tail -n +1000 file1 从1000行开始显示，显示1000行以后的
cat filename | head -n 3000 | tail -n +1000  显示1000行到3000行
cat filename | tail -n +3000 | head -n 1000  从第3000行开始，显示1000(即显示3000~3999行)

```

### 三. 文件搜索

find 命令，用来查找系统的

```
find / -name file1 从 '/' 开始进入根文件系统搜索文件和目录
find / -user user1 搜索属于用户 'user1' 的文件和目录
find /usr/bin -type f -atime +100 搜索在过去100天内未被使用过的执行文件
find /usr/bin -type f -mtime -10 搜索在10天内被创建或者修改过的文件
whereis halt 显示一个二进制文件、源码或man的位置
which halt 显示一个二进制文件或可执行文件的完整路径
```

删除大于 50M 的文件：

```
find /var/mail/ -size +50M -exec rm {} ＼;
```

### 四. 文件的权限 - 使用 "+" 设置权限，使用 "-" 用于取消

chmod 命令，改变文件 / 文件夹权限

```
ls -lh 显示权限
chmod ugo+rwx directory1 设置目录的所有人(u)、群组(g)以及其他人(o)以读（r，4 ）、写(w，2)和执行(x，1)的权限
chmod go-rwx directory1  删除群组(g)与其他人(o)对目录的读写执行权限
```

chown 命令，改变文件的所有者

```
chown user1 file1 改变一个文件的所有人属性
chown -R user1 directory1 改变一个目录的所有人属性并同时改变改目录下所有文件的属性
chown user1:group1 file1 改变一个文件的所有人和群组属性
```

chgrp 命令，改变文件所属用户组

```
chgrp group1 file1 改变文件的群组
```

### 五. 文本处理

grep 命令，分析一行的信息，若当中有我们所需要的信息，就将该行显示出来，该命令通常与管道命令一起使用，用于对一些命令的输出进行筛选加工等等

```
grep Aug /var/log/messages  在文件 '/var/log/messages'中查找关键词"Aug" 

grep ^Aug /var/log/messages 在文件 '/var/log/messages'中查找以"Aug"开始的词汇
grep [0-9] /var/log/messages 选择 '/var/log/messages' 文件中所有包含数字的行

grep Aug -R /var/log/* 在目录 '/var/log' 及随后的目录中搜索字符串"Aug" 

sed 's/stringa1/stringa2/g' example.txt 将example.txt文件中的 "string1" 替换成 "string2" 

sed '/^$/d' example.txt 从example.txt文件中删除所有空白行
```

paste 命令

```
paste file1 file2 合并两个文件或两栏的内容
paste -d '+' file1 file2 合并两个文件或两栏的内容，中间用"+"区分
```

sort 命令

```
sort file1 file2 排序两个文件的内容
sort file1 file2 | uniq 取出两个文件的并集(重复的行只保留一份)
sort file1 file2 | uniq -u 删除交集，留下其他的行
sort file1 file2 | uniq -d 取出两个文件的交集(只留下同时存在于两个文件中的文件)
```

comm 命令

```
comm -1 file1 file2 比较两个文件的内容只删除 'file1' 所包含的内容
comm -2 file1 file2 比较两个文件的内容只删除 'file2' 所包含的内容
comm -3 file1 file2 比较两个文件的内容只删除两个文件共有的部分
```

### 六、打包和压缩文件

tar 命令，对文件进行打包，默认情况并不会压缩，如果指定了相应的参数，它还会调用相应的压缩程序（如 gzip 和 bzip 等）进行压缩和解压

```
-c ：新建打包文件
-t ：查看打包文件的内容含有哪些文件名
-x ：解打包或解压缩的功能，可以搭配-C（大写）指定解压的目录，注意-c,-t,-x不能同时出现在同一条命令中
-j ：通过bzip2的支持进行压缩/解压缩
-z ：通过gzip的支持进行压缩/解压缩
-v ：在压缩/解压缩过程中，将正在处理的文件名显示出来
-f filename ：filename为要处理的文件
-C dir ：指定压缩/解压缩的目录dir
```

压缩：tar -jcv -f filename.tar.bz2 要被处理的文件或目录名称 查询：tar -jtv -f filename.tar.bz2 解压：tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录

```
bunzip2 file1.bz2 解压一个叫做 'file1.bz2'的文件
bzip2 file1 压缩一个叫做 'file1' 的文件
gunzip file1.gz 解压一个叫做 'file1.gz'的文件
gzip file1 压缩一个叫做 'file1'的文件
gzip -9 file1 最大程度压缩
rar a file1.rar test_file 创建一个叫做 'file1.rar' 的包
rar a file1.rar file1 file2 dir1 同时压缩 'file1', 'file2' 以及目录 'dir1' 
rar x file1.rar 解压rar包

zip file1.zip file1 创建一个zip格式的压缩包
unzip file1.zip 解压一个zip格式压缩包
zip -r file1.zip file1 file2 dir1 将几个文件和目录同时压缩成一个zip格式的压缩包
```

### **七. 系统和关机（关机、重启和登出）**

```
shutdown -h now 关闭系统(1)
init 0 关闭系统(2)
telinit 0 关闭系统(3)
shutdown -h hours:minutes & 按预定时间关闭系统
shutdown -c 取消按预定时间关闭系统
shutdown -r now 重启(1)
reboot 重启(2)
logout 注销
time 测算一个命令（即程序）的执行时间
```

### **八、进程相关的命令**

jps 命令，显示当前系统的 java 进程情况，及其 id 号

jps(Java Virtual Machine Process Status Tool) 是 JDK 1.5 提供的一个显示当前所有 java 进程 pid 的命令，简单实用，非常适合在 linux/unix 平台上简单察看当前 java 进程的一些简单情况。

ps 命令，用于将某个时间点的进程运行情况选取下来并输出，process 之意

```
-A ：所有的进程均显示出来
-a ：不与terminal有关的所有进程
-u ：有效用户的相关进程
-x ：一般与a参数一起使用，可列出较完整的信息
-l ：较长，较详细地将PID的信息列出

ps aux # 查看系统所有的进程数据
ps ax # 查看不与terminal有关的所有进程
ps -lA # 查看系统所有的进程数据
ps axjf # 查看连同一部分进程树状态
```

kill 命令, 用于向某个工作（%jobnumber）或者是某个 PID（数字）传送一个信号，它通常与 ps 和 jobs 命令一起使用

命令格式 : kill[命令参数][进程 id]

命令参数:

```
-l 信号，若果不加信号的编号参数，则使用“-l”参数会列出全部的信号名称
-a 当处理当前进程时，不限制命令名和进程号的对应关系
-p 指定kill 命令只打印相关进程的进程号，而不发送任何信号
-s 指定发送信号
-u 指定用户
```

实例 1：列出所有信号名称 命令：kill -l 输出：

```
[root@localhost test6]# kill -l
 1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL
 5) SIGTRAP 6) SIGABRT 7) SIGBUS 8) SIGFPE
 9) SIGKILL 10) SIGUSR1 11) SIGSEGV 12) SIGUSR2
13) SIGPIPE 14) SIGALRM 15) SIGTERM 16) SIGSTKFLT
17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG 24) SIGXCPU
25) SIGXFSZ 26) SIGVTALRM 27) SIGPROF 28) SIGWINCH
29) SIGIO 30) SIGPWR 31) SIGSYS 34) SIGRTMIN
35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3  38) SIGRTMIN+4
39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12
47) SIGRTMIN+13 48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14
51) SIGRTMAX-13 52) SIGRTMAX-12 53) SIGRTMAX-11 54) SIGRTMAX-10
55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7  58) SIGRTMAX-6
59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

说明：

只有第 9 种信号 (SIGKILL) 才可以无条件终止进程，其他信号进程都有权利忽略。   下面是常用的信号：

```
HUP 1 终端断线
INT 2 中断（同 Ctrl + C）
QUIT 3 退出（同 Ctrl + \）
TERM 15 终止
KILL    9    强制终止
CONT 18    继续（与STOP相反， fg/bg命令）
STOP    19    暂停（同 Ctrl + Z）
```

实例 2：得到指定信号的数值

```
[root@localhost test6]# kill -l KILL
[root@localhost test6]# kill -l SIGKILL
[root@localhost test6]# kill -l TERM
[root@localhost test6]# kill -l SIGTERM
[root@localhost test6]#
```

实例 3：先用 ps 查找进程，然后用 kill 杀掉

```
命令：kill 3268
[root@localhost test6]# ps -ef|grep vim 
root 3268  2884  0 16:21 pts/1    00:00:00 vim install.log
root 3370  2822  0 16:21 pts/0    00:00:00 grep vim
[root@localhost test6]# kill 3268
```

实例 4：彻底杀死进程

```
命令：kill –9 3268 // -9 强制杀掉进程
```

killall 命令，向一个命令启动的进程发送一个信号，用于杀死指定名字的进程  

命令格式 : killall[命令参数][进程名]

```
命令参数：
-Z 只杀死拥有scontext 的进程
-e 要求匹配进程名称
-I 忽略小写
-g 杀死进程组而不是进程
-i 交互模式，杀死进程前先询问用户
-l 列出所有的已知信号名称
-q 不输出警告信息
-s 发送指定的信号
-v 报告信号是否成功发送
-w 等待进程死亡
--help 显示帮助信息
--version 显示版本显示
```

示例

```
1：杀死所有同名进程
    killall nginx
    killall -9 bash

2.向进程发送指定信号
    killall -TERM ngixn 或者 killall -KILL nginx
```

top 命令，是 Linux 下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于 Windows 的任务管理器。  

如何杀死进程：

```
（1）图形化界面的方式
（2）kill -9 pid （-9表示强制关闭）
（3）killall -9 程序的名字
（4）pkill 程序的名字
```

查看进程端口号：

```
netstat -tunlp|grep 端口号
```

# Linux `find` 命令详解与实用技巧

`find` 是一个功能极其强大的文件查找命令，它能根据丰富的条件递归地在目录树中搜索文件和目录，并对其执行相应操作。

**基本语法：** `find [查找路径] [查找条件] [操作]`

## 1. 按名称查找

这是最常用的查找方式。

```bash
# 在根目录 (/) 查找所有名为 "default" 的文件或目录
# 2>/dev/null: 将错误输出（如权限不足的目录）重定向到空设备，保持输出干净
find / -name "default" 2>/dev/null

# -iname: 忽略文件名的大小写
find / -iname "default" 2>/dev/null

# 使用通配符查找所有 .conf 文件
find /etc -name "*.conf"
```

## 2. 按类型查找

使用 `-type` 参数可以精确查找文件类型。

```
# 只查找普通文件 (f: file)
find /var/log -type f -name "*.log"

# 只查找目录 (d: directory)
find /usr/share -type d -name "doc"

# 只查找符号链接 (l: symbolic link)
find /usr/bin -type l
```

## 3. 按时间查找

基于文件的修改时间 (`-mtime`)、访问时间 (`-atime`) 或状态变更时间 (`-ctime`) 进行查找。

```bash
# 查找 /etc 下，24小时内被修改过的文件 (-1)
find /etc -mtime -1

# 查找 /home 下，超过30天未被修改过的文件 (+30)
find /home -mtime +30

# 查找 /var/log 下，7天前正好那天被修改过的日志文件
find /var/log -name "*.log" -mtime 7
```

## 4. 按大小查找

可以指定精确大小，或大于 (`+`)、小于 (`-`) 某个阈值。

```bash
# 查找大于 100MB 的文件
# 单位: c(字节), k(KB), M(MB), G(GB)
find / -type f -size +100M 2>/dev/null

# 查找大小在 100MB 到 300MB 之间的文件
find / -type f -size +100M -size -300M 2>/dev/null
```

## 5. 按用户和权限查找

查找特定用户、用户组或权限的文件。

```bash
# 查找用户 apache 所拥有的所有文件和目录
find / -user apache 2>/dev/null

# 查找用户组 nginx 所拥有的文件和目录
find / -group nginx 2>/dev/null

# 查找权限正好为 644 的文件
find /etc -type f -perm 644
```

## 6. 对查找结果执行操作

`find` 的强大之处在于它可以直接对查找到的结果执行命令。

### 6.1 使用 `-exec`

这是 `find` 内置的命令执行方式。

Bash

```bash
# 找到用户 dmxy 的文件，并将其安全地拷贝到 /root/dmxy 目录
# {}: 代表 find 找到的每一个文件
# \;: 代表 -exec 命令的结束符
find / -user dmxy -exec cp -a {} /root/dmxy \; 2>/dev/null

# 更高效的 -exec 用法：使用 "+" 替代 "\;"
# 这会将多个查找结果作为参数一次性传递给命令，而不是每个文件执行一次命令
find / -user dmxy -exec ls -ld {} + 2>/dev/null
```

### 6.2 配合 `xargs`

`xargs` 可以从标准输入读取数据，并将其作为参数传递给后续命令。

```bash
# 查找用户 apache 的文件，并用 ls -ld 查看其详细信息
find / -user apache 2>/dev/null | xargs ls -ld
```

> **安全实践**：如果文件名可能包含空格或特殊字符，`-exec` 更安全。或者使用 `find` 的 `-print0` 和 `xargs` 的 `-0` 参数来正确处理。 `find / -user apache -print0 | xargs -0 ls -ld`

### 6.3 使用 `-delete`

对于删除操作，`find` 内置的 `-delete` 比 `-exec rm {} \;` 更高效、更安全。

```bash
# 查找并删除所有 .tmp 后缀的临时文件
find / -type f -name "*.tmp" -delete
```

## 7. 组合条件查询

`find` 默认使用“与”(`-a`)逻辑组合条件，还可以使用“或”(`-o`)和“非”(`!`)。

```bash
# 查找 /tmp 下大于 10MB 的 .log 文件 (隐式 "与")
find /tmp -size +10M -name "*.log"

# 查找 /home 下的 .txt 文件 或者 .md 文件 ("或")
find /home -name "*.txt" -o -name "*.md"

# 查找 /etc 下，不属于 root 用户的文件 ("非")
find /etc ! -user root
```

## 使用`find`命令查找大文件

`find`命令是 Linux 系统管理员工具库中最强大的工具之一。它允许你根据不同的标准（包括文件大小）搜索文件和目录。
 例如，如果在当前工作目录中要搜索大小超过 100MB 的文件，请使用以下命令：

```sh
sudo find . -xdev -type f -size +100M
```

`.` 代表当前目录。如要搜索其它目录替换`.`为要搜索目录的路径。

输出将显示的文件列表，不会包含其它信息。

```
/var/lib/libvirt/images/centos-7-desktop_default.img
/var/lib/libvirt/images/bionic64_default.img
/var/lib/libvirt/images/winqcow2
/var/lib/libvirt/images/debian-9_default.img
/var/lib/libvirt/images/ubuntu-18-04-desktop_default.img
/var/lib/libvirt/images/centos-7_default.img
```

`find`命令还可以与其他命令结合使用，例如`ls`或`sort`对这些文件执行操作。

在下面的示例中，我们传递`find`命令的输出到`ls` ，`ls`将打印已找到的每个文件的大小，然后将将输出传递给`sort`命令，以根据文件大小的第 5 列对其进行排序。

```sh
find . -xdev -type f -size +100M -print | xargs ls -lh | sort -k5,5 -h -r
```

输出像这样：

```
-rw-------  1 root   root 40967M Jan  5 14:12 /var/lib/libvirt/images/winqcow2
-rw-------  1 root   root  3725M Jan  7 22:12 /var/lib/libvirt/images/debian-9_default.img
-rw-------  1 root   root  1524M Dec 30 07:46 /var/lib/libvirt/images/centos-7-desktop_default.img
-rw-------  1 root   root   999M Jan  5 14:43 /var/lib/libvirt/images/ubuntu-18-04-desktop_default.img
-rw-------  1 root   root   562M Dec 31 07:38 /var/lib/libvirt/images/centos-7_default.img
-rw-------  1 root   root   378M Jan  7 22:26 /var/lib/libvirt/images/bionic64_default.img
```

如果输出包含大量信息，你可以使用该`head`命令仅打印前 10 行：

```sh
find . -xdev -type f -size +100M -print | xargs ls -lh | sort -k5,5 -h -r | head
```

分解命令：`find . -xdev -type f -size +100M -print`

*   仅搜索当前工作目录（`.`）中的 文件 (`-type f`)，大于 100MB（`-size +100M`），不要查找其他文件系统上的目录（`-xdev`）并在标准输出上打印完整文件名，然后是新的一行（`-print`） 。
*   `xargs ls -lh`- `find`命令的输出通过管道`xargs`执行，`ls -lh`命令将以长列表可读格式打印输出。
*   `sort -k5,5 -h -r`- 基于第 5 列（`-k5,5`）对行进行排序，以可读格式（`-h`）的值并反转结果（`-r`）。
*   `head` ：仅打印管道输出的前 10 行。

> `find`命令带有许多强大的选项。例如，你可以搜索超过多少天的大文件，具有特定扩展名的大文件或属于特定用户的大文件。

### 使用`du`命令查找大文件和目录

`du`命令用于估计文件空间使用情况，对于查找占用大量磁盘空间的目录和文件特别有用。

以下命令将打印最大的文件和目录：

```sh
du -ahx . | sort -rh | head -5
```

第一列包含文件大小，第二列包含文件名：

```
55G    .
24G    ./.vagrant.d/boxes
24G    ./.vagrant.d
13G    ./Projects
2G    ./.minikube
```

命令说明：

*   `du -ahx .`：估算当前工作目录（`.`）中的磁盘空间使用情况，包括文件和目录（`a`），以比较接近人的常见可读格式打印大小（`h`）并跳过不同文件系统上的目录（`x`）。
*   `sort -rh`：通过可读格式（`-h`）的值并反转结果（`-r`）来对输出行进行排序。
*   `head -5` ：仅打印管道输出的前 5 行。



---

[Linux上查找最大文件的3种方法_无忧杂货铺的博客-CSDN博客_](https://blog.csdn.net/qq_41816540/article/details/103046568)

# Linux `ls` 命令详解与实用指南

`ls` (list) 是 Linux 系统中最基础也最重要的命令之一，用于列出目录中的文件和子目录。通过丰富的选项，`ls` 可以提供详细、多样化的信息。

## 1. `ls` 常用选项

以下是 `ls` 命令最常用的一些选项，它们经常被组合使用。

| 选项     | 完整写法           | 作用                                                         |
| -------- | ------------------ | ------------------------------------------------------------ |
| **`-l`** |                    | **长格式 (long)** 显示。列出文件的详细信息，如权限、所有者、大小、修改时间等。 |
| **`-a`** | `--all`            | **全部 (all)** 文件。显示所有文件，包括以 `.` 开头的隐藏文件。 |
| **`-h`** | `--human-readable` | **人性化 (human-readable)** 显示文件大小。自动以 K, M, G 等单位显示，必须与 `-l` 配合使用。 |
| `-d`     | `--directory`      | **目录本身 (directory)**。当参数是目录时，仅显示目录本身的信息，而不是其内容。 |
| `-t`     |                    | 按**修改时间 (time)** 排序，最新的文件会显示在最前面。       |
| `-r`     | `--reverse`        | **反向 (reverse)** 排序。通常与 `-t` 组合使用 (`-ltr`)，将最新的文件显示在最下面。 |
| `-i`     | `--inode`          | 显示每个文件的 **inode** 索引节点号。                        |
| `-R`     | `--recursive`      | **递归 (recursive)** 显示。列出所有子目录中的内容。          |

**常用组合：** `ls -lah` 是日常使用频率最高的组合之一，它能以人性化的方式显示所有文件的详细信息。

## 2. 解读 `ls -l` 的详细输出

`ls -l` 的输出包含了关于文件或目录的丰富信息。让我们逐列进行详细解读。

**示例输出：**

```
drwxr-xr-x. 2 fanyang fanyang 4096 Sep 21 10:30 Documents
-rw-r--r--. 1 root    root    5624 Apr  6 15:19 config.txt
lrwxrwxrwx. 1 root    root      20 Sep 22 17:00 link_to_config -> /etc/config.conf
```

**输出列详解：**

| #    | 列内容         | 含义                                                    |
| ---- | -------------- | ------------------------------------------------------- |
| 1    | `drwxr-xr-x.`  | **文件类型、权限 和 特殊属性**                          |
| 2    | `2`            | **硬链接数** (对于目录，通常表示其包含的子目录数 + 2)   |
| 3    | `fanyang`      | **文件所有者** (User)                                   |
| 4    | `fanyang`      | **文件所属组** (Group)                                  |
| 5    | `4096`         | **文件大小** (单位：字节)。使用 `-h` 选项可人性化显示。 |
| 6    | `Sep 21 10:30` | **最后修改时间** (Month, Day, Time)                     |
| 7    | `Documents`    | **文件名或目录名**                                      |

**文件类型与权限的深入解读:**

第一列 `drwxr-xr-x.` 是信息量最大的一列，由三部分组成：

**第一部分：文件类型 (第1个字符)**

| 字符    | 类型            | 描述               |
| ------- | --------------- | ------------------ |
| **`-`** | 普通文件        | (Regular file)     |
| **`d`** | 目录            | (Directory)        |
| **`l`** | 符号链接/软链接 | (Symbolic link)    |
| `b`     | 块设备文件      | (Block device)     |
| `c`     | 字符设备文件    | (Character device) |
| `s`     | 套接字文件      | (Socket)           |
| `p`     | 管道文件        | (Named pipe)       |

**第二部分：权限位 (第2-10个字符)** 这9个字符被分为三组，每组都是 `rwx` (读、写、执行) 的组合：

- **`r` (Read)**: 读取权限 (值为 4)
- **`w` (Write)**: 写入权限 (值为 2)
- **`x` (Execute)**: 执行权限 (值为 1)
- **`-`**: 无相应权限 (值为 0)

| 字符组       | 含义                     | 示例 `rwxr-xr-x` 中的含义    |
| ------------ | ------------------------ | ---------------------------- |
| **第2-4位**  | **所有者 (User)** 权限   | `rwx` (可读、可写、可执行)   |
| **第5-7位**  | **所属组 (Group)** 权限  | `r-x` (可读、不可写、可执行) |
| **第8-10位** | **其他人 (Others)** 权限 | `r-x` (可读、不可写、可执行) |

**第三部分：特殊属性 (权限位后的字符)**

| 字符    | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| **`.`** | (点) 表示该文件拥有 **SELinux** 安全上下文。                 |
| **`+`** | (加号) 表示该文件设置了 **ACL (Access Control List)** 扩展权限。 |
| (空)    | 无特殊属性。                                                 |

## 3. 相关命令

在浏览文件系统时，`ls` 通常与以下命令配合使用，以获取不同的信息维度。

### 3.1 `tree` - 递归树状显示

当需要清晰地查看目录的层级结构时，`tree` 命令比 `ls -R` 的输出更直观。

```
# 以树状结构显示 /etc/sysconfig 目录
tree /etc/sysconfig
```

### 3.2 `du` - 查看目录空间占用

`ls -l` 显示的是文件本身的大小，但对于目录，它只显示目录元数据的大小（通常是 4096 字节），而不是目录内所有文件的总大小。要查看目录的真实空间占用，应使用 `du`。

```
# 查看 /etc 目录的总大小 (-s: summary, -h: human-readable)
du -sh /etc
```

# Linux `dd` 命令详解与实用指南

`dd` 是一个核心的 Linux/Unix 工具，用于以底层、按块的方式复制和转换数据。它功能强大，常被称为“数据毁灭者”(Data Destroyer)，因为一个微小的输入错误（如颠倒 `if` 和 `of`）就可能导致整个磁盘的数据被擦除。

> **❗ 终极警告：** 在执行任何 `dd` 命令前，请**反复确认**输入文件 (`if=`) 和输出文件 (`of=`) 的路径是否完全正确。错误的 `of` 参数将会无提示地覆盖目标设备上的所有数据。

## 1. `dd` 命令语法与核心参数

基本语法为 `dd if=<来源> of=<目标> [选项]`

| 参数              | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| `if=`             | **Input File**，数据来源。例如 `/dev/zero`, `/dev/sda`, 或一个文件。 |
| `of=`             | **Output File**，数据目标。例如一个文件，或一个设备 `/dev/sdb`。 |
| `bs=`             | **Block Size**，块大小。定义一次读写的数据块大小，如 `bs=4M`。是影响读写性能的关键参数。 |
| `count=`          | **Count**，复制块的数量。总复制量 = `bs` * `count`。         |
| `status=progress` | **(推荐使用)** 显示一个实时的进度条，让您了解复制进度。      |
| `conv=fsync`      | 在写操作结束后，强制将内存缓冲区的数据同步到物理磁盘，确保数据落盘。 |
| `oflag=direct`    | 使用直接 I/O 模式写入数据，绕过系统缓存，用于获取更真实的磁盘性能测试结果。 |

## 2. 实用场景示例

### 2.1 测试磁盘**写**性能

此命令从 `/dev/zero` 读取无限的零数据，并向 `testfile` 文件写入 1GB 数据，通过 `oflag=direct` 绕过缓存来测试磁盘的原始写入速度。

```bash
# 写 1GB 数据到当前目录的 testfile 文件中
dd if=/dev/zero of=testfile bs=1M count=1024 oflag=direct status=progress
```

### 2.2 测试磁盘**读**性能

此命令从物理磁盘 `/dev/sda` 读取数据，并将其“丢弃”到 `/dev/null`，从而测试磁盘的原始读取速度。

> **注意**：从正在使用的系统盘读取可能会影响性能和测试准确性。

```bash
# 测试 /dev/sda 的读取速度
dd if=/dev/sda of=/dev/null bs=4M status=progress
```

### 2.3 创建指定大小的文件

快速生成一个大文件，常用于创建交换文件 (swapfile) 或虚拟机磁盘。

```bash
# 创建一个 1GB 大小的文件
dd if=/dev/zero of=/root/largefile bs=1M count=1000
```

### 2.4 磁盘与分区克隆

**这是 `dd` 最危险也是最强大的功能之一。**

```bash
# --- 整盘克隆 ---
# 将 /dev/sda 的全部内容完整复制到 /dev/sdb。
# 警告：此操作将完全抹掉 /dev/sdb 上的所有数据！
dd if=/dev/sda of=/dev/sdb bs=4M conv=fsync status=progress

# --- 分区克隆 ---
# 仅复制 /dev/sda 的第一个分区到 /dev/sdb 的第一个分区
dd if=/dev/sda1 of=/dev/sdb1 bs=4M conv=fsync status=progress
```

### 2.5 创建与恢复磁盘镜像

将整个磁盘或分区备份成一个文件，或从文件恢复。

```bash
# --- 创建磁盘镜像 (备份) ---
# 将 /dev/sda 整盘备份为 home 目录下的 sda_backup.img 文件
dd if=/dev/sda of=~/sda_backup.img bs=4M conv=fsync status=progress

# --- 从镜像恢复 ---
# 将备份文件恢复到 /dev/sdb 磁盘
# 警告：同样会抹掉 /dev/sdb 的所有数据！
dd if=~/sda_backup.img of=/dev/sdb bs=4M conv=fsync status=progress
```

### 2.6 制作 USB 启动盘

将 Linux 发行版的 `.iso` 镜像文件直接写入 U 盘。

> **注意**：`of` 的目标是整个 U 盘设备 (如 `/dev/sdc`)，而不是某个分区 (如 `/dev/sdc1`)。

```bash
# 将 ubuntu.iso 写入 /dev/sdc (请用 lsblk 命令确认您的 U 盘设备名)
dd if=~/Downloads/ubuntu-22.04.iso of=/dev/sdc bs=4M conv=fsync status=progress
```

### 2.7  填充稀疏文件 (Thick Provisioning)

`qemu-img create` 创建的 qcow2 文件默认是稀疏（或称“精简置备”）的，它只在需要时才分配空间。使用 `dd` 可以将其完全填充为“厚置备”，预先分配所有空间。

```bash
# 1. 创建一个 100G 的稀疏 qcow2 文件 (此文件在磁盘上初始大小很小)
qemu-img create -f qcow2 my_vm_disk.qcow2 100G

# 2. 使用 dd 向其写入零，直到写满 100G，将其转换为厚置备
# 注意：此操作会消耗 100G 的物理磁盘空间，且可能需要很长时间
dd if=/dev/zero of=my_vm_disk.qcow2 bs=1M status=progress conv=fsync
```

# shutdown 特权命令

## 1. 功能

`shutdown` 命令用于安全地关闭或重启 Linux 系统。它会通知所有登录的用户系统即将关闭，并阻止新的登录。只有超级用户（root）或具有相应权限的用户才能执行此命令

使用 `shutdown` 命令的主要优点是它会正常地终止系统中的所有进程，并确保文件系统的数据同步，从而防止数据丢失或文件系统损坏

## 2. 语法

命令的基本语法结构如下：

`shutdown [选项] 时间 [警告消息]`

-   **[选项]**: 控制关机或重启的具体行为，例如是关机还是重启。

-   **时间**: 指定何时执行关机或重启操作。

-   **[警告消息]**: 可选参数，向所有登录用户广播的自定义消息。

## 3. 选项

以下是 `shutdown` 命令常用的选项：

| 选项        | 长格式       | 描述                                                         |
| ----------- | ------------ | ------------------------------------------------------------ |
| `-r`        | `--reboot`   | 重启系统。                                                   |
| `-h`        | `--halt`     | 关闭系统后停止 CPU 的活动（Halt）。                          |
| `-P`        | `--poweroff` | 关闭系统并切断电源（Power Off）。这是最常用的关机选项。      |
| `-c`        | `--cancel`   | 取消一个已经计划的 `shutdown` 操作。                         |
| `-k`        |              | 发送警告消息，但并不实际执行关机或重启操作。常用于提醒用户系统即将维护。 |
| `--help`    |              | 显示命令的帮助信息。                                         |
| `--version` |              | 显示命令的版本信息。                                         |

## 4. 时间参数格式

`时间` 参数可以有以下几种格式：

| 时间格式 | 描述                                         | 示例                                   |
| -------- | -------------------------------------------- | -------------------------------------- |
| `now`    | 立即执行关机或重启。                         | `shutdown -h now`                      |
| `+m`     | 表示在 `m` 分钟后执行。                      | `shutdown -r +15` （15分钟后重启）     |
| `hh:mm`  | 指定一个绝对时间（24小时制），例如 `22:00`。 | `shutdown -P 22:00` （在晚上10点关机） |

## 5. 警告消息

这是一个可选参数。如果指定了警告消息，该消息将在关机倒计时期间广播给所有已登录的用户。这对于通知用户保存工作非常有用

## 6. 常用示例

### 6.1 立即关机

要立即关闭系统并切断电源，可以使用以下命令：

```bash
shutdown -P now
```

或者使用 `-h` 选项，效果类似：

```bash
shutdown -h now
```

### 6.2 计划在10分钟后重启

要让系统在10分钟后自动重启，并向用户显示一条消息：

```bash
shutdown -r +10 "系统将在10分钟后重启以进行更新，请保存您的工作。"
```

### 6.3 在指定时间关机

要在当晚 11:30 关闭系统：

```bash
shutdown -P 22:30 "系统将在 22:30 关闭，请确保所有任务已完成。"
```

### 6.4 取消已计划的关机

如果您之前设置了一个计划关机任务，但现在需要取消它，可以使用 `-c` 选项：

```bash
shutdown -c
```

您还可以附带一条取消消息：

```bash
shutdown -c "系统维护已推迟，关机已取消。"
```

### 6.5 仅发送警告而不关机

如果您想提醒用户系统将在稍后进行维护，但暂时不执行任何关机操作，可以使用 `-k` 选项：

```bash
shutdown -k "警告：系统将于今晚午夜进行维护，届时将无法访问。"
```

## 7. 注意事项

-   **权限**: 执行 `shutdown` 命令需要超级用户（root）权限。普通用户可以使用 `sudo` 来执行，例如 `sudo shutdown -h now`。
-   **无选项**: 如果执行 `shutdown` 命令时不带任何选项，默认行为（在一些系统中）通常是在1分钟后进入单用户模式。但为了清晰和安全，建议总是明确指定 `-h` 或 `-r` 选项。
-   **替代命令**: 在现代 Linux 系统中，`poweroff`、`reboot` 和 `halt` 等命令通常是 `systemctl` 的符号链接，也可以用来关机和重启，但 `shutdown` 提供了更多控制选项，尤其是在多用户环境中。

# Linux Shell I/O 重定向与管道详解

> 在 Linux Shell 中，**I/O 重定向** 和 **管道** 是两个极其强大和核心的功能，它们允许我们灵活地控制命令的输入和输出流向，将简单的命令组合成强大的工具链。

## 1. 核心概念：三个标准流

当你在终端执行一个命令时，Linux 系统会为该进程默认打开三个标准的数据流：

| 名称                  | 文件描述符 | 默认来源/去向 | 描述                         |
| --------------------- | ---------- | ------------- | ---------------------------- |
| **标准输入 (stdin)**  | `0`        | 键盘          | 命令读取数据的地方。         |
| **标准输出 (stdout)** | `1`        | 屏幕/终端     | 命令输出**正常**结果的地方。 |
| **标准错误 (stderr)** | `2`        | 屏幕/终端     | 命令输出**错误**信息的地方。 |

重定向就是改变这些数据流的默认“源”和“目的地”的过程。

## 2. 输出重定向 (Redirecting Output)

输出重定向用于将命令的执行结果（标准输出或标准错误）保存到文件，而不是显示在屏幕上。

### 2.1 重定向标准输出 (stdout, 描述符 `1`)

这是最常见的重定向，操作符 `>` 默认就是针对标准输出。

`>`: **覆盖**重定向。如果目标文件不存在，则创建；如果已存在，则**清空**其内容后写入。

`>>`: **追加**重定向。如果目标文件不存在，则创建；如果已存在，则将输出**追加**到文件末尾。

```bash
# 将 /etc 目录的列表覆盖写入到 a.txt
ls -l /etc > a.txt

# 将当前日期追加到 a.txt 文件末尾
date >> a.txt
```

### 2.2 重定向标准错误 (stderr, 描述符 `2`)

用于捕获命令执行期间产生的错误信息。

`2>`: **覆盖**重定向标准错误。

`2>>`: **追加**重定向标准错误。

```bash
# find 命令在查找时常有权限不足的错误，我们将这些错误信息保存到 find_errors.log
find / -name "*.conf" 2> find_errors.log

# 再次执行其他可能出错的命令，将错误追加到同一个日志文件
another_command 2>> find_errors.log
```

### 2.3 重定向所有输出 (stdout & stderr)

有时候，我们希望将命令的所有输出（无论是正常结果还是错误信息）都保存到同一个地方。

`&>`: **覆盖**重定向所有输出。

`&>>`: **追加**重定向所有输出。

```bash
# 将 yum 安装过程的所有信息（包括进度和可能出现的错误）都覆盖写入到 install.log
yum install -y httpd &> install.log

# 将另一个命令的所有输出追加到日志文件
another_command &>> install.log
```

> **传统写法**: `> install.log 2>&1` 是 `&> install.log` 的等效传统写法，表示先将 stdout 重定向到文件，然后将 stderr (`2`) 重定向到与 stdout (`&1`) 相同的地方。

### 2.4 “黑洞”设备 (`/dev/null`)

`/dev/null` 是一个特殊的设备文件，它会“吞噬”掉所有写入它的数据。当您不关心某个命令的输出，只想让它安静地执行时，就可以将输出重定向到这里。

```bash
# 只关心命令是否成功执行，不看任何正常输出
command > /dev/null

# 忽略所有错误信息，只看正常输出
command 2> /dev/null

# 忽略所有输出，让命令在后台“安静”地运行
command &> /dev/null
```

## 3. 输入重定向 (Redirecting Input)

输入重定向用于改变命令的数据来源，使其从文件或其他来源读取数据，而不是从键盘。

### 3.1 从文件重定向输入 (`<`)

这是最直接的输入重定向，让命令从指定文件中读取内容。

```bash
# wc -l 命令会统计 unsorted_list.txt 文件中的行数
wc -l < unsorted_list.txt
```

### 3.2 Here Document (`<<`)

Here Document 允许您在脚本中直接提供多行输入，直到遇到一个预定义的结束符。

```bash
# 将多行文本直接写入 message.txt 文件，直到遇到行首的 "EOF"
cat > message.txt << EOF
This is the first line.
This is the second line.
Hello, world.
EOF
```

## 4. 管道 (`|`)

管道是 Linux 命令行的精髓。它**不是**将输出重定向到文件，而是将**一个命令的标准输出 (stdout) 直接连接到另一个命令的标准输入 (stdin)**，形成一个处理流水线。

```bash
# 1. 查看系统中所有进程
ps aux

# 2. 使用管道，将 ps 的输出交给 grep 来筛选包含 "httpd" 的行
ps aux | grep "httpd"

# 3. 复杂的管道链：统计系统日志中包含 "error" 关键字的行数
cat /var/log/messages | grep -i "error" | wc -l
```

# Vim/Vi 编辑器核心技巧与配置指南

`Vim` (Vi IMproved) 是 `Vi` 编辑器的增强版，也是现代 Linux 系统中 `vi` 命令的实际提供者。它是一款功能强大、高度可配置的文本编辑器，其精髓在于**模式化操作**和丰富的快捷键。

## 1. Vim 的核心模式

理解 Vim 的模式是掌握它的第一步。最常用的有三种：

| 模式                               | 英文              | 作用                                                     | 进入/退出方式                           |
| ---------------------------------- | ----------------- | -------------------------------------------------------- | --------------------------------------- |
| **普通模式 (Normal Mode)**         | Normal Mode       | **默认模式**。用于移动光标、删除、复制、粘贴等命令操作。 | 按 `Esc` 键可从任何其他模式返回此模式。 |
| **插入模式 (Insert Mode)**         | Insert Mode       | 用于输入和编辑文本，此时 Vim 就像一个普通的记事本。      | 在普通模式下按 `i`, `a`, `o` 等键进入。 |
| **命令行模式 (Command-line Mode)** | Command-line Mode | 用于执行更复杂的操作，如保存、退出、搜索、替换等。       | 在普通模式下按 `:` 键进入。             |

## 2. 核心操作快捷键 (在普通模式下)

### 2.1 基础光标移动

- `h`, `j`, `k`, `l`：左、下、上、右
- `w` / `b`：向前 / 向后移动一个单词
- `^` / `$`：移动到行首 / 行尾
- `gg` / `G`：移动到文件第一行 / 最后一行
- `:n` 或 `nG`：跳转到第 `n` 行

### 2.2 基础文本编辑

- `x`：删除光标处的字符
- `dw`：删除一个单词
- `dd`：删除当前整行
- `yy`：复制当前整行
- `p`：在光标后粘贴
- `u`：撤销上一步操作
- `Ctrl + r`：重做 (Redo)
- `.`：重复上一次的修改命令

## 3. 批量注释与取消注释

### 方法一：使用命令行替换 (您提供的方法)

这是一种精确、可脚本化的方式，利用了 Vim 的替换命令。

```
" 将第 n1 行到 n2 行的行首 (^) 替换为 "#"
:n1,n2s/^/#/g

" 将第 n1 行到 n2 行的行首 "#" 替换为空，即取消注释
:n1,n2s/^#//g
```

### 方法二：使用可视块模式 (Visual Block Mode - 更直观)

1. **进入可视块模式**：将光标移动到要注释的第一行行首，按下 `Ctrl + v`。
2. **选择行**：使用 `j` 键向下移动光标，选中所有需要注释的行的第一列。
3. **插入注释符**：按下大写的 `I` 键，此时会进入“块插入模式”。
4. **输入注释符**：输入 `#` (或其他注释符，如 `//`)。
5. **应用到所有行**：按下 `Esc` 键，您会发现 `#` 已经同时添加到了所有选定行的行首。

> **取消注释**：同样使用 `Ctrl + v` 选中行首的 `#` 字符，然后直接按 `x` 或 `d` 即可全部删除。

## 4. 自定义 Vim (`~/.vimrc`)

`~/.vimrc` 是 Vim 的个人配置文件，您可以在此文件中添加配置和自定义快捷键，打造属于自己的高效编辑器。

### 4.1 推荐的基础配置

将以下配置加入您的 `~/.vimrc` 文件，可以让 Vim 更好用。

```
" 开启语法高亮
syntax on

" 显示行号
set number

" 设置 Tab 键宽度为 4 个空格
set tabstop=4

" 将 Tab 自动转换为空格
set expandtab

" 启用搜索结果高亮
set hlsearch

" 在输入搜索内容时实时高亮
set incsearch
```

### 4.2 自定义快捷键 (`map`)

我们可以将一系列复杂操作映射到一个简单的快捷键上。

```
" --- 您提供的快捷键配置详解 ---

" 将 vp 映射为在行首添加 '#' 并返回普通模式
" I: 在行首进入插入模式; #: 输入#号; <ESC>: 返回普通模式
nnoremap vp I#<ESC>

" 将 vb 映射为删除行首的 '#'
" 0: 跳转到行首; x: 删除一个字符
nnoremap vb 0x
```

> **最佳实践**: 推荐使用 `nnoremap` 代替 `map`。`noremap` 表示“非递归映射”，可以防止自定义的快捷键之间发生意想不到的连锁反应，使配置更稳定。

### 4.3 自定义缩写 (`ab`)

当您在**插入模式**下输入一个缩写并敲击空格后，它会自动展开成完整的文本。

```
" --- 您提供的缩写配置 ---

" 当输入 "exm" + 空格时，自动替换为 "example.com"
ab exm example.com
```

# Linux `ping` 命令详解与网络诊断技巧

> `ping` (Packet Internet Groper) 是 Linux 系统中最基础也最重要的网络诊断工具。它通过发送 ICMP (Internet Control Message Protocol) `ECHO_REQUEST` 报文到目标主机，并等待 `ECHO_RESPONSE` 报文返回，来检测网络的连通性和测量往返延迟。

**核心用途：**

- **测试网络连通性**：判断本机与目标主机之间网络是否通畅。
- **测试 DNS 解析**：通过 `ping` 域名，可以验证 DNS 是否能正确解析到 IP 地址。
- **测量网络延迟 (Latency)**：`time=` 值反映了数据包往返所需的时间，是衡量网络质量的关键指标。

## 1. `ping` 命令输出解读

理解 `ping` 的输出是有效使用它的第一步。

**示例输出：**

```bash
$ ping baidu.com
PING baidu.com (39.156.66.10) 56(84) bytes of data.
64 bytes from 39.156.66.10 (39.156.66.10): icmp_seq=1 ttl=52 time=29.3 ms
64 bytes from 39.156.66.10 (39.156.66.10): icmp_seq=2 ttl=52 time=29.2 ms
--- baidu.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 29.215/29.257/29.300/0.042 ms
```

**关键字段详解：**

| 字段                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| `from ...`           | 响应主机的 IP 地址。                                         |
| `icmp_seq`           | ICMP 报文的序列号，从 1 开始递增。                           |
| `ttl` (Time to Live) | **生存时间**。数据包在网络中可通过的路由器（跳数）上限，每经过一个路由器，该值减 1。可用于大致判断目标主机的操作系统类型。 |
| `time`               | **往返时间 (RTT)**。数据包从发送到收到响应所用的时间（毫秒），**值越小表示网络延迟越低**。 |
| `packet loss`        | **丢包率**。反映了网络连接的稳定性，理想情况下应为 `0%`。    |
| `rtt min/avg/max`    | RTT 的最小值 / 平均值 / 最大值，是对网络延迟和抖动的综合评估。 |

## 2. `ping` 常用选项

| 选项              | 含义                                                         | 示例                   |
| ----------------- | ------------------------------------------------------------ | ---------------------- |
| **`-c count`**    | **Count**。发送指定**数量**的报文后自动停止。                | `ping -c 4 8.8.8.8`    |
| **`-i interval`** | **Interval**。设置发送每个报文之间的**间隔**时间（秒），默认为 1 秒。 | `ping -i 0.2 8.8.8.8`  |
| **`-W timeout`**  | **Timeout**。设置等待响应的**超时**时间（秒）。如果在此时间内未收到响应，则认为该报文丢失。 | `ping -W 1 8.8.8.8`    |
| `-s size`         | **Size**。设置 ICMP 数据包的**大小**（字节）。可用于测试 MTU (最大传输单元)。 | `ping -s 1472 8.8.8.8` |
| `-I interface`    | **Interface**。指定从哪个**网络接口**发送报文，用于有多网卡的主机。 | `ping -I eth1 8.8.8.8` |

## 3. 高级应用: 扫描局域网存活主机

您的脚本是一个很好的 `ping` 应用范例，它展示了如何自动化地探测一个网段。下面是经过注释和优化的版本。

### 3.1. 基础脚本 (串行扫描)

这是您提供的原始脚本，它逐个 IP 进行探测，简单易懂。

```bash
#!/bin/bash

# 提示用户输入网段的前三个部分
read -p "请输入您要扫描的网段 (例如 192.168.1): " lan

echo "正在扫描网段 ${lan}.0/24 ..."

# 循环遍历 1 到 254 的所有主机地址
for (( i = 1; i <= 254; i++ )); do
    address="${lan}.${i}"

    # -c1: 只发送一个包
    # -W0.5: 等待超时为 0.5 秒 (原始的 0.01 可能过于激进)
    # &>/dev/null: 丢弃所有正常和错误输出，保持界面干净
    ping ${address} -c1 -W0.5 &>/dev/null

    # $?: 获取上一条命令的退出状态码。0 表示成功 (主机存活)
    if [[ $? == 0 ]]; then
        echo -e "\033[32mHost ${address} is up\033[0m"
    fi
done

echo "扫描完成。"
```

### 3.2. 优化脚本 (并行扫描)

串行扫描效率较低，因为每个不在线的主机都会让脚本等待超时。我们可以通过并行化来**极大提升扫描速度**。

```bash
#!/bin/bash

read -p "请输入您要扫描的网段 (例如 192.168.1): " lan

echo "正在并行扫描网段 ${lan}.0/24 ..."

for (( i = 1; i <= 254; i++ )); do
    # 将每个 ping 命令放入后台执行 (&)
    (
        address="${lan}.${i}"
        ping ${address} -c1 -W0.5 &>/dev/null
        if [[ $? == 0 ]]; then
            echo -e "\033[32mHost ${address} is up\033[0m"
        fi
    ) &
done

# wait 命令会等待所有后台任务执行完毕
wait
echo "扫描完成。"
```

### 3.3. 更专业的工具推荐

虽然 `ping` 脚本很实用，但对于网络扫描这类任务，使用专门设计的工具会更高效、更强大。

**`fping`**: 一个专门用于 `ping` 多个主机的工具，性能远高于 `ping` 脚本。

```bash
# 安装: yum install fping -y
# 使用: -a 表示只显示存活主机, -g 表示生成一个地址范围
fping -a -g 192.168.1.0/24
```

**`nmap`**: 强大的网络扫描与安全审计工具，主机发现是其基本功能。

```bash
# 安装: yum install nmap -y
# 使用: -sn 表示只进行 ping 扫描，不进行端口扫描
nmap -sn 192.168.1.0/24
```

# rsync 深度使用与自动化备份指南

> 本指南旨在提供一份关于 `rsync` (Remote Sync) 命令的全面深度手册。内容从 `rsync` 的核心概念与常用选项入手，逐步深入到远程同步、自动化备份脚本，并最终提供一个功能完备、可重复使用的通用同步脚本实践。

## 1. `rsync` 简介

`rsync` 是一款功能强大且极其灵活的文件同步工具。它通过其著名的 “delta-transfer” 算法，可以实现本地和远程系统之间的高效增量同步，即只传输文件的差异部分，从而大大节省带宽和时间。

**核心优势**：

- **增量同步**：只传输修改过的文件内容。
- **保持属性**：可以完整地保留文件的权限、时间戳、所有者等元数据。
- **高灵活性**：支持本地复制、远程推送、远程拉取等多种模式。
- **安全性**：可通过 SSH 协议进行加密传输。

## 2. 核心概念与常用选项

### 2.1 关键：源路径末尾的斜杠 (`/`)

这是 `rsync` 中最重要也最容易混淆的概念，它决定了同步的是目录本身还是目录的内容。

**不带斜杠 (`/`)**：`rsync -a /path/source /path/destination`

- **效果**：将 `source` **目录本身** 复制到 `destination` 目录下，最终路径为 `/path/destination/source`。

**带斜杠 (`/`)**：`rsync -a /path/source/ /path/destination`

- **效果**：将 `source` 目录下的**所有内容**直接复制到 `destination` 目录下。

### 2.2 常用选项详解

| 选项                       | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| **`-a, --archive`**        | **归档模式**。是 `rsync` 最常用的选项，相当于 `-rlptgoD` 的组合，可以递归同步并保留绝大多数文件属性。 |
| **`-v, --verbose`**        | **详细模式**。显示同步过程中的详细信息。                     |
| **`-h, --human-readable`** | **人性化显示**。以 K, M, G 等易读的单位显示文件大小。        |
| **`-P`**                   | 相当于 `--progress --partial`。显示每个文件的传输进度，并支持断点续传。 |
| **`--delete`**             | **删除模式**。让目标目录与源目录严格保持一致，会**删除**目标目录中源目录不存在的文件。**此为危险操作，使用前请确认！** |
| **`-n, --dry-run`**        | **试运行模式**。只模拟同步过程，列出将要执行的操作，但**不实际修改任何文件**。**强烈建议在执行带有 `--delete` 的命令前，先用此参数进行测试。** |
| `--exclude=PATTERN`        | **排除**。忽略掉匹配指定模式的文件或目录，例如 `--exclude="*.log"`。 |
| `--chown=USER:GROUP`       | 在目标端强制设置同步后文件的所有者和所属组。                 |
| `--chmod=PERMS`            | 在目标端强制设置同步后文件的权限，例如 `--chmod=D755,F644` (目录755, 文件644)。 |

## 3. 基础与远程同步示例

### 3.1 本地目录同步

```bash
# 将 source 目录下的内容同步到 destination 目录
# -a 归档模式, -v 显示详情, -h 人性化显示, -P 显示进度
rsync -avhP --delete source/ destination/
```

### 3.2 推送到远程服务器 (Push)

```bash
# 将本地的 /path/source/ 目录内容，推送到远程服务器的目标目录
rsync -avhP --delete /path/source/ root@172.24.0.246:/remote/destination/
```

### 3.3 从远程服务器拉取 (Pull)

```bash
# 将远程服务器的目录内容，拉取到本地
rsync -avhP --delete root@172.24.0.246:/remote/source/ /local/destination/
```

## 4. 高级应用：NAS 备份自动化

这是一个结合了 `wol` (网络唤醒)、`nfs` 挂载和 `ssh` 远程关机的完整备份流程示例。

### 4.1 场景说明

脚本的目标是：自动唤醒一台 NAS 设备，挂载其 NFS 共享，使用 `rsync` 将本地数据备份过去，完成后再自动关闭 NAS，实现无人值守的节能备份。

### 4.2 脚本分解与解析

```bash
#!/bin/bash
SOURCE_DIRS=(Backup Photos Documents)
NAS_NFS_PATH="172.24.0.249:/fs/1000/nfs"
LOCAL_MOUNT_POINT="/fnos"
REMOTE_TARGET="root@172.24.0.246:/backup"
NAS_MAC="70:85:c2:6d:f4:f6"
REMOTE_SSH_HOST="root@172.24.0.246"

# 1. 唤醒 NAS
echo "[$(date)] 正在发送网络唤醒信号到 ${NAS_MAC}..."
wol ${NAS_MAC}

# 2. 等待 NAS 启动
echo "[$(date)] 等待 5 分钟让 NAS 完全启动..."
sleep 300

# 3. 挂载 NFS 共享
echo "[$(date)] 正在挂载 NFS 共享..."
mount -t nfs ${NAS_NFS_PATH} ${LOCAL_MOUNT_POINT}

# 4. 循环执行 rsync 同步
for DIR in "${SOURCE_DIRS[@]}"; do
    echo "[$(date)] 正在同步目录: ${DIR}..."
    # --no-perms: 不同步权限
    # --chown & --chmod: 在目标端强制设置权限和所有者
    # 这在源和目标文件系统不兼容（如 Linux ext4 -> NAS 的 ZFS/Btrfs）时非常有用
    rsync -avhP --delete --no-perms --chown=root:root --chmod=F644,D755 \
        "${LOCAL_MOUNT_POINT}/${DIR}/" "${REMOTE_TARGET}/${DIR}/"
done

# 5. 远程修复权限 (作为双重保险)
echo "[$(date)] 正在远程修复权限..."
ssh ${REMOTE_SSH_HOST} 'find /backup/ -type d -exec chmod 755 {} \;'
ssh ${REMOTE_SSH_HOST} 'find /backup/ -type f -exec chmod 644 {} \;'

# 6. 等待并关闭 NAS
echo "[$(date)] 同步完成，等待 5 分钟后关闭 NAS..."
sleep 300
ssh ${REMOTE_SSH_HOST} 'init 0'

# 7. 卸载本地挂载点
umount ${LOCAL_MOUNT_POINT}
echo "[$(date)] 任务完成。"
```

## 5. 终极实践：构建通用同步脚本

### 5.1 脚本功能与设计

- **多任务同步**：通过配置数组，可以一次性执行多个源到目标的同步任务。
- **模式化操作**：支持“保留权限”、“统一权限”和“试运行”三种模式。
- **安全锁机制**：通过锁文件防止脚本重复执行，避免冲突。
- **彩色日志**：输出带有颜色和时间戳的日志，清晰易读。
- **错误处理**：能捕获 `rsync` 的退出码，并对不同类型的错误进行提示。

### 5.2 如何使用

1. 将下面的脚本内容保存为 `sync.sh` 文件。
2. 赋予执行权限：`chmod +x sync.sh`。
3. **核心步骤**：修改脚本开头的 `SYNC_SOURCES` 和 `SYNC_DESTINATIONS` 数组，定义您自己的同步任务。
4. 执行脚本，并提供一种操作模式作为参数：
    - **试运行 (最安全)**：`./sync.sh dry-run`
    - **保留源文件权限**：`./sync.sh preserve`
    - **在目标端强制统一权限**：`./sync.sh unify`

### 5.3 脚本源码 (`sync.sh`)

*(此处保留并展示您提供的完整 `sync.sh` 脚本，其本身已非常完善)*

```bash
#!/bin/bash
#
# 功能: 终极版 rsync 脚本，支持多目录同步、权限模式、彩色日志、锁机制和试运行。
#

# ========================= 1. 配置区域 =========================
# --- 多目录同步配置 ---
# 请确保 SYNC_SOURCES 和 SYNC_DESTINATIONS 两个数组的元素数量完全相同！
SYNC_SOURCES=(
    "/data/src/documents/"
    "/data/src/photos/"
)
SYNC_DESTINATIONS=(
    "user@remote:/backup/documents/"
    "user@remote:/backup/photos/"
)

# --- 其他配置 ---
LOCK_FILE="/tmp/rsync_sync.lock"
SSH_OPTS="" # 可选: -e 'ssh -i /path/to/key'

# ========================= 2. 脚本初始化 =========================
C_RESET='\033[0m'; C_GREEN='\033[32m'; C_YELLOW='\033[33m'; C_RED='\033[31m'; C_BLUE='\033[34m'
log_info() { echo -e "${C_GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] - $1${C_RESET}"; }
log_warn() { echo -e "${C_YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] - $1${C_RESET}"; }
log_error() { echo -e "${C_RED}[$(date +'%Y-%m-%d %H:%M:%S')] - $1${C_RESET}"; }
log_header() { echo -e "${C_BLUE}==================== $1 ====================${C_RESET}"; }
abort() { log_error "$1"; exit 1; }
cleanup() { rm -f "$LOCK_FILE"; }
trap cleanup EXIT INT TERM

# ========================= 3. 主程序 =========================
if ! command -v rsync &> /dev/null; then abort "rsync 命令未找到，请先安装。"; fi

if [ -e "$LOCK_FILE" ]; then
    OLD_PID=$(cat "$LOCK_FILE")
    if ps -p "$OLD_PID" > /dev/null; then
        log_warn "脚本已在运行 (PID: $OLD_PID)。为防止冲突，本次任务退出。"
        exit 0
    else
        log_warn "发现残留的锁文件，但进程不存在，已自动清理。"
    fi
fi
echo $$ > "$LOCK_FILE"

MODE="$1"
if [ -z "$MODE" ]; then
    echo "用法: $0 [preserve|unify|dry-run]"
    abort "未提供操作模式。"
fi

log_info "脚本启动，选择模式: $MODE"

if [ ${#SYNC_SOURCES[@]} -ne ${#SYNC_DESTINATIONS[@]} ]; then
    abort "配置错误: 源和目标的数量不匹配！"
fi

for i in "${!SYNC_SOURCES[@]}"; do
    SRC=${SYNC_SOURCES[$i]}
    DST=${SYNC_DESTINATIONS[$i]}
    log_header "任务 $((i+1))/${#SYNC_SOURCES[@]}"
    log_info "源: $SRC"; log_info "目标: $DST"

    RSYNC_BASE_ARGS=("-rtz" "--delete" "--partial") # -r: recursive, -t: times, -z: compress
    RSYNC_EXTRA_ARGS=()

    case "$MODE" in
        "preserve") RSYNC_EXTRA_ARGS+=("-pgo" "-v");; # -p: perms, -g: group, -o: owner
        "unify") RSYNC_EXTRA_ARGS+=("-v" "--chmod=D755,F644");;
        "dry-run") RSYNC_EXTRA_ARGS+=("-pgo" "-v" "-n" "--info=stats2");;
        *) abort "未知模式: '$MODE'。";;
    esac

    CMD_TO_RUN="rsync ${RSYNC_BASE_ARGS[*]} ${RSYNC_EXTRA_ARGS[*]} ${SSH_OPTS} \"$SRC\" \"$DST\""
    log_info "即将执行命令: $(echo ${CMD_TO_RUN} | xargs)"
    
    eval "${CMD_TO_RUN}"
    RSYNC_STATUS=$?

    if [ ${RSYNC_STATUS} -eq 0 ]; then
        log_info "任务 $((i+1)) 同步成功！"
    elif [ ${RSYNC_STATUS} -eq 24 ]; then
        log_warn "任务 $((i+1)) 同步完成，但部分文件在传输中消失 (退出码: 24)。"
    else
        log_error "任务 $((i+1)) 同步失败！(退出码: ${RSYNC_STATUS})。"
    fi
    echo
done

log_info "所有同步任务均已处理完毕。"
exit 0
```

# `mwget`：一款多线程命令行下载工具

本指南将详细介绍 `mwget` 的安装与使用。`mwget` 是一款开源的、支持多线程的命令行下载工具，可以看作是标准 `wget` 命令的一个增强版。

## 1.  `mwget` 简介与适用场景

**`mwget` 是什么？** `mwget` (Multi-thread Wget) 模仿了 `wget` 的操作方式，但其核心优势在于**支持多线程下载**。它会将一个文件分割成多个部分，同时开启多个连接进行下载，最后再将这些部分合并成一个完整的文件。

**何时使用 `mwget`？** 当您需要从一个带宽充足的服务器下载单个大文件时，`mwget` 通常能比单线程的 `wget` 获得**显著的速度提升**。对于下载大量小文件，其优势不明显。

## 2. 从源码编译安装 (RHEL/CentOS 7+)

`mwget` 并非主流发行版的默认软件包，通常需要从源码进行编译安装。

### 2.1 安装编译依赖

```bash
# 安装编译所需的 C++ 编译器、openssl 开发库和 intltool
yum install -y gcc-c++ openssl-devel intltool
```

### 2.2 下载源码包

```bash
# 从官方源下载 mwget 0.1.0 的源码包
wget http://jaist.dl.sourceforge.net/project/kmphpfm/mwget/0.1/mwget_0.1.0.orig.tar.bz2
```

### 2.3 编译与安装

```bash
# 1. 解压源码包并进入目录
tar xf mwget_0.1.0.orig.tar.bz2 && cd mwget_0.1.0.orig

# 2. 运行配置脚本
./configure

# 3. (关键修复) 修正源码中缺失的头文件引用
# 在较新的编译器 (GCC) 版本下，旧的源码可能会因缺少 <cstring> 头文件而编译失败。
# 以下三条命令为其添加必要的头文件引用。
sed -i '/^#include <iostream>/a #include <cstring>' src/httpplugin.cpp
sed -i '/^#include <iostream>/a #include <cstring>' src/ftpplugin.cpp
sed -i '/^#include <errno.h>/a #include <cstring>' src/downloader.cpp

# 4. 执行编译和安装
make -j$(nproc) && make install
```

## 3. `mwget` 使用方法

`mwget` 的使用非常简单，与 `wget` 类似。

### 3.1 基本下载

```bash
# 直接跟上文件的 URL 即可开始下载
mwget http://mirror.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso
```

### 3.2 常用选项

| 选项            | 含义                                  | 示例                             |
| --------------- | ------------------------------------- | -------------------------------- |
| **`-n <数字>`** | **(核心)** 指定下载所用的**线程数**。 | `mwget -n 8 <URL>`               |
| `-o <文件名>`   | 将下载的文件**另存为**指定的文件名。  | `mwget -o centos.iso <URL>`      |
| `-p <目录>`     | 将文件下载到指定的**目录**中。        | `mwget -p /tmp/downloads/ <URL>` |
| `-r <数字>`     | 下载失败后的**重试次数**。            | `mwget -r 5 <URL>`               |
| `-t <秒>`       | 连接**超时**时间。                    | `mwget -t 10 <URL>`              |

### 3.3 实践示例

```bash
# 使用 16 个线程下载一个大文件，并将其保存到 /tmp 目录
mwget -n 16 -p /tmp/ http://example.com/large-video-file.mp4
```