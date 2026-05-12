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

### 详细解析

```shell
#user  nobody;   # 用户  用户组  ；一般只有类unix系统有，windows不需要
worker_processes  1;   # 工作进程，一般配置为cpu核心数的2倍

#  错误日志的配置路径
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;


# 进程id存放路径
#pid        logs/nginx.pid;




# 指定进程可以打开的最大描述符的数量
# 整个系统的最大文件描述符打开数目可以通过命令 ulimit -n  查看
# 所以一般而言，这个值可以是 ulimit -n 除以nginx进程数
#  worker_rlimit_nofile 65535;

events {
    # 使用epoll的I/O 模型；epoll 使用于Linux内核2.6版本及以后的系统
    use epoll;
    # 每一个工作进程的最大连接数量； 理论上而言每一台nginx服务器的最大连接数为： worker_processes*worker_connections
    worker_connections  1024;

    # 超时时间
    #keepalive_timeout 60

    # 客户端请求头部的缓冲区大小，客户端请求一般会小于一页； 可以根据你的系统的分页大小来设定， 命令 getconf PAGESIZE 可以获得当前系统的分页大小（一般4K）
    #client_header_buffer_size 4k;

    # 为打开的文件指定缓存，默认是不启用； max指定缓存数量，建议和打开文件数一致；inactive是指经过这个时间后还没有被请求过则清除该文件的缓存。
    #open_file_cache max=65535 inactive=60s;

    # 多久会检查一次缓存的有效信息
    #open_file_cache_valid 80s;

    # 如果在指定的参数open_file_cache的属性inactive设置的值之内，没有被访问这么多次（open_file_cache_min_uses），则清除缓存
    # 则这里指的是 60s内都没有被访问过一次则清除 的意思
    # open_file_cache_min_uses 1;
}



# 设定http服务；可以利用它的反向代理功能提供负载均衡支持
http {
    #设定mime类型,类型由mime.types文件定义；  可以 cat nginx/conf/mime.types  查看下支持哪些类型
    include       mime.types;
    # 默认mime类型；  application/octet-stream 指的是原始二进制流
    default_type  application/octet-stream;



    # --------------------------------------------------------------------------   #
    # 日志格式设置：
    #   $remote_addr、$http_x_forwarded_for     可以获得客户端ip地址
    #   $remote_user                            可以获得客户端用户名
    #   $time_local                             记录访问的时区以及时间
    #   $request                                请求的url与http协议
    #   $status                                 响应状态成功为200
    #   $body_bytes_sent                        发送给客户端主体内容大小
    #   $http_referer                           记录从哪个页面过来的请求
    #   $http_user_agent                        客户端浏览器信息
    #
    #   注意事项：
    #           通常web服务器(我们的tomcat)放在反向代理(nginx)的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。
    #           反向代理服务器(nginx)在转发请求的http头信息中，可以增加$http_x_forwarded_for信息，记录原有客户端的IP地址和原来客户端的请求的服务器地址。
    # --------------------------------------------------------------------------   #
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    
    #log_format log404 '$status [$time_local] $remote_addr $host$request_uri $sent_http_location';


    # 日志文件；  【注意：】用了log_format则必须指定access_log来指定日志文件
    #access_log  logs/access.log  main;
    #access_log  logs/access.404.log  log404;


    # 保存服务器名字的hash表由 server_names_hash_bucket_size、server_names_hash_max_size 控制
    # server_names_hash_bucket_size 128;

    # 限制通过nginx上传文件的大小
    #client_max_body_size 300m;


    # sendfile 指定 nginx 是否调用sendfile 函数（零拷贝 方式）来输出文件；
    # 对于一般常见应用，必须设为on。
    # 如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile        on;

    # 此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用；TCP_CORK TCP_NODELAY 选项可以是否打开TCP的内格尔算法
    #tcp_nopush     on;


    #tcp_nodelay on;

    # 后端服务器连接的超时时间_发起握手等候响应超时时间
    #proxy_connect_timeout 90; 

    # 连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
    #proxy_read_timeout 180;

    # 后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
    #proxy_send_timeout 180;


    # proxy_buffering 这个参数用来控制是否打开后端响应内容的缓冲区，如果这个设置为on，以下的proxy_buffers才生效
    #proxy_buffering on

    # 设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，
    # 默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
    #proxy_buffer_size 256k;

    # 设置用于读取应答（来自被代理服务器--如tomcat）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
    #proxy_buffers 4 256k;


    # 同一时间处理的请求buffer大小；也可以说是一个最大的限制值--控制同时传输到客户端的buffer大小的。
    #proxy_busy_buffers_size 256k;

    # 设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
    #proxy_temp_file_write_size 256k;


    # proxy_temp_path和proxy_cache_path指定的路径必须在同一分区
    #proxy_temp_path /app/tmp/proxy_temp_dir;

    # 设置内存缓存空间大小为200MB，1天没有被访问的内容自动清除，硬盘缓存空间大小为10GB。
    #proxy_cache_path /app/tmp/proxy_cache_dir levels=1:2 keys_zone=cache_one:200m inactive=1d max_size=30g;


    # 如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。
    # 如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
    # 无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
    #client_body_buffer_size 512k;   # 默认是页大小的两倍

    # 表示使nginx阻止HTTP应答代码为400或者更高的应答。可以结合error_page指向特定的错误页面展示错误信息
    #proxy_intercept_errors on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;



    # 负载均衡 START>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    # upstream 指令定义的节点可以被proxy_pass指令引用；二者结合用来反向代理+负载均衡配置
    # 【内置策略】：轮询、加权轮询、ip_hash、最少连接 默认编译进了nginx
    # 【扩展策略】：fair、通用hash、一致性hash 默认没有编译进nginx
    #-----------------------------------------------------------------------------------------------#
    # 【1】默认是轮询；如果后端服务器down掉，能自动剔除。
    #   upstream bakend {
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #        server 192.168.75.134:8080;
    #    }
    #
    #【2】权重轮询(加权轮询)：这样配置后，如果总共请求了3次，则前面两次请求到130，后面一次请求到132
    #   upstream bakend {
    #        server 192.168.75.130:8080 weight=2;
    #        server 192.168.75.132:8080 weight=1;
    #   }
    #
    #【3】ip_hash：这种配置会使得每个请求按访问者的ip的hash结果分配，这样每个访客固定访问一个后端服务器，这样也可以解决session的问题。
    #   upstream bakend {
    #        ip_hash;
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #   }
    #
    #【4】最少连接：将请求分配给连接数最少的服务器。Nginx会统计哪些服务器的连接数最少。
    #   upstream bakend {
    #        least_conn;
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #   }
    #
    #
    #【5】fair策略(需要安装nginx的第三方模块fair)：按后端服务器的响应时间来分配请求，响应时间短的优先分配。
    #   upstream bakend {
    #        fair;
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #   }
    #
    #【6】url_hash策略（也是第三方策略）：按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
    #               在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method指定hash算法
    #   upstream bakend {
    #        server 192.168.75.130:8080;
    #        server 192.168.75.132:8080;
    #        hash $request_uri;
    #        hash_method crc32;
    #   }
    #
    #【7】其他设置，主要是设备的状态设置
    #   upstream bakend{
    #       ip_hash;
    #       server 127.0.0.1:9090 down;   # down 表示该机器处于下线状态不可用
    #       server 127.0.0.1:8080 weight=2;
    #       server 127.0.0.1:6060;
    #       
    #       # max_fails 默认为1； 最大请求失败的次数，结合fail_timeout使用；
    #       # 以下配置表示 192.168.0.100:8080在处理请求失败3次后，将在15s内不会受到任何请求了
    #       # fail_timeout 默认为10秒。某台Server达到max_fails次失败请求后，在fail_timeout期间内，nginx会认为这台Server暂时不可用，不会将请求分配给它。
    #       server 192.168.0.100:8080 weight=2 max_fails=3 fail_timeout=15;
    #       server 192.168.0.101:8080 weight=3;
    #       server 192.168.0.102:8080 weight=1;
    #       # 限制分配给某台Server处理的最大连接数量，超过这个数量，将不会分配新的连接给它。默认为0，表示不限制。注意：1.5.9之后的版本才有这个配置
    #       server 192.168.0.103:8080 max_conns=1000;
    #       server 127.0.0.1:7070 backup;  # 备份机；其他机器都不可用时，这台机器就上场了
    #       server example.com my_dns_resolve;  # 指定域名解析器；my_dns_resolve需要在http节点配置resolver节点如：resolver 10.0.0.1;
    #   }
    #
    #
    #
    #负载均衡 END<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
    #-----------------------------------------------------------------------------------------------#


    ###配置虚拟机START>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    #
    #server{
    #    listen 80;
    #    配置监听端口
    #    server_name image.***.com;
    #    配置访问域名
    #    location ~* \.(mp3|exe)$ {
    #        对以“mp3或exe”结尾的地址进行负载均衡
    #        proxy_pass http://img_relay$request_uri;
    #        设置被代理服务器的端口或套接字，以及URL
    #        proxy_set_header Host $host;
    #        proxy_set_header X-Real-IP $remote_addr;
    #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #        以上三行，目的是将代理服务器收到的用户的信息传到真实服务器上
    #    }
    #
    #
    #
    #    location /face {
    #        if ($http_user_agent ~* "xnp") {
    #            rewrite ^(.*)$ http://211.151.188.190:8080/face.jpg redirect;
    #        }
    #        proxy_pass http://img_relay$request_uri;
    #        proxy_set_header Host $host;
    #        proxy_set_header X-Real-IP $remote_addr;
    #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #        error_page 404 502 = @fetch;
    #    }
    #
    #    location @fetch {
    #        access_log /data/logs/face.log log404;
    #        rewrite ^(.*)$ http://211.151.188.190:8080/face.jpg redirect;
    #    }
    #
    #    location /image {
    #        if ($http_user_agent ~* "xnp") {
    #            rewrite ^(.*)$ http://211.151.188.190:8080/face.jpg redirect;
    #
    #        }
    #        proxy_pass http://img_relay$request_uri;
    #        proxy_set_header Host $host;
    #        proxy_set_header X-Real-IP $remote_addr;
    #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #        error_page 404 502 = @fetch;
    #    }
    #
    #    location @fetch {
    #        access_log /data/logs/image.log log404;
    #        rewrite ^(.*)$ http://211.151.188.190:8080/face.jpg redirect;
    #    }
    #}
    #
    #
    #
    ###其他举例
    #
    #server{
    #
    #    listen 80;
    #
    #    server_name *.***.com *.***.cn;
    #
    #    location ~* \.(mp3|exe)$ {
    #        proxy_pass http://img_relay$request_uri;
    #        proxy_set_header Host $host;
    #        proxy_set_header X-Real-IP $remote_addr;
    #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #    }
    #
    #    location / {
    #        if ($http_user_agent ~* "xnp") {
    #            rewrite ^(.*)$ http://i1.***img.com/help/noimg.gif redirect;
    #        }
    #
    #        proxy_pass http://img_relay$request_uri;
    #        proxy_set_header Host $host;
    #        proxy_set_header X-Real-IP $remote_addr;
    #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #        #error_page 404 http://i1.***img.com/help/noimg.gif;
    #        error_page 404 502 = @fetch;
    #    }
    #
    #    location @fetch {
    #        access_log /data/logs/baijiaqi.log log404;
    #        rewrite ^(.*)$ http://i1.***img.com/help/noimg.gif redirect;
    #    }
    #}
    #
    #server{
    #    listen 80;
    #    server_name *.***img.com;
    #
    #    location ~* \.(mp3|exe)$ {
    #        proxy_pass http://img_relay$request_uri;
    #        proxy_set_header Host $host;
    #        proxy_set_header X-Real-IP $remote_addr;
    #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #    }
    #
    #    location / {
    #        if ($http_user_agent ~* "xnp") {
    #            rewrite ^(.*)$ http://i1.***img.com/help/noimg.gif;
    #        }
    #
    #        proxy_pass http://img_relay$request_uri;
    #        proxy_set_header Host $host;
    #        proxy_set_header X-Real-IP $remote_addr;
    #        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #        #error_page 404 http://i1.***img.com/help/noimg.gif;
    #        error_page 404 = @fetch;
    #    }
    #
    ##access_log off;
    #
    #    location @fetch {
    #        access_log /data/logs/baijiaqi.log log404;
    #        rewrite ^(.*)$ http://i1.***img.com/help/noimg.gif redirect;
    #    }
    #}
    #
    #server{
    #    listen 8080;
    #    server_name ngx-ha.***img.com;
    #
    #    location / {
    #        stub_status on;
    #        access_log off;
    #    }
    #}
    #
    #server {
    #    listen 80;
    #    server_name imgsrc1.***.net;
    #    root html;
    #}
    #
    #
    #
    #server {
    #    listen 80;
    #    server_name ***.com w.***.com;
    #    # access_log /usr/local/nginx/logs/access_log main;
    #    location / {
    #        rewrite ^(.*)$ http://www.***.com/ ;
    #    }
    #}
    #
    #server {
    #    listen 80;
    #    server_name *******.com w.*******.com;
    #    # access_log /usr/local/nginx/logs/access_log main;
    #    location / {
    #        rewrite ^(.*)$ http://www.*******.com/;
    #    }
    #}
    #
    #server {
    #    listen 80;
    #    server_name ******.com;
    #
    #    # access_log /usr/local/nginx/logs/access_log main;
    #
    #    location / {
    #        rewrite ^(.*)$ http://www.******.com/;
    #    }
    #
    #    location /NginxStatus {
    #        stub_status on;
    #        access_log on;
    #        auth_basic "NginxStatus";
    #        auth_basic_user_file conf/htpasswd;
    #    }
    #
    #    #设定查看Nginx状态的地址
    #    location ~ /\.ht {
    #        deny all;
    #    }#禁止访问.htxxx文件
    #
    #}
    #配置虚拟机END<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            root   /app;
	    index   zyt505050.html;
	    #index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

## Nginx 反向代理

- **`upstream`**: 定义一个后端服务器集群。
- **`server`**: 在 `upstream` 块中定义一台后端服务器。
- **`least_conn`**: 一种负载均衡算法，将请求发送到当前活动连接数最少的服务器。
- **`proxy_pass`**: 在 `location` 块中，将请求转发到指定的 `upstream` 集群。

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

#### 负载均衡技术

负载均衡实现多种，硬件到软件，商业的、开源的。常见的负载均衡方式：

### HTTP重定向负载均衡（较差）

需要**HTTP重定向服务器**。

#### 流程

- 浏览器先去HTTP重定向服务器请求，获取到实际被定向到的服务器；浏览器再请求被实际定向到的服务器。

#### 特点

简单易实现。

#### 缺点

- 浏览器需要两次请求才能完成一次完整的访问，性能以及体验太差。
- 依赖于**重定向服务器**自身处理能力，集群伸缩性规模有限。
- 使用HTTP 302进行重定向可能使搜索引擎判断为SEO作弊，降低搜索排名。

### DNS域名解析负载均衡（一般）

需要**DNS服务器**。

#### 流程

- 浏览器请求某个域名，DNS服务器返回web服务器集群中的某个ip地址。
- 浏览器再去请求DNS返回的服务器地址。


#### 特点

该方案要要求在DNS服务器中配置多个A记录，例如：

| www.example.com IN A | 10.192.168.66 |
| :------------------- | :------------ |
| www.example.com IN A | 10.192.168.77 |
| www.example.com IN A | 10.192.168.88 |

该方案将负载均衡的工作交给了DNS，节省了网站管理维护负载均衡服务器的麻烦。

#### 缺点

- 目前的DNS是**多级解析**的，每一级别都可能缓存了A记录，当某台业务服务器下线后，即使修改了DNS的A记录，要使其生效还需要一定时间。这期间，可能导致用户会访问已下线的服务器造成访问失败。
- DNS负载均衡的控制权在域名服务商那里，网站无法对其做出更多的改善和管理。

>**温馨提示**
>
>事实上，可能是部分使用DNS域名解析，利用域名解析作为第一级的负载均衡手段，域名解析得到的是同样提供负载均衡的内部服务器，这组服务器再进行负载均衡，请求分发到真是的web应用服务器。


### 反向代理负载均衡（良）

反向代理服务器需要配置双网卡和内外部两套IP地址。即**反向代理服务器需要有外部IP、内部IP**。

#### 特点

部署简单，负载均衡功能和反向代理服务器功能集成在一块。

#### 缺点

反向代理服务器是所有请求和相应的中转站，性能可能成为瓶颈。


### IP负载均衡（良）

#### 特点

优点是在**内核进程**中完成了数据分发，而反向代理负载均衡是在应用程序中分发数据，IP负载均衡处理性能更优。

#### 缺点

所有请求、响应都经由负载均衡服务器，导致**集群最大响应数据吞吐量不得不受制于负载均衡服务器网卡带宽**。


### 数据链路层负载均衡（优）

#### 特点

- 数据链路层负载均衡也叫**三角传输模式**。
- 负载均衡数据分发过程中**不修改IP地址，而是修改MAC地址**。
- 由于实际处理请求的真实物理IP地址和数据请求目的IP地址一致，所以不需要通过负载均衡服务器进行地址转换，可以将响应数据包直接返回给用户浏览器，**避免负载均衡服务器网卡宽带成为瓶颈**。

这种负载均衡方式也叫作**直接路由方式（DR）**。

>使用三角传输模式的链路层负载均衡是目前大型网站使用最广泛的一种负载均衡手段。
>
>在Linux平台上最好的链路层负载均衡开源产品是LVS（Linux Virutal Server）。

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

### Keepalived工作原理

> Keepalived类似一个工作在layer3,4&7的交换机。
>
> 
>
> Layer3,4&7工作在IP/TCP协议栈的IP层，TCP层，及应用层,原理分别如下：
>
> Layer3：Keepalived使用Layer3的方式工作式时，Keepalived会定期向服务器群中的服务器发送一个ICMP的数据包（即我们平时用的Ping程序）,如果发现某台服务的IP地址没有激活，Keepalived便报告这台服务器失效，并将它从服务器群中剔除，这种情况的典型例子是某台服务器被 非法关机。Layer3的方式是以服务器的IP地址是否有效作为服务器工作正常与否的标准。
>
> Layer4:如果您理解了Layer3的方式，Layer4就容易了。Layer4主要以TCP端口的状态来决定服务器工作正常与否。如web server的服务端口一般是80，如果Keepalived检测到80端口没有启动，则Keepalived将把这台服务器从服务器群中剔除。
>
> Layer7：Layer7就是工作在具体的应用层了，比Layer3,Layer4要复杂一点，在网络上占用的带宽也要大一些。Keepalived将根据用户的设定检查服务器程序的运行是否正常，如果与用户的设定不相符，则Keepalived将把服务器从服务器群中剔除。
>
> ---- 来自搜狗百科

### Keepalived作用

主要用作真实服务器的健康状态检查以及负载均衡的主机和备机之间故障切换实现。

### Keepalived高可用架构

Keepalived高可用简单架构概念视图：

当主down机后，会提升备为主，保证服务可用。

当主再次上线时，备会退位，主继续服务。

```shell
Keepalived 本质是基于 VRRP 协议的四层高可用方案，通过 VIP 漂移实现服务冗余，结合健康检查实现故障自动切换，不解析业务流量，适用于 LVS/Nginx 等负载均衡入口的高可用保障。
```

### Keepalived组成

Keepalived 模块化设计，主要有三个模块，分别是`core`、`check`和 `vrrp`。 

- core
  - **keepalived的核心，负责主进程的启动和维护、全局配置文件的加载解析** 
- check
  - **负责healthchecker(健康检查)，包括各种健康检查方式，以及对应的配置的解析包括LVS的配置解析；可基于脚本检查对IPVS后端服务器健康状况进行检查。** 
- vrrp
  - **VRRPD子进程就是来实现VRRP（虚拟路由冗余协议）协议的**

![image-20260415083754300](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260415083754300.png)

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

```shell
# 源码方式安装
#下载
wget http://www.keepalived.org/software/keepalived-1.2.23.tar.gz
#解压
tar -zxvf keepalived-1.2.23.tar.gz
cd keepalived-1.2.23
#安装
./configure --prefix=/usr/local/keepalived   #prefix指定安装目录
make
make install


vi /usr/local/keepalived/etc/keepalived/keepalived.conf
替换为你自身逻辑的配置文件的内容。


vi /usr/local/keepalived/etc/sysconfig/keepalived
添加以下内容：
KEEPALIVED_OPTIONS="-D -f /usr/local/keepalived/etc/keepalived/keepalived.conf" #不使用默认的，自己显示指定keepalived配置文件路径


# 因为我们使用非默认路径（/usr/local）安装keepalived，需要设置一些软链接以保证keepalived能正常启动：
ln -s /usr/local/keepalived/sbin/keepalived  /usr/bin #将keepalived主程序加入到环境变量
ln -s /usr/local/keepalived/etc/rc.d/init.d/keepalived  /etc/init.d/ #keepalived启动脚本，放到/etc/init.d/目录下就可以使用service命令便捷调用
ln -s /usr/local/keepalived/etc/sysconfig/keepalived  /etc/sysconfig/ #keepalived启动脚本变量引用文件，默认文件路径是/etc/sysconfig/，也可以不做软链接，直接修改启动脚本中文件路径即可


启动服务
service keepalived start

# 可以检查下服务是否正常（没有消息就是最好的消息）
chkconfig keepalived on

# 查看绑定好的虚拟ip（vip）
ip addr
```

在执行 `yum install keepalived -y`

```shell
# 完整keepalived.conf 配置解析
#全局配置
global_defs { 
	#故障通知邮箱（一般生产才用）
   notification_email { 
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   # 发邮件配置（报警用）
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   
   # 本节点标识（日志用）
   router_id LVS_DEVEL  
   
   # VRRP 行为控制 vrrp_strict：严格模式（更安全） vrrp_skip_check_adv_addr：跳过地址检查
   vrrp_skip_check_adv_addr
   vrrp_strict
   
   # ARP 广播控制（VIP漂移时用）
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

# 定义一个 VRRP 实例
vrrp_instance VI_1 {
    # 当前节点是主
    state MASTER
    # 绑定网卡
    interface eth0
    # 虚拟路由ID（主备必须一样）
    virtual_router_id 51
    # 优先级（决定主备）
    priority 100
    # 心跳间隔（1秒）
    advert_int 1
    # 主备认证（必须一致）
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP（VIP）可多个
    virtual_ipaddress {
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}
# LVS负载均衡
virtual_server 192.168.200.100 443 {
	# 健康检查间隔（6秒）
    delay_loop 6
    # 负载均衡算法 rr = 轮询
    lb_algo rr
    # LVS模式：NAT 地址转换（简单） DR 直连（高性能） TUN 隧道
    lb_kind NAT
    # 会话保持（50秒）
    persistence_timeout 50
    # 协议
    protocol TCP
	
	# 真实服务器
    real_server 192.168.201.100 443 {
        weight 1
        # 健康检查方式（HTTPS）
        SSL_GET {
       		 # 检查： 访问 / 返回内容的MD5必须匹配
            url {
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            # 超时与重试机制
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

# 带备用服务器
virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
	
	# 所有后端挂了 → 转发到“兜底服务器”
    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

```shell
| 项目 | 你现在      | 这个配置    |
| -- | -------- | ------- |
| 层级 | 7层（HTTP） | 4层（TCP） |
| 软件 | Nginx    | LVS     |
| 功能 | 反向代理     | 高性能转发   |
| 性能 | 一般       | 极高      |

Keepalived不仅可以实现高可用（VRRP），
还可以结合LVS实现四层负载均衡。

在生产环境中：

1. VRRP负责VIP漂移，保证服务不宕机
2. LVS负责流量分发，保证高性能
3. 健康检查决定后端节点是否参与调度

相比Nginx，LVS工作在内核层，
性能更高，但功能更简单（不解析HTTP）
```

```shell
LVS 是一个工作在内核层的四层负载均衡器，
只负责请求转发，不解析业务内容，
特点是性能极高，但功能简单。

适用于高并发场景，
通常与 Nginx 配合使用。
```

### 准备两台 Nginx

克隆一份 Linux 系统即可。记得修改 IP 地址。

### 主节点（192.168.157.133）配置：

`/etc/keepalived/keepalived.conf`

```shell
global_defs {
   # 故障通知邮箱列表（多个邮箱用空格或换行分隔）
   #notification_email {
    #  acassen@firewall.loc
    #  failover@firewall.loc
    #  sysadmin@firewall.loc
   #}
   # 发件人邮箱地址
   #notification_email_from Alexandre.Cassen@firewall.loc
   # SMTP 服务器地址（用于发送邮件告警）
   #smtp_server 192.168.200.130
   # SMTP 连接超时时间（秒）
   #smtp_connect_timeout 30
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
   weight -20
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
      192.168.157.100/24 dev ens33 label ens33:0
      # 格式：VIP/子网掩码 dev 网卡名 label 别名
      # /24 表示子网掩码 255.255.255.0，根据你的网络调整
   }
}
```

### 备节点（192.168.157.136）配置

### `/etc/keepalived/keepalived.conf`

```shell
global_defs {
   #notification_email {
    #  acassen@firewall.loc
   #   failover@firewall.loc
    #  sysadmin@firewall.loc
   #}
   #notification_email_from Alexandre.Cassen@firewall.loc
   #smtp_server 192.168.200.130
   #smtp_connect_timeout 30
   # 备节点 router_id 不同，便于区分
   router_id nginx_backup
}

vrrp_script chk_http_port {
   script "/usr/local/src/nginx_check.sh"
   interval 2
   weight -20
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
      192.168.157.100/24 dev ens33 label ens33:0
   }
}
```

### 在 `/usr/local/src` 添加检测脚本

脚本名称 `nginx_check.sh`

```shell
#!/bin/bash
# Nginx 健康检查脚本
# 功能：检测 nginx 进程是否存在，若不存在则尝试启动；若启动失败则停止 keepalived，触发 VIP 漂移

# 检查 nginx 是否存活
if ! pidof nginx > /dev/null; then
    systemctl start nginx
    sleep 2
fi

# 再检查一次
if ! pidof nginx > /dev/null; then
    exit 1
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
ip addr show ens33 | grep 192.168.157.100
# 或在主节点执行
ip a
```

正常访问验证（基线测试）

```shell
# 浏览器访问
http://192.168.157.100

# VIP + nginx 工作正常

# 报错
[root@localhost ~]# curl 192.168.157.100
<!DOCTYPE html>
<html>
<head>
<title>Error</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>An error occurred.</h1>
<p>Sorry, the page you are looking for is currently unavailable.<br/>
Please try again later.</p>
<p>If you are the system administrator of this resource then you should check
the error log for details.</p>
<p><em>Faithfully yours, nginx.</em></p>
</body>
</html>

Keepalived 高可用只能保证 VIP 不宕机，
但不能保证后端服务一定可用。

如果后端服务（Nginx/Tomcat）异常，
用户仍然会访问到错误页面。

因此必须结合：
- 健康检查（track_script）
- 应用层检测（HTTP检测）
来保证真正的高可用。
```

### **模拟故障测试**

- **方法一**：停止主节点 nginx

  ```bash
  systemctl stop nginx
  ```

  ```shell
  # 观察备节点是否在 6 秒内（2s*3fall）接管 VIP。
  [root@localhost ~]# ip a
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
  2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
      link/ether 00:0c:29:a9:06:44 brd ff:ff:ff:ff:ff:ff
      inet 192.168.157.136/24 brd 192.168.157.255 scope global ens33
         valid_lft forever preferred_lft forever
      inet 192.168.157.100/24 scope global secondary ens33:0    #备节点接管VIP
         valid_lft forever preferred_lft forever
      inet6 fe80::789b:8aea:bd9:cb0a/64 scope link
         valid_lft forever preferred_lft forever
      inet6 fe80::635a:7777:87b0:6deb/64 scope link tentative dadfailed
         valid_lft forever preferred_lft forever
  3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN
      link/ether 02:42:ea:c1:67:dd brd ff:ff:ff:ff:ff:ff
      inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
         valid_lft forever preferred_lft forever
         
  #当 nginx 故障时，Keepalived 能自动切换 VIP，实现服务不中断（高可用生效）
  ```

  **方法二**：停止主节点 keepalived

  ```bash
  systemctl stop keepalived
  #VIP 应立即漂移到备节点。
  #Keepalived进程是VIP控制核心，停止即释放VIP
  ```

- **方法三**：关闭主节点防火墙或断网

  ```bash
  iptables -I INPUT -p vrrp -j DROP
  
  单独上述是不能实现切换到备节点，必须结合
  iptables -I OUTPUT -p vrrp -j DROP
  达到预期效果，VRRP 是 单向“广播/组播”协议，BACKUP 不依赖 INPUT 来判断 MASTER 是否存活
  iptables -I INPUT -p vrrp -j DROP是阻止“进入本机的 VRRP包”，但是MASTER 仍然在发送 VRRP，BACKUP 仍然能收到（因为是 OUTGOING），而iptables -I OUTPUT -p vrrp -j DROP是MASTER 发不出 VRRP 心跳包了，结果BACKUP自然 收不到 VRRP 心跳
  或者可以用ip link set ens33 down进行测试
  
  #Keepalived切换本质：BACKUP是否收到MASTER VRRP心跳
  
  #阻断 VRRP 心跳 备节点认为主挂了
  
  #VRRP依赖网络心跳，网络异常也会触发切换
  ```

###  **查看日志**

```bash
tail -f /var/log/messages | grep Keepalived
# 或
journalctl -u keepalived -f
```

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

### 使用 Let's Encrypt 获取免费证书 (推荐)

Let's Encrypt 提供免费、自动化的 SSL 证书。

```bash
# 1. 安装 Certbot 客户端
yum install certbot python3-certbot-nginx -y

# 2. 自动获取并配置证书
# certbot 会自动修改您的 Nginx 配置以启用 HTTPS
certbot --nginx -d myapp.example.com
```

### 使用 OpenSSL 创建自签名证书 (测试/开发)

自签名证书适用于内部测试或开发环境，但在公共网络中浏览器会提示不安全。

```bash
# 1. 创建私钥 (去除口令)
openssl genrsa -out server.key 2048

# 2. 创建证书签名请求 (CSR)
openssl req -new -key server.key -out server.csr

# 3. 生成自签名证书
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.pem
```



```nginx
server {
    listen 443 ssl http2;
    server_name www.example.com;

    # 证书文件路径
    ssl_certificate      /path/to/your/server.pem; # 或 fullchain.pem
    ssl_certificate_key  /path/to/your/server.key; # 或 privkey.pem

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

## 问题集锦

### 出现大量TIME_WAIT的情况

1）导致 nginx端出现大量TIME_WAIT的情况有两种：

keepalive_requests设置比较小，高并发下超过此值后nginx会强制关闭和客户端保持的keepalive长连接；（主动关闭连接后导致nginx出现TIME_WAIT）
keepalive设置的比较小（空闲数太小），导致高并发下nginx会频繁出现连接数震荡（超过该值会关闭连接），不停的关闭、开启和后端server保持的keepalive长连接；
2）导致后端server端出现大量TIME_WAIT的情况：
nginx没有打开和后端的长连接，即：没有设置proxy_http_version 1.1;和proxy_set_header Connection “”;从而导致后端server每次关闭连接，高并发下就会出现server端出现大量TIME_WAIT

```shell
http {
    server {
        location /  {
            proxy_http_version 1.1; // 这两个最好也设置
            proxy_set_header Connection "";
        }
    }
}
```

### Nginx服务端404以及502等页面配置

#### 进入nginx.conf配置文件:

```shell
vi /usr/local/nginx/conf/nginx.conf
```

新手请记得备份一下再操作。

#### 添加页面重定向

#### ```http```内添加一行 ```fastcgi_intercept_errors on;``` 开启页面重定向功能。

```
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    # 404 500
    fastcgi_intercept_errors on;
	...
```

#### ```server``` 内添加页面指向


```
server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        # 假如服务器ip是192.168.10.125，则访问 192.168.10.125 实际访问的目录是：/usr/local/nginx/html/index.html
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://127.0.0.1:3000;
        }

        # 配置 404 状态页面指向，指向的是：root 中的 html 的绝对路径是 /usr/local/nginx/html/404.html 文件
        error_page  404              /404.html;
        location = /404.html {
            root html;
        }
		...
		error_page   500  /500.html;
        location = /500.html {
            root   html;
        }

        error_page   502   /502.html;
        location = /502.html {
            root   html;
        }
		...
```



#### 检查nginx配置文件是否准确
```
/usr/local/nginx/sbin/nginx -t
```

#### 重启nginx
```
/usr/local/nginx/sbin/nginx -s reload
```


#### 访问服务失败

访问服务器ip时，失败，通过```tail -n 5  /usr/local/nginx/logs/error.log```查看nginx错误日志发现：

```
2019/01/29 16:01:10 [error] 4351#0: *21 connect() failed (111: Connection refused) while connecting to ups
tream, client: xxx.xxx.xxx.xxx, server: localhost, request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:3
000/", host: "xxx.xxx.xxx.xxx:80"
```

原因可能是：

#### php-fpm没有安装

新买的阿里云服务器 就属于这种情况,有nginx，但是没安装php-fpm
这种情况下可参考
centos安装php php-fpm 以及 配置nginx

### Nginx的页面乱码解决方法

在server段里加字符集配置：
```
default_type 'text/html';
charset utf-8;
```

```
检查:
/usr/local/nginx/sbin/nginx -t

重启:
/usr/local/nginx/sbin/nginx -s reload
```

### 将nginx设置为系统service【掌握】

源码编译的一个缺陷是没法将安装好的应用设置为系统的service， 即无法使用 service 服务名 start | stop | restart 等命令统一操作。

以nginx为例，需要做一些配置，该配置文件的样本示例： https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/

- `vi /etc/init.d/nginx`

```bash
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

# 配置nginx命令的位置
# 修改为你的nginx可执行命令的路径如： /usr/local/nginx/sbin/nginx
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

# 指向你的配置文件路径，如：/usr/local/nginx/conf/nginx.conf
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $prog -HUP
    retval=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

- 添加可执行权限： 

  ```bash
  chmod a+x /etc/init.d/nginx
  ```

- 将一个新服务添加到启动列表中(关于chkconfig 命令更多详情，可以参考[这里](https://mp.weixin.qq.com/s?src=11&timestamp=1577072100&ver=2051&signature=y-131MCP3pZ*oQ6x7ylxbr-*Z1WND5itrTbxzh7eXzoCfrhsHc7X7dUqPjjaP91Nv4YJYxTuWmqcr9yZrRLyoSgQtIl1CXDCGlHiM9tC4S4LvdPCn-sfQq5ZOYpa6DQx&new=1) )

  ```bash
  chkconfig --add /etc/init.d/nginx
  ```

- 配置启动

  ```bash
  chkconfig nginx on
  ```

OK。 就可以使用：  `service nginx start ` 之类的命令了。

## 资料链接

- [大型网站技术架构淘宝大牛](https://www.cnblogs.com/edisonchou/category/585873.html)
- [淘宝大牛的博客干货资料](https://www.cnblogs.com/edisonchou/p/4281978.html)
- [nginx实现请求的负载均衡 + keepalived实现nginx的高可用](https://www.cnblogs.com/youzhibing/p/7327342.html)
- [nginx优化之keepalive](https://www.cnblogs.com/sunsky303/p/10648861.html)
- [Nginx+keepalived搭建---nginx反向代理为什么这么快](https://www.zhihu.com/question/19761434/answer/250280897)
- [Keepalived介绍以及-安装与配置](https://blog.csdn.net/xyang81/article/details/52554398)

- [Linux高可用之Keepalived-简书](https://www.jianshu.com/p/b050d8861fc1)

- [SegmentFault负载均衡算法实现详细介绍](https://segmentfault.com/a/1190000004492447)

- [云栖社区-负载均衡调度算法](https://cloud.tencent.com/developer/information/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95)
- [淘宝大牛分享经验](https://www.cnblogs.com/edisonchou/p/3851333.html)
- [菜鸟教程-负载均衡算法大全](https://www.runoob.com/w3cnote/balanced-algorithm.html)

- [LVS实战](https://www.cnblogs.com/liwei0526vip/p/6370103.html)

