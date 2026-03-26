+++
date = '2024-03-18T08:01:59+08:00'
draft = false
title = 'Docker 常见疑难杂症解决方案'
slug = "7k38hq60mo"
description = "Docker 常见疑难杂症解决方案"
categories = ["📒docker"]
tags = ["docker","问题集锦"]
summary = "Docker 常见疑难杂症解决方案"
+++

Docker 常见疑难杂症解决方案

## 1.Docker 迁移存储目录 

默认情况系统会将 [Docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247509626&idx=2&sn=a1612d83aec08936950cc5bed3dc96af&chksm=e918c366de6f4a706cb9502afcb7f1e73911f053f4e4dadd17ccea1abd0e616e115e51024590&scene=21#wechat_redirect) 容器存放在/var/lib/[docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247509225&idx=2&sn=df58e5a088f56e0db6c09720317da9e3&chksm=e918c1f5de6f48e3d72df38b1f1972b99e8acf2bc860bc4f06fc7946faf8c65559e3837231d1&scene=21#wechat_redirect) 目录下

**问题起因**:今天通过监控系统，发现公司其中一台服务器的磁盘快慢，随即上去看了下，发现 /var/lib/[docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247508774&idx=2&sn=8802a06a3f2cc1617ff5e32353b11c2d&chksm=e918c63ade6f4f2c6aeaddfad813000b9f0a3a5ae5b535f39976e47bdc1df2bbec05c337d5ca&scene=21#wechat_redirect) 这个目录特别大。由上述原因，我们都知道，在 /var/lib/docker 中存储的都是相关于容器的存储，所以也不能随便的将其删除掉。

那就准备迁移 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247504153&idx=1&sn=208009779ddb18d84fd198770d67656c&chksm=e918b405de6f3d1321f11b4a1be539f01a7bd79f7a37560caec825bb3055844910d2576b371e&scene=21#wechat_redirect) 的存储目录吧，或者对 /var 设备进行扩容来达到相同的目的。更多关于 dockerd 的详细参数，请点击查看 官方文档 地址。

但是需要注意的一点就是，尽量不要用软链， 因为一些 [docker ](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247502654&idx=1&sn=9458abf0f3adc055759f4498a97b50f9&chksm=e918ae22de6f273421893f8e0203130b1ca4a90805cb315cca236760b32abe37b19e4cfe9c9f&scene=21#wechat_redirect)容器编排系统不支持这样做，比如我们所熟知的 [k8s ](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247509120&idx=2&sn=7a1541742111f5faee1716aac58bbf28&chksm=e918c19cde6f488a1e67f9a83ced62449027412e90ab349027e458c9a68c15dfdadb2843db63&scene=21#wechat_redirect)就在内。

```
# 发现容器启动不了了
ERROR：cannot  create temporary directory!

# 查看系统存储情况
$ du -h --max-depth=1
```

**解决方法1**：添加软链接

```
# 1.停止docker服务
$ sudo systemctl stop docker

# 2.开始迁移目录
$ sudo mv /var/lib/docker /data/

# 3.添加软链接
# sudo ln -s /data/docker /var/lib/docker

# 4.启动docker服务
$ sudo systemctl start docker
```

**解决方法2**：改动 [docker 配置](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247501440&idx=1&sn=0e5865b7060cd2ae31edd331086a048e&chksm=e918a39cde6f2a8a254eb2253fe49f0ce3f000e92f824776cb9ec5177e8eee0554ada9ec10d9&scene=21#wechat_redirect)文件

```
# 3.改动docker启动配置文件
$ sudo vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --graph=/data/docker/

# 4.改动docker启动配置文件
$ sudo vim /etc/docker/daemon.json
{
    "live-restore": true,
    "graph": [ "/data/docker/" ]
}
```

**操作注意事项**：在迁移 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247502315&idx=2&sn=ba6611fe2bea274498dd379a58211231&chksm=e918acf7de6f25e148eb211df35c3bddf4f5ebdaecc91311f53aa6aacdfbea43efc749b25f5d&scene=21#wechat_redirect) 目录的时候注意使用的命令，要么使用 mv 命令直接移动，要么使用 cp  命令复制文件，但是需要注意同时复制文件权限和对应属性，不然在使用的时候可能会存在权限问题。如果容器中，也是使用 root  用户，则不会存在该问题，但是也是需要按照正确的操作来迁移目录。

```
# 使用mv命令
$ sudo mv /var/lib/docker /data/docker

# 使用cp命令
$ sudo cp -arv /data/docker /data2/docker
```

下图中，就是因为启动的容器使用的是普通用户运行进程的，且在运行当中需要使用 /tmp 目录，结果提示没有权限。在我们导入容器镜像的时候，其实是会将容器启动时需要的各个目录的权限和属性都赋予了。如果我们直接是 cp  命令单纯复制文件内容的话，就会出现属性不一致的情况，同时还会有一定的安全问题。

![图片](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304163547659.webp)

2.[Docker ](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247500917&idx=2&sn=8f93bd64875f72a94f222994fbe4c295&chksm=e918a169de6f287f8a3fb4fddf1bff02629753a21c28d7358f94879f9dc3c67ba12019680d42&scene=21#wechat_redirect)设备空间不足

Increase Docker container size from default 10GB on rhel7.

问题起因一：容器在导入或者启动的时候，如果提示磁盘空间不足的，那么多半是真的因为物理磁盘空间真的有问题导致的。如下所示，我们可以看到 / 分区确实满了。

```
# 查看物理磁盘空间
$ df -Th
Filesystem    Size    Used    Avail    Use%    Mounted on
/dev/vda1      40G     40G       0G    100%    /
tmpfs         7.8G       0     7.8G      0%    /dev/shm
/dev/vdb1     493G    289G     179G     62%    /mnt
```

如果发现真的是物理磁盘空间满了的话，就需要查看到底是什么占据了如此大的空间，导致因为容器没有空间无法启动。其中，[docker ](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247499520&idx=2&sn=2e4079c35f786fc29dd3971802cfd531&chksm=e9189a1cde6f130a3cb23b15e7e66ee0fe9af838f4bfe60096141b9db75c23c870e3c9e3885d&scene=21#wechat_redirect)自带的命令就是一个很好的能够帮助我们发现问题的工具。

```
# 查看基本信息
# 硬件驱动使用的是devicemapper，空间池为docker-252
# 磁盘可用容量仅剩16.78MB，可用供我们使用
$ docker info
Containers: 1
Images: 28
Storage Driver: devicemapper
 Pool Name: docker-252:1-787932-pool
 Pool Blocksize: 65.54 kB
 Backing Filesystem: extfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data Space Used: 1.225 GB
 Data Space Total: 107.4 GB
 Data Space Available: 16.78 MB
 Metadata Space Used: 2.073 MB
 Metadata Space Total: 2.147 GB
```

**解决方法**：通过查看信息，我们知道正是因为 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247500702&idx=2&sn=0fb77cd1830fe7f637bb6e4f3da2b13b&chksm=e918a682de6f2f9417f49d9d64253e12d38822df6f28458b502c2be8eddb5177d5b76153cdac&scene=21#wechat_redirect) 可用的磁盘空间不足，所以导致启动的时候没有足够的空间进行加载启动镜像。解决的方法也很简单，第一就是清理无效数据文件释放磁盘空间(清除日志)，第二就是修改 [docker 数据](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247497216&idx=2&sn=41af594fb528892a17e899f4231dc255&chksm=e918931cde6f1a0a9a350e37d282153ad4226fbf26299fa629ee2e8748c5e0109e179e844908&scene=21#wechat_redirect)的存放路径(大分区)。

```
# 显示哪些容器目录具有最大的日志文件
$ du -d1 -h /var/lib/docker/containers | sort -h

# 清除您选择的容器日志文件的内容
$ cat /dev/null > /var/lib/docker/containers/container_id/container_log_name
```

**问题起因二**：显然我遇到的不是上一种情况，而是在启动容器的时候，容器启动之后不久就显示是 unhealthy 的状态，通过如下日志发现，原来是复制配置文件启动的时候，提示磁盘空间不足。

后面发现是因为 [CentOS7 ](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247494213&idx=2&sn=10c4b4a5a21ee2a4c261577d6daaae3c&chksm=e9188f59de6f064fb617efb308126a4103eba00c55789f875e3a65c23fba0437a478dc94ba11&scene=21#wechat_redirect)的系统使用的 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247495700&idx=2&sn=1da61769a7fcf457aaf1cb8c5735313f&chksm=e9189508de6f1c1ef1f1a1806839ff91a4c4dde906640cb8d895bb89f04e7361ff040e4b3014&scene=21#wechat_redirect) 容器默认的创建大小就是 10G 而已，然而我们使用的容器却超过了这个限制，导致无法启动时提示空间不足。

```
2019-08-16 11:11:15,816 INFO spawned: 'app-demo' with pid 835
2019-08-16 11:11:16,268 INFO exited: app (exit status 1; not expected)
2019-08-16 11:11:17,270 INFO gave up: app entered FATAL state, too many start retries too quickly
cp: cannot create regular file '/etc/supervisor/conf.d/grpc-app-demo.conf': No space left on device
cp: cannot create regular file '/etc/supervisor/conf.d/grpc-app-demo.conf': No space left on device
cp: cannot create regular file '/etc/supervisor/conf.d/grpc-app-demo.conf': No space left on device
cp: cannot create regular file '/etc/supervisor/conf.d/grpc-app-demo.conf': No space left on device
```

**解决方法1**：改动 [docker ](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247493129&idx=2&sn=2667f7f3a17103941236959f6d173bb3&chksm=e9188315de6f0a03fd8cc13a66f8247cd65e1cbf9f5475ec4f4a996e57f782a429d0eb6c8933&scene=21#wechat_redirect)启动配置文件

```
# /etc/docker/daemon.json
{
    "live-restore": true,
    "storage-opt": [ "dm.basesize=20G" ]
}
```

**解决方法2**：改动 systemctl 的 docker 启动文件

```
# 1.stop the docker service
$ sudo systemctl stop docker

# 2.rm exised container
$ sudo rm -rf /var/lib/docker

# 2.edit your docker service file
$ sudo vim /usr/lib/systemd/system/docker.service

# 3.find the execution line
ExecStart=/usr/bin/dockerd
and change it to:
ExecStart=/usr/bin/dockerd --storage-opt dm.basesize=20G

# 4.start docker service again
$ sudo systemctl start docker

# 5.reload daemon
$ sudo systemctl daemon-reload
```

**问题起因三**：还有一种情况也会让容器无法启动，并提示磁盘空间不足，但是使用命令查看发现并不是因为物理磁盘真的不足导致的。而是，因为对于分区的 inode 节点数满了导致的。

```
# 报错信息
No space left on device
```

**解决方法**：因为 ext3 文件系统使用 inode table 存储 inode 信息，而 xfs 文件系统使用 B+ tree  来进行存储。考虑到性能问题，默认情况下这个 B+ tree 只会使用前 1TB 空间，当这 1TB 空间被写满后，就会导致无法写入 inode  信息，报磁盘空间不足的错误。我们可以在 mount 时，指定 inode64 即可将这个 B+ tree 使用的空间扩展到整个文件系统。

```
# 查看系统的inode节点使用情况
$ sudo df -i

# 尝试重新挂载
$ sudo mount -o remount -o noatime,nodiratime,inode64,nobarrier /dev/vda1
```

**补充知识**：文件储存在硬盘上，硬盘的最小存储单位叫做“扇区”(Sector)。每个扇区储存 512 字节(相当于0.5KB)。[操作系统](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247506120&idx=2&sn=1da03a24c4152e3a94bf396d5406ce9b&chksm=e918bdd4de6f34c21572c851fc499cc8ac5c3fe65599cdf3d98da16fbb757752fffecc86edb3&scene=21#wechat_redirect)读取硬盘的时候，不会一个个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个“块”(block)。这种由多个扇区组成的”块”，是文件存取的最小单位。”块”的大小，最常见的是4KB，即连续八个 sector 组成一个 block  块。文件数据都储存在”块”中，那么很显然，我们还必须找到一个地方储存文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等等。这种储存文件元信息的区域就叫做“[索引节点”(inode](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247506224&idx=2&sn=551adf9199a8ec76c1b0b4a481620cb2&chksm=e918bc2cde6f353a97a1751aad3ed4f58364c9f40b4bdc3f39b5d2d1f96538775955a3acc372&scene=21#wechat_redirect))。每一个文件都有对应的 inode，里面包含了除了文件名以外的所有文件信息。

inode 也会消耗硬盘空间，所以硬盘格式化的时候，操作系统自动将硬盘分成两个区域。一个是数据区，存放文件数据；另一个是 inode 区(inode  table)，存放 inode 所包含的信息。每个 inode 节点的大小，一般是 128 字节或 256 字节。inode  节点的总数，在格式化时就给定，一般是每1KB或每2KB就设置一个 inode 节点。

```
# 每个节点信息的内容
$ stat check_port_live.sh
  File: check_port_live.sh
  Size: 225           Blocks: 8          IO Block: 4096   regular file
Device: 822h/2082d    Inode: 99621663    Links: 1
Access: (0755/-rwxr-xr-x)  Uid: ( 1006/  escape)   Gid: ( 1006/  escape)
Access: 2019-07-29 14:59:59.498076903 +0800
Modify: 2019-07-29 14:59:59.498076903 +0800
Change: 2019-07-29 23:20:27.834866649 +0800
 Birth: -

# 磁盘的inode使用情况
$ df -i
Filesystem                 Inodes   IUsed     IFree IUse% Mounted on
udev                     16478355     801  16477554    1% /dev
tmpfs                    16487639    2521  16485118    1% /run
/dev/sdc2               244162560 4788436 239374124    2% /
tmpfs                    16487639       5  16487634    1% /dev/shm
```

## 3.Docker 缺共享链接库

[Docker 命令](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247485839&idx=1&sn=842a3d5ef663ac07d9397d0d0cff8ebb&chksm=e91b6c93de6ce585418d974dcc6991c23df1bb24a1c1f65b66260055cbcebdd634184a848141&scene=21#wechat_redirect)需要对/tmp 目录下面有访问权限

**问题起因**：给系统安装完 compose 之后，查看版本的时候，提示缺少一个名为 libz.so.1 的共享链接库。第一反应就是，是不是系统少安装那个软件包导致的。随即，搜索了一下，将相关的依赖包都给安装了，却还是提示同样的问题。

```
# 提示错误信息
$ docker-compose --version
error while loading shared libraries: libz.so.1: failed to map segment from shared object: Operation not permitted
```

**解决方法**：后来发现，是因为系统中 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247490874&idx=3&sn=7d501f99e538d696deaaea61aa771d41&chksm=e91b7826de6cf13086f386de7b4f4ae7b271f1b57087f81ef5f7ed558000e9a5497b7c264172&scene=21#wechat_redirect) 没有对 /tmp 目录的访问权限导致，需要重新将其挂载一次，就可以解决了。

```
# 重新挂载
$ sudo mount /tmp -o remount,exec
```

## 4.Docker 容器文件损坏

对 dockerd 的配置有可能会影响到系统稳定

**问题起因**：容器文件损坏，经常会导致容器无法操作。正常的 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247490579&idx=3&sn=52b999d1a4ac48abdc44878c6dbf4580&chksm=e91b790fde6cf019c81ab519ab00a84979db875b48dfb73fdc2ac2c70be8c73ee97eecbbe09e&scene=21#wechat_redirect) 命令已经无法操控这台容器了，无法关闭、重启、删除。正巧，前天就需要这个的问题，主要的原因是因为重新对 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247490417&idx=2&sn=fa6eacdb2fcf0ea6ed060366f8fe4595&chksm=e91b7e6dde6cf77b7f1acbda77bb380a146cd7401b1452ba8ad89ca0b3cf18efe950ac383437&scene=21#wechat_redirect) 的默认容器进行了重新的分配限制导致的。

```
# 操作容器遇到类似的错误
b'devicemapper: Error running deviceCreate (CreateSnapDeviceRaw) dm_task_run failed'
```

**解决方法**：可以通过以下操作将容器删除/重建。

```
# 1.关闭docker
$ sudo systemctl stop docker

# 2.删除容器文件
$ sudo rm -rf /var/lib/docker/containers

# 3.重新整理容器元数据
$ sudo thin_check /var/lib/docker/devicemapper/devicemapper/metadata
$ sudo thin_check --clear-needs-check-flag /var/lib/docker/devicemapper/devicemapper/metadata

# 4.重启docker
$ sudo systemctl start docker
```

## 5.Docker 容器优雅重启

不停止服务器上面运行的容器，重启 dockerd 服务是多么好的一件事

**问题起因**：默认情况下，当 [Docker 守护程序](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247489798&idx=1&sn=7996426d140655dbf5f791a5c2cc7e88&chksm=e91b7c1ade6cf50cfda7fc4ffd5d00fe469b315949bd488ede822d562af2eed7ad0931462f6f&scene=21#wechat_redirect)终止时，它会关闭正在运行的容器。从 Docker-ce 1.12 开始，可以在配置文件中添加 live-restore 参数，以便在守护程序变得不可用时容器保持运行。需要注意的是 Windows 平台暂时还是不支持该参数的配置。

```
# Keep containers alive during daemon downtime
$ sudo vim /etc/docker/daemon.yaml
{
  "live-restore": true
}

# 在守护进程停机期间保持容器存活
$ sudo dockerd --live-restore

# 只能使用reload重载
# 相当于发送SIGHUP信号量给dockerd守护进程
$ sudo systemctl reload docker

# 但是对应网络的设置需要restart才能生效
$ sudo systemctl restart docker
```

**解决方法**：可以通过以下操作将容器删除/重建。

```
# /etc/docker/daemon.yaml
{
    "registry-mirrors": ["https://vec0xydj.mirror.aliyuncs.com"],  # 配置获取官方镜像的仓库地址
    "experimental": true,  # 启用实验功能
    "default-runtime": "nvidia",  # 容器的默认OCI运行时(默认为runc)
    "live-restore": true,  # 重启dockerd服务的时候容易不终止
    "runtimes": {  # 配置容器运行时
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "default-address-pools": [  # 配置容器使用的子网地址池
        {
            "scope": "local",
            "base":"172.17.0.0/12",
            "size":24
        }
    ]
}
```

## 6.Docker 容器无法删除

找不到对应容器进程是最吓人的

**问题起因**：今天遇到 [docker 容器](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247490081&idx=1&sn=5ff951cfdc9e575ff5c5d78e0983a5b7&chksm=e91b7f3dde6cf62b166050ae6c1687bec7f3d6c782ec0687cabf3c07bcc0563914cb58830843&scene=21#wechat_redirect)无法停止/终止/删除，以为这个容器可能又出现了 dockerd 守护进程托管的情况，但是通过`ps -ef <container id>`无法查到对应的运行进程。哎，后来开始开始查 supervisor 以及 Dockerfile  中的进程，都没有。这种情况的可能原因是容器启动之后，之后，主机因任何原因重新启动并且没有优雅地终止容器。剩下的文件现在阻止你重新生成旧名称的新容器，因为系统认为旧容器仍然存在。

```
# 删除容器
$ sudo docker rm -f f8e8c3..
Error response from daemon: Conflict, cannot remove the default name of the container
```

**解决方法**：找到 /var/lib/docker/containers/ 下的对应容器的文件夹，将其删除，然后重启一下 dockerd 即可。我们会发现，之前无法删除的容器没有了。

```
# 删除容器文件
$ sudo rm -rf /var/lib/docker/containers/f8e8c3...65720

# 重启服务
$ sudo systemctl restart docker.service
```

## 7.Docker 容器中文异常

容器存在问题话，记得优先在官网查询

**问题起因**：今天登陆之前部署的 [MySQL 数据库](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247505367&idx=2&sn=8eaf87e3d8d108fbee06799ff9f80cbd&chksm=e918b0cbde6f39dd540ac9758c5fe1b15c7abd47b2245c8c6d446a67a266170d3de0c316e712&scene=21#wechat_redirect)查询，发现使用 SQL 语句无法查询中文字段，即使直接输入中文都没有办法显示。

```
# 查看容器支持的字符集
root@b18f56aa1e15:# locale -a
C
C.UTF-8
POSIX
```

**解决方法**：Docker 部署的 MySQL 系统使用的是 POSIX 字符集。然而 POSIX 字符集是不支持中文的，而 C.UTF-8  是支持中文的只要把系统中的环境 LANG 改为 "C.UTF-8" 格式即可解决问题。同理，在 K8S 进入 pod  不能输入中文也可用此方法解决。

```
# 临时解决
docker exec -it some-mysql env LANG=C.UTF-8 /bin/bash

# 永久解决
docker run --name some-mysql \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    -d mysql:tag --character-set-server=utf8mb4 \
    --collation-server=utf8mb4_unicode_ci
```

## 8.Docker 容器网络互通

了解 Docker 的四种网络模型

**问题起因**：在本机部署 [Nginx 容器](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247509451&idx=1&sn=31082f7c17d04ff42002d988a30885e0&chksm=e918c0d7de6f49c107db26408a6199e7606e2ce746f5b949149b7f689eb2b086d318d99915b9&scene=21#wechat_redirect)想代理本机启动的 Python 后端服务程序，但是对代码服务如下的配置，结果访问的时候一直提示 502 错误。

```
# 启动Nginx服务
$ docker run -d -p 80:80 $PWD:/etc/nginx nginx
nginx
server {
    ...
    location /api {
        proxy_pass http://localhost:8080
    }
    ...
}
```

**解决方法**：后面发现是因为 [nginx.conf 配置](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247509016&idx=1&sn=0e6d0e99b0902a39da7fb083cb885fb0&chksm=e918c104de6f481284122ebfb704aeef803df4391f8b395b0ef730fe0de4c60f48bf8db2d060&scene=21#wechat_redirect)文件中的 localhost 配置的有问题，由于 Nginx 是在容器中运行，所以 localhost 为容器中的 localhost，而非本机的 localhost，所以导致无法访问。

可以将 [nginx.conf ](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247507251&idx=2&sn=82bbf6075d50f9b5172dc61d950af762&chksm=e918b82fde6f31393185eb26a5f8fd03ef5f4d9a4aeb894c79c52923ceafccc611cf927b8310&scene=21#wechat_redirect)中的 localhost 改为宿主机的 IP 地址，就可以解决 502 的错误。

```
# 查询宿主机IP地址 => 172.17.0.1
$ ip addr show docker0
docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:d5:4c:f2:1e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:d5ff:fe4c:f21e/64 scope link
       valid_lft forever preferred_lft forever
nginx
server {
    ...
    location /api {
        proxy_pass http://172.17.0.1:8080
    }
    ...
}
```

当容器使用 host 网络时，容器与宿主共用网络，这样就能在容器中访问宿主机网络，那么容器的 localhost 就是宿主机的 localhost 了。

```
# 服务的启动方式有所改变(没有映射出来端口)
# 因为本身与宿主机共用了网络，宿主机暴露端口等同于容器中暴露端口
$ docker run -d -p 80:80 --network=host $PWD:/etc/nginx nginxx
```

## 9.Docker 容器总线错误

总线错误看到的时候还是挺吓人了

**问题起因**：在 docker 容器中运行程序的时候，提示 bus error 错误。

```
# 总线报错
$ inv app.user_op --name=zhangsan
Bus error (core dumped)
```

**解决方法**：原因是在 docker 运行的时候，shm 分区设置太小导致 share memory 不够。不设置 –shm-size 参数时，docker 给容器默认分配的 shm 大小为 64M，导致程序启动时不足。

```
# 启动docker的时候加上--shm-size参数(单位为b,k,m或g)
$ docker run -it --rm --shm-size=200m pytorch/pytorch:latest
```

**解决方法**：还有一种情况就是容器内的磁盘空间不足，也会导致 bus error 的报错，所以清除多余文件或者目录，就可以解决了。

```
# 磁盘空间不足
$ df -Th
Filesystem     Type     Size  Used Avail Use% Mounted on
overlay        overlay    1T    1T    0G 100% /
shm            tmpfs     64M   24K   64M   1% /dev/shm
```

## 10.Docker NFS 挂载报错

总线错误看到的时候还是挺吓人了

**问题起因**：我们将服务部署到 openshift 集群中，启动服务调用资源文件的时候，报错信息如下所示。从报错信息中，得知是在 Python3 程序执行  read_file()  读取文件的内容，给文件加锁的时候报错了。但是奇怪的是，本地调试的时候发现服务都是可以正常运行的，文件加锁也是没问题的。后来发现，在  openshift 集群中使用的是 NFS 挂  载的共享磁盘。

```
# 报错信息
Traceback (most recent call last):
    ......
    File "xxx/utils/storage.py", line 34, in xxx.utils.storage.LocalStorage.read_file
OSError: [Errno 9] Bad file descriptor
# 文件加锁代码
...
    with open(self.mount(path), 'rb') as fileobj:
        fcntl.flock(fileobj, fcntl.LOCK_EX)
        data = fileobj.read()
    return data
...
```

**解决方法**：从下面的信息得知，要在 Linux 中使用 flock() 的话，就需要升级内核版本到 2.6.11+ 才行。后来才发现，这实际上是由 RedHat  內核中的一个错误引起的，并在 kernel-3.10.0-693.18.1.el7 版本中得到修复。所以对于 NFSv3 和 NFSv4  服务而已，就需要升级 Linux 内核版本才能够解决这个问题。

```
# https://t.codebug.vip/questions-930901.htm
$ In Linux kernels up to 2.6.11, flock() does not lock files over NFS (i.e.,
the scope of locks was limited to the local system). [...] Since Linux 2.6.12,
NFS clients support flock() locks by emulating them as byte-range locks on the entire file.
```

## 11.Docker 默认使用网段

启动的容器网络无法相互通信，很是奇怪！

**问题起因**：我们在使用 Docker  启动服务的时候，发现有时候服务之前可以相互连通，而有时间启动的多个服务之前却出现了无法访问的情况。究其原因，发现原来是因为使用的内部私有地址网段不一致导致的。有点服务启动到了 172.17 - 172.31 的网段，有的服务跑到了 192.169.0 - 192.168.224  的网段，这样导致服务启动之后出现无法访问的情况。

![图片](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304163603620.webp)

**解决方法**：上述问题的处理方式，就是手动指定 Docker 服务的启动网段，就可以了。

```
# 查看docker容器配置
$ cat /etc/docker/daemon.json
{
    "registry-mirrors": ["https://vec0xydj.mirror.aliyuncs.com"],
    "default-address-pools":[{"base":"172.17.0.0/12","size":24}],
    "experimental": true,
    "default-runtime": "nvidia",
    "live-restore": true,
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

## 12.Docker 服务启动串台

使用 docker-compose 命令各自启动两组服务，发现服务会串台！

**问题起因**：在两个不同名称的目录目录下面，使用 docker-compose 来启动服务，发现当 A 组服务启动完毕之后，再启动 B 组服务的时候，发现 A  组当中对应的一部分服务又重新启动了一次，这就非常奇怪了！因为这个问题的存在会导致，A 组服务和 B 组服务无法同时启动。之前还以为是工具的  Bug，后来请教了“上峰”，才知道了原因，恍然大悟。

```
# 服务目录结构如下所示
A: /data1/app/docker-compose.yml
B: /data2/app/docker-compose.yml
```

**解决方法**：发现 A 和 B 两组服务会串台的原因，原来是 docker-compose 会给启动的容器加 label 标签，然后根据这些 label  标签来识别和判断对应的容器服务是由谁启动的、谁来管理的，等等。而这里，我们需要关注的 label 变量是  com.docker.compose.project，其对应的值是使用启动配置文件的目录的最底层子目录名称，即上面的 app  就是对应的值。我们可以发现， A 和 B 两组服务对应的值都是  app，所以启动的时候被认为是同一个，这就出现了上述的问题。如果需要深入了解的话，可以去看对应源代码。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
# 可以将目录结构调整为如下所示
A: /data/app1/docker-compose.yml
B: /data/app2/docker-compose.yml

A: /data1/app-old/docker-compose.yml
B: /data2/app-new/docker-compose.yml
```

或者使用 docker-compose 命令提供的参数 -p 来规避该问题的发生。

```
# 指定项目项目名称
$ docker-compose -f ./docker-compose.yml -p app1 up -d
```

## 13.Docker 命令调用报错

在编写脚本的时候常常会执行 docker 相关的命令，但是需要注意使用细节！

**问题起因**：CI 更新环境执行了一个脚本，但是脚本执行过程中报错了，如下所示。通过对应的输出信息，可以看到提示说正在执行的设备不是一个 tty。

![图片](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304163613775.webp)

随即，查看了脚本发现报错地方是执行了一个 exec 的 docker 命令，大致如下所示。很奇怪的是，手动执行或直接调脚本的时候，怎么都是没有问题的，但是等到 CI 调用的时候怎么都是有问题。后来好好看下下面这个命令，注意到 -it 这个参数了。



```
# 脚本调用docker命令
docker exec -it <container_name> psql -Upostgres ......

我们可以一起看下 exec 命令的这两个参数，自然就差不多理解了。

-i/-interactive #即使没有附加也保持 STDIN 打开；如果你需要执行命令则需要开启这个选项
-t/–tty #分配一个伪终端进行执行；一个连接用户的终端与容器 stdin 和 stdout 的桥梁
```

**解决方法**：docker exec 的参数 -t 是指 Allocate a pseudo-TTY 的意思，而 CI 在执行 job 的时候并不是在 TTY 终端中执行，所以 -t 这个参数会报错。

![图片](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304163619288.webp)

## 14.Docker 定时任务异常

在 Crontab 定时任务中也存在 Docker 命令执行异常的情况！

**问题起因**：今天发现了一个问题，就是在备份 Mysql 数据库的时候，使用 docker 容器进行备份，然后使用 Crontab 定时任务来触发备份。但是发现备份的 MySQL 数据库居然是空的，但是手动执行对应命令切是好的，很奇怪。

```
# Crontab定时任务
0 */6 * * * \
    docker exec -it <container_name> sh -c \
        'exec mysqldump --all-databases -uroot -ppassword ......'
```

**解决方法**：后来发现是因为执行的 docker 命令多个 -i 导致的。因为 Crontab 命令执行的时候，并不是交互式的，所以需要把这个去掉才可以。总结就是，如果你需要回显的话则需要 -t 选项，如果需要交互式会话则需要 -i 选项。

```
-i/-interactive #即使没有附加也保持 STDIN 打开；如果你需要执行命令则需要开启这个选项
-t/–tty  #分配一个伪终端进行执行；一个连接用户的终端与容器 stdin 和 stdout 的桥梁
```

## 15.Docker 变量使用引号

compose 里边环境变量带不带引号的问题！

**问题起因**：使用过 compose 的同学可能都遇到过，我们在编写启动配置文件的时候，添加环境变量的时候到底是使用单引号、双引号还是不使用引号。时间长了，可能我们总是三者是一样的，可以相互使用。但是，直到最后我们发现坑越来越多，越来越隐晦。

反正我是遇到过很多是因为添加引号导致的服务启动问题，后来得出的结论就是一律不适用引号。裸奔，体验前所未有的爽快！直到现在看到了 Github 中对应的 issus 之后，才终于破案了。

```
# TESTVAR="test"
在Compose中进行引用TESTVAR变量，无法找到

# TESTVAR=test
在Compose中进行引用TESTVAR变量，可以找到

# docker run -it --rm -e TESTVAR="test" test:latest
后来发现docker本身其实已经正确地处理了引号的使用
```

**解决方法**：得到的结论就是，因为 Compose 解析 yaml 配置文件，发现引号也进行了解释包装。这就导致原本的 TESTVAR="test" 被解析成了  'TESTVAR="test"'，所以我们在引用的时候就无法获取到对应的值。现在解决方法就是，不管是我们直接在配置文件添加环境变量或者使用  env_file 配置文件，能不使用引号就不适用引号。

## 16. Docker 删除镜像报错

无法删除镜像，归根到底还是有地方用到了！

**问题起因**：清理服器磁盘空间的时候，删除某个镜像的时候提示如下信息。提示需要强制删除，但是发现及时执行了强制删除依旧没有效果。

```
# 删除镜像
$ docker rmi 3ccxxxx2e862
Error response from daemon: conflict: unable to delete 3ccxxxx2e862 (cannot be forced) - image has dependent child images

# 强制删除
$ dcoker rmi -f 3ccxxxx2e862
Error response from daemon: conflict: unable to delete 3ccxxxx2e862 (cannot be forced) - image has dependent child images
```

**解决方法**：后来才发现，出现这个原因主要是因为 TAG，即存在其他镜像引用了这个镜像。这里我们可以使用如下命令查看对应镜像文件的依赖关系，然后根据对应 TAG 来删除镜像。

```
# 查询依赖 - image_id表示镜像名称
$ docker image inspect --format='{{.RepoTags}} {{.Id}} {{.Parent}}' $(docker image ls -q --filter since=<image_id>)

# 根据TAG删除镜像
$ docker rmi -f c565xxxxc87f
bash
# 删除悬空镜像
$ docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
```

## 17.Docker 普通用户切换

切换 Docker 启动用户的话，还是需要注意下权限问题的！

**问题起因**：我们都知道在 Docker 容器里面使用 root 用户的话，是不安全的，很容易出现越权的安全问题，所以一般情况下，我们都会使用普通用户来代替 root  进行服务的启动和管理的。今天给一个服务切换用户的时候，发现 Nginx 服务一直无法启动，提示如下权限问题。因为对应的配置文件也没有配置 var 相关的目录，无奈 🤷‍♀ ！️

```
# Nginx报错信息
nginx: [alert] could not open error log file: open() "/var/log/nginx/error.log" failed (13: Permission denied)
2020/11/12 15:25:47 [emerg] 23#23: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
```

**解决方法**：后来发现还是 nginx.conf 配置文件，配置的有问题，需要将 Nginx 服务启动时候需要的文件都配置到一个无权限的目录，即可解决。

```
nginx
user  www-data;
worker_processes  1;

error_log  /data/logs/master_error.log warn;
pid        /dev/shm/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    gzip               on;
    sendfile           on;
    tcp_nopush         on;
    keepalive_timeout  65;

    client_body_temp_path  /tmp/client_body;
    fastcgi_temp_path      /tmp/fastcgi_temp;
    proxy_temp_path        /tmp/proxy_temp;
    scgi_temp_path         /tmp/scgi_temp;
    uwsgi_temp_path        /tmp/uwsgi_temp;

    include /etc/nginx/conf.d/*.conf;
}
```

## 18. **Docker 不稳定**

通过实践，我发现 Docker 还是挺容易挂的，尤其是长时间跑高之后。为了保证 Docker 服务的持续运行，除了要让 Docker 开机自启动之外，还需要对 Docker 服务进行监控，一旦发现服务挂了就马上重启服务。

可以通过一条简单的 crontab 定时任务解决：

```text
# 适用于 CentOS 7，如果 Docker 正在服务，不会产生负面影响
* * * * * systemctl start docker
```

## 19.定期清理

时间长了，宿主机会有很多不需要的镜像、停止的容器等，如果有需要，同样可以通过定时任务进行清理。

```text
# 每天凌晨 2 点清理容器和镜像
0 2 * * * docker container prune --force && docker image prune --force
# 更凶残地方式
0 2 * * * docker system prune --force
```

## 20.yum源报错

[CentOS镜像-CentOS镜像下载安装-开源镜像站-阿里云](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.1feb1b11LmEYJg)

问题提出：

### cannot find a valid baseurl for repo:base/7/x86_64的解决方案

报错说明：

出现cannot find a valid baseurl for repo:base/7/x86_64错误通常是由于YUM仓库源无法找到或无法访问，导致YUM无法正常工作。这种情况常见于CentOS 7系统。解决这个问题需要检查几个方面，如网络连接、DNS设置和YUM仓库源配置。以下是详细的排查解决方法。

检查网络连接
可以通过以下命令检查系统是否能访问外部网站：

```shell
ping -c 4 google.com
```

如果不能ping通，可能是网络配置问题。你需要确保网络连接正常，可能需要重新启动网络服务：

```shell
systemctl restart network
```

方法二：检查DNS设置
如果你的网络连接正常但依然不能访问仓库，可能是DNS问题。

更新DNS配置
编辑/etc/resolv.conf文件，确保其中包含有效的DNS服务器，例如Google的公共DNS：

```shell
vi /etc/resolv.conf
```

添加以下行：

```shell
nameserver 8.8.8.8
nameserver 8.8.4.4
```

保存文件并退出。

检查是否能解析域名
再次检查系统是否能解析域名：

```shell
ping -c 4 google.com
```

方法三：检查YUM仓库配置
如果网络连接和DNS设置都正常，可能是YUM仓库配置有问题，需要检查并更新YUM仓库源。

更新YUM仓库源
备份现有的YUM配置文件：

```shell
 cp -r /etc/yum.repos.d /etc/yum.repos.d.backup
```

编辑或替换仓库配置文件
检查/etc/yum.repos.d/CentOS-Base.repo文件。确保baseurl和gpgcheck配置正确。你可以手动编辑这个文件，或者更换为可靠的YUM仓库源。
使用阿里云或其他国内镜像源
如果你在国内，使用国内的镜像源通常可以提供更快和更稳定的访问速度。以下是如何配置阿里云镜像源：

更新YUM仓库源为阿里云镜像源：

```shell
vi /etc/yum.repos.d/CentOS-Base.repo
```

将内容替换为以下内容：

```shell
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

保存文件并退出。

清理并重建缓存

```shell
yum clean all
yum makecache
yum update
```

