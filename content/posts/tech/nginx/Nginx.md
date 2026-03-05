+++
date = '2024-07-22T06:48:18+08:00'
draft = false
title = 'Nginx'
slug = "mutl9oasar"
description = "Nginx学习笔记"
categories = ["📒nginx"]
tags = ["nginx"]
summary = "Nginx学习笔记"
+++

# Nginx

[TOC]



## Nginx 简介

### 背景介绍

Nginx（“engine x”）一个具有高性能的【HTTP】和【反向代理】的【WEB服务器】，同时也是一个【POP3/SMTP/IMAP代理服务器】，是由伊戈尔·赛索耶夫(俄罗斯人)使用C语言编写的，Nginx的第一个版本是2004年10月4号发布的0.1.0版本。另外值得一提的是伊戈尔·赛索耶夫将Nginx的源码进行了开源，这也为Nginx的发展提供了良好的保障。

![1573470187616](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162046180.webp)

#### 名词解释

1. WEB服务器：

WEB服务器也叫网页服务器，英文名叫Web Server，主要功能是为用户提供网上信息浏览服务。

2. HTTP:

HTTP是超文本传输协议的缩写，是用于从WEB服务器传输超文本到本地浏览器的传输协议，也是互联网上应用最为广泛的一种网络协议。HTTP是一个客户端和服务器端请求和应答的标准，客户端是终端用户，服务端是网站，通过使用Web浏览器、网络爬虫或者其他工具，客户端发起一个到服务器上指定端口的HTTP请求。

3. POP3/SMTP/IMAP：

POP3(Post Offic Protocol 3)邮局协议的第三个版本，

SMTP(Simple Mail Transfer Protocol)简单邮件传输协议，

IMAP(Internet Mail Access Protocol)交互式邮件存取协议，

通过上述名词的解释，我们可以了解到Nginx也可以作为电子邮件代理服务器。

4. 反向代理

正向代理

![1573489359728](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162052071.webp)

反向代理

![1573489653799](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162059419.webp)

### 常见服务器对比

在介绍这一节内容之前，我们先来认识一家公司叫Netcraft。

```
Netcraft公司于1994年底在英国成立，多年来一直致力于互联网市场以及在线安全方面的咨询服务，其中在国际上最具影响力的当属其针对网站服务器、SSL市场所做的客观严谨的分析研究，公司官网每月公布的调研数据（Web Server Survey）已成为当今人们了解全球网站数量以及服务器市场分额情况的主要参考依据，时常被诸如华尔街杂志，英国BBC，Slashdot等媒体报道或引用。
```

我们先来看一组数据，我们先打开Nginx的官方网站  <http://nginx.org/>,找到Netcraft公司公布的数据，对当前主流服务器产品进行介绍。

![1581394945120](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162104281.webp)

上面这张图展示了2019年全球主流Web服务器的市场情况，其中有Apache、Microsoft-IIS、google Servers、Nginx、Tomcat等，而我们在了解新事物的时候，往往习惯通过类比来帮助自己理解事物的概貌。所以下面我们把几种常见的服务器来给大家简单介绍下：

#### IIS

​	全称(Internet Information Services)即互联网信息服务，是由微软公司提供的基于windows系统的互联网基本服务。windows作为服务器在稳定性与其他一些性能上都不如类UNIX操作系统，因此在需要高性能Web服务器的场合下，IIS可能就会被"冷落".

#### Tomcat

​	Tomcat是一个运行Servlet和JSP的Web应用软件，Tomcat技术先进、性能稳定而且开放源代码，因此深受Java爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web应用服务器。但是Tomcat天生是一个重量级的Web服务器，对静态文件和高并发的处理比较弱。

#### Apache

​	Apache的发展时期很长，同时也有过一段辉煌的业绩。从上图可以看出大概在2014年以前都是市场份额第一的服务器。Apache有很多优点，如稳定、开源、跨平台等。但是它出现的时间太久了，在它兴起的年代，互联网的产业规模远远不如今天，所以它被设计成一个重量级的、不支持高并发的Web服务器。在Apache服务器上，如果有数以万计的并发HTTP请求同时访问，就会导致服务器上消耗大量能存，操作系统内核对成百上千的Apache进程做进程间切换也会消耗大量的CUP资源，并导致HTTP请求的平均响应速度降低，这些都决定了Apache不可能成为高性能的Web服务器。这也促使了Lighttpd和Nginx的出现。

#### Lighttpd

​	Lighttpd是德国的一个开源的Web服务器软件，它和Nginx一样，都是轻量级、高性能的Web服务器，欧美的业界开发者比较钟爱Lighttpd,而国内的公司更多的青睐Nginx，同时网上Nginx的资源要更丰富些。

#### 其他的服务器

Google Servers，Weblogic, Webshpere(IBM)...

经过各个服务器的对比，种种迹象都表明，Nginx将以性能为王。这也是我们为什么选择Nginx的理由。

#### Nginx的优点

##### (1)速度更快、并发更高

单次请求或者高并发请求的环境下，Nginx都会比其他Web服务器响应的速度更快。一方面在正常情况下，单次请求会得到更快的响应，另一方面，在高峰期(如有数以万计的并发请求)，Nginx比其他Web服务器更快的响应请求。Nginx之所以有这么高的并发处理能力和这么好的性能原因在于Nginx采用了多进程和I/O多路复用(epoll)的底层实现。

##### (2)配置简单，扩展性强

Nginx的设计极具扩展性，它本身就是由很多模块组成，这些模块的使用可以通过配置文件的配置来添加。这些模块有官方提供的也有第三方提供的模块，如果需要完全可以开发服务自己业务特性的定制模块。

##### (3)高可靠性

Nginx采用的是多进程模式运行，其中有一个master主进程和N多个worker进程，worker进程的数量我们可以手动设置，每个worker进程之间都是相互独立提供服务，并且master主进程可以在某一个worker进程出错时，快速去"拉起"新的worker进程提供服务。

##### (4)热部署

现在互联网项目都要求以7*24小时进行服务的提供，针对于这一要求，Nginx也提供了热部署功能，即可以在Nginx不停止的情况下，对Nginx进行文件升级、更新配置和更换日志文件等功能。

##### (5)成本低、BSD许可证

BSD是一个开源的许可证，世界上的开源许可证有很多，现在比较流行的有六种分别是GPL、BSD、MIT、Mozilla、Apache、LGPL。这六种的区别是什么，我们可以通过下面一张图来解释下：

Nginx本身是开源的，我们不仅可以免费的将Nginx应用在商业领域，而且还可以在项目中直接修改Nginx的源码来定制自己的特殊要求。这些点也都是Nginx为什么能吸引无数开发者继续为Nginx来贡献自己的智慧和青春。OpenRestry [Nginx+Lua]   Tengine[淘宝]

### Nginx的功能特性及常用功能

Nginx提供的基本功能服务从大体上归纳为"基本HTTP服务"、“高级HTTP服务”和"邮件服务"等三大类。

#### 基本HTTP服务

Nginx可以提供基本HTTP服务，可以作为HTTP代理服务器和反向代理服务器，支持通过缓存加速访问，可以完成简单的负载均衡和容错，支持包过滤功能，支持SSL等。

- 处理静态文件、处理索引文件以及支持自动索引；
- 提供反向代理服务器，并可以使用缓存加上反向代理，同时完成负载均衡和容错；
- 提供对FastCGI、memcached等服务的缓存机制，，同时完成负载均衡和容错；
- 使用Nginx的模块化特性提供过滤器功能。Nginx基本过滤器包括gzip压缩、ranges支持、chunked响应、XSLT、SSI以及图像缩放等。其中针对包含多个SSI的页面，经由FastCGI或反向代理，SSI过滤器可以并行处理。
- 支持HTTP下的安全套接层安全协议SSL.
- 支持基于加权和依赖的优先权的HTTP/2

#### 高级HTTP服务

- 支持基于名字和IP的虚拟主机设置
- 支持HTTP/1.0中的KEEP-Alive模式和管线(PipeLined)模型连接
- 自定义访问日志格式、带缓存的日志写操作以及快速日志轮转。
- 提供3xx~5xx错误代码重定向功能
- 支持重写（Rewrite)模块扩展
- 支持重新加载配置以及在线升级时无需中断正在处理的请求
- 支持网络监控
- 支持FLV和MP4流媒体传输

#### 邮件服务

Nginx提供邮件代理服务也是其基本开发需求之一，主要包含以下特性：

- 支持IMPA/POP3代理服务功能
- 支持内部SMTP代理服务功能

#### Nginx常用的功能模块

```
静态资源部署
Rewrite地址重写
	正则表达式
反向代理
负载均衡
	轮询、加权轮询、ip_hash、url_hash、fair
Web缓存
环境部署
	高可用的环境
用户认证模块...
```

Nginx的核心组成

```
nginx二进制可执行文件
nginx.conf配置文件
error.log错误的日志记录
access.log访问日志记录
```

## Nginx 环境准备

### Nginx 版本介绍

Nginx的官方网站为: http://nginx.org

打开源码可以看到如下的页面内容

![1580461114467](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162119770.webp)

Nginx的官方下载网站为<http://nginx.org/en/download.html>，当然你也可以之间在首页选中右边的`download`进入版本下载网页。在下载页面我们会看到如下内容：

![1580463222053](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162125357.webp)

### 获取Nginx源码

<http://nginx.org/download/>

打开上述网站，就可以查看到Nginx的所有版本，选中自己需要的版本进行下载。下载我们可以直接在windows上下载然后上传到服务器，也可以直接从服务器上下载，这个时候就需要准备一台服务器。

![1580610584036](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162131932.webp)



### 准备服务器系统

环境准备

```
VMware WorkStation
Centos7
MobaXterm
	xsheel,SecureCRT
网络
```

(1)确认centos的内核

准备一个内核为2.6及以上版本的操作系统，因为linux2.6及以上内核才支持epoll,而Nginx需要解决高并发压力问题是需要用到epoll，所以我们需要有这样的版本要求。

我们可以使用`uname -a`命令来查询linux的内核版本。

![1581416022481](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162138793.webp)

(2)确保centos能联网

```
ping www.baidu.com
```

(3)确认关闭防火墙

这一项的要求仅针对于那些对linux系统的防火墙设置规则不太清楚的，建议大家把防火墙都关闭掉，因为我们此次课程主要的内容是对Nginx的学习，把防火墙关闭掉，可以省掉后续Nginx学习过程中遇到的诸多问题。

关闭的方式有如下两种：

```
systemctl stop firewalld      关闭运行的防火墙，系统重新启动后，防火墙将重新打开
systemctl disable firewalld   永久关闭防火墙，，系统重新启动后，防火墙依然关闭
systemctl status firewalld	 查看防火墙状态
```

（4）确认停用selinux

selinux(security-enhanced linux),美国安全局对于强制访问控制的实现，在linux2.6内核以后的版本中，selinux已经成功内核中的一部分。可以说selinux是linux史上最杰出的新安全子系统之一。虽然有了selinux，我们的系统会更安全，但是对于我们的学习Nginx的历程中，会多很多设置，所以这块建议大家将selinux进行关闭。

sestatus查看状态

![1581419845687](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162156200.webp)

如果查看不是disabled状态，我们可以通过修改配置文件来进行设置,修改SELINUX=disabled，然后重启下系统即可生效。

```
vim /etc/selinux/config
```

![1581419902873](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162201159.webp)

### Nginx安装方式介绍

Nginx的安装方式有两种分别是:

```
通过Nginx源码
	通过Nginx源码简单安装 (1)
	通过Nginx源码复杂安装 (3)
通过yum安装 (2)
```

如果通过Nginx源码安装需要提前准备的内容：

##### GCC编译器

Nginx是使用C语言编写的程序，因此想要运行Nginx就需要安装一个编译工具。GCC就是一个开源的编译器集合，用于处理各种各样的语言，其中就包含了C语言。

使用命令`yum install -y gcc`来安装

安装成功后，可以通过`gcc --version`来查看gcc是否安装成功

##### PCRE

Nginx在编译过程中需要使用到PCRE库（perl Compatible Regular Expressoin 兼容正则表达式库)，因为在Nginx的Rewrite模块和http核心模块都会使用到PCRE正则表达式语法。

可以使用命令`yum install -y pcre pcre-devel`来进行安装

安装成功后，可以通过`rpm -qa pcre pcre-devel`来查看是否安装成功

##### zlib

zlib库提供了开发人员的压缩算法，在Nginx的各个模块中需要使用gzip压缩，所以我们也需要提前安装其库及源代码zlib和zlib-devel

可以使用命令`yum install -y zlib zlib-devel`来进行安装

安装成功后，可以通过`rpm -qa zlib zlib-devel`来查看是否安装成功

##### OpenSSL

OpenSSL是一个开放源代码的软件库包，应用程序可以使用这个包进行安全通信，并且避免被窃听。

SSL:Secure Sockets Layer安全套接协议的缩写，可以在Internet上提供秘密性传输，其目标是保证两个应用间通信的保密性和可靠性。在Nginx中，如果服务器需要提供安全网页时就需要用到OpenSSL库，所以我们需要对OpenSSL的库文件及它的开发安装包进行一个安装。

可以使用命令`yum install -y openssl openssl-devel`来进行安装

安装成功后，可以通过`rpm -qa openssl openssl-devel`来查看是否安装成功

上述命令，一个个来的话比较麻烦，我们也可以通过一条命令来进行安装

`yum install -y gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel`进行全部安装。



#### 方案一：Nginx的源码简单安装

>   我使用的是这个方案

(1)进入官网查找需要下载版本的链接地址，然后使用wget命令进行下载

```shell
wget http://nginx.org/download/nginx-1.18.0.tar.gz
```

(2)建议大家将下载的资源进行包管理

```shell
mkdir -p /opt/nginx # 创建目录
mv nginx-1.18.0.tar.gz /opt/nginx # 移动nginx压缩包
```

(3)解压缩

```shell
tar -xzf nginx-1.18.0.tar.gz #解压缩
```

(4)进入资源文件中，发现configure

```shell
cd /opt/nginx/nginx-1.18.0 # 进入到目录
./configure # 执行configure文件
```

(5)编译

```shell
make
```

(6)安装

```shell
make install
```

默认安装地址：`/usr/local/nginx/`

(7)启动

```shell
cd /usr/local/nginx/sbin #进入目录
./nginx #执行文件
```

(8)打开80端口

```shell
firewall-cmd --permanent --add-port=80/tcp #打开端口
firewall-cmd --reload #刷新
```

(9)访问Nginx

![image-20210608112755686](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162209308.webp)



#### 方案二：yum安装

使用源码进行简单安装，我们会发现安装的过程比较繁琐，需要提前准备GCC编译器、PCRE兼容正则表达式库、zlib压缩库、OpenSSL安全通信的软件库包，然后才能进行Nginx的安装。

（1）安装yum-utils

```
sudo yum  install -y yum-utils
```

（2）添加yum源文件

```
vim /etc/yum.repos.d/nginx.repo
```

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

（3）查看是否安装成功

```
yum list | grep nginx
```

![1581416861684](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162214684.webp)

（4）使用yum进行安装

```
yun install -y nginx
```

（5）查看nginx的安装位置

```
whereis nginx
```

![1581416981939](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162221393.webp)

（6）启动测试

#### 源码简单安装和yum安装的差异：

这里先介绍一个命令: `./nginx -V`,通过该命令可以查看到所安装Nginx的版本及相关配置信息。

简单安装

![1586016586042](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162227054.webp)

yum安装

![1586016605581](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162231599.webp)

##### 解压Nginx目录

执行`tar -zxvf nginx-1.16.1.tar.gz`对下载的资源进行解压缩，进入压缩后的目录，可以看到如下结构

![1581421319232](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162237612.webp)

内容解释：

auto:存放的是编译相关的脚本

CHANGES:版本变更记录

CHANGES.ru:俄罗斯文的版本变更记录

conf:nginx默认的配置文件

configure:nginx软件的自动脚本程序,是一个比较重要的文件，作用如下：

​	（1）检测环境及根据环境检测结果生成C代码

​	（2）生成编译代码需要的Makefile文件

contrib:存放的是几个特殊的脚本文件，其中README中对脚本有着详细的说明

html:存放的是Nginx自带的两个html页面，访问Nginx的首页和错误页面

LICENSE:许可证的相关描述文件

man:nginx的man手册

README:Nginx的阅读指南

src:Nginx的源代码

#### 方案三:Nginx的源码复杂安装

这种方式和简单的安装配置不同的地方在第一步，通过`./configure`来对编译参数进行设置，需要我们手动来指定。那么都有哪些参数可以进行设置，接下来我们进行一个详细的说明。

PATH:是和路径相关的配置信息

with:是启动模块，默认是关闭的

without:是关闭模块，默认是开启的

我们先来认识一些简单的路径配置已经通过这些配置来完成一个简单的编译：

--prefix=PATH

```
指向Nginx的安装目录，默认值为/usr/local/nginx   
```

--sbin-path=PATH

```
指向(执行)程序文件(nginx)的路径,默认值为<prefix>/sbin/nginx
```

--modules-path=PATH

```
指向Nginx动态模块安装目录，默认值为<prefix>/modules
```

--conf-path=PATH

```
指向配置文件(nginx.conf)的路径,默认值为<prefix>/conf/nginx.conf
```

--error-log-path=PATH 

```
指向错误日志文件的路径,默认值为<prefix>/logs/error.log
```

--http-log-path=PATH  

```
指向访问日志文件的路径,默认值为<prefix>/logs/access.log
```

--pid-path=PATH

```
指向Nginx启动后进行ID的文件路径，默认值为<prefix>/logs/nginx.pid
```

--lock-path=PATH

```
指向Nginx锁文件的存放路径,默认值为<prefix>/logs/nginx.lock
```

要想使用可以通过如下命令

```
./configure --prefix=/usr/local/nginx \
--sbin-path=/usr/local/nginx/sbin/nginx \
--modules-path=/usr/local/nginx/modules \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--error-log-path=/usr/local/nginx/logs/error.log \
--http-log-path=/usr/local/nginx/logs/access.log \
--pid-path=/usr/local/nginx/logs/nginx.pid \
--lock-path=/usr/local/nginx/logs/nginx.lock
```

在使用上述命令之前，需要将之前服务器已经安装的nginx进行卸载，卸载的步骤分为三步骤：

步骤一：需要将nginx的进程关闭

```
./nginx -s stop
```

步骤二:将安装的nginx进行删除

```
rm -rf /usr/local/nginx
```

步骤三:将安装包之前编译的环境清除掉

```
make clean
```

### Nginx目录结构分析

在使用Nginx之前，我们先对安装好的Nginx目录文件进行一个分析，在这块给大家介绍一个工具tree，通过tree我们可以很方面的去查看centos系统上的文件目录结构，当然，如果想使用tree工具，就得先通过`yum install -y tree`来进行安装，安装成功后，可以通过执行`tree /usr/local/nginx`(tree后面跟的是Nginx的安装目录)，获取的结果如下：

![1581439634265](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162244524.webp)

conf:nginx所有配置文件目录

​    CGI(Common Gateway Interface)通用网关【接口】，主要解决的问题是从客户端发送一个请求和数据，服务端获取到请求和数据后可以调用调用CGI【程序】处理及相应结果给客户端的一种标准规范。

​	fastcgi.conf:fastcgi相关配置文件

​	fastcgi.conf.default:fastcgi.conf的备份文件

​	fastcgi_params:fastcgi的参数文件

​	fastcgi_params.default:fastcgi的参数备份文件

​	scgi_params:scgi的参数文件

​	scgi_params.default：scgi的参数备份文件

​    uwsgi_params:uwsgi的参数文件

​	uwsgi_params.default:uwsgi的参数备份文件

​	mime.types:记录的是HTTP协议中的Content-Type的值和文件后缀名的对应关系

​	mime.types.default:mime.types的备份文件

​	nginx.conf:这个是Nginx的核心配置文件，这个文件非常重要，也是我们即将要学习的重点

​	nginx.conf.default:nginx.conf的备份文件

​	koi-utf、koi-win、win-utf这三个文件都是与编码转换映射相关的配置文件，用来将一种编码转换成另一种编码

html:存放nginx自带的两个静态的html页面

​	50x.html:访问失败后的失败页面

​	index.html:成功访问的默认首页

logs:记录入门的文件，当nginx服务器启动后，这里面会有 access.log error.log 和nginx.pid三个文件出现。

sbin:是存放执行程序文件nginx

​	nginx是用来控制Nginx的启动和停止等相关的命令。





## Nginx 常用命令

使用 Nginx 操作前必须进入 Nginx 的目录 `usr/local/nginx/sbin`

>   查看 Nginx 版本号

`./nginx -v`

![image-20210608115209023](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162251639.webp)

>   启动 Nginx

`./nginx`

![image-20210608115459765](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162256199.webp)

>   关闭 Nginx

`./nginx -s stop`

![image-20210608115421139](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162303019.webp)

>   重新加载 Nginx

`./nginx -s reload` 

作用：刷新 nginx.conf 文件，不会关闭 Nginx

![image-20210608115609148](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162308857.webp)





## Nginx 配置文件

默认 Nginx 配置文件的位置：`/usr/local/nginx/conf/nginx.conf`

可以使用 `whereis nginx` 查看 Nginx 配置文件的位置

![image-20210608115800821](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162314013.webp)



### Nginx 配置文件组成部分

![image-20210608120917468](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162319972.webp)

#### 全局块

从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

-   `worker_processes 1`：==进程数设置==，这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约。一般来说，拥有几个逻辑CPU，就设置为几个worker_processes 为宜，但是 worker_processes 超过8个就没有多大意义了。



#### events 块

events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等

-   `worker_connections 1024`：表示每个 work process 支持的最大连接数为 1024.这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。



#### http 块

这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。需要注意的是：http 块也可以包括 **<span style='color: red;'>http 全局块、erver 块</span>**。

##### http 全局块

http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。



##### server 块

这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。 每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。

>   1、全局 server 块

 最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

>   2、location 块

 ==一个 server 块可以配置多个 location 块。==这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（实现效果：使用 nginx 反向代理，访问 www.123.com 直接跳转到 127.0.0.1:8080也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。





## Nginx 反向代理

### Nginx 反向代理 ( 1 )

实现效果：使用 nginx 反向代理，访问 www.123.com 直接跳转到 127.0.0.1:8080

#### 启动 Tomcat

启动就不写了



#### 修改 Windows 系统下的 hosts 文件

```shell
# 文件末尾添加
192.168.200.130 www.123.com
```

`ipconfig /flushdns ` 清除DNS缓存内容



#### 修改 nginx.conf 文件，增加反向代理配置

![image-20210608142159538](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162327536.webp)

然后重启 Nginx

如上配置，我们监听 80 端口，访问域名为 www.123.com，不加端口号时默认为 80 端口，故访问该域名时会跳转到 127.0.0.1:8080 路径上。在浏览器端输入 www.123.com 结果如下：

![image-20210608142239170](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162335353.webp)



### Nginx 反向代理 ( 2 )

#### 环境搭建

`cp -r tomcat tomcat2` 复制一份 Tomcat，并修改 `conf/server.xml`，防止两个 Tomcat 端口冲突

修改 Tomcat 关闭时的端口，默认8005

![image-20210608143538399](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162343347.webp)

修改 Tomcat 启动后的占用端口，默认8080

![image-20210608143645680](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162350767.webp)

在8080端口的 Tomcat 的 webapps 文件夹中创建文件夹 `mkdir test`，再创建文件 `a.html` 

```html
<h1>8080</h1>
```

在8081端口的 Tomcat 的 webapps 文件夹中创建文件夹 `mkdir dev`，再创建文件 `a.html` 

```html
<h1>8081</h1>
```

最后启动 Tomcat



#### 修改 Nginx.con 文件

在 http 块中添加如下配置

```js
server {
	listen 9001;
	server_name 192.168.130.200;

	location ~/dev/  {
		proxy_pass htpp://127.0.0.1:8081;
	}

	location ~/test/ {
		proxy_pass http://127.0.0.1:8080;
	}
}
```

![image-20210608151208466](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162357535.webp)

#### localhost 指令说明

```
location [=|~|~*|^~] uri {
	
}
```

`= `：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。

` ~`：用于表示 uri 包含正则表达式，并且==区分大小写==。

` ~*`：用于表示 uri 包含正则表达式，并且==不区分大小写==。

` ^~`：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。



#### 打开9001端口

`firewall-cmd --permanent --add-port=9001/tcp`

`firewall-cmd --reload`



#### 测试

重新启动 Nginx

访问 `http://192.168.200.130:9001/test/a.html` 和 `http://192.168.200.130:9001/dev/a.html`

![image-20210608152052807](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162404418.webp)

![image-20210608152116213](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162410504.webp)





## Nginx 负载均衡

#### 实现效果

浏览器地址栏输入地址 http://192.168.200.130/dev/a.html，平均转发到8080和8081端口中，实现负载均衡



#### 环境搭建

准备两台 Tomcat 服务器，一台8080，一台8081

在两台 Tomcat 中的 webapps 目录中，创建 dev 文件夹，在 dev 文件夹中创建页面 a.html，用于测试



#### 修改 nginx.conf 文件

在 http 块中添加如下配置

```js
upstream myserver{
	server 192.168.200.130:8080;
	server 192.168.200.130:8081;
}
```

![image-20210608153353155](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162417680.webp)

作用：将多个服务设置到一个组中，Nginx 可以通过组名访问到组中对应的服务。

修改 server 块

![image-20210608153720646](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162422237.webp)

监听80端口，当访问 192.168.130.200 时 Nginx 会转发到 myserver 组中，并进行负载均衡。



#### 测试

重启 Nginx，浏览器访问 http://192.168.200.130/dev/a.html，8080和8081会交替出现

![image-20210608154109222](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162428266.webp)

![image-20210608154109222](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162435644.webp)



#### 负载均衡策略

##### 轮询

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

默认就是轮询，上面就时轮询的案例。



##### weight

weight 代表权重，默认为 1,权重越高被分配的客户端越多，指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。 例如：

```js
upstream server_pool{ 
	server 192.168.5.21 weight=10; 
	server 192.168.5.22 weight=10; 
}
```



##### ip_hash

每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。

```js
upstream server_pool{ 
	ip_hash; 
	server 192.168.5.21:80; 
	server 192.168.5.22:80; 
}
```



##### fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```js
upstream server_pool{ 
    server 192.168.5.21:80; 
    server 192.168.5.22:80; 
    fair; 
}
```





## Nginx 动静分离

### 概述

Nginx 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用 Nginx 处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种，

-   一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；
-   另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。

通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以设置浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。



### 准备工作

创建文件夹，结构如下

![image-20210608164622953](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162444068.webp)

>   html

创建 `/data/html/a.html `

```html
<h1>TEST</h1>
```

>   images

随便复制一张图片过来放在 `/data/images/` 目录下



### 修改 nginx.conf 文件

![image-20210608165605104](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162450205.webp)

当我们在浏览器访问 http://192.168.200.130/html/a.html 时 Nginx 会返回给我们 `/data/html/a.html` 文件。

`root`：文件所在的目录，会拼接在 `/html/a.html` 后面构成一个完整的路径，并将内容返回给浏览器。

`autoindex on`：列出目录全部内容，浏览器访问 http://192.168.200.130/images 没有指定文件名，浏览器会显示 `/data/images` 文件夹中的内容全部显示出来



### 测试

>   访问 http://hong/html/a.html

![image-20210608170439305](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162457451.webp)

>   访问 http://192.168.200.130/images/

==记得后面有一个 `/`，一定不能漏了==

![image-20210608171144621](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162503886.webp)





## Nginx 高可用

![image-20210609112427875](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162510011.webp)

-   需要两台 Nginx 服务器
-   需要 keepalived
-   需要虚拟 ip



### 安装 keepalived

`yum install keepalived -y` 进行安装

可能出现已下错误

```shell
错误：软件包：1:net-snmp-agent-libs-5.7.2-49.el7_9.1.x86_64 (updates)
          需要：libmysqlclient.so.18()(64bit)
错误：软件包：2:postfix-2.10.1-9.el7.x86_64 (@anaconda)
          需要：libmysqlclient.so.18(libmysqlclient_18)(64bit)
错误：软件包：2:postfix-2.10.1-9.el7.x86_64 (@anaconda)
          需要：libmysqlclient.so.18()(64bit)
错误：软件包：1:net-snmp-agent-libs-5.7.2-49.el7_9.1.x86_64 (updates)
          需要：libmysqlclient.so.18(libmysqlclient_18)(64bit)
 您可以尝试添加 --skip-broken 选项来解决该问题
** 发现 3 个已存在的 RPM 数据库问题， 'yum check' 输出如下：
libkkc-0.3.1-9.el7.x86_64 有缺少的需求 libmarisa.so.0()(64bit)
2:postfix-2.10.1-9.el7.x86_64 有缺少的需求 libmysqlclient.so.18()(64bit)
2:postfix-2.10.1-9.el7.x86_64 有缺少的需求 libmysqlclient.so.18(libmysqlclient_18)(64bit)
```

>   解决方案

`wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-community-libs-compat-5.7.25-1.el7.x86_64.rpm`

`rpm -ivh mysql-community-libs-compat-5.7.25-1.el7.x86_64.rpm`

在执行 `yum install keepalived -y`



### 准备两台 Nginx

克隆一份 Linux 系统即可。记得修改 IP 地址。



### 修改 `/etc/keepalived/keepalivec.conf` 配置文件 

配置备 Nginx 时需要注意：需要修改 state 为 BACKUP , priority 比 MASTER 低，virtual_router_id 和 master 的值一致

```shell
global_defs {
	notification_email {
		acassen @firewall.loc
		failover @firewall.loc
		sysadmin @firewall.loc
	}
	notification_email_from Alexandre.Cassen @firewall.loc
	smtp_server 192.168.200.130 # IP
	smtp_connect_timeout 30
	router_id hong # 主机名称, 在 /etc/hosts 文件中查看和设置;
}
vrrp_script chk_http_port {
	script "/usr/local/src/nginx_check.sh"
	interval 2 #（ 检测脚本执行的间隔）
	weight 2
}
vrrp_instance VI_1 {
	state BACKUP # 备份服务器上将 MASTER 改为 BACKUP
	interface ens33 # 网卡, [ip a] 查看
	virtual_router_id 51 # 主、 备机的 virtual_router_id 必须相同
	priority 90 # 主、 备机取不同的优先级， 主机值较大， 备份机值较小
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 1111
	}
	virtual_ipaddress {
		192.168.200.50 # VRRP H 虚拟地址
	}
}
```



### 在 `/usr/local/src` 添加检测脚本

脚本名称 `nginx_check.sh`

```shell
#!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
 /usr/local/nginx/sbin/nginx
 sleep 2
 if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
 killall keepalived
 fi
fi
```



### 测试

-   删除动静分离的配置，不然会有问题
-   `cd /usr/local/nginx/sbin/ `
-   `./nginx` 启动 Nginx
-   `systemctl start keepalived.service` 启动 keepalived
-   两台 Nginx 都要启动

访问 http://192.168.200.50/ 虚拟 IP

![image-20210609124047972](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162518717.webp)

关闭主机的 Nginx 再次访问

![image-20210609124047972](assets/202507250946730.png)

## Nginx 原理

### master 和 worker

![image-20210609161903425](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162525551.webp)

![image-20210609162018944](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162531140.webp)



### worker 如何进行工作的

![image-20210609162132461](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162539033.webp)



### 一个 master 和多个 woker 有好处

（1）可以使用 nginx –s reload 热部署，利用 nginx 进行热部署操作

（2）每个 woker 是独立的进程，如果有其中的一个 woker 出现问题，其他 woker 独立的，继续进行争抢，实现请求过程，不会造成服务中断



### 设置多少个 woker 合适

worker 数和服务器的 cpu 数相等是最为适宜的



### 连接数 worker_connection

>   发送请求，占用了 woker 的几个连接数？

答案：2 或者 4 个

>   nginx 有一个 master，有四个 woker，每个 woker 支持最大的连接数 1024，支持的最大并发数是多少？

普通的静态访问最大并发数是：` worker_connections * worker_processes / 2`**，** 

而如果是 HTTP 作 为反向代理来说，最大并发数量应该是 `worker_connections * worker_processes / 4`。

```
"Binds": [
                "/mydata/nginx/conf/:/etc/nginx",
                "/mydata/nginx/html:/usr/share/nginx/html",
                "/mydata/nginx/logs:/var/log/nginx"
            ],
```





## Nginx 补充知识

当我们访问 http://ip ( 不写端口号默认访问80端口 ) 时会访问了 Nginx 安装目录下的 `html/index.html` 文件；

如果我们需要访问 `html` 文件夹下的内容不需要配置 `nginx.conf` 文件，因为默认就是访问 `html` 文件夹；

比如：http://192.168.200.130/login/login.html，浏览器会直接输出 `html/login/login.html` 文件的内容。

