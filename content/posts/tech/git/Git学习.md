+++
date = '2024-07-05T22:46:29+08:00'
draft = false
title = 'Git学习'
slug = "len2vr2ppm"
description = "Git学习"
categories = ["📒版本控制"]
tags = ["Git"]
summary = "Git学习"
+++
﻿
# 版本控制

### 什么是版本控制

版本控制（Revision control）是一种在开发的过程中用于管理我们对文件、目录或工程等内容的修改历史，方便查看更改历史记录，备份以便恢复以前的版本的软件工程技术。

+ 实现跨区域多人协同开发
+ 追踪和记载一个或者多个文件的历史记录
+ 组织和保护你的源代码和文档
+ 统计工作量
+ 并行开发、提高开发效率
+ 跟踪记录整个软件的开发过程
+ 减轻开发人员的负担，节省时间，同时降低人为错误

简单说就是用于管理多人协同开发项目的技术。

没有进行版本控制或者版本控制本身缺乏正确的流程管理，在软件开发过程中将会引入很多问题，如软件代码的一致性、软件内容的冗余、软件过程的事物性、软件开发过程中的并发性、软件源代码的安全性，以及软件的整合等问题。

多人开发就必须要使用版本控制！

### 常见的版本控制工具

主流的版本控制器有如下这些：

+ **Git**
+ **SVN**（Subversion）
+ **CVS**（Concurrent Versions System）
+ **VSS**（Micorosoft Visual SourceSafe）
+ **TFS**（Team Foundation Server）
+ Visual Studio Online

版本控制产品非常的多（Perforce、Rational ClearCase、RCS（GNU Revision Control System）、Serena Dimention、SVK、BitKeeper、Monotone、Bazaar、Mercurial、SourceGear Vault），现在影响力最大且使用最广泛的是Git与SVN

>**为什么要使用版本控制?**

软件开发中采用版本控制系统是个明智的选择。
有了它你就可以将某个文件回溯到之前的状态,甚至将整个项目都回退到过去某个时间点的状态。
就算你乱来一气把整个项目中的文件改的改删的删，你也照样可以轻松恢复到原先的样子。
但额外增加的工作量却微乎其微。你可以比较文件的变化细节,查出最后是谁修改了哪个地方,从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等。

> **版本控制分类**

**密集中化的版本控制系统:**

集中化的版本控制系统诸如CVS, SVN以及Perforce等,都有一-个单一-的集中管理的服务器保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器,取出最新的文件或者提交更新。多年以来,这已成为版本控制系统的标准做法，这种做法带来了许多好处,现在,每个人都可以在一定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统;要远比在各 个客户端上维护本地数据库来得轻松容易。事分两面，有好有坏。这么做最显而易见的缺点是中央服务器的单点故障。如果服务器宕机- -小时，那么在这- -小时内，谁都无法提交更新,也就无法协同工作。

**密分布式的版本控制系统**

由于上面集中化版本控制系统的那些缺点，于是分布式版本控制系统面世了-
在这类系统中，像Git, BitKeeper等,==客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。==

更进-步，许多这类系统都可以指定和若干不同的远端代码仓库进行交互。这样,你就可以在同一一个项目中分别和不同工作小组的人相互协作。

分布式的版本控制系统在管理项目时存放的不是项目版本与版本之间的差异.它存的是索引(所需磁盘空间很少所以每个客户端都可以放下整个项目的历史记录)

### Git与SVN的主要区别

SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而工作的时候，用的都是自己的电脑，所以首先要从中央服务器得到最新的版本，然后工作，完成工作后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，对网络带宽要求较高。

Git是分布式版本控制系统，没有中央服务器，每个人的电脑就是一个完整的版本库，工作的时候不需要联网了，因为版本都在自己电脑上。协同的方法是这样的：比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。Git可以直接看到更新了哪些代码和文件！

**Git是目前世界上最先进的分布式版本控制系统。**

# Git结构

### 三个区域

Git本地有三个工作区域：工作目录（Working Directory）、暂存区(Stage/Index)、资源库(Repository或Git Directory)。如果在加上远程的git仓库(Remote Directory)就可以分为四个工作区域。文件在这四个区域之间的转换关系如下：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241639112.webp)

+ Workspace：工作区，就是你平时存放项目代码的地方
+ Index / Stage：暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
+ Repository：仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
+ Remote：远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换

本地的三个区域确切的说应该是git仓库中HEAD指向的版本：

+ Directory：使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。
+ WorkSpace：需要通过Git进行版本控制的目录和文件，这些目录和文件组成了工作空间。
+ .git：存放Git管理信息的目录，初始化仓库的时候自动创建。
+ Index/Stage：暂存区，或者叫待提交更新区，在提交进入repo之前，我们可以把所有的更新放在暂存区。
+ Local Repo：本地仓库，一个存放在本地的版本库；HEAD会只是当前的开发分支（branch）。
+ Stash：隐藏，是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

# 代码托管中心

我们已经有了本地库，本地库可以帮我们进行版本控制，为什么还需要代码托管中心呢?

它的任务是帮我们维护远程库,

下面说一下本地库和远程库的交互方式， 也分为两种:

(1)团队内部协作

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241639247.webp)


(2)跨团队协作

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241639417.webp)


托管中心种类:

​	局域网环境下:可以搭建 GitLab服务器作为代码托管中心，GitLab可以自己去搭建
​	外网环境下:可以由GitHub或者Gitee作为代码托管中心，GitHub或者Gitee是现成的托管中心，不用自己去搭建

# 工作流程

git的工作流程一般是这样的：

１、在工作目录中添加、修改文件；

２、将需要进行版本管理的文件放入暂存区域；

３、将暂存区域的文件提交到git仓库。

因此，git管理的文件有三种状态：已修改（modified）,已暂存（staged）,已提交(committed)

# Git下载安装

### 软件下载

打开 [git官网] https://git-scm.com/，下载git对应操作系统的版本。

所有东西下载慢的话就可以去找镜像！

官网下载太慢，我们可以使用淘宝镜像下载：http://npm.taobao.org/mirrors/git-for-windows/

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241639050.webp)


下载对应的版本即可安装！

### 安装

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640033.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241639434.webp)


### 启动Git

鼠标右键--->点击Git Bash here，打开Git终端



**Git Bash：** Unix与Linux风格的命令行，使用最多，推荐最多

**Git CMD：** Windows风格的命令行

**Git GUI** ：图形界面的Git，不建议初学者使用，尽量先熟悉常用命令

### 查看Git配置

所有的配置文件，其实都保存在本地！

```bash
git config -l  #查看配置 
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640385.webp)



查看不同级别的配置文件：

```bash
#查看系统config
git config --system --list　　
#查看当前用户（global）配置
git config --global  --list
```

### 初始化本地仓库

1. 创建一个文件夹，名为GitResp，作为本地仓库

2. 打开Git终端：

   点击Git Bash Here:

   再右键-->选择options

   进入以后先对字体和编码进行设置：

 ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640766.webp)


   在Git中命令跟Linux是一样的：

   （1）查看git安装版本

   ```bash
   git --version
   ```

     (2) 清屏

   ```bash
   clear
   ```

     (3)设置签名

   ​       设置用户名与邮箱

   ```bash
   git config --global user.name "XXX"  #名称
   git config --global user.email 1354353@qq.com   #邮箱
   ```

3. 初始化仓库操作

   用cd命令进入之前创建文件夹所在路径或者直接进入该文件夹右击打开Git Bash here

   执行下列命令

   ```bash
   git init
   ```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640683.webp)


   .git文件隐藏

 ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640817.webp)


   查看.git下内容

   ```bash
   ll
   ```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640368.webp)


   注意事项：.git目录下的本地库相关的子目录和子文件不要删除和修改。

# Git常用命令

### add和commit命令

1.先创建一个文件

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640339.webp)


2.将文件提交到暂存区：

```bash
git add demo.txt
```

3.将暂存区的内容提交到本地库：

```bash
git commit -m "这是我提交的第一个文件 demo.txt" demo.txt
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640142.webp)


注意事项：

​	1.不放在本地仓库中的文件，git是不进行管理

​	2.即使放在本地仓库的文件。git也不管理，必须通过add，commit命令操作才可以将内容提交到本地库。

### status命令

```bash
git status #看的是工作区和暂存区的状态
```

创建一个文件，然后查看状态：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241640432.webp)


然后将demo02.txt通过git add命令提交至：暂存区

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641350.webp)

查看状态：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641788.webp)


利用git commit 命令将文件提交至：本地库

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641863.webp)



现在修改demo.txt文件中的内容，然后再查看状态：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641478.webp)

重新添加至：暂存区并查看状态：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641309.webp)


然后将暂存区的文件提交至本地库，并查看状态：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641168.webp)


### log命令

```bash
git log  #查看提交历史
git log -p -2 #-p 显示每次提交所引入的差异（按补丁的格式输出）  -2 限制显示的日志条目数量
git log --stat  #可以看到每次提交的简略统计信息
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641339.webp)


当历史记录过多的时候，查看日志的时候，有分页效果，分屏效果，一页展示不下

下一页：空格

上一页：b

到页尾显示END

退出：q



日志展示方式：

1.方式一：git log  分页

2.方式二：git log --pretty=oneline

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641818.webp)


3.方式三：git log --oneline

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641233.webp)


4.方式四：git reflog

多了信息：HEAD@{数字}

这个数字的含义：指针回到当前这个历史版本需要走多少步

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241641216.webp)


### reset命令

```bash
git reset --hard 索引   #前进或者后退历史版本
hard参数    #本地库的指针移动的同时，重置暂存区，重置工作区
mixed参数   #本地库的指针移动的同时，重置暂存区，但是工作区不动
soft参数    #本地库的指针移动的时候，暂存区，工作区都不动
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642682.webp)


### 删除/找回文件

1.新建一个demo3.txt文件

2.添加到暂存区

3.提交到本地库

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642281.webp)


4.删除工作区中的Test2.txt

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642259.webp)


5.删除操作同步到暂存区

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642072.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642497.webp)


6.删除操作同步到本地库

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642901.webp)


7.查看日志

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642073.webp)


8.找回本地库中删除的文件，实际上就是将历史版本切换到刚才添加文件的那个版本即可：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642556.webp)


1.删除工作区数据：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642219.webp)


2.同步到缓存区：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241642272.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241643735.webp)


3.恢复暂存区中数据：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241643664.webp)


上述命令相当于将2c0518a指针回滚了一次，下列命令也可以达到目的

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241643266.webp)


### diff命令

```bash
git diff [文件名]    #将工作区中的文件和暂存区中文件进行比较
git diff     #比较工作区中和暂存区中所有文件的差异
git diff [历史版本][文件名]  #比较暂存区和工作区中的内容
```

1.先创建一个文件，编写内容，然后添加到暂存区，在提交到本地库：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644095.webp)


2.更改工作区中demo4.txt中内容，增加内容：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644518.webp)


**导致：工作区  和  暂存区 不一致，比对：**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644931.webp)


**多个文件的比对**

更改工作区中demo3.txt中内容，增加内容：ccc，然后比对

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644006.webp)


**比较暂存区和工作区中的内容**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644411.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644930.webp)


更改工作区中demo4.txt中内容，增加内容：ssss，然后添加到暂存区，然后比对：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644405.webp)

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644183.webp)
### 获取帮助

使用Git时需要获取帮助，有三种方法：

```bash
git help
```

![image-20210127202634657](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644246.webp)

```bash
git help branch -h
```

![image-20210127202832316](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241644037.webp)

### 打标签

列出标签

```bash
git tag
```

### 创建标签

Git 支持两种标签：轻量标签（lightweight）与附注标签（annotated）。

轻量标签很像一个不会改变的分支——它只是某个特定提交的引用。

而附注标签是存储在 Git 数据库中的一个完整对象， 它们是可以被校验的，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息，并且可以使用 GNU Privacy Guard （GPG）签名并验证。

### 附注标签

```bash
git tag -a v1.4 -m "my version 1.4"
#-m 选项指定了一条将会存储在标签中的信息。 如果没有为附注标签指定一条信息，Git 会启动编辑器要求你输入信息。
```

### 轻量标签

```bash
git tag 标签名
```

### 后期打标签

你也可以对过去的提交打标签。

```bash
git tag -a 标签名 索引
```

### 共享标签

默认情况下，`git push` 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上。

```bash
 git push origin 标签名
```

如果想要一次性推送很多标签，也可以使用带有 `--tags` 选项的 `git push` 命令。 这将会把所有不在远程仓库服务器上的标签全部传送到那里。

```bash
git push origin --tags
```

### 删除标签

```bash
git tag -d 标签名
#注意上述命令并不会从任何远程仓库中移除这个标签

git push origin --delete 标签名  #可以移除远程库标签
```



# 分支

### 什么是分支

在版本控制过程中，使用多条线同时推进多个任务。这里面说的多条线，就是多个分支。

### 通过一张图展示

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645217.webp)


### 分支的好处

同时多个分支可以并行开发，互相不耽误，互相不影响，提高开发效率

如果有一个分支功能开发失败，直接删除这个分支就可以了，不会对其它分支产生任何影响。

### 操作分支

1.在工作区创建一个Test2.txt文件，编写内容，然后提交到暂存区，提交到本地库：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645748.webp)


2.查看分支

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645553.webp)


3.创建分支

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645984.webp)


再查看

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645117.webp)


4.切换分支

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645619.webp)


再查看

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645945.webp)


### 冲突问题及解决

1.进入分支，增加内容，提交到本地库

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645541.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645732.webp)


2.将分支切换到master,并查看分支

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645460.webp)
```bash
git checkout -b 分支名  #创建分支并切换到该分支，相当于git branch和git checkout两个操作
```
然后在主分支下 加入内容：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645802.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241645070.webp)


3.再次切换到branch01分支查看：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646036.webp)


4.将branch01分支 合并到  主分支：

（1）进入主分支：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646815.webp)


（2）将branch01中的内容和主分支的内容进行合并：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646381.webp)


查看文件：出现冲突：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646597.webp)


解决：

公司内部决定，或者认为决定，留下想要的即可：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646169.webp)


将工作区中内容添加到暂存区:

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646311.webp)


然后进行commit命令提交

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646175.webp)


# 远程管理

### 注册GIthub

https://github.com/

### 创建一个远程仓库

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646531.webp)


### 创建远程库地址别名

远程库的地址：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646503.webp)


点击进入

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646676.webp)


在Git本地将地址保存，通过别名

查看别名

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241646846.webp)


起别名：（按照需求起，这里为origin）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647439.webp)


再查看别名：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647573.webp)


### 推送操作

```bash
git push 远程库别名 推送分支
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647306.webp)


推送成功以后，查看自己的远程库：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647435.webp)


### 克隆操作

1.获取远程库地址

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647478.webp)


2.任意盘下新建一个文件夹，用于存放克隆的文件,然后打开Git Bash Here进行克隆操作:

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647925.webp)


3.进入文件夹，查看克隆到本地的文件

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647805.webp)


克隆操作可以帮我们完成：

（1）初始化本地库

（2）将远程库内容完整的克隆到本地

（3）替我们创建远程库的别名：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647656.webp)


### 邀请加入团队

1.创建一个文件，编辑内容，并更新本地库信息：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647488.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241647542.webp)


**发现push内容到远程仓库中并没有要求录入账号密码或者提示错误。**

原因：git使用的时候在本地有缓存：

将缓存删除：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648908.webp)


重新测试，发现出错

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648227.webp)


必须要加入团队：

登录项目经理的账号，邀请普通成员：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648091.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648116.webp)


登录被邀请者的账号，接受邀请：（在地址栏录入邀请链接即可）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648828.webp)


再重新执行push操作，会发现成功推送到远程库：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648972.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648204.webp)


### 远程库修改的拉取操作

1.拉取操作pull操作，相当于fetch+merge

2.项目经理先确认远程库内容是否更新了：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648033.webp)


3.项目经理进行拉取：

（1）先是抓取操作：fetch：

```bash
git fetch 远程库别名 远程库上对应的分支
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241648645.webp)


在抓取操作执行后，只是将远程库的内容下载到本地，但是工作区中的文件并没有更新。工作区中还是原先的内容：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649881.webp)


抓取后可以去远程库看看内容是否正确：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649759.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649612.webp)


然后发现内容都正确，就可以进行合并了：

（2）进行合并：merge：（合并之前切换回来）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649767.webp)


远程库的拉取可以直接利用pull命令来完成：

```bash
git pull origin master
```

fetch+merge  --->为了保险起见

pull  ----->简单代码

### 协同开发合作时冲突以及解决

1.向远程库推送数据：（项目经理）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649446.webp)


2.删除凭据，做拉取操作：（普通开发人员）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649349.webp)


到现在为止，现在远程合作没有任何问题。

现在操作同一个文件的同一个位置的时候，就会引起冲突：

3.再次做了推送操作：（普通开发人员）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649698.webp)


改动位置

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649560.webp)


4.改动demo6.txt中内容，然后进行推送：（项目经理）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649440.webp)


==注意：所编辑的文件demo6.txt不在同一个文件夹下==

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241649988.webp)

发现推送失败！！！

在冲突的情况下，先应该拉取下来，然后修改冲突，然后再推送到远程服务器：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650500.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650641.webp)


人为解决冲突：（仅留下需要的那个）

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650296.webp)


解决完冲突以后，向服务器推送：

```bash
git add demo6.txt
git commit -m "解决了冲突"   #注意在提交冲突时不可以带文件名，否则提交失败
git push origin master
```

### 跨团队合作

![在这里插入图片描述](assets/202310241650173.png)


1.得到**项目经理**的远程库的地址：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650531.webp)


地址：https://github.com/solitude114/GitResp.git

2.**开发人员B**进行fork操作：

登录开发人员B账号：复制上面地址，然后点击其中的fork操作：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20210124175055564.webp)

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650677.webp)


3.然后就可以克隆到本地，并且进行修改：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650068.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650515.webp)


然后更改数据：添加到暂存区，然后提交到本地库，然后push到远程库：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650126.webp)


然后重新推送：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650517.webp)


4.进行pull request操作：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDg3MDcw,size_16,color_FFFFFF,t_70.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241650702.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651984.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651690.webp)


5.项目经理进行审核操作：

登录项目经理账号，然后审核：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651942.webp)


可以互相留言：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651855.webp)

登录**开发人员B**账号查看：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651969.webp)


并回复：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651184.webp)


登录**项目经理**账号：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651106.webp)

确定通过以后，可以进行合并：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651813.webp)


### 免密操作

1.进入用户的主目录中：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651474.webp)


2.执行命令，生成一个.ssh的目录：

```bash
ssh-keygen -t rsa -C Github邮箱地址
然后三次回车确认默认值即可
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651815.webp)


发现.ssh目录下有俩个文件：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241651046.webp)


3.打开id_rsa.pub文件，复制里面的内容：

4.打开Github账号：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652273.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652351.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652811.webp)


5.生成密钥以后，就可以正常进行push操作了：

对ssh远程地址起别名：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652011.webp)


展示别名：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652064.webp)


6.测验：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652207.webp)


ssh方式的好处：不用每次都进行身份验证

缺陷：只能针对一个账号

# IDEA集成Git

### 初始化-添加-提交

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652743.webp)


本地库的初始化操作:

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652592.webp)


然后生成一个.git文件

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652233.webp)


创建一个文件会自动提示是否添加到暂存区：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652718.webp)


将整个模块添加到暂存区：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652643.webp)


可以看到全部变成绿色：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652607.webp)


还可以提交到本地库：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241652128.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653115.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDg3MDcw,size_16,color_FFFFFF,t_70.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653676.webp)


### 拉取-推送

因为他们是两个不同的项目，要把两个不同的项目合并，git需要添加一句代码，在git pull之后，这句代码是在git 2.9.2版本发生的,最新的版本需要添加 **--allow-unrelated-histories** 告诉 git允许不相关历史合并。
假如我们的源是origin，分支是master，那么我们需要这样写 **git pull origin master --allow-unrelated-histories**
这个方法只解决因为两个仓库有不同的开始点，也就是两个仓库没有共同的commit出现的无法提交。如果使用本文的方法还无法提交，需要看一下是不是发生了冲突，解决冲突再提交
push推送: **git push -u origin master -f**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653130.webp)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDg3MDcw,size_16,color_FFFFFF,t_70.webp)


在IDEA中进行推送：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653295.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653901.webp)

也可以提交和推送一起执行：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653593.webp)


一般在开发中先pull操作，再push操作，不会直接进行push操作！！

### 克隆

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20210124175741512.webp)


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653344.webp)


克隆到本地后：

这个目录既变成了一个本地仓库，又变成了工作空间。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653699.webp)


### 冲突以及解决

1.在你push以后，有冲突的时候提示合并操作：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241653496.webp)


2.合并即可

### 如何避免冲突

1.团队开发的时候避免在一个文件中改代码

2.在修改一个文件前，在push之前，先pull操作

# 附加

### 忽略文件

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件的模式。 

```bash
cat .gitignore
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。

- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。

- 匹配模式可以以（`/`）开头防止递归。

- 匹配模式可以以（`/`）结尾指定目录。

- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。

  

# 常见问题解决办法

## Git Gui 查看分支历史中文乱码解决

1. 在Git Gui工具栏上选择-编辑-选项。
2. 选择：Default File Contents Encoding， 改为`UTF-8`。

## Git warning: LF will be replaced by CRLF

    git config --global core.autocrlf  false

## .git 文件太大时怎样处理

clone的时候，可以指定深度，如下，为1即表示只克隆最近一次commit.

    git clone git://xxoo --depth 1

## Git 生成变更历史文件

一些项目要求生成变更日志changelog，通过键入<sup>[1]</sup>：

    git log > ChangeLog

## 利用Dropbox或者Google Drive建立私人仓库

    cd /Dropbox/repo.git
    git init --bare
    cd ~/local_repository
    git remote add origin ~/Dropbox/repo.git
    git add .
    git commit -a -m "init repo."
    git push origin master

# 参考文献及其它内容

1. [Git 魔法（教程）](http://www-cs-students.stanford.edu/~blynn/gitmagic/intl/zh_cn/ch02.html)
2. [Astral: 整理Starred项目](https://app.astralapp.com/dashboard)
