+++
date = '2024-03-21T22:22:09+08:00'
draft = false
title = 'tomcat部署和安装'
slug = "frk61ebu2f"
description = "tomcat部署和安装"      
summary = "tomcat部署和安装"                       
categories = ["📒tomcat"]                                      
tags = ["tomcat"]                                                  
+++
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
sourse /etc/profile.d/java.sh
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
