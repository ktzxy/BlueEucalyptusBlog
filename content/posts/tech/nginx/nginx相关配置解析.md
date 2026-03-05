+++
date = '2024-08-15T10:38:44+08:00'
draft = false
title = 'nginx的相关配置解析'
description = "nginx的相关配置解析"
categories = ["📒nginx"]
tags = ["nginx"]
summary = "nginx的相关配置解析"
+++
# Nginx服务端404以及502等页面配置

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


### 访问服务失败

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

# Nginx实现虚拟主机、反向代理、负载均衡、高可用

## 一 虚拟主机

概念： 

​		虚拟主机是一种特殊的模拟硬件的软件技术，它可以将网络上的一台物理计算机映射成多个虚拟主机，每个虚拟主机可以独立对外提供www服务，这样就可以实现一台物理主机对外提供多个web服务了。并且每个虚拟主机之间是独立的，互不影响的。

nginx支持三种类型的虚拟主机配置：

- 1、基于ip的虚拟主机
- 2、基于域名的虚拟主机 
- 3、基于端口的虚拟主机

这里我们主要讲一下基于域名的虚拟主机配置，也是使用最多的。

###  基于域名的虚拟主机配置

### 需求

两个域名都指向一台机器， 使用不同的域名访问会得到不同的内容。

### 分析

- 使用一台虚拟机作为物理机。
- 使用两个域名：www.szlocal1.com、www.szlocal2.com
- 分别为两个域名创建访问资源目录：  usr/local/szlocal1/index.html、usr/local/szlocal2/index.html

### 实战

**虚拟主机配置**

在`/usr/local/nginx/conf/nginx.conf`文件，添加两个虚拟主机，即添加两个server配置项：

```nginx
	# 添加szlocal1
	server {
        listen       80;
        server_name  www.szlocal1.com;

        location / {
            root   /usr/local/szlocal1;
            index  index.html index.htm;
        }
    }
	
	# 添加szlocal2
	server {
        listen       80;
        server_name  www.szlocal2.com;

        location / {
            root   /usr/local/szlocal2;
            index  index.html index.htm;
        }
    }
```

**在宿主机中访问**

​		这样我们的基于域名的虚拟主机就配置好了，但是在浏览中会访问不到，因为DNS服务器中并没有我们刚配置的的两个域名，我们要模拟的话，可以配置我们宿主机的hosts文件，windows系统hosts所在的路径是： 

`C:\Windows\System32\drivers\etc`

在hosts文件中添加以下内容：

```
192.168.75.135 www.szlocal1.com
192.168.75.135 www.szlocal2.com
```

在浏览器中访问www.szlocal1.com、www.szlocal2.com，即可分别看到页面输出： szlocal1、szlocal2

## 二 反向代理

概念：

​		**反向代理**（Reverse Proxy）：指以代理服务器来接受internet上的连接请求，然后将请求转发给==内部网络==上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。==反向代理，客户端并不知道目标资源在哪里，主动权也不在客户端==。

​		反向代理特点： 保护和隐藏原始资源服务器。

​		**正向代理**：和反向代理不同之处在于，典型的==正向代理是一种用户知道目标地址并主动使用的代理方式==。例如Chrome浏览器中安装了switchysharp以后，通过switchysharp方便地进行代理转发服务。而为此用户必须要提前在switchysharp中做好设置才能达到相应的效果。

### 需求

在一台虚拟机中使用nginx配置反向代理，代理到tomcat服务。

### 分析

一台虚拟机：192.168.75.135

安装了nginx、tomcat

需要使用虚拟主机的配置，这里增加一个配置为 www.szlocal3.com

### 实战

在`/usr/local/nginx/conf/nginx.conf`文件，添加一个server配置项，并且增加==proxy_pass（反向代理）==的配置。

```nginx
# 添加szlocal3 虚拟主机
	server {
        listen       80;
        server_name  www.szlocal3.com;

        location / {
			proxy_pass http://localhost:8080;
        }
    }
```

在宿主机windows的hosts配置: 

```
192.168.75.135 www.szlocal3.com
```

然后浏览器访问： www.szlocal3.com 即可。

说明： 访问www.szlocal3.com时，是监听了80端口，然后反向代理到了该机器的8080端口。用户并不知道资源在8080的tomcat服务中。

## 三 负载均衡

概念：

​		负载均衡（Load Balance，LB）：意思是当一台机器支撑不住访问流量的时候，可以通过水平扩展、增加廉价的机器设备来分担访问请求。

​		举个栗子：假设你开发了一个web应用，使用的是单机的tomcat，并发请求仅仅能支撑400，做活动期间达到了900的请求，这时候，一台机器搞不定，就需要再扩展2台，可以支撑1200的请求，最开始的1台+后面的2台就可以分摊请求，达到负载均衡的目的。

​		所以说，负载均衡可以增加系统的吞吐。

Nginx就是一个负载均衡服务器：

> 数据转发功能，为nginx提供了跨越单机的横向处理能力，使nginx摆脱只能为终端节点提供单一功能的限制，而使它具备了网路应用级别的拆分、封装和整合的战略功能。在云模型大行其道的今天，数据转发是nginx有能力构建一个网络应用的关键组件。

Nginx中upstream模块就拥有数据转发功能，实现负载均衡。upstream不产生自己的内容，而是通过请求后端服务器得到内容，所以才称为upstream（上游）。upstream按照轮询（默认）方式进行负载，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

Nginx负载均衡配置步骤：

1. 在http节点添加upstream节点，如：

```nginx
# 设置当前机器为upstream配置列表中服务的上游节点（意思是一个nginx可以是多个应用服务的上游节点）
upstream myapp{
	server 192.168.75.130:8080;
    server 192.168.75.134:8080;
}
```

2. 在server节点下的location节点中添加 proxy_pass 配置项， 格式：  proxy_pass  http://upstream后面的名字

```nginx
# 添加szlocal4 虚拟主机
	server {
        listen       80;
		# 虚拟主机
        server_name  www.szlocal4.com;

        location / {
			root   html;
			index  index.html index.htm;
			# 反向代理到upstream
			proxy_pass http://myapp;
        }
    }
```



### 需求

实现一个简单的负载均衡，访问www.szlocal4.com，会在两台机器之间来回切换。

### 分析

- 需要设置基于域名的虚拟主机www.szlocal4.com
- 负载均衡使用 upstream
- 结合反向代理 proxy_pass 

### 实战

1. 在192.168.75.130、192.168.75.134 两台虚拟机中开启tomcat服务, tomcat版本号不一致，便于后面访问时观察页面变化。

2. 在Nginx负载均衡服务器中配置：

   ```nginx
   # 在http节点中配置
   upstream myapp{
   	server 192.168.75.130:8080;
       server 192.168.75.134:8080;
   }
   ```

3. 配置一个server节点，配置了虚拟主机www.szlocal4.com ;  并且反向代理结合upstream的名字myapp。

   ```nginx
   # 添加szlocal4 虚拟主机
   	server {
           listen       80;
   		# 虚拟主机
           server_name  www.szlocal4.com;
   
           location / {
   			root   html;
   			index  index.html index.htm;
   			# 反向代理到upstream，这样访问www.szlocal4.com前先访问其上游节点
   			proxy_pass http://myapp;
           }
       }
   ```

4. 设置宿主机，修改windows的hosts，添加：

   ```
   192.168.75.135 www.szlocal4.com
   ```

5. 重启192.168.75.135的nginx

6. 浏览器访问； 会看到在两个tomcat服务之间来回切换，见下图：（这里为了以示区分，在2台机器上的tomcat版本不一致）。

## 四 高可用

如果nginx服务存在异常，上面的负载均衡架构就面临问题，服务不可用。

这个时候就会采用高可用方案了，简单的高可用是一般主从备份机制。

> 我们可以使用2台机器，作为主、备服务器，各自运行nginx服务。
>
> 需要一个监控程序，来控制传输主、备之间的心跳信息（我是否还活着的信息），如果备份机在一段时间内没有收到主的发送信息，则认为主已经挂了，自己上去挑大梁，接管主服务继续提供负载均衡服务。
>
> 当备再次可以从主那里获得信息时，释放主服务，这样原来的主又可以再次提供负载均衡服务了。

这里有3个角色： 一主、一备、监控程序。其中的监控程序或者软件我们可以采用Keepalived。

## Keepalived

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

一般的高可用web架构组件：LVS+keepalived+nginx

### Keepalived高可用架构

Keepalived高可用简单架构概念视图：

当主down机后，会提升备为主，保证服务可用。

当主再次上线时，备会退位，主继续服务。



### Keepalived组成

Keepalived 模块化设计，主要有三个模块，分别是`core`、`check`和 `vrrp`。 

- core
  - **keepalived的核心，负责主进程的启动和维护、全局配置文件的加载解析** 
- check
  - **负责healthchecker(健康检查)，包括各种健康检查方式，以及对应的配置的解析包括LVS的配置解析；可基于脚本检查对IPVS后端服务器健康状况进行检查。** 
- vrrp
  - **VRRPD子进程就是来实现VRRP（虚拟路由冗余协议）协议的**

### 安装Keepalived

- 方式一：可以直接使用yum安装。

```bash
yum install -y keepalived
```



- 方拾二： 使用源码安装方式。

```bash
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

# 测试
使用配置好的虚拟ip，在浏览器访问测试一下即可。


```

也可以这样设置，**配置文件**：安装完成之后，可以把配置文件放到目录：  `/etc/keepalived` 

后面则统一修改 `/etc/keepalived/keepalived.conf` 配置文件即可。

/usr/local/keepalived/etc/keepalived/keepalived.conf

/etc/keepalived/keepalived.conf

**启动|停止|重启服务**

```bash
service keepalived start|stop|restart
```

**检查服务**

```bash
chkconfig keepalived on
```

**查看keepalived版本**

```shell
keepalived -v
Keepalived v1.2.23 (12/20,2019)
```



### 需求

使用keepalived结合nginx，搭建一套简单的高可用集群方案。

并且测试主宕机的情况；

演示主上线的情况。

### 分析

- 选择4台虚拟机：2台做nginx的主、备；2台做tomcat集群。
  - 为了使nginx命令方便的话，可以将其所在目录添加到：`/etc/profile` 文件中。
- 选择192.168.75.132、192.168.75.135安装keepalived和nginx并分别做主、备。
- 选择192.168.75.130、192.168.75.134安装tomcat做应用集群。

### 实战

1. 在192.168.75.132、192.168.75.135分别安装keepalived并配置

   ```bash
   #下载
   mkdir /usr/local/keepalived
   cd /usr/local/keepalived
   wget http://www.keepalived.org/software/keepalived-1.2.23.tar.gz
   # 或者 wget http://www.keepalived.org/software/keepalived-1.2.23.tar.gz --no-check-certificate  跳过证书验证
   #解压
   tar -zxvf keepalived-1.2.23.tar.gz
   # cd 到解压后的目录
   cd keepalived-1.2.23
   # 源码安装三板斧搞起来
   ./configure --prefix=/usr/local/keepalived   #prefix指定安装目录
   make
   make install
   ```


​	在192.168.75.132(作为主)修改配置文件 keepalived.conf：

```bash
! Configuration File for keepalived

# 全局配置
global_defs {
   # 发生切换时，需要通知的邮件； 一行一个用户的email地址；这里假定了几个人..
   notification_email {
     zs@163.com
     ls@163.com
     ww@163.com
   }
   
   notification_email_from admin@163.com    				# 发件人，谁发的；这里假定了一个管理员邮箱admin@163.com
   smtp_server smtp.163.com                                 # smtp服务器地址，这里示例使用163的公开地址
   smtp_connect_timeout 30									# smtp连接超时时间设置
   router_id LVS_DEVEL										# 运行keepalived的一个路由标志id
}

# VRRP 配置
vrrp_instance VI_1 {
    state MASTER											# 配置标志，MASTER 为主；BACKUP 为备
    interface eth0						# 该keepalived实例绑定的网卡; RHEL7以前可以设置为eth0,7以及之后可以设置为ens33
    virtual_router_id 51				#VRRP组名，两个节点的设置必须一样，以指明各个节点属于同一VRRP组
    priority 100											# 主节点的优先级（1-254之间），备用节点必须比主节点优先级低
    advert_int 1											# 主、备之间检查是否一致的时间间隔：单位秒
    
	# 认证配置   设置验证信息，主、备节点必须一致
	authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {					# 指定虚拟IP, 主、备节点设置必须一样
		# 可以设置多个虚拟ip，换行即可；随便写；这个地址是虚拟的，并不需要实体机器
		# 会将该vip绑定到当前机器的网卡eth0上（因为vrrp_instance的interface指定了eth0）
        192.168.75.88					
    }
}
```

在192.168.75.135（作为备）修改配置文件 keepalived.conf：

- vrrp_instance 下面的state改为了 **BACKUP**。
- vrrp_instance  下面的额priority改为了**90** ，比主的100低即可。

- 其他的配置项一样。

2. 启动keepalived服务

```bash
service keepalived start
```

- 可以使用 `ip  addr` 命令查看vip是否已经绑定


3. **测试正常情况**

   在Master和Backup上分别启动keepalived。此时VIP(192.168.75.88)是可访问的，也可以ping通，通过虚拟ip（192.168.75.88）访问，可以看到一直访问的是主（192.168.75.132）：

4. **测试Master宕掉的情况**

**现在把主（192.168.75.132）的keepalived关掉**，在浏览器上再次访问192.168.75.88，会发现此时访问的是备（192.168.75.135）了：

5. 测试Master宕掉又恢复的情况

   重新把192.168.75.132的keepalived启动后，会看到浏览器输入192.168.75.88访问的一直是192.168.75.132（主）。

**综上所述，当前的架构方案，在keepalived出现问题时，是可以及时更换主备的。**

## 原始高可用方案存在的问题

我们把主的nginx服务关闭，再访问192.168.75.88，发现服务访问失败，因为此时一直访问的是主192.168.75.132的nginx。但是http://192.168.75.132:8080/服务其实是可用的。

这不是我们想要的结果，因为此时备（192.168.75.135）的nginx服务是OK的。能不能在主的nginx挂掉后，自动地切换到备的nginx服务呢？

毋庸置疑，肯定是可以的，我们还需要更多的配置。

**截至目前，该方案不能保证主nginx挂掉后服务仍可用。所以，还需要继续提高高可用性。**



我们需要在`keepalived.conf`中增加对nginx服务的检测，根据nginx服务的状态做出一些处理。

### vrrp_script节点

​		此模块专门用于对集群中服务资源进行监控 。与此模块同时使用的还有 track_script 模块，在此模块中可以引入监控脚本、命令组合、shell 语句等 ，以实现对服务、端口等多方面的监控。

​		track_script 模块主要用来调用 vrrp_script 模块使 keepalived执行对集群服务资源的检测。

​		vrrp_script 模块中还可以定义对服务资源检测的时间间隔、权重等参数，通过 vrrp_script 和 track_script 组合，可以实现对集群资源的监控并改变优先级，进而实现 keepalived 主备节点切换。



我们可以在`keepalived.conf`文件中新增一个`vrrp_script`节点，来指定监控一个shell脚本文件: `/etc/keepalived/check_and_start_nginx.sh`。

```shell
# 新增一个vrrp_script节点，用来监控nginx
vrrp_script chk_nginx {
    script "/etc/keepalived/check_and_start_nginx.sh"   	# 检测nginx服务并尝试重启
    interval 2                    							# 每2s检查一次
    weight -5                     							# 检测失败（脚本返回非0）则优先级减少5个值
    fall 3                        							# 如果连续失败次数达到此值，则认为服务器已down
    rise 2                        							# 如果连续成功次数达到此值，则认为服务器已up，但不修改优先级
}
```



还需要在虚拟节点中增加`track_script`子节点引用`vrrp_script`节点：

```shell
# VRRP 配置
vrrp_instance VI_1 {
    state MASTER											# 配置标志，MASTER 为主；BACKUP 
    
    # ...... 省略
	
	# 引用VRRP脚本，即在 vrrp_script 中定义的
    track_script {                # 引用VRRP脚本
        chk_nginx          
    }     
}
```





### shell脚本编写

`vi /etc/keepalived/check_and_start_nginx.sh` 编写以下内容：

```shell
#!/bin/bash

# 查看nginx进程是否正在运行，为0则表示已经down掉
nginx_result=$(ps -C nginx --no-heading|wc -l)
if [ "${nginx_result}" = "0" ]; then
	# 使用service的前提需要将nginx设置为servic； 或者直接使用nginx所在绝对路径比如： /usr/local/nginx/sbin/nginx
    service nginx start   # 或者使用   /usr/local/nginx/sbin/nginx
    sleep 2
    nginx_result=$(ps -C nginx --no-heading|wc -l)
    if [ "${nginx_result}" = "0" ]; then
		# 如果重启nginx服务还是不行的话，就把keepalived也停止；
		# 这样会访问备keepalived，从而保证主的nginx挂掉备也能使用
        /etc/rc.d/init.d/keepalived stop
    fi
fi
```

**注意！！！！**：  需要加执行权限：   

```shell
chmod a+x /etc/keepalived/check_and_start_nginx.sh 
```

### 主的keepalived.conf

以下是完整的主的keepalived.conf（/etc/keepalived/keepalived.conf）文件配置：

```shell
! Configuration File for keepalived

# 全局配置
global_defs {
   # 发生切换时，需要通知的邮件； 一行一个用户的email地址；这里假定了几个人..
   notification_email {
     zs@163.com
     ls@163.com
     ww@163.com
   }
   
   notification_email_from admin@163.com    				# 发件人，谁发的；这里假定了一个管理员邮箱admin@163.com
   smtp_server smtp.163.com                                 # smtp服务器地址，这里示例使用163的公开地址
   smtp_connect_timeout 30									# smtp连接超时时间设置
   router_id LVS_DEVEL										# 运行keepalived的一个路由标志id
}

# 新增一个vrrp_script节点，用来监控nginx
vrrp_script chk_nginx {
    script "/etc/keepalived/check_and_start_nginx.sh"   	# 检测nginx服务并尝试重启
    interval 2                    							# 每2s检查一次
    weight -5                     							# 检测失败（脚本返回非0）则优先级减少5个值
    fall 3                        							# 如果连续失败次数达到此值，则认为服务器已down
    rise 2                        							# 如果连续成功次数达到此值，则认为服务器已up，但不修改优先级
}

# VRRP 配置
vrrp_instance VI_1 {
    state MASTER											# 配置标志，MASTER 为主；BACKUP 为备
    interface eth0						# 该keepalived实例绑定的网卡; RHEL7以前可以设置为eth0,7以及之后可以设置为ens33
    virtual_router_id 51				#VRRP组名，两个节点的设置必须一样，以指明各个节点属于同一VRRP组
    priority 100											# 主节点的优先级（1-254之间），备用节点必须比主节点优先级低
    advert_int 1											# 主、备之间检查是否一致的时间间隔：单位秒
    
	# 认证配置   设置验证信息，主、备节点必须一致
	authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {					# 指定虚拟IP, 主、备节点设置必须一样
		# 可以设置多个虚拟ip，换行即可；随便写；这个地址是虚拟的，并不需要实体机器
		# 会将该vip绑定到当前机器的网卡eth0上
        192.168.75.88					
    }
	
	# 引用VRRP脚本，即在 vrrp_script 中定义的
    track_script {                # 引用VRRP脚本
        chk_nginx          
    }     
}

```



- 启动keepalived：  `service keepalived start`



现在主（192.168.75.132）和备（192.168.75.135）的keepalived、nginx都已经启动；

此时 浏览器访问vip ： http://192.168.75.88/， 回请求到主192.168.75.132：

现在将主的keepalived关闭，再访问http://192.168.75.88/，会请求到备192.168.75.135。

再将主的keepalived启动，又会访问到主。

**注意**：  原来我们将主的nginx关闭，访问 http://192.168.75.88/ 时会一直调用主，导致服务不可用。现在我们在主的keepalived中配置了监控nginx服务，看他是否会尝试重启主的nginx服务并且保证给外界提供访问。

我们在主上关闭nginx服务。稍等一会，访问 http://192.168.75.88/  ，发现可以访问到主的页面，这样keepalived监控了nginx的状态并将nginx服务启动了，保证了高可用，现在不需要人工干预去启动服务了。



# 多学一招

## 将nginx设置为系统service【掌握】

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

# 负载均衡实现实践

## 负载均衡技术

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

## 负载均衡算法

如何从web服务器列表中计算得到一台目标web服务器地址，就是负载均衡算法。


### 准备数据

表示一个web服务器ip、权重的字典映射。

```java
private Map<String,Integer> serverMap = new HashMap<String,Integer>(){{
    put("192.168.1.100",1);
    put("192.168.1.101",1);
    put("192.168.1.102",4);
    put("192.168.1.103",1);
    put("192.168.1.104",1);
    put("192.168.1.105",3);
    put("192.168.1.106",1);
    put("192.168.1.107",2);
    put("192.168.1.108",1);
    put("192.168.1.109",1);
    put("192.168.1.110",1);
}};
```

### 轮询

这是最简单的调度算法，调度器将收到的请求循环分配到服务器集群中的每台机器，这种算法平等地对待每一台服务器，而不管服务器上实际的负载状况和连接状态，适合所有服务器有相同或者相近性能的情况.


轮询记录一个字段，记录下一次的字典位置。

```java
private Integer pos = 0;
public void roundRobin(){
    List<String> keyList = new ArrayList<String>(serverMap.keySet());
    String server = null;
    synchronized (pos){
        if(pos > keyList.size()){
            pos = 0;
        }
        server = keyList.get(pos);
        pos++;
    }
    System.out.println(server);
}
```


### 加权轮询

加权轮询，旨在轮询的基础上，增加权重的影响。
用一个列表获取web服务器的字典映射，权重大的在列表中出现的次数就多。

```java
public void weightRoundRobin(){
    // 所有的服务器字典集合
    Set<String> keySet = serverMap.keySet();
    // 权重影响的可选列表(权重大的出现的次数较多)
    List<String> servers = new ArrayList<String>();
    for(Iterator<String> it = keySet.iterator();it.hasNext();){
        String server = it.next();
        int weithgt = serverMap.get(server);
        for(int i=0;i<weithgt;i++){
           servers.add(server);
        }
    }
    String server = null;
    synchronized (pos){
        // 如果位置大于服务器个数，则选择第一个服务器
        if(pos > keySet.size()){
            pos = 0;
        }
        
        // 其他则每一次选择，都自增位置，从【权重影响的可选列表】中获取索引位置上的服务器
        server = servers.get(pos);
        pos++;
    }
    System.out.println(server);
}
```

### 随机

维护一个服务列表，随机调取

```java
public void random(){
    List<String> keyList = new ArrayList<String>(serverMap.keySet());
    Random random = new Random();
    int idx = random.nextInt(keyList.size());
    String server = keyList.get(idx);
    System.out.println(server);
}
```

### 权重随机

维护一个列表, 但是会读取web服务器的权重，权重大的出现次数较多，再随机调取。

```java
public void weightRandom(){
    Set<String> keySet = serverMap.keySet();
    List<String> servers = new ArrayList<String>();
    for(Iterator<String> it = keySet.iterator();it.hasNext();){
        String server = it.next();
        int weithgt = serverMap.get(server);
        for(int i=0;i<weithgt;i++){
            servers.add(server);
        }
    }
    String server = null;
    Random random = new Random();
    int idx = random.nextInt(servers.size());
    server = servers.get(idx);
    System.out.println(server);
}
```

### 最少连接

选择与当前连接数最少的服务器通信。

#### 加权最少链接

为最少连接算法中的每台服务器附加权重的算法，该算法事先为每台服务器分配处理连接的数量，并将客户端请求转至连接数最少的服务器上。

### 散列（哈希）

#### 普通哈希

可以将请求源地址的ip；将其哈希值与web服务器字典集合映射做取模运算，获取合适的web服务器发送请求。

```java
public void hash(){
    List<String> keyList = new ArrayList<String>(serverMap.keySet());
    // 可能是源地址-请求地址的ip；将其哈希值与web服务器字典集合映射做取模运算，获取合适的web服务器发送请求。
    String remoteIp = "192.168.2.215";
    int hashCode = remoteIp.hashCode();
    int idx = hashCode % keyList.size();
    String server = keyList.get(Math.abs(idx));
    System.out.println(server);
}
```

#### 一致性哈希

- 一致性Hash，相同参数的请求总是发到同一提供者。当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
  实现可以参考dubbo的一致性哈希算法： [```org.apache.dubbo.rpc.cluster.loadbalance.ConsistentHashLoadBalance```](https://github.com/apache/dubbo/blob/master/dubbo-cluster/src/main/java/org/apache/dubbo/rpc/cluster/loadbalance/ConsistentHashLoadBalance.java)

## 参考干货资料

- [SegmentFault负载均衡算法实现详细介绍](https://segmentfault.com/a/1190000004492447)
- [云栖社区-负载均衡调度算法](https://cloud.tencent.com/developer/information/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E8%B0%83%E5%BA%A6%E7%AE%97%E6%B3%95)
- [淘宝大牛分享经验](https://www.cnblogs.com/edisonchou/p/3851333.html)
- [菜鸟教程-负载均衡算法大全](https://www.runoob.com/w3cnote/balanced-algorithm.html)

- [LVS实战](https://www.cnblogs.com/liwei0526vip/p/6370103.html)



# Nginx配置文件nginx.conf详细介绍

### Nginx配置文件nginx.conf全解

nginx配置文件nginx.conf的配置http、upstream、server、location等；

nginx负载均衡算法：轮询、加权轮询、ip_hash、url_hash等策略配置；

nginx日志文件access_log配置；

代理服务缓存proxy_buffer设置。

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



# 资料链接

- [大型网站技术架构淘宝大牛](https://www.cnblogs.com/edisonchou/category/585873.html)
- [淘宝大牛的博客干货资料](https://www.cnblogs.com/edisonchou/p/4281978.html)
- [nginx实现请求的负载均衡 + keepalived实现nginx的高可用](https://www.cnblogs.com/youzhibing/p/7327342.html)
- [nginx优化之keepalive](https://www.cnblogs.com/sunsky303/p/10648861.html)
- [Nginx+keepalived搭建---nginx反向代理为什么这么快](https://www.zhihu.com/question/19761434/answer/250280897)
- [Keepalived介绍以及-安装与配置](https://blog.csdn.net/xyang81/article/details/52554398)

- [Linux高可用之Keepalived-简书](https://www.jianshu.com/p/b050d8861fc1)



