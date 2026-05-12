+++
date = '2024-03-20T22:22:09+08:00'
draft = false
title = 'tomcat学习'
slug = "bhytpcx85p"
description = "tomcat学习"      
summary = "tomcat 脚本；集群"                       
categories = ["📒tomcat"]                                      
tags = ["tomcat"]                                                  

+++

# 启动脚本

```shell
#!/bin/bash

set -e
set -o pipefail

TOMCAT_HOME="/data/tomcat/tomcat-8484"
PID_FILE="$TOMCAT_HOME/tomcat.pid"
LOG_FILE="$TOMCAT_HOME/logs/control.log"

START="$TOMCAT_HOME/bin/startup.sh"
STOP="$TOMCAT_HOME/bin/shutdown.sh"

log() {
    echo "$(date '+%F %T') [INFO] $1" | tee -a $LOG_FILE
}

get_pid() {
    pgrep -f "org.apache.catalina.startup.Bootstrap" | grep "$TOMCAT_HOME" || true
}

start() {
    if [ -n "$(get_pid)" ]; then
        log "Tomcat 已运行"
        exit 1
    fi

    log "启动 Tomcat..."
    $START

    sleep 3

    pid=$(get_pid)
    if [ -n "$pid" ]; then
        echo $pid > $PID_FILE
        log "启动成功 PID=$pid"
    else
        log "启动失败"
        exit 1
    fi
}

stop() {
    pid=$(get_pid)

    if [ -z "$pid" ]; then
        log "Tomcat 未运行"
        return
    fi

    log "停止 Tomcat PID=$pid"
    $STOP

    # 等待优雅退出
    for i in {1..10}; do
        sleep 1
        if ! ps -p $pid &>/dev/null; then
            log "停止成功"
            rm -f $PID_FILE
            return
        fi
    done

    log "优雅停止失败，强制杀进程"
    kill -15 $pid
    sleep 2

    if ps -p $pid &>/dev/null; then
        kill -9 $pid
    fi

    rm -f $PID_FILE
}

logs() {
    tail -f $TOMCAT_HOME/logs/catalina.out
}

case "$1" in
    start) start ;;
    stop) stop ;;
    restart)
        stop
        sleep 2
        start
        ;;
    logs) logs ;;
    *)
        echo "Usage: $0 {start|stop|restart|logs}"
        exit 1
        ;;
esac
```

```shell
sed -i "s/ //" tomcat-8484.sh #设置脚本文件为unix格式
chmod 777 ./tomcat-8484.sh
```

**有这样一个场景，公司为了安全起见，需要对所有登录Linux服务器做安全限制，要求除了管理员其他要登录linux服务器的员工不能用最高权限账号登录，要创建新的用户，对目录及文件权限做出控制，只能对需要操作的目录允许读，写，执行权限，其他目录只有读的权限，并且所有tomcat不能直接在bin中用startup.sh,shutdown.sh进行启动和停止，要通过写shell脚本进行此操作，也就是说有两个步骤，创建用户并设置权限，写tomcat启动脚本**

```shell
groupadd tomcat
useradd -g tomcat -s /bin/bash tomcat
passwd -l tomcat   # 禁止密码登录
passwd tomcat # 设置密码
chown -R tomcat:tomcat /data #分配权限给用户

#限制访问目录
setfacl -m u:tomcat:rwx /data/tomcat
setfacl -R -m u:tomcat:rx /
```

#  多实例部署和安装

准备安装包：

```shell
# 放到 opt目录下
jdk-8u371-linux-x64.rpm
apache-tomcat-9.0.78.tar.gz

# 执行脚本
chmod +x deploy_tomcat.sh
./deploy_tomcat.sh
```

**脚本**

```shell
#!/bin/bash

set -e
set -o pipefail

# ========================
# 基础变量（可改）
# ========================
TOMCAT_VERSION="9.0.78"
TOMCAT_DIR="/usr/local/tomcat"
TOMCAT_USER="tomcat"
JDK_RPM="jdk-8u371-linux-x64.rpm"
TOMCAT_TAR="apache-tomcat-9.0.78.tar.gz"

DOMAIN1="test1.com"
DOMAIN2="test2.com"

APP1="test1"
APP2="test2"

# ========================
# 日志函数
# ========================
log() {
    echo -e "\033[32m[INFO]\033[0m $1"
}

error_exit() {
    echo -e "\033[31m[ERROR]\033[0m $1"
    exit 1
}

# ========================
# 检查 root
# ========================
if [ "$EUID" -ne 0 ]; then
    error_exit "请使用 root 用户运行"
fi

cd /opt || error_exit "/opt 不存在"

# ========================
# 安装 JDK
# ========================
if ! command -v java &>/dev/null; then
    log "安装 JDK..."
    rpm -ivh $JDK_RPM || error_exit "JDK 安装失败"

    cat >/etc/profile.d/java.sh <<EOF
export JAVA_HOME=/usr/java/latest
export PATH=\$JAVA_HOME/bin:\$PATH
EOF

    source /etc/profile.d/java.sh
else
    log "JDK 已存在，跳过"
fi

# ========================
# 创建 tomcat 用户
# ========================
if ! id $TOMCAT_USER &>/dev/null; then
    log "创建 tomcat 用户"
    useradd -r -s /bin/false $TOMCAT_USER
fi

# ========================
# 安装 Tomcat
# ========================
if [ ! -d "$TOMCAT_DIR" ]; then
    log "安装 Tomcat..."
    tar zxf $TOMCAT_TAR
    mv apache-tomcat-$TOMCAT_VERSION $TOMCAT_DIR
    chown -R $TOMCAT_USER:$TOMCAT_USER $TOMCAT_DIR
else
    log "Tomcat 已存在，跳过"
fi

# ========================
# 创建应用目录
# ========================
mkdir -p $TOMCAT_DIR/webapps/$APP1
mkdir -p $TOMCAT_DIR/webapps/$APP2

echo "This is $APP1 page!" >$TOMCAT_DIR/webapps/$APP1/index.jsp
echo "This is $APP2 page!" >$TOMCAT_DIR/webapps/$APP2/index.jsp

# ========================
# 虚拟主机配置（官方推荐）
# ========================
mkdir -p $TOMCAT_DIR/conf/Catalina

cat >$TOMCAT_DIR/conf/Catalina/$DOMAIN1.xml <<EOF
<Context docBase="$TOMCAT_DIR/webapps/$APP1" reloadable="true"/>
EOF

cat >$TOMCAT_DIR/conf/Catalina/$DOMAIN2.xml <<EOF
<Context docBase="$TOMCAT_DIR/webapps/$APP2" reloadable="true"/>
EOF

chown -R $TOMCAT_USER:$TOMCAT_USER $TOMCAT_DIR

# ========================
# systemd 服务
# ========================
if [ ! -f /etc/systemd/system/tomcat.service ]; then
    log "创建 systemd 服务"

    cat >/etc/systemd/system/tomcat.service <<EOF
[Unit]
Description=Apache Tomcat
After=network.target

[Service]
Type=forking

User=$TOMCAT_USER
Group=$TOMCAT_USER

Environment=JAVA_HOME=/usr/java/latest
Environment=CATALINA_HOME=$TOMCAT_DIR
Environment=CATALINA_BASE=$TOMCAT_DIR

ExecStart=$TOMCAT_DIR/bin/startup.sh
ExecStop=$TOMCAT_DIR/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

    systemctl daemon-reload
    systemctl enable tomcat
fi

# ========================
# 获取IP并写hosts（幂等）
# ========================
local_ip=$(hostname -I | awk '{print $1}')

grep -q "$DOMAIN1" /etc/hosts || echo "$local_ip $DOMAIN1" >>/etc/hosts
grep -q "$DOMAIN2" /etc/hosts || echo "$local_ip $DOMAIN2" >>/etc/hosts

# ========================
# 启动 Tomcat
# ========================
log "启动 Tomcat..."
systemctl restart tomcat

sleep 3

# ========================
# 验证
# ========================
log "验证访问..."

curl -s http://$DOMAIN1:8080 | grep "$APP1" && log "$DOMAIN1 正常" || log "$DOMAIN1 访问异常"
curl -s http://$DOMAIN2:8080 | grep "$APP2" && log "$DOMAIN2 正常" || log "$DOMAIN2 访问异常"

log "部署完成！"
```





```shell
#!/bin/bash
# 切换到/opt目录
cd /opt
# 安装Java Development Kit (JDK) 8
rpm -ivh jdk-8u371-linux-x64.rpm

# 向环境变量配置文件中添加Java环境变量
echo "export JAVA_HOME=/usr/java/jdk1.8.0-x64
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
export PATH=$JAVA_HOME/bin:$PATH" >>/etc/profile.d/java.sh
# 使环境变量配置立即生效
source /etc/profile.d/java.sh

# 切换到/opt目录，解压Apache Tomcat 9
cd /opt
tar zxvf apache-tomcat-9.0.78.tar.gz
# 将解压的Tomcat移动到/usr/local/tomcat下
mv -f apache-tomcat-9.0.78 /usr/local/tomcat
# 启动Tomcat
/usr/local/tomcat/bin/startup.sh

# 检查Tomcat启动是否成功
if [ $? -eq 0 ]; then
    echo "tomcat启动成功"
else
    echo "tomcat启动失败"
fi

# 在Tomcat的webapps目录下创建kgc和benet应用目录
mkdir /usr/local/tomcat/webapps/kgc
mkdir /usr/local/tomcat/webapps/benet
# 向kgc和benet应用目录下的index.jsp文件写入内容
echo "This is kgc page\!" >/usr/local/tomcat/webapps/kgc/index.jsp
echo "This is benet page\!" >/usr/local/tomcat/webapps/benet/index.jsp

# 更新Tomcat的server.xml配置文件，以添加对kgc和benet域名的配置
# 创建临时文件来存储更新后的配置
temp_file=$(mktemp)
# 使用sed命令将配置插入到临时文件中
sed "160a\\t<Host name="www.kgc.com" appBase="webapps" unpackWARs="true" autoDeploy="true" xmlValidation="false" xmlNamespaceAware="false">\n\t<Context docBase="/usr/local/tomcat/webapps/kgc" path="" reloadable="true"\n\t/>\n\t</Host>\n\t<Host name="www.benet.com" appBase="webapps" unpackWARs="true" autoDeploy="true" xmlValidation="false" xmlNamespaceAware="false">\n<Context docBase="/usr/local/tomcat/webapps/benet" path="" reloadable="true"\n/>\n</Host>" /usr/local/tomcat/conf/server.xml >"$temp_file" &&
    # 检查是否成功写入临时文件
    if [ $? -eq 0 ]; then
        # 在替换原始配置文件之前创建一个备份
        cp /usr/local/tomcat/conf/server.xml /usr/local/tomcat/conf/server.xml.bak
        # 替换原始配置文件
        mv -f "$temp_file" /usr/local/tomcat/conf/server.xml &&
            echo "Configuration updated successfully."
    else
        # 如果写入临时文件失败，记录错误消息并清理临时文件
        echo "Failed to update configuration."
        rm "$temp_file"
    fi
# 重写kgc和benet应用目录下的index.jsp文件内容，确保内容是最新的
echo "This is kgc page\!" >/usr/local/tomcat/webapps/kgc/index.jsp
echo "This is benet page\!" >/usr/local/tomcat/webapps/benet/index.jsp

# 获取本地eth0网卡的IP地址，并将其与kgc和benet域名一起添加到hosts文件中
local_ip=$(ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d '/' -f 1)
echo "$local_ip www.kgc.com www.benet.com" >> /etc/hosts
# 停止并重新启动Tomcat，以确保配置和内容更新生效
/usr/local/tomcat/bin/shutdown.sh 
/usr/local/tomcat/bin/startup.sh
# 验证kgc和benet应用是否可以通过域名访问
curl www.kgc.com:8080/kgc/index.jsp
curl www.benet.com:8080/benet/index.jsp
```

# 文件说明

```shell
●bin:存放启动和关闭Tomcat的脚本文件，比较常用的是 catalina.sh、startup.sh、shutdown.sh三个文件
●conf：存放Tomcat 服务器的各种配置文件，比较常用的是 server.xml、context.xml、tomcat-users.xml、web.xml 四个文件。
●server.xml: Tomcat的主配置文件，包含Service，Connector，Engine，Realm，Valve,Hosts主组件的相关配置信息;
●context.xml:所有host的默认配置信息;
●tomcat-user.xml:Realm认证时用到的相关角色、用户和密码等信息，Tomcat自带的manager默认情况下会用到此文件，在Tomcat中添加/删除用户，为用户指|定角色等将通过编辑此文件实现;
●web.xml:遵循Servlet规范标准的配置文件，用于配置servlet，并为所有的web应用程序提供包括MIME映射等默认配置信息;
●lib：存放Tomcat运行需要的库文件的jar 包，一般不作任何改动，除非连接第三方服务，比如 redis，那就需要添加相对应的jar 包
●logs：存放 Tomcat 执行时的日志
●temp：存放 Tomcat 运行时产生的文件
●webapps：存放 Tomcat 默认的 Web 应用部署目录
●work：Tomcat工作日录，存放jsp编译后产生的class文件，一般清除Tomcat缓存的时候会使用到
●src:存放Tomcat 的源代码
●doc:存放Tomcat文档
```

```shell
CLASSPATH：编译、运行Java程序时，JRE会去该变量指定的路径中搜索所需的类（.class）文件。
dt.jar：是关于运行环境的类库，主要是可视化的 swing 的包。
tools.jar：主要是一些jdk工具的类库，包括javac、java、javap（jdk自带的一个反编译工具）、javadoc等。
JDK ：java development kit （java开发工具）
JRE ：java runtime environment
（java运行时环境）
JVM ：java virtual machine （java虚拟机），使java程序可以在多种平台上运行class文件。
```
```shell
#### 扩展和优化 Tomcat 的 catalina.sh 文件以调整 JVM 参数

CATALINA_OPTS="-server \
                -Xms2048m -Xmx2048m \
                -Xmn512m \
                -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m \
                -XX:NewRatio=3 \
                -XX:SurvivorRatio=8 \
                -XX:+UseG1GC \
                -XX:MaxGCPauseMillis=200 \
                -XX:InitiatingHeapOccupancyPercent=70 \
                -XX:+PrintGCDetails \
                -XX:+PrintGCDateStamps \
                -XX:+PrintGCCause \
                -Xloggc:/var/log/tomcat/tomcat_gc.log \
                -Djava.awt.headless=true \
                -Dcom.sun.management.jmxremote.port=10086 \
                -Dcom.sun.management.jmxremote.ssl=false \
                -Dcom.sun.management.jmxremote.authenticate=false"


-server: 启用服务器模式的 JVM。
-Xms 和 -Xmx: 分别设置堆的初始大小和最大大小，保持一致可以减少堆大小调整带来的开销。
-Xmn: 设置年轻代（Young Generation）大小。
-XX:MetaspaceSize 和 -XX:MaxMetaspaceSize: 设置元空间（MetaSpace，取代了 PermGen）的初始和最大大小。
-XX:NewRatio: 设置老年代与年轻代的大小比例。
-XX:SurvivorRatio: 设置 Eden 区与 Survivor 区的大小比例。
-XX:+UseG1GC: 使用 G1 垃圾收集器，根据实际需求可选用其他适合的 GC 策略。
-XX:MaxGCPauseMillis: 设置期望的最大 GC 停顿时间。
-XX:InitiatingHeapOccupancyPercent: 设置触发并发标记周期的堆占用百分比。
-XX:+PrintGCDetails 等：开启详细的 GC 日志记录，便于分析和调优。
# 禁用显式gc
-XX:+DisableExplicitGC # 自动将System.gc() 调用转换成一个空操作，即应用中调用
System.gc()会变成一个空操作，避免程序员在代码里进行System.gc()这种危险操作。System.gc()除
非是到了万不得已的情况下使用，都应该交给JVM。
-Xloggc: 设置 GC 日志文件路径。
-Djava.awt.headless=true: 防止在无图形界面环境中出现相关异常。
-Dcom.sun.management.jmxremote.*: 开启 JMX 远程监控，便于通过 JConsole 或其他工具监控 JVM 状态。
```

# 集群学习

## 单机多实例 Tomcat

> 🎯 目标：在一台机器上跑 **2个独立 Tomcat 实例**

```shell
# 安装 JDK 如果是 CentOS / Rocky / AlmaLinux：
yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel

# 验证安装
java -version

# 获取 JAVA_HOME
readlink -f $(which java)

# 配置环境变量
vi /etc/profile

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.412.b08-1.el7_9.x86_64
export PATH=$JAVA_HOME/bin:$PATH

#执行
source /etc/profile

# 验证
echo $JAVA_HOME
ls $JAVA_HOME/bin/java
ls $JAVA_HOME/bin/javac
```

准备tomcat

```shell
mkdir -p /data/tomcat
cd /data/tomcat

tar -zxf apache-tomcat-9.0.78.tar.gz

cp -r apache-tomcat-9.0.78 tomcat-8081
cp -r apache-tomcat-9.0.78 tomcat-8082
```

```shell
vi /data/tomcat/tomcat-8081/conf/server.xml
```

```shell
<!-- 关闭端口 -->
<Server port="8006" shutdown="SHUTDOWN">

<!-- HTTP端口 -->
<Connector port="8081" protocol="org.apache.coyote.http11.Http11NioProtocol"/>

<!-- AJP端口 -->
<Connector port="8010" protocol="AJP/1.3" redirectPort="8443"/>
```

```shell
vi /data/tomcat/tomcat-8082/conf/server.xml
```

```shell
<!-- 关闭端口 -->
<Server port="8007" shutdown="SHUTDOWN">

<!-- HTTP端口 -->
<Connector port="8082" protocol="org.apache.coyote.http11.Http11NioProtocol"/>

<!-- AJP端口 -->
<Connector port="8011" protocol="AJP/1.3" redirectPort="8443"/>
```

```shell
echo "THIS IS TOMCAT 8081" > /data/tomcat/tomcat-8081/webapps/ROOT/index.jsp
echo "THIS IS TOMCAT 8082" > /data/tomcat/tomcat-8082/webapps/ROOT/index.jsp
```

```shell
/data/tomcat/tomcat-8081/bin/startup.sh
/data/tomcat/tomcat-8082/bin/startup.sh
```

```shell
http://你的IP:8081
http://你的IP:8082
```

## 用 Nginx 做负载均衡

> 实现：用户 → Nginx → Tomcat8081 / Tomcat8082（随机）

```shell
# 安装nginx
yum install -y nginx
# 启动
systemctl start nginx
systemctl enable nginx

# 配置
vi /etc/nginx/nginx.conf
# 找到 http {}，加入
upstream tomcat_cluster {
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
}
# 修改 server 配置
server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_pass http://tomcat_cluster;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# 重启 Nginx
nginx -t   # 检查配置
systemctl restart nginx
```

**报错**

```shell
[root@localhost ~]# yum install -y nginx Loaded plugins: fastestmirror Loading mirror speeds from cached hostfile * base: mirrors.aliyun.com * extras: mirrors.aliyun.com * updates: mirrors.aliyun.com No package nginx available. Error: Nothing to do

CentOS 7 默认源没有 nginx
#创建 repo 文件
vi /etc/yum.repos.d/nginx.repo
# 写入
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
# 清缓存
yum clean all
yum makecache
```

```shell
[root@localhost ~]# curl 127.0.0.1:8081
THIS IS TOMCAT 8081
[root@localhost ~]# curl 127.0.0.1:8082
THIS IS TOMCAT 8082
[root@localhost ~]# curl 127.0.0.1
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

[root@localhost ~]# tail -f /var/log/nginx/error.log
2026/04/13 16:10:05 [warn] 34388#34388: *1 upstream server temporarily disabled while connecting to upstream, client: 192.168.157.1, server: localhost, request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8082/", host: "192.168.157.129"
2026/04/13 16:10:05 [crit] 34388#34388: *1 connect() to 127.0.0.1:8081 failed (13: Permission denied) while connecting to upstream, client: 192.168.157.1, server: localhost, request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8081/", host: "192.168.157.129"
2026/04/13 16:10:05 [warn] 34388#34388: *1 upstream server temporarily disabled while connecting to upstream, client: 192.168.157.1, server: localhost, request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8081/", host: "192.168.157.129"
2026/04/13 16:10:06 [error] 34388#34388: *1 no live upstreams while connecting to upstream, client: 192.168.157.1, server: localhost, request: "GET / HTTP/1.1", upstream: "http://tomcat_cluster/", host: "192.168.157.129"


#SELinux 拦截 
getenforce   # 输出Enforcing
# 临时解决
[root@localhost ~]# setenforce 0  
[root@localhost ~]# curl 127.0.0.1
THIS IS TOMCAT 8082
# 永久解决
setsebool -P httpd_can_network_connect 1
```

```shell
# 访问
http://ip
页面在两个 Tomcat 之间切换
```

## Session 丢失问题

/root/data/tomcat/tomcat-8081/webapps/ROOT/index.jsp

```shell
<%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>

<%
request.setCharacterEncoding("UTF-8");

// 获取参数
String user = request.getParameter("user");

// ⭐ 登录逻辑（POST + 重定向）
if("POST".equalsIgnoreCase(request.getMethod())){
    if(user != null && !user.trim().isEmpty()){
        session.setAttribute("user", user);
        session.setAttribute("flag", "THIS_IS_8081");

        // ⭐ 关键：重定向，清掉参数
        response.sendRedirect("/");
        return;
    }
}

// 登出
if("true".equals(request.getParameter("logout"))){
    session.invalidate();
    response.sendRedirect("/");
    return;
}
%>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Tomcat 8081</title>
</head>
<body>

<h2>Tomcat 8081</h2>

<% if(session.getAttribute("user") == null){ %>

    <!-- ⭐ 改成 POST -->
    <form method="post">
        <input name="user" placeholder="Username"/>
        <button type="submit">Login</button>
    </form>

<% } else { %>

    <p>User: <b><%=session.getAttribute("user")%></b></p>
    <p>Session ID: <%=session.getId()%></p>
    <p>FLAG: <%=session.getAttribute("flag")%></p>

    <a href="?logout=true">Logout</a>

<% } %>

</body>
</html>
```

/root/data/tomcat/tomcat-8082/webapps/ROOT/index.jsp

```shell
<%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>

<%
request.setCharacterEncoding("UTF-8");

// 获取参数
String user = request.getParameter("user");

// ⭐ 登录逻辑（POST + 重定向）
if("POST".equalsIgnoreCase(request.getMethod())){
    if(user != null && !user.trim().isEmpty()){
        session.setAttribute("user", user);
        session.setAttribute("flag", "THIS_IS_8082");

        // ⭐ 关键：重定向，清掉参数
        response.sendRedirect("/");
        return;
    }
}

// 登出
if("true".equals(request.getParameter("logout"))){
    session.invalidate();
    response.sendRedirect("/");
    return;
}
%>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Tomcat 8082</title>
</head>
<body>

<h2>Tomcat 8082</h2>

<% if(session.getAttribute("user") == null){ %>

    <!-- ⭐ 改成 POST -->
    <form method="post">
        <input name="user" placeholder="Username"/>
        <button type="submit">Login</button>
    </form>

<% } else { %>

    <p>User: <b><%=session.getAttribute("user")%></b></p>
    <p>Session ID: <%=session.getId()%></p>
    <p>FLAG: <%=session.getAttribute("flag")%></p>

    <a href="?logout=true">Logout</a>

<% } %>

</body>
</html>
```

访问ip，登录不上

```tex
负载均衡 + Session 不共享 = 用户登录状态不一致 / 登录失效

                用户浏览器
                      │
                      ▼
                   Nginx
                 /        \
                ▼          ▼
         Tomcat 8081    Tomcat 8082
            │                │
     Session(A)         Session(B)
   user=admin ✔         无 ❌
   
Nginx（负载均衡器）:把用户请求分发到不同 Tomcat
Apache Tomcat（应用服务器）:处理请求 + 保存 Session（默认在本机内存）
Session（关键）:服务器端存储用户状态的数据（登录信息）
本质原因：Session 默认是“本地内存存储”，不是共享的

常见现象：
✔ 登录后偶尔掉线
✔ 页面时好时坏
✔ 用户状态不一致
✔ 购物车丢失

在负载均衡环境下，请求会被分发到不同应用服务器，
而 Session 默认存储在单个服务器内存中，不具备共享能力。

因此，当用户登录后，请求如果被分发到其他服务器，
由于该服务器没有对应 Session，会导致用户状态丢失，
表现为登录失效、状态不一致等问题。

解决该问题通常有三种方式：
1. 会话粘性（ip_hash）
2. Session 复制（不推荐）
3. 使用 Redis 等统一存储（主流方案）
```

## 方法一：Sticky Session

```bash
# 原来的分发（轮询）
请求1 → 8081  
请求2 → 8082  
请求3 → 8081  
# 加了 ip_hash 后
同一个客户端IP → 永远固定一台
例如
你的电脑 → 永远 8081
别人电脑 → 可能 8082
```

```shell
vi /etc/nginx/conf.d/default.conf
# 修改 upstream
upstream tomcat_cluster {
    ip_hash;   # ⭐ 加这一行（核心）
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
}
# 重启
systemctl restart nginx
```

```shell
ip_hash 可以实现会话粘性，使同一客户端请求固定访问同一台服务器，
从而避免 Session 丢失问题。

但该方案并未真正解决 Session 共享问题，
存在单点故障和负载不均问题，因此在生产环境中一般不推荐使用，
更多采用 Redis 等集中式存储方案。

1. ip_hash 是基于客户端 IP 做哈希分发
2. 同一台机器（127.0.0.1）访问 → 永远落在同一台 Tomcat
3. Session 是通过 Cookie（JSESSIONID）区分，而不是 IP
4. 浏览器无痕模式不会清除当前会话的 Cookie，只是在关闭窗口后才清除
```

## 方法二：Tomcat Session 复制（DeltaManager）

```shell
# DeltaManager 是什么
当 Session 发生变化时，Tomcat 之间自动复制 Session

Tomcat1 ←→ Tomcat2 ←→ Tomcat3
     \         |        /
      \____ Session 同步 ____/


```

server.xml

```shell
<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">

    <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster">

        <Manager className="org.apache.catalina.ha.session.DeltaManager"
                 expireSessionsOnShutdown="false"
                 notifyListenersOnReplication="true"/>

        <Channel className="org.apache.catalina.tribes.group.GroupChannel">

            <Membership className="org.apache.catalina.tribes.membership.McastService"
                        address="228.0.0.4"
                        port="45564"
                        frequency="500"
                        dropTime="3000"/>

            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                      port="4000"
                      autoBind="100"/>

            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter"/>

        </Channel>

        <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"/>
        <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>

    </Cluster>
</Engine>
```

```shell
1. 用户访问 Tomcat1
2. 创建 Session（user=admin）
3. setAttribute 触发复制
4. Session 广播到 Tomcat2
5. Tomcat2 内存生成同样 Session

优点：
✔ 不依赖外部组件
✔ 配置简单（相对）
✔ 原生支持

缺点：
❌ 每次 Session 变化都要复制
❌ 网络开销大
❌ 节点越多越慢（指数级）
❌ 不适合大规模集群
```

## 方法三：Redis 实现 Session 共享

```shell
                浏览器
                   │
                   ▼
                Nginx
             /         \
            ▼           ▼
     Tomcat 8081    Tomcat 8082
            \           /
             ▼         ▼
                 Redis
          （统一 Session 存储）
```

```shell
# 方式1：Tomcat 原生扩展（不推荐）
<Manager className="xxx.RedisSessionManager"
         host="127.0.0.1"
         port="6379"/>
# 方式2：Spring Session（主流）
spring:
  session:
    store-type: redis

  redis:
    host: 127.0.0.1
    port: 6379
```

```shell
核心思想：将 Session 从 Tomcat 内存中“剥离”，统一存储到 Redis

核心优势
1. 真正无状态架构
Tomcat 不再保存状态
2. 支持横向扩展
随便加 Tomcat 节点
无需改代码
3. 故障自动恢复
某台 Tomcat 挂了
Session 仍在 Redis
4. 支持负载均衡任意切换
Nginx 随机分发请求
不会丢登录状态

结论1：
DeltaManager = “复制 Session”
Redis = “集中存储 Session”
结论2：
复制模式 → 强耦合（节点之间绑定）
Redis 模式 → 解耦（统一状态中心）
结论3：
分布式系统的核心：
状态不能放在应用服务器
结论4：
Tomcat 负责计算
Redis 负责状态
Nginx 负责流量
```

```shell
Tomcat 集群属于传统架构中的进阶知识，主要用于理解
“Session 在分布式环境下的处理方式”。

但在现代互联网架构中，已经逐渐被
“Redis Session”或“JWT 无状态登录”所取代。

因此，Tomcat 集群的学习目标不是用于生产落地，
而是用于建立分布式状态管理的认知基础。
```

```shell
1. 为什么负载均衡会导致登录失效？ 
负载均衡会把请求分发到不同的 Tomcat 实例，而 Session 是保存在单个 Tomcat 内存中的，
导致后续请求访问到其他节点时无法获取原有 Session，从而出现登录失效。
# 负载均衡解决的是“流量分发问题”，但不会解决“状态一致性问题”。
2. Session 为什么不能放在 Tomcat？ 
因为 Tomcat 是多实例部署的，每个实例的内存是独立的，
Session 放在本地内存会导致状态无法共享，不具备分布式能力。
#在分布式系统中，应用节点必须无状态（Stateless），否则无法实现水平扩展。
3. Redis 为什么能解决问题？
Redis 作为独立的集中式存储，所有 Tomcat 实例共享同一份 Session 数据，
从而实现跨节点访问一致性，解决了登录状态丢失问题。
#Redis 将 Session 从应用服务器中剥离，实现状态外置，从而让应用服务变成无状态服务，支持水平扩展。
```

# 拓展

```shell
Tomcat 集群学习的核心意义不在于掌握其配置本身，
而在于理解分布式系统中的四个关键问题：

1. 负载均衡（流量如何分发）
2. 服务扩展（如何横向扩展）
3. 状态管理（Session 如何共享）
4. 高可用（节点故障如何处理）

在现代架构中，Tomcat 仅作为应用容器存在，
真正的核心在于 Nginx、Redis 以及高可用机制的组合。

因此，运维学习应从 Tomcat 集群出发，
逐步扩展到负载均衡、缓存系统、高可用架构，
最终形成完整的分布式系统认知体系。
```

[Redis 安装部署 | 🏘️Home](https://ktzxy.top/posts/sfmj09zg3t/)

[Redis6 | 🏘️Home](https://ktzxy.top/posts/0utglweo6k/)

[Nginx | 🏘️Home](https://ktzxy.top/posts/mutl9oasar/)

