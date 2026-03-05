+++
date = '2024-03-20T22:22:09+08:00'
draft = false
title = 'tomcat启动脚本'
slug = "bhytpcx85p"
description = "tomcat启动脚本"      
summary = "tomcat启动脚本"                       
categories = ["📒tomcat"]                                      
tags = ["tomcat"]                                                  
+++
```shell
#!/bin/bash

# 设置 Tomcat 目录和相关命令路径
tomcat_home="/data/tomcat/tomcat-8484"
SHUTDOWN="$tomcat_home/bin/shutdown.sh"
STARTTOMCAT="$tomcat_home/bin/startup.sh"

# 添加错误处理
set -e

# 参数校验
if [ $# -ne 1 ]; then
    echo "Usage: $0 {start|stop|restart|logs}"
    exit 1
fi

# 根据传入参数执行不同操作
case $1 in
    start)
        echo "启动 $tomcat_home"
        # 检查 Tomcat 是否已经在运行
        if ps -p $(pgrep -f "catalina\.home=$tomcat_home") &>/dev/null; then
            echo "Tomcat 已经在运行"
            exit 1
        fi
        $STARTTOMCAT
        cd $tomcat_home/logs
        tail -f catalina.out
        ;;
    stop)
        echo "关闭 $tomcat_home"
        # 优化停止 Tomcat 的方式
        if [ -f "$SHUTDOWN" ]; then
            $SHUTDOWN
        else
            netstat -anp | grep 8484 | grep -v grep | awk '{print $7}' | sed -e 's//java//g' | sed -e 's/^/kill -9 /g' | sh
        fi
        ;;
    restart)
        echo "重启 $tomcat_home"
        $0 stop
        sleep 5
        $0 start
        ;;
    logs)
        echo "查看 Tomcat 日志："
        cd $tomcat_home/logs
        tail -f catalina.out
        ;;
    *)
        echo "Invalid option. Usage: $0 {start|stop|restart|logs}"
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
groupadd tomcat #加组 
useradd -g tomcat -s /usr/sbin/nologin tomcat #向组加用户 
usermod -L tomcat #锁定密码，使密码无效 
passwd tomcat # 设置密码
chown -R tomcat:tomcat /data #分配权限给用户
[root@localhost data]# ls -l total 0 drwxr-xr-x. 4 tomcat tomcat 79 May 20 08:03 tomcat [root@localhost data]#
```