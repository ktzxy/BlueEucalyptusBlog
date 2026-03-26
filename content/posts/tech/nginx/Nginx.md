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

## Nginx 简介

### 背景介绍

Nginx（“engine x”）一个具有高性能的【HTTP】和【反向代理】的【WEB服务器】，同时也是一个【POP3/SMTP/IMAP代理服务器】，是由伊戈尔·赛索耶夫(俄罗斯人)使用C语言编写的，Nginx的第一个版本是2004年10月4号发布的0.1.0版本。另外值得一提的是伊戈尔·赛索耶夫将Nginx的源码进行了开源，这也为Nginx的发展提供了良好的保障。

![1573470187616](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162046180.webp)

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

![1573489359728](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162052071.webp)

反向代理

![1573489653799](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162059419.webp)

### 常见服务器对比

```
Netcraft公司于1994年底在英国成立，多年来一直致力于互联网市场以及在线安全方面的咨询服务，其中在国际上最具影响力的当属其针对网站服务器、SSL市场所做的客观严谨的分析研究，公司官网每月公布的调研数据（Web Server Survey）已成为当今人们了解全球网站数量以及服务器市场分额情况的主要参考依据，时常被诸如华尔街杂志，英国BBC，Slashdot等媒体报道或引用。
```

我们先来看一组数据，我们先打开Nginx的官方网站  <http://nginx.org/>,找到Netcraft公司公布的数据，对当前主流服务器产品进行介绍。

![1581394945120](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162104281.webp)

上面这张图展示了2019年全球主流Web服务器的市场情况，其中有Apache、Microsoft-IIS、google Servers、Nginx、Tomcat等，而我们在了解新事物的时候，往往习惯通过类比来帮助自己理解事物的概貌。所以下面我们把几种常见的服务器来给大家简单介绍下：

#### IIS

​	全称(Internet Information Services)即互联网信息服务，是由微软公司提供的基于windows系统的互联网基本服务。windows作为服务器在稳定性与其他一些性能上都不如类UNIX操作系统，因此在需要高性能Web服务器的场合下，IIS可能就会被"冷落".

#### Tomcat

​	Tomcat是一个运行Servlet和JSP的Web应用软件，Tomcat技术先进、性能稳定而且开放源代码，因此深受Java爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web应用服务器。但是Tomcat天生是一个重量级的Web服务器，对静态文件和高并发的处理比较弱。

#### Apache

​	Apache的发展时期很长，同时也有过一段辉煌的业绩。从上图可以看出大概在2014年以前都是市场份额第一的服务器。Apache有很多优点，如稳定、开源、跨平台等。但是它出现的时间太久了，在它兴起的年代，互联网的产业规模远远不如今天，所以它被设计成一个重量级的、不支持高并发的Web服务器。在Apache服务器上，如果有数以万计的并发HTTP请求同时访问，就会导致服务器上消耗大量内存，操作系统内核对成百上千的Apache进程做进程间切换也会消耗大量的CUP资源，并导致HTTP请求的平均响应速度降低，这些都决定了Apache不可能成为高性能的Web服务器。这也促使了Lighttpd和Nginx的出现。

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

![image-20260324085618306](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260324085618306.webp)

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

```nginx
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

![image-20260324085903679](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260324085903679.webp)

Nginx的官方下载网站为<http://nginx.org/en/download.html>，当然你也可以之间在首页选中右边的`download`进入版本下载网页。在下载页面我们会看到如下内容：

![image-20260324085947454](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260324085947454.webp)

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
wget http://nginx.org/download/nginx-1.28.2.tar.gz
```

(2)建议大家将下载的资源进行包管理

```shell
mkdir -p /opt/nginx # 创建目录
mv nginx-1.28.2.tar.gz /opt/nginx # 移动nginx压缩包
```

(3)解压缩

```shell
tar -xzf nginx-1.28.2.tar.gz #解压缩
```

(4)进入资源文件中，发现configure

```shell
cd /opt/nginx/nginx-1.28.2 # 进入到目录
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

![image-20210608112755686](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162209308.webp)



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

（4）使用yum进行安装

```
yun install -y nginx
```

（5）查看nginx的安装位置

```
whereis nginx
```

![1581416981939](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162221393.webp)

（6）启动测试

#### 源码简单安装和yum安装的差异：

这里先介绍一个命令: `./nginx -V`,通过该命令可以查看到所安装Nginx的版本及相关配置信息。

简单安装

![1586016586042](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162227054.webp)

yum安装

![1586016605581](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162231599.webp)

##### 解压Nginx目录

执行`tar -zxvf nginx-1.16.1.tar.gz`对下载的资源进行解压缩，进入压缩后的目录，可以看到如下结构

![1581421319232](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162237612.webp)

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

![1581439634265](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162244524.webp)

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

![image-20210608115209023](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162251639.webp)

>   启动 Nginx

`./nginx`

![image-20210608115459765](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162256199.webp)

>   关闭 Nginx

`./nginx -s stop`

![image-20210608115421139](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162303019.webp)

>   重新加载 Nginx

`./nginx -s reload` 

作用：刷新 nginx.conf 文件，不会关闭 Nginx

![image-20210608115609148](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162308857.webp)





## Nginx 配置文件

默认 Nginx 配置文件的位置：`/usr/local/nginx/conf/nginx.conf`

可以使用 `whereis nginx` 查看 Nginx 配置文件的位置

![image-20210608115800821](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162314013.webp)

### Nginx 配置文件组成部分

![image-20210608120917468](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162319972.webp)

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

#### 修改 Windows 系统下的 hosts 文件

```shell
# 文件末尾添加
192.168.200.130 www.123.com
```

`ipconfig /flushdns ` 清除DNS缓存内容

#### 修改 nginx.conf 文件，增加反向代理配置

```nginx
server {
    listen 80;
    server_name www.123.com;

    location / {
        proxy_pass http://127.0.0.1:8080;

        # 【必配】传递真实 IP 和主机头
        proxy_set_header Host $host;             # 让 Tomcat 知道用户访问的是 www.123.com
        proxy_set_header X-Real-IP $remote_addr; # 传递真实 IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # 传递代理链
        
        # 【可选但推荐】如果是 HTTPS 终结，需告诉后端原始协议
        # proxy_set_header X-Forwarded-Proto $scheme; 
    }
}
```

然后重启 Nginx

如上配置，我们监听 80 端口，访问域名为 www.123.com，不加端口号时默认为 80 端口，故访问该域名时会跳转到 127.0.0.1:8080 路径上。在浏览器端输入 www.123.com 结果如下：

![image-20210608142239170](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162335353.webp)



### Nginx 反向代理 ( 2 )

#### 环境搭建

`cp -r tomcat tomcat2` 复制一份 Tomcat，并修改 `conf/server.xml`，防止两个 Tomcat 端口冲突

修改 Tomcat 关闭时的端口，默认8005

![image-20210608143538399](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162343347.webp)

修改 Tomcat 启动后的占用端口，默认8080

![image-20210608143645680](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162350767.webp)

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

	location /dev/ {
    	proxy_pass http://127.0.0.1:8081; 
	}

	location /test/ {
		proxy_pass http://127.0.0.1:8080;
	}
}
```

`proxy_pass` 末尾的斜杠 `/`,不带`/`，则会在`webapps/ROOT/dev/` 下找文件，反之，会在 `webapps/ROOT/a.html` 找文件

以下是**完善后**的配置，包含了真实 IP 传递、超时控制、缓冲优化和错误处理

```nginx
server {
    listen 9001;
    server_name 192.168.130.200; # 或者写 localhost

    # --- 优化点 1: 使用前缀匹配而非正则，性能更好 ---
    # 访问 /dev/xxx -> 转发给 8081
    location /dev/ {
        # 注意：这里没有末尾斜杠，保留 /dev/ 传递给后端
        proxy_pass http://127.0.0.1:8081;

        # --- 优化点 2: 传递真实用户信息 (至关重要) ---
        proxy_set_header Host $host;             # 告诉后端原始域名是哪个
        proxy_set_header X-Real-IP $remote_addr; # 传递真实用户 IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # 传递代理链
        
        # --- 优化点 3: 超时控制 (防止后端卡死拖垮 Nginx) ---
        proxy_connect_timeout 5s;   # 连接后端超时
        proxy_send_timeout 10s;     # 发送数据给后端超时
        proxy_read_timeout 30s;     # 等待后端响应超时 (根据业务调整)

        # --- 优化点 4: 缓冲设置 (提升用户体验，保护后端) ---
        proxy_buffering on;         # 开启缓冲
        proxy_buffer_size 4k;       # 缓冲区大小
        proxy_buffers 8 4k;         # 缓冲区数量和大小
        
        # --- 优化点 5: 错误重试 (高可用) ---
        # 如果后端返回 502/503/504 或超时，自动尝试下一次（如果有 upstream 组）
        proxy_next_upstream error timeout http_502 http_503 http_504;
    }

    # 访问 /test/xxx -> 转发给 8080
    location /test/ {
        proxy_pass http://127.0.0.1:8080;
        
        # 同样需要传递 Header 和设置超时
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        proxy_connect_timeout 5s;
        proxy_read_timeout 30s;
    }
    
    # 可选：全局错误页面定制
    error_page 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

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

![image-20210608152052807](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162404418.webp)

![image-20210608152116213](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162410504.webp)





## Nginx 负载均衡

#### 实现效果

浏览器地址栏输入地址 http://192.168.200.130/dev/a.html，平均转发到8080和8081端口中，实现负载均衡

#### 环境搭建

准备两台 Tomcat 服务器，一台8080，一台8081

在两台 Tomcat 中的 webapps 目录中，创建 dev 文件夹，在 dev 文件夹中创建页面 a.html，用于测试

#### 修改 nginx.conf 文件

在 http 块中添加如下配置

```nginx
upstream myserver{
	server 192.168.200.130:8080;
	server 192.168.200.130:8081;
}
```

![image-20210608153353155](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162417680.webp)

- **默认算法**：**Weighted Round Robin (加权轮询)**。
- **行为**：Nginx 按时间顺序逐一分配请求。第1个给8080，第2个给8081，第3个给8080...
- **优点**：简单，负载绝对平均。
- 缺点/局限：
  1. **无视服务器性能差异**：如果8080是高性能服务器（32核），8081是低配服务器（2核），轮询会导致低配机累死，高配机闲死。
  2. **无视连接长短**：如果某个请求需要处理10秒，另一个只需0.1秒，轮询可能导致长连接堆积在某台机器上。
  3. **Session 丢失问题**：用户第一次请求落到8080（登录了），第二次请求落到8081（没登录状态），导致用户被迫重新登录。

#### 补充：四大负载均衡算法

针对上述局限，Nginx `upstream` 模块提供了多种策略。根据具体的业务场景选择：

##### 1. 加权轮询 (`weight`) —— **最常用**

**场景**：服务器硬件配置不一致。
**配置**：

```nginx
upstream myserver {
    # 权重越高，分到的请求越多。默认 weight=1
    server 192.168.200.130:8080 weight=3; # 性能强，承担3份流量
    server 192.168.200.130:8081 weight=1; # 性能弱，承担1份流量
}
```

**理解**：每4个请求中，3个去8080，1个去8081。

##### 2. IP 哈希 (`ip_hash`) —— **解决 Session 共享问题**

**场景**：后端服务器没有做 Session 共享（如 Redis 集中存储），需要确保同一用户始终访问同一台服务器。
**配置**：

```nginx
upstream myserver {
    ip_hash; # 开启 IP 哈希算法
    server 192.168.200.130:8080;
    server 192.168.200.130:8081;
}
```

**原理**：根据客户端 IP 地址的 hash 值取模，将请求固定分配到某台后端。
**注意**：

- 此时 `weight` 参数失效。
- 如果局域网出口 IP 相同（如公司所有人通过同一个公网 IP 上网），所有人都可能被分到同一台服务器，导致负载不均。

##### 3. 最少连接 (`least_conn`) —— **长连接/耗时任务首选**

**场景**：请求处理时间长短不一，或维持长连接（如 WebSocket, 数据库代理）。
**配置**：

```nginx
upstream myserver {
    least_conn; 
    server 192.168.200.130:8080;
    server 192.168.200.130:8081;
}
```

**原理**：Nginx 会实时统计每台后端的活跃连接数，将新请求发给**当前连接数最少**的那台。这能避免某台机器因为处理慢请求而“堵车”。

##### 4. URL 哈希 (`hash $request_uri`) —— **缓存命中优化**

**场景**：后端有本地缓存（如 Squid, Varnish 或应用层缓存），希望同一 URL 的请求总是落到同一台机器，提高缓存命中率。
**配置**：

```nginx
upstream myserver {
    hash $request_uri consistent; # consistent 表示一致性哈希，节点变动时影响最小
    server 192.168.200.130:8080;
    server 192.168.200.130:8081;
}
```

作用：将多个服务设置到一个组中，Nginx 可以通过组名访问到组中对应的服务。

#### 修改 server 块

仅仅配置 `upstream` 是不够的，在 `location` 中转发时，必须传递正确的信息给后端 Tomcat，否则 Tomcat 获取不到真实用户 IP，日志也是 Nginx 的 IP。

**完整优化后的 `server` 块配置：**

```shell
server {
    listen 80;
    server_name 192.168.200.130;

    location /dev/ {
        # 1. 转发到 upstream 组
        proxy_pass http://myserver; 

        # 2. 传递真实用户 IP (Tomcat 需配置 RemoteIpValve 才能识别)
        proxy_set_header Host $host;             # 保留原始 Host
        proxy_set_header X-Real-IP $remote_addr; # 传递真实 IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # 传递代理链 IP
        
        # 3. 超时设置 (防止后端卡死拖垮 Nginx)
        proxy_connect_timeout 5s;   # 连接后端超时
        proxy_send_timeout 10s;     # 向后端发送数据超时
        proxy_read_timeout 30s;     # 等待后端响应超时 (根据业务调整，长轮询需加大)

        # 4. 缓冲设置 (保护后端，快速响应用户)
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        
        # 5. 错误页面定制 (可选)
        proxy_next_upstream error timeout http_502 http_503 http_504; # 遇到这些错误自动切换下一台
    }
}
```

监听80端口，当访问 192.168.130.200 时 Nginx 会转发到 myserver 组中，并进行负载均衡。

#### 测试

重启 Nginx，浏览器访问 http://192.168.200.130/dev/a.html，8080和8081会交替出现

![image-20210608154109222](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162428266.webp)

![image-20210608154109222](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162435644.webp)

#### 故障转移与健康检查

如果关掉 8080 的 Tomcat，Nginx 默认会怎么做？

- **默认行为**：Nginx 会把请求发给 8080 -> 连接失败 -> 自动尝试发给 8081（故障转移）。
- **问题**：默认情况下，Nginx 认为失败一次就标记为不可用，但过一段时间（默认 `fail_timeout=10s`）又会把它拉回来重试。如果 8080 一直没起，每次重试都会导致用户请求延迟（因为要等超时）。

##### 优化配置：`max_fails` 和 `fail_timeout`

```nginx
upstream myserver {
    # max_fails: 允许失败的最大次数（默认1次）。超过这个次数，认为服务器挂了。
    # fail_timeout: 在多少秒内统计失败次数；以及服务器挂掉后，暂停多久不再分发请求。
    
    server 192.168.200.130:8080 max_fails=3 fail_timeout=30s;
    server 192.168.200.130:8081 max_fails=3 fail_timeout=30s;
    
    # backup: 备用服务器。只有当所有非 backup 服务器都挂了，才启用它。
    # server 192.168.200.130:8082 backup; 
}
```

**效果**：如果 8080 连续失败 3 次，Nginx 会在接下来的 30 秒内彻底不理它，直接全部分发给 8081，避免用户感知到延迟。30 秒后，Nginx 会再试探性地发一个请求给 8080，如果成功了，就恢复负载均衡。

> **💡 进阶提示**：开源版 Nginx **不支持**主动式健康检查（即 Nginx 定期主动发心跳包探测）。它只能被动检测（用户请求失败了才知道）。如果需要主动探测（如每秒 ping 一次），需要购买 **Nginx Plus** 商业版，或者使用第三方模块（如 `nginx_upstream_check_module`），但在 Kubernetes 环境中通常由 K8s 负责健康检查。

## Nginx 动静分离

### 概述

Nginx 动静分离简单来说就是把==动态跟静态请求分开==，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用 Nginx 处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种，

-   一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；
-   另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。

通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以设置浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。



### 准备工作

创建文件夹，结构如下

![image-20210608164622953](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162444068.webp)

>   html

创建 `/data/html/a.html `

```html
<h1>TEST</h1>
```

>   images

随便复制一张图片过来放在 `/data/images/` 目录下

### 修改 nginx.conf 文件

![image-20210608165605104](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162450205.webp)

### 核心配置指令的深度辨析：`root` vs `alias`

`root`，这是最常用的，但在动静分离中，`alias` 往往更灵活。

#### 1. `root` (拼接模式)

- **逻辑**：`完整路径 = root 指定路径 + location 匹配到的 URI`

- 你的例子：

  ```nginx
  location /html/ {
      root /data;
  }
  ```

  - 访问：`http://ip/html/a.html`
  - 映射：`/data` + `/html/a.html` = **`/data/html/a.html`**
  - **特点**：URL 中的路径必须存在于文件系统中。

#### 2. `alias` (替换模式) —— **动静分离推荐**

- **逻辑**：`完整路径 = alias 指定路径` (location 匹配到的部分被**替换**掉)

- 场景：假设你的静态资源散落在服务器各处，或者你想让 URL 看起来更简洁。

  ```nginx
  location /static/ {
      alias /opt/resources/files/;  # 注意：alias 结尾建议加斜杠
  }
  ```

  - 访问：`http://ip/static/logo.png`
  - 映射：**`/opt/resources/files/logo.png`** (`/static/` 被丢弃了)
  - **优势**：文件系统结构可以和 URL 结构完全解耦。

> **💡 最佳实践建议**：
> 如果目录结构一致（如 `/data/images` 对应 `/images`），用 `root` 效率高一点；
> 如果目录结构不一致（如 `/var/www/assets` 对应 `/img`），必须用 `alias`。

### 缓存策略进阶：`expires` vs `Cache-Control`

#### 1. 不常变动的资源（版本化文件名）

对于 `jquery-3.6.0.js`, `logo.v1.png` 这种带版本号的文件，一旦发布几乎不改。

- **策略**：**永久缓存**。

- 配置：

  ```nginx
  location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
      expires max; # 设置为最大时间（通常是 2037 年）
      add_header Cache-Control "public, immutable"; # 告诉浏览器：别问了，这文件永远不变，直接用本地的！
  }
  ```

- **效果**：用户第一次访问后，后续几天甚至几个月都不会再向服务器发送任何请求（连 304 验证都没有），极大节省带宽。

#### 2. 经常变动的资源（HTML 入口文件）

对于 `index.html`，它可能经常引用新的 JS/CSS 版本。

- **策略**：**不缓存或短时间缓存**。

- 配置：

  ```nginx
  location / {
      expires -1; # 禁止缓存
      add_header Cache-Control "no-cache, no-store, must-revalidate";
  }
  ```

- **效果**：每次访问都重新下载 HTML，确保用户能获取到最新的资源链接（从而加载新的带版本的 JS/CSS）。

#### 3. 关于 304 状态码的补充

你提到的“比对最后更新时间”是 `Last-Modified` 机制。

- **流程**：浏览器发送 `If-Modified-Since` -> 服务器比对时间 -> 没变返回 `304 Not Modified` (无 body) -> 变了返回 `200` + 新内容。
- **更强机制 `ETag`**：Nginx 默认开启 `etag on`。它通过计算文件内容的哈希值来比对，比时间戳更精准（防止文件内容没变但时间戳变了导致的无效下载）。

### 性能优化：为什么 Nginx 处理静态这么快？

除了“不用转发给 Tomcat”，Nginx 内部还有两个黑科技加速静态文件传输，建议在 `http` 块中开启：

#### 1. `sendfile on` (零拷贝技术)

- **传统方式**：磁盘 -> 内核缓冲区 -> 用户缓冲区 (Nginx) -> 内核缓冲区 (Socket) -> 网卡。涉及多次 CPU 拷贝和上下文切换。

- **Nginx 方式**：磁盘 -> 内核缓冲区 -> 网卡。

- 配置：

  ```nginx
  sendfile on;
  tcp_nopush on; # 配合 sendfile，积攒足够数据包再发送，减少网络包数量
  ```

#### 2. `open_file_cache` (文件描述符缓存)

- **原理**：Nginx 会缓存已打开文件的描述符、文件大小、修改时间等信息。

- **作用**：当高并发访问同一批静态文件时，不需要每次都去磁盘 `stat` 文件属性，直接从内存读取，极大降低磁盘 I/O。

- 配置示例：

  ```nginx
  http {
      open_file_cache max=10000 inactive=20s; # 最多缓存 1 万个文件，20 秒没访问则清除
      open_file_cache_valid 30s; # 每隔 30 秒检查一次文件是否被修改
      open_file_cache_min_uses 2; # 最少被访问 2 次才放入缓存
  }
  ```

### 动静分离的完整实战配置模板

```nginx
server {
    listen 80;
    server_name 192.168.200.130;

    # --- 1. 动态请求转发给 Tomcat (或其他后端) ---
    location / {
        # 如果没有匹配到下面的静态规则，就走到这里
        proxy_pass http://127.0.0.1:8080; 
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # --- 2. 静态资源：图片/视频 (大文件，长期缓存) ---
    # 使用正则匹配后缀名
    location ~* \.(jpg|jpeg|png|gif|mp4|swf|flv|wmv|avi)$ {
        root /data; 
        # 开启 autoindex 方便调试，生产环境建议关闭 (off) 以防泄露目录结构
        autoindex off; 
        
        expires 30d; # 缓存 30 天
        add_header Cache-Control "public";
        
        # 防盗链 (可选)：只允许自己的域名访问图片
        valid_referers none blocked server_names *.example.com example.com;
        if ($invalid_referer) {
            return 403;
        }
    }

    # --- 3. 静态资源：CSS/JS (小文件，版本控制，永久缓存) ---
    location ~* \.(css|js)$ {
        root /data;
        expires max; # 永久缓存
        add_header Cache-Control "public, immutable"; # 关键：强制浏览器不使用验证
    }

    # --- 4. 特殊目录：使用 alias 映射 ---
    # 比如上传的文件在 /opt/uploads，但希望 URL 是 /files/...
    location /files/ {
        alias /opt/uploads/; 
        expires 1d;
    }
}
```

### 测试

>   访问 http://hong/html/a.html

![image-20210608170439305](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162457451.webp)

>   访问 http://192.168.200.130/images/

==记得后面有一个 `/`，一定不能漏了==

![image-20210608171144621](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162503886.webp)

## Nginx 高可用

![image-20210609112427875](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162510011.webp)

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

### 主节点（192.168.17.129）配置：

`/etc/keepalived/keepalived.conf`

```shell
global_defs {
   # 故障通知邮箱列表（多个邮箱用空格或换行分隔）
   notification_email {
      acassen@firewall.loc
      failover@firewall.loc
      sysadmin@firewall.loc
   }
   # 发件人邮箱地址
   notification_email_from Alexandre.Cassen@firewall.loc
   # SMTP 服务器地址（用于发送邮件告警）
   smtp_server 192.168.200.130
   # SMTP 连接超时时间（秒）
   smtp_connect_timeout 30
   # 路由器 ID，用于标识本台机器（主备建议不同，方便日志排查）
   router_id nginx_master
}

# 定义健康检查脚本
vrrp_script chk_http_port {
   # 执行脚本的路径（必须存在且可执行）
   script "/usr/local/src/nginx_check.sh"
   # 每隔 2 秒执行一次检查
   interval 2
   # 如果检查失败，优先级降低 2（原优先级 - weight）
   weight 2
   # 可选：连续失败多少次才判定为失败（默认 3）
   fall 3
   # 可选：连续成功多少次才恢复（默认 3）
   rise 2
}

# VRRP 实例配置
vrrp_instance VI_1 {
   # 主节点状态设为 MASTER
   state MASTER
   # 绑定网卡名称（请用 'ip a' 命令确认实际网卡名，如 ens33, eth0 等）
   interface ens33
   # 虚拟路由器 ID，主备必须一致（范围 1-255）
   virtual_router_id 51
   # 优先级：主节点设高值（如 100），备节点设低值（如 90）
   priority 100
   # VRRP 通告间隔（秒）
   advert_int 1
   # 认证配置（主备必须一致）
   authentication {
      auth_type PASS          # 简单密码认证
      auth_pass 1111          # 密码（最多8字符）
   }
   # 关联健康检查脚本
   track_script {
      chk_http_port
   }
   # 虚拟 IP 地址（即对外服务的 VIP）
   virtual_ipaddress {
      192.168.17.50/24 dev ens33 label ens33:0
      # 格式：VIP/子网掩码 dev 网卡名 label 别名
      # /24 表示子网掩码 255.255.255.0，根据你的网络调整
   }
}
```

### 备节点（192.168.17.131）配置

### `/etc/keepalived/keepalived.conf`

```shell
global_defs {
   notification_email {
      acassen@firewall.loc
      failover@firewall.loc
      sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.130
   smtp_connect_timeout 30
   # 备节点 router_id 不同，便于区分
   router_id nginx_backup
}

vrrp_script chk_http_port {
   script "/usr/local/src/nginx_check.sh"
   interval 2
   weight 2
   fall 3
   rise 2
}

vrrp_instance VI_1 {
   # 备节点状态设为 BACKUP
   state BACKUP
   interface ens33
   virtual_router_id 51        # 必须与主节点一致
   priority 90                 # 优先级低于主节点
   advert_int 1
   authentication {
      auth_type PASS
      auth_pass 1111           # 必须与主节点一致
   }
   track_script {
      chk_http_port
   }
   virtual_ipaddress {
      192.168.17.50/24 dev ens33 label ens33:0
   }
}
```

### 在 `/usr/local/src` 添加检测脚本

脚本名称 `nginx_check.sh`

```shell
#!/bin/bash
# Nginx 健康检查脚本
# 功能：检测 nginx 进程是否存在，若不存在则尝试启动；若启动失败则停止 keepalived，触发 VIP 漂移

# 统计 nginx 进程数量（注意使用英文短横线 - ）
A=$(ps -C nginx --no-header | wc -l)

# 如果没有 nginx 进程
if [ $A -eq 0 ]; then
    # 尝试启动 nginx（根据你的实际安装路径调整）
    /usr/local/nginx/sbin/nginx
    
    # 等待 2 秒让进程启动
    sleep 2
    
    # 再次检查是否启动成功
    if [ $(ps -C nginx --no-header | wc -l) -eq 0 ]; then
        # 如果仍失败，停止 keepalived，释放 VIP
        killall keepalived
        exit 1
    fi
fi

# 检查成功，退出码 0
exit 0
```

### 设置脚本权限：

```shell
chmod +x /usr/local/src/nginx_check.sh
chown root:root /usr/local/src/nginx_check.sh
```

### 测试与验证步骤

#### 启动

```shell
# 主备节点分别执行
systemctl start keepalived
systemctl enable keepalived
```

### **查看 VIP 是否绑定**

```shell
ip addr show ens33 | grep 192.168.17.50
# 或在主节点执行
ip a
```

### **模拟故障测试**

- **方法一**：停止主节点 nginx

  ```bash
  systemctl stop nginx
  ```

  观察备节点是否在 6 秒内（2s*3fall）接管 VIP。

- **方法二**：停止主节点 keepalived

  ```bash
  systemctl stop keepalived
  ```

  VIP 应立即漂移到备节点。

- **方法三**：关闭主节点防火墙或断网

  ```bash
  iptables -I INPUT -p vrrp -j DROP
  ```

###  **查看日志**

```bash
tail -f /var/log/messages | grep Keepalived
# 或
journalctl -u keepalived -f
```

### 测试

-   删除动静分离的配置，不然会有问题
-   `cd /usr/local/nginx/sbin/ `
-   `./nginx` 启动 Nginx
-   `systemctl start keepalived.service` 启动 keepalived
-   两台 Nginx 都要启动

访问 http://192.168.200.50/ 虚拟 IP

![image-20210609124047972](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162518717.webp)

关闭主机的 Nginx 再次访问

## Nginx 原理

### master 和 worker

![image-20210609161903425](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162525551.webp)

![image-20210609162018944](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162531140.webp)



### worker 如何进行工作的

![image-20210609162132461](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304162539033.webp)



### 一个 master 和多个 woker 有好处

（1）可以使用 nginx –s reload 热部署，利用 nginx 进行热部署操作

（2）每个 woker 是独立的进程，如果有其中的一个 woker 出现问题，其他 woker 独立的，继续进行争抢，实现请求过程，不会造成服务中断

底层机制：

- 零宕机升级（Zero Downtime Upgrade）
  - 当执行 `nginx -s reload` 时，Master 进程会检查新配置语法。如果正确，它会启动新的 Worker 进程，并发送信号给旧的 Worker 进程，让它们在处理完**当前正在处理的请求**后优雅退出。
  - **关键点**：在这个过程中，监听端口（Socket）始终由 Master 持有或平滑移交，因此**没有任何一个连接会被强制断开**。
- 惊群效应（Thundering Herd）的解决
  - 在旧版本或某些配置下，新连接到来时，所有休眠的 Worker 都会被唤醒争抢，只有一个能抢到，其他白忙活。
  - **优化**：现代 Nginx 默认开启了 `accept_mutex on;`（或在较新版本中自动优化），确保同一时刻只有一个 Worker 去接受新连接，极大降低了上下文切换开销。

### 设置多少个 woker 合适

worker 数和服务器的 cpu 数相等是最为适宜的

- 原因

  Nginx 是多进程单线程

  模型（每个 Worker 内部是单线程事件驱动）。

  - 如果设置过多：会导致频繁的 CPU 上下文切换（Context Switch），反而降低性能。
  - 如果设置过少：无法充分利用多核 CPU 的并行处理能力。

- 特殊情况

  - 如果服务器还运行了其他重负载应用（如 Java/Python），可以适当减少 Nginx 的 worker 数（例如 `auto` 或 `核数 - 1`）。
  - 配置写法推荐：`worker_processes auto;`（Nginx 会自动检测并设置为 CPU 核数，最省心）。

### 连接数 worker_connection

>   发送请求，占用了 woker 的几个连接数？

答案：2 或者 4 个

>   nginx 有一个 master，有四个 woker，每个 woker 支持最大的连接数 1024，支持的最大并发数是多少？

普通的静态访问最大并发数是：` worker_connections * worker_processes / 2`**，** 

- **静态资源场景（分母为 2）**：
  - 客户端发起请求 -> 建立连接（占用 1 个连接）。
  - Nginx 读取文件 -> 发送给客户端 -> 关闭连接（占用 1 个连接，或者是保持长连接但逻辑上算一次交互）。
  - 实际上，对于短连接，通常理解为：**1 个连接处理 1 个请求**。但在高并发估算中，考虑到握手和挥手过程，以及部分长连接复用，经验值常除以 2 作为保守估计（或者理解为：每个请求需要“接收”和“发送”两个阶段的操作资源）。
  - **更精准的理解**：如果是 `keepalive` 开启，一个连接可以处理多个请求。公式 `worker_connections * worker_processes / 2` 是一个**保守的安全阈值**，防止内存耗尽。

而如果是 HTTP 作 为反向代理来说，最大并发数量应该是 `worker_connections * worker_processes / 4`。

- 客户端 <-> Nginx（占用 1 个连接）。
- Nginx <-> 后端服务器（占用 1 个连接）。
- **双向占用**：处理一个用户请求，Nginx 需要维持**至少 2 个**网络连接（上游 + 下游）。
- 再加上握手、超时等待、后端响应慢导致的连接堆积，资源消耗翻倍。
- 所以：`总连接数 / 2 (双向) / 2 (安全冗余) = / 4`。

## Nginx HTTPS / SSL 终结

```nginx
server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate     /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;

    # 推荐的安全协议和加密套件
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header X-Forwarded-Proto $scheme; # 告诉后端原始协议是 HTTPS
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # HTTP 自动跳转 HTTPS
    listen 80;
    return 301 https://$server_name$request_uri;
}
```

## Nginx 访问控制与安全加固

### 常见用途：

- 限制特定 IP 或网段访问（如后台管理只允许内网访问）。
- 防止恶意爬虫、DDoS 攻击。
- 隐藏敏感路径或文件。

### 核心配置示例：

#### 1. IP 白名单/黑名单

```nginx
location /admin/ {
    allow 192.168.1.0/24;   # 允许内网
    allow 10.0.0.5;         # 允许特定 IP
    deny all;               # 其他全部拒绝
    proxy_pass http://admin_backend;
}
```

#### 2. 限制请求频率（防刷）

```nginx
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

    server {
        location /login/ {
            limit_req zone=one burst=5 nodelay; # 每秒1个请求，突发允许5个
            proxy_pass http://auth_service;
        }
    }
}
```

#### 3. 隐藏版本号 & 禁用危险方法

```nginx
http {
    server_tokens off; # 隐藏 Nginx 版本号

    if ($request_method !~ ^(GET|HEAD|POST|PUT|DELETE)$ ) {
        return 405;
    }
}
```

#### 4. 禁止访问敏感文件

```nginx
location ~ /\.ht {
    deny all;
}

location ~* \.(git|svn|env|bak|sql)$ {
    deny all;
}
```

## Nginx 缓存优化

#### 开启代理缓存

```nginx
http {
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m use_temp_path=off;

    server {
        location /api/ {
            proxy_cache my_cache;
            proxy_cache_valid 200 302 10m;
            proxy_cache_valid 404 1m;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
            proxy_cache_lock on;
            add_header X-Cache-Status $upstream_cache_status; # 调试用

            proxy_pass http://backend;
        }
    }
}
```

#### 静态资源强缓存（结合 expires）

```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
    access_log off; # 关闭日志减少 IO
}
```

## nginx 日志增强与监控

#### 自定义日志格式

```nginx
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'rt=$request_time uct="$upstream_connect_time" '
                    'uht="$upstream_header_time" urt="$upstream_response_time" '
                    'cs=$upstream_cache_status';

    access_log /var/log/nginx/access.log main;
}
```

#### 关键指标说明：

- `rt`: 总请求耗时
- `uct`: 连接上游耗时
- `uht`: 等待上游响应头耗时
- `urt`: 上游总响应耗时
- `cs`: 缓存命中状态（HIT/MISS/BYPASS）

## Nginx URL 重写与重定向

#### 301 永久重定向

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://www.example.com$request_uri;
}
```

#### URL 重写（伪静态）

```nginx
location /article/ {
    rewrite ^/article/([0-9]+)\.html$ /article.php?id=$1 last;
}
```

#### 去除尾部斜杠或添加斜杠

```nginx
# 去除尾部斜杠
rewrite ^(.*)/$ $1 permanent;

# 添加尾部斜杠（目录）
if (-d $request_filename) {
    rewrite ^(.*)$ $1/ permanent;
}
```

## Nginx 限流与熔断

#### 按 IP 限流

```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    proxy_pass http://api_backend;
}
```

#### 按区域限流（地理围栏）

```nginx
geo $country_code {
    default US;
    192.168.1.0/24 CN;
    10.0.0.0/8 RU;
}

map $country_code $deny_country {
    default 0;
    RU 1;
    KP 1;
}

server {
    if ($deny_country) {
        return 403;
    }
}
```

