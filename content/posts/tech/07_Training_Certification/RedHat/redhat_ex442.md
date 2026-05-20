+++
date = '2026-05-20T09:51:07+08:00'
draft = false
title = 'RedHat_EX442'
slug = "9vutuqcycr"
description = "RedHat_EX442"
summary = "RedHat_EX442"
categories = ["📒考试"]
tags = ["RedHat"]
comments = true
series = ["RedHat 认证考试"]
showSeries = true
+++

# EX442 解法试题

## 1. 查看硬件参数

![image-20240822152314679](https://typora.fanyops.com/documents/2024/0822/image-20240822152314679.png)

```bash
# 在 node1 上，分析 dmidecode.out 文件中 cpu 每核心的 L1 和 L2 cache 容量。
#（有时候会考察每个核心的L1d或者L1i cache的容量，最后得到的结果需要除以2，因为L1 Cache由data cache和instruction cache组成）

[root@node1 ~]# dmidecode --from-dump dmidecode > dmidecode.txt
[root@node1 ~]# cat dmidecode.txt | grep -i l1-cache -A5
 Socket Designation: L1-Cache
 Configuration: Enabled, Not Socketed, Level 1
 Operational Mode: Write Back
 Location: Internal
 Installed Size: 1280 kB
 Maximum Size: 1280 kB
--
 Socket Designation: L1-Cache
 Configuration: Enabled, Not Socketed, Level 1
 Operational Mode: Write Back
 Location: Internal
 Installed Size: 1280 kB
 Maximum Size: 1280 kB
[root@node1 ~]# cat dmidecode.txt | grep -i l2-cache -A5
 Socket Designation: L2-Cache
 Configuration: Enabled, Not Socketed, Level 2
 Operational Mode: Varies With Memory Address
 Location: Internal
 Installed Size: 20 MB
 Maximum Size: 20 MB
--
 Socket Designation: L2-Cache
 Configuration: Enabled, Not Socketed, Level 2
 Operational Mode: Varies With Memory Address
 Location: Internal
 Installed Size: 20 MB
 Maximum Size: 20 MB
[root@node1 ~]# cat dmidecode.txt | grep Count
 Core Count: 20
 Thread Count: 40
 Core Count: 20
 Thread Count: 40

# 计算单核Cache并填到题目上
# 总的L1 Cache为1280KB
# 总的L2 Cache为20MB
# CPU的核心数量为20，单核L1 Cache为1280KB/20=64KB，单核L1d和L1i Cache为1280KB/20/2=32KB，单核L2 Cache为20MB/20=1MB=1024KB。
```

## 2. 分析sar的输出结果

![image-20240822152603693](https://typora.fanyops.com/documents/2024/0822/image-20240822152603693.png)

```bash
# 在 node1 上，分析采集自其它计算机的 sar.data 文件，回答以下问题：系统的最高进程数量是多少？

[root@node1 ~]# yum install sysstat -y
[root@node1 ~]# bunzip2 sar.data.bz2
[root@node1 ~]# sar -q -f sar.data | awk '{print $4}' | sort -n | tail -1
807
```

## 3. 分析sar的磁盘输出

```bash
# 在 node1 上，分析 sar.data 文件，回答以下问题：（注：这个文件采集自含有逻辑卷的计算机，请忽略逻辑卷，而只分析物理驱动器）关于拥有最高写 I/O 的设备，最高的写 I/O 是？（四舍五入到百位）

[root@node1 ~]# sar -d -p -f sar.data | awk '{print $6}' | sort -n | tail -1
91189.60
# 最高的写IO是91189.60，四舍五入到百位就是91200

[root@node1 ~]# sar -d -p -f sar.data | grep 91189.60
01:10:33 AM       sdb    180.00      0.00  91189.60    506.61    105.42    587.26      4.46     80.32
# 最高的写IO设备是sdb

# 什么时候，以分钟标识，相对于监视周期的开始，系统在什么时候出现了导致内存内容被写入磁盘的突发活动?
[root@node1 ~]# sar -d -p -f sar.data | less
01:00:27 AM       DEV       tps     rkB/s     wkB/s   areq-sz    aqu-sz     await     svctm     %util
01:04:58 AM       sdc      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
01:05:03 AM       sda    126.60      0.00  63801.40    503.96    117.13    817.65      6.47     81.88
01:05:03 AM  dev253-0    154.20      0.00  77934.20    505.41    117.94    675.31      5.31     81.88
01:05:03 AM  dev253-1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

## 4. 分析sar的网络输出

```bash
# 在 node1 上，分析采集自其它计算机的 sar.data 文件，回答以下问题：在单一网卡上持续了多长时间（分钟）的爆发性流量？
# 1. 确定排行前20的流量值
[root@node1 ~]# sar -n DEV -f sar.data | awk '{print $4}' | sort -n | tail -20
6486.20
6510.40
6660.00
6724.60
6777.00
6788.40
6790.00
6911.20
7003.80
8311.60
11038.20
11534.80
14060.60
14174.80
16386.80
18895.40
19228.20
19547.20
22112.60
22265.60
# 2. 根据排名前20的流量爆发值查看爆发性流量属于哪个网卡？
[root@node1 ~]# for i in `sar -n DEV -f sar.data | awk '{print $4}' | sort -n | tail -20`; do 
sar -n DEV -f sar.data | grep $i | awk '{print $3,$4,$1}';
done
eth0 6486.20 01:21:03
eth0 6510.40 01:19:23
eth0 6660.00 01:39:23
eth0 6724.60 01:17:08
eth0 6777.00 01:38:48
eth0 6788.40 01:43:28
eth0 6790.00 01:15:28
eth0 6911.20 01:40:33
eth0 7003.80 01:13:48
eth1 8311.60 01:07:48
eth1 11038.20 01:09:28
eth1 11534.80 01:09:23
eth1 14060.60 01:06:03
eth1 14174.80 01:07:43
eth1 16386.80 01:07:13
eth1 18895.40 01:08:53
eth1 19228.20 01:06:38
eth1 19547.20 01:09:58
eth1 22112.60 01:10:33
eth1 22265.60 01:08:18
# 3. 分析eth1是什么什么时候开始出现爆发性流量，持续了多久
[root@node1 ~]# sar -n DEV -f sar.data | grep eth1 | less
01:05:33 AM      eth1   2854.80  10199.60    198.95  14527.41      0.00      0.00      0.00      0.00
......
01:11:03 AM      eth1   1179.60   3729.00     82.04   5328.48      0.00      0.00      0.00      0.00

# 分析后发现从1点05开始~1点11结束，持续了6分钟。将6填入答案
```

## 5. 分析system call

![image-20240822152635285](https://typora.fanyops.com/documents/2024/0822/image-20240822152635285.png)

```bash
# 在 node1 上，找到 updatedb 命令的最频繁 system call
[root@node1 ~]# yum install strace mlocate -y
[root@node1 ~]# strace -c updatedb
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 35.05    0.290398          24     12097           lstat
 15.42    0.127774          16      7577         1 openat
 12.58    0.104261           8     12088           getdents64
  8.05    0.066700           5     12089           fcntl
  6.77    0.056102           7      7577           close
  5.86    0.048514           8      6045           chdir
  4.81    0.039814           6      6065           fstat
  4.49    0.037220           6      6044           fchdir
  4.42    0.036656           6      6052           poll
  1.57    0.012989          50       259           write
  0.43    0.003586          99        36           read
  0.23    0.001931        1931         1           execve
  0.08    0.000680         113         6           mprotect
  0.08    0.000627          62        10           mmap
  0.04    0.000364          36        10           brk
  0.02    0.000184          92         2         1 arch_prctl
  0.02    0.000183          91         2           munmap
  0.02    0.000139          46         3           lseek
  0.01    0.000113         113         1         1 access
  0.01    0.000056          18         3           rt_sigaction
  0.01    0.000046          11         4           rt_sigprocmask
  0.01    0.000045          22         2           socket
  0.01    0.000045          22         2         2 connect
  0.00    0.000026          26         1           rename
  0.00    0.000022          22         1           chown
  0.00    0.000015          15         1           sysinfo
  0.00    0.000013          13         1           getpid
  0.00    0.000010          10         1           chmod
------ ----------- ----------- --------- --------- ----------------
100.00    0.828513          10     75980         5 total

# 将 lstat 填入答案
```

## 6. 限制内存分配

![image-20240822152646867](https://typora.fanyops.com/documents/2024/0822/image-20240822152646867.png)

```bash
# 配置 node1，当 greedy 程序运行时，报错信息如下：“unable to allocate memory”，
# 当 checklimit 命令运行时，显示：“Success: Address space limit is okay”，设置要求重启也生效。
[root@node1 ~]# echo 'vm.overcommit_memory=2' >> /etc/sysctl.conf
[root@node1 ~]# sysctl -p
vm.overcommit_memory = 2

[root@node1 ~]# chmod +x greedy checklimit 
[root@node1 ~]# ./greedy 
Process 11083 allocating 536870912 bytes of memory.
Process 11083 will sleep for 30 seconds.
Process 11084 allocating 536870912 bytes of memory.
Error: 11084 unable to allocate memory.
^C
[root@node1 ~]# ./checklimit 
Success: Address space limit is okay

# 如果更改了内核可调项之后运行 greedy 程序，系统死机了，那么你就把节点断电再开机，然后看节点上有没有空余的磁盘，根据磁盘的情况，分 2G/4G/8G 的 swap，并且写入 /etc/fstab 持久性的挂载 swap 空间，然后再次运行 greedy 程序。
# 这个做法主要是为了防止，红帽评分脚本运行greedy程序死机了，导致其它的没有评分的题目没办法继续评分导致考试挂掉。
```

## 7. 配置内存共享

![image-20240822152703319](https://typora.fanyops.com/documents/2024/0822/image-20240822152703319.png)

```bash
# 在 node1 上，配置 SYSV 共享内存的数量在 1GiB 至 1.5GiB 之间，重启后也生效。
# 只要配置最大值在1GiB~1.5GiB之间即可。这里我们使用1.4GiB来做。
# 由于kernel.shmall单位页大小，单页大小是4k，所以我们需要换算成KB，然后再除以页大小。
[gzy@EX442 ~]$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
1.4*1024*1024/4
367001

[root@node1 ~]# echo 'kernel.shmall=367001' >> /etc/sysctl.conf 
[root@node1 ~]# sysctl -p
vm.overcommit_memory = 2
kernel.shmall = 367001

[root@node1 ~]# ipcs -l

------ Messages Limits --------
max queues system wide = 32000
max size of message (bytes) = 8192
default max size of queue (bytes) = 16384

------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 18014398509465599
max total shared memory (kbytes) = 1468004
min seg size (bytes) = 1

------ Semaphore Limits --------
max number of arrays = 32000
max semaphores per array = 32000
max semaphores system wide = 1024000000
max ops per semop call = 500
semaphore max value = 32767
```

## 8. 分析虚拟内存占用

```bash
# 在 node1 上，计算 memory_usage 软件的虚拟内存占用（单位为 page）
# 查看一个程序使用的内存占用（page，虚拟内存还是物理内存要看清题目）
# 例如查看 sshd 占用【考试的时候会给一个特定的程序】
[root@node1 ~]# ps aux | grep crond
root        1178  0.0  0.1 237676  3520 ?        Ss   13:38   0:00 /usr/sbin/crond -n
root       11107  0.0  0.0 221928  1104 pts/0    S+   20:28   0:00 grep --color=auto crond
[root@node1 ~]# ps axo pid,comm,vsz | grep 1178
   1178 crond           237676
237676/4
```

## 9. 配置systemtap

```bash
# 在 node1 上，root 家目录下有一个文件 count_jiffies.stp，配置系统使得 root 用户可以运行这个脚本
# (http://debuginfo.centos.org/8-stream/x86_64/Packages/)
cat > /etc/yum.repos.d/baseos-debuginfo.repo << END
[baseos-debuginfo]
baseurl = http://192.168.122.248/web/mirror/rocky/8.10/baseos-debuginfo/
enabled = 1
gpgcheck = 0
name = baseos-debuginfo
END

[root@node1 ~]# yum install systemtap -y
[root@node1 ~]# stap-prep
[root@node1 ~]# stap -v count_jiffies.stp

[root@node1 ~]# yum list | grep kernel-debuginfo
kernel-debuginfo.x86_64                                4.18.0-348.el8                           @kernel-debuginfo
kernel-debuginfo-common-x86_64.x86_64                  4.18.0-348.el8                           @kernel-debuginfo
[root@node1 ~]# uname -a
Linux node1.domain1.example.com 4.18.0-348.el8.x86_64 #1 SMP Mon Oct 4 12:17:22 EDT 2021 x86_64 x86_64 x86_64 GNU/Linux
```

## 10. 针对cache的利用率，选择合适的应用

![image-20240822152717141](https://typora.fanyops.com/documents/2024/0822/image-20240822152717141.png)

```bash
# 在 node1 上，账户的目录有两个版本的应用程序：cache-a 和 cache-b。两个版本的应用程序功能一样，但是其中一个在使用缓存方面不合理。
# 确认哪个应用程序有更好的缓存性能并复制这个程序到/usr/local/bin

[root@node1 ~]# yum install valgrind -y
[root@node1 ~]# chmod +x cache-a cache-b 
[root@node1 ~]# valgrind --tool=cachegrind ./cache-a
[root@node1 ~]# valgrind --tool=cachegrind ./cache-b
[root@node1 ~]# cp cache-b /usr/local/bin/
[root@node1 ~]# restorecon -RvF /usr/local/bin/
Relabeled /usr/local/bin/cache-b from unconfined_u:object_r:bin_t:s0 to system_u:object_r:bin_t:s0
```

![image-20240822095958788](https://typora.fanyops.com/documents/2024/0822/image-20240822095958788.png)

## 11. 配置网络buffer

![image-20240822152743516](https://typora.fanyops.com/documents/2024/0822/image-20240822152743516.png)

```bash
# 在 node1 上，配置网络缓存使得每个 UDP 连接（输入、输出），保证能有最小 128 KiB最大 192KiB 的 buffer，重启后仍然生效

[root@node1 ~]# sysctl -a | grep mem | grep net.core
net.core.optmem_max = 81920
net.core.rmem_default = 212992
net.core.rmem_max = 212992
net.core.wmem_default = 212992
net.core.wmem_max = 212992

128*1024=131072
192*1024=196608

cat >> /etc/sysctl.conf << END
net.core.rmem_default = 131072
net.core.rmem_max = 196608
net.core.wmem_default = 131072
net.core.wmem_max = 196608
END

sysctl -p
```

## 12. 配置sar

![image-20240822153005420](https://typora.fanyops.com/documents/2024/0822/image-20240822153005420.png)

```bash
# 在 node1 上，配置每隔 2 分钟采集一次 sar 数据
cp -r /usr/lib/systemd/system/sysstat-collect.timer /etc/systemd/system
vi /etc/systemd/system/sysstat-collect.timer
[Unit]
Description=Run system activity accounting tool every 10 minutes
[Timer]
OnCalendar=*:00/03
#每隔三分钟收集一次
[Install]
WantedBy=sysstat.service

systemctl daemon-reload
systemctl start sysstat-collect.timer
systemctl enable sysstat-collect.timer

# 然后过几分钟之后到/var/log/sa/下看看是否有收集到的信息
sar -f sa08
```

## 13. 基于BDP配置网络buffer

![image-20240822173219473](https://typora.fanyops.com/documents/2024/0822/image-20240822173219473.png)

```bash
# node1 和其它系统之间的绝大多数网络连接都将经由一个低地轨道卫星链路。
# 连接的延迟为 500 毫秒，带宽为1.5Mib/s(mebibits/second)。调整系统，使得所有的 TCP 连接都能：
# 1.用于缓存每个连接的缓存最小值，足够适用于这个延迟以及带宽
# 2.用于缓存每个连接的缓存缺省值等于用于缓存的最小值
# 3.用于缓存每个连接的缓存最大值，等于 1.5 倍缓存最小值
# 设置重启后仍然生效

# BDP = 带宽*延迟
1.5Mib * 500ms = (1.5*1024*1024 b/s) * (0.5s) = 786432 bit
786432 bit * 1/8 B/b = 98304 Bytes = 96 Kib
98304 * 1.5 = 147456

1. 缓存单位为页，页大小为4 KiB，96KiB / 4KiB = 24
2. 缓存每个链接的缓存最小值为 98304
3. 缓存每个链接的缓存缺省值等于最小值 98304
4. 缓存每个连接的缓存最大值等于 1.5 倍缓存最小值为 147456

[root@node1 ~]# sysctl -a | grep tcp | grep mem >> /etc/sysctl.conf 
[root@node1 ~]# vi /etc/sysctl.conf
[root@node1 ~]# sysctl -p
vm.overcommit_memory = 2
kernel.shmall = 367001
net.core.rmem_default = 131072
net.core.rmem_max = 196608
net.core.wmem_default = 131072
net.core.wmem_max = 196608
net.ipv4.tcp_mem = 24 24 36
net.ipv4.tcp_rmem = 98304 98304 147456
net.ipv4.tcp_wmem = 98304 98304 147456
```

## 14. 配置服务内存

```bash
# 在 node1 上，创建一个服务满足以下要求：
# 1.服务名为：monitoring_service
# 2.此服务运行/usr/local/bin/monitoring_app
# 3.此服务随操作系统自动启动
# 4.该服务使用不得超过 10Mb 内存，如果超过，将会被终止
# 可以通过查看/var/log/messages 查看服务运行信息

[root@node1 ~]# cp monitoring_app /usr/local/bin/
[root@node1 ~]# chmod +x /usr/local/bin/monitoring_app
[root@node1 ~]# cp /usr/lib/systemd/system/sshd.service /etc/systemd/system/monitoring_service.service
[root@node1 ~]# vi /etc/systemd/system/monitoring_service.service
[root@node1 ~]# cat /etc/systemd/system/monitoring_service.service
[Unit]
Description=test

[Service]
ExecStart="/usr/local/bin/monitoring_app" &
MemoryAccounting=true
MemoryLimit=10M

[Install]
WantedBy=multi-user.target

[root@node1 ~]# restorecon -RvF /usr/local/bin/monitoring_app
Relabeled /usr/local/bin/monitoring_app from unconfined_u:object_r:bin_t:s0 to system_u:object_r:bin_t:s0
[root@node1 ~]# systemctl --now enable monitoring_service.service 
Created symlink /etc/systemd/system/multi-user.target.wants/monitoring_service.service ?? /etc/systemd/system/monitoring_service.service.
[root@node1 ~]# tail /var/log/messages 
Oct 10 01:30:28 node1 systemd[1]: sysstat-collect.service: Succeeded.
Oct 10 01:30:28 node1 systemd[1]: Started system activity accounting tool.
Oct 10 01:33:28 node1 systemd[1]: Starting system activity accounting tool...
Oct 10 01:33:28 node1 systemd[1]: sysstat-collect.service: Succeeded.
Oct 10 01:33:28 node1 systemd[1]: Started system activity accounting tool.
Oct 10 01:33:54 node1 systemd[1]: Reloading.
Oct 10 01:33:54 node1 systemd[1]: Started test.
Oct 10 01:33:54 node1 /usr/local/bin/monitoring_app[32295]: Carpe Cerevisi!
Oct 10 01:34:09 node1 /usr/local/bin/monitoring_app[32295]: 4...5...6...7...8...9...
Oct 10 01:34:24 node1 /usr/local/bin/monitoring_app[32295]: Carpe diem!
```

## 15. 配置磁盘脏页回收

```bash
# 在 node1 上，配置系统使得已经被修改的数据能够在被考虑流入磁盘前，在内存中至少保留 60 秒，重启后仍然生效

[root@node1 ~]# sysctl -a | grep dirty_e
vm.dirty_expire_centisecs = 3000
[root@node1 ~]# echo 'vm.dirty_expire_centisecs = 6100' >> /etc/sysctl.conf 
[root@node1 ~]# sysctl -p
vm.dirty_expire_centisecs = 6100
```

## 16. 分析应用性能

```bash
# 在 node1 的 root 目录中有一个压缩包，名为 application.tgz 包含了一个 client-server应用程序。
# 这些程序都被设计为在同一个系统上使用client 应用程序能够接受键盘输入并且发到服务器，输入 ctrl-D，将结束客户端程序。
# 服务端应用程序做的事情一样: 它们接受来自 client 的输入并且复制到文件/tmp/application.out.一旦进入生产环节，
# 服务端应用程序将会运行很长时间而不能重新启动，并且每天需要处理数百个连接请求。你必须分析服务端应用程序，将答案填写在下方：
# 这个应用存在什么问题？

# 这是一道选择题，数据内存泄漏 memory leak [两个空，一个填memory 一个填leak]
[root@node1 ~]# tar xf application.tgz
[root@node1 ~]# valgrind --tool=memcheck ./server
==33070== Memcheck, a memory error detector
==33070== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==33070== Using Valgrind-3.17.0 and LibVEX; rerun with -h for copyright info
==33070== Command: ./server
==33070==

^C==33070== 
==33070== Process terminating with default action of signal 2 (SIGINT)
==33070==    at 0x4F43D48: accept (in /usr/lib64/libc-2.28.so)
==33070==    by 0x400842: main (in /root/server)
==33070== 
==33070== HEAP SUMMARY:
==33070==     in use at exit: 196,608 bytes in 4 blocks
==33070==   total heap usage: 4 allocs, 0 frees, 196,608 bytes allocated
==33070== 
==33070== LEAK SUMMARY:
==33070==    definitely lost: 196,608 bytes in 4 blocks
==33070==    indirectly lost: 0 bytes in 0 blocks
==33070==      possibly lost: 0 bytes in 0 blocks
==33070==    still reachable: 0 bytes in 0 blocks
==33070==         suppressed: 0 bytes in 0 blocks
==33070== Rerun with --leak-check=full to see details of leaked memory
==33070== 
==33070== For lists of detected and suppressed errors, rerun with: -s
==33070== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

#=================================================================================
[root@node1 ~]# ./client 
Type ctrl-D to exit this utility.
pwd
123
```

## 17. 配置一个模块

```bash
# 在 node1 上，将安装一个 SCSI 控制器和一个 SCSI 磁带备份硬件。
# 请确保 SCSI 磁带控制器模块（st.ko）的加载，缺省缓存大小需要设置为 16KiB，重启后仍然生效。

[root@node1 ~]# modinfo st
[root@node1 ~]# echo 'options st buffer_kbs=16' >> /etc/modprobe.d/st.conf    
[root@node1 ~]# cat /etc/modprobe.d/st.conf 
options st buffer_kbs=16
[root@node1 ~]# echo 'st' >> /etc/modules-load.d/st.conf
[root@node1 ~]# cat /etc/modules-load.d/st.conf 
st
[root@node1 ~]# lsmod | grep st
st                     65536  0
```

## 18. 配置tuned

```bash
# 在 node1 上，创建一个 tuned profile，要求如下：
# Profile 名为 rh442
# Profile 调用自系统推荐的 profile
# 磁盘的 readahead 参数为 3092KiB
# 重启后 profile 仍然生效

[root@node1 ~]# tuned-adm recommend 
virtual-guest
[root@node1 ~]# mkdir /etc/tuned/rh442
[root@node1 ~]# vi /etc/tuned/rh442/tuned.conf
[root@node1 ~]# cat /etc/tuned/rh442/tuned.conf
[main]
include=virtual-guest

[disk]
readahead=3092
[root@node1 ~]# tuned-adm profile rh442
[root@node1 ~]# tuned-adm active 
Current active profile: rh442

# 结果以扇区的数量显示 (sector=512B)
blockdev --getra /dev/vda
3092*1024/512=6184
```

## 19. 编写一个cgroup配置文件

```bash
# 在 node1 上，创建一个 cgroup slice，名为 rh442，要求如下：
# 此 slice 需要限制 1024CPU shares 和 256M 内存
# httpd 服务调用此 slice，服务开机自动启动

[root@node1 ~]# yum install httpd -y
[root@node1 ~]# cp /usr/lib/systemd/system/httpd.service /etc/systemd/system
[root@node1 ~]# vi /etc/systemd/system/rh442.slice
[root@node1 ~]# cat /etc/systemd/system/rh442.slice
[Unit]
Description=rh442

[Slice]
MemoryAccounting=true
MemoryLimit=256M
CPUAccounting=true
CPUShares=1024

[root@node1 ~]# mkdir /etc/systemd/system/httpd.service.d
[root@node1 ~]# vi /etc/systemd/system/httpd.service.d/10-httpd.conf
[root@node1 ~]# cat /etc/systemd/system/httpd.service.d/10-httpd.conf
[Service]
Slice=rh442
[root@node1 ~]# systemctl enable --now httpd.service 
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service ?? /etc/systemd/system/httpd.service.

[root@node2 ~]# systemd-cgls | grep httpd
|   |_|_?10854 grep --color=auto httpd
| |_httpd.service
|   |_10637 /usr/sbin/httpd -DFOREGROUND
|   |_10638 /usr/sbin/httpd -DFOREGROUND
|   |_10639 /usr/sbin/httpd -DFOREGROUND
|   |_10640 /usr/sbin/httpd -DFOREGROUND
|   |_10641 /usr/sbin/httpd -DFOREGROUND
```

## 20. 分析pcp数据文件

![image-20240822173240190](https://typora.fanyops.com/documents/2024/0822/image-20240822173240190.png)

```bash
# 在 node1 上，pmlogger 日志 20141231.06.00.01.folio 中包含了大约 1 小时的系统运行数据，分析并
# 回答以下问题：
# 1.什么设备具有最高的峰值 I/O write rate
# 2.此设备的峰值 write rate 是多少 MB/s
# 3.另一个设备的的峰值 write rate 是多少 MB/s

cat 20200207.13.07.40.folio
yum –y install pcp
systemctl enable pmcd --now

pminfo | grep disk | grep write

#查看vda的
pmval -a 20150223.12.24.0 disk.dev.write_bytes | awk '{print $2}' | sort -n
#查看vdb的
pmval -a 20150223.12.24.0 disk.dev.write_bytes | awk '{print $3}' | sort -n
#对比vda和vdb最高的io谁大，第一题就填谁，写速率需要计算，结果除以1000就可以了。
```

## 21. TLB缓存调优

![image-20240822173249934](https://typora.fanyops.com/documents/2024/0822/image-20240822173249934.png)

```bash
# 在 node1 上，在/root 中有两个版本的程序，它们都分配一个 64 MiB 的共享内存段。
# 配置系统，使它可以运行这些程序中的一个或另一个：
# 1.运行程序的进程的 page table 所消耗的内存量被最小化
# 2.假设这个系统主要只运行这个程序，那么在程序运行时 TLB 刷新会最小化
# 3.配置更改在重启后仍然生效

# 程序 hugepage.shm 使用 SYSV 共享内存来分配 64MiB 内存段。程序 hugepage.fs 使用一个伪文件系统来处理内存分配。
# 需要配置系统以支持其中一个或另一个程序。不需要将系统配置为同时支持这两种功能。将决定使用的程序复制到/usr/local/bin 目录中
# 如果选择支持 hugepage.fs 程序，应该将伪文件系统挂载到目录/bigpages。该文件系统应该在系统重新启动后自动挂载。运行 hugepage 时。
# 在 fs 程序中，系统将提示输入伪文件系统挂载点下的文件名称：应使用/bigpages/memory两个程序都将在分配共享内存后暂停，并在准备就绪时提示继续。
# 这样，就可以在程序运行时检查系统，以确认内存被正确分配.

[root@node1 ~]# echo 'vm.nr_hugepages = 32' >> /etc/sysctl.conf 
[root@node1 ~]# sysctl -p
vm.nr_hugepages = 32
[root@node1 ~]# mkdir /bigpages
[root@node1 ~]# echo 'nodev /bigpages hugetlbfs defaults 0 0' > /etc/fstab
[root@node1 ~]# mount -a
[root@node1 ~]# chmod +x /usr/local/bin/hugepage.fs
[root@node1 ~]# cp hugepage.fs /usr/local/bin

# 验证
[root@node1 ~]# tail -n8 /proc/meminfo 
HugePages_Total:      32
HugePages_Free:       32
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:           65536 kB
DirectMap4k:      110448 kB
DirectMap2M:     1986560 kB
[root@node1 ~]# /usr/local/bin/hugepage.fs 
Enter a filename on your hugetlbfs filesystem mount point: /bigpages/442test
Memory allocation successful - returned address is 0x7fb57a800000
Type 'continue' and press 'Enter' to proceed: continue
First hex is 0
First hex is 3020100
Type 'continue' and press 'Enter' to proceed: ^C
[root@node1 ~]# tail -n8 /proc/meminfo 
HugePages_Total:      32
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:           65536 kB
DirectMap4k:      110448 kB
DirectMap2M:     1986560 kB
```

## 22. 应用分析

```bash
numactl --membind 1 -- bigmem 100M
cat /proc/1502/numa_maps | grep -i bind

# 在 node2 上，分析 firewalld 程序运行在哪一个 NUMA 节点上。
systemctl status firewalld
cat /proc/xxx/numa_maps | grep bind:

xxx代表processid，你通过systemctl status firewalld获得firewalld进程的process id
```

## 23. 网络分析

```bash
# 在node1上，有两个yum仓库提供ansible-doc软件包，一个yum仓库的响应时间比另外一个更好。更新node1上的yum配置，使用延迟最低的yum仓库，如下所示:
# http://a.example.com/repo
# http://b.example.com/repo

yum –y install perf
perf stat yum -y install ansible-doc

# 选context-switches/cpu-migrations/page-faults 小的，越小代表延迟越低。保留性能好的软件仓库配置。
# page-faults，cycles，context-switches 对比
[root@node1 ~]# perf stat yum -y install ansible-doc
Last metadata expiration check: 1:05:56 ago on Fri 23 Aug 2024 11:05:28 PM EDT.
No match for argument: ansible-doc
Error: Unable to find a match: ansible-doc

 Performance counter stats for 'yum -y install ansible-doc':

          1,498.75 msec task-clock                #    0.995 CPUs utilized          
                30      context-switches          #    0.020 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
            21,982      page-faults               #    0.015 M/sec                  
   <not supported>      cycles                                                      
   <not supported>      instructions                                                
   <not supported>      branches                                                    
   <not supported>      branch-misses                                               

       1.506036471 seconds time elapsed

       1.216103000 seconds user
       0.216021000 seconds sys
```

