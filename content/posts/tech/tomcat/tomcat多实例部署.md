+++
date = '2024-03-20T22:22:09+08:00'
draft = false
title = 'tomcat多实例部署'
description = "tomcat多实例部署"      
summary = "tomcat多实例部署"                       
categories = ["📒tomcat"]                                      
tags = ["tomcat"]                                                  
+++
```shell
#!/bin/bash

# 定义软件包名称和版本
jdk_package="jdk-8u371-linux-x64.rpm"
tomcat_package="apache-tomcat-9.0.78.tar.gz"
tomcat_version="9.0.78"
tomcat_base_dir="/usr/local/tomcat"

# 检查是否以root身份运行
if [ "$(id -u)" != "0" ]; then
  echo "Please run as root."
  exit 1
fi

# 检查软件包是否存在
if [ ! -f "/opt/${jdk_package}" ] || [ ! -f "/opt/${tomcat_package}" ]; then
  echo "JDK or Tomcat package not found. Please make sure ${jdk_package} and ${tomcat_package} are in /opt."
  exit 1
fi

# 安装JDK
rpm -ivh /opt/${jdk_package}

# 解压Tomcat
tar -xf /opt/${tomcat_package} -C /opt || { echo "Tomcat extraction failed."; exit 1; }

# 创建Tomcat目录并移动Tomcat
mkdir -p ${tomcat_base_dir} && mv -f /opt/apache-tomcat-${tomcat_version} ${tomcat_base_dir}/tomcat1 

# 复制Tomcat实例
cp -a ${tomcat_base_dir}/tomcat1 ${tomcat_base_dir}/tomcat2 || { echo "Tomcat copy failed."; exit 1; }

# 编辑环境变量文件
echo "export CATALINA_HOME1=${tomcat_base_dir}/tomcat1
export CATALINA_BASE1=${tomcat_base_dir}/tomcat1
export TOMCAT_HOME1=${tomcat_base_dir}/tomcat1
#tomcat2
export CATALINA_HOME2=${tomcat_base_dir}/tomcat2
export CATALINA_BASE2=${tomcat_base_dir}/tomcat2
export TOMCAT_HOME2=${tomcat_base_dir}/tomcat2" | tee -a /etc/profile.d/tomcat.sh && source /etc/profile.d/tomcat.sh || { echo "Environment variables configuration failed."; exit 1; }

# 修改Tomcat配置文件
sed -i.bak -e '123s/<!--//g'\
-e '130s/<!--//g' $CATALINA_HOME1/conf/server.xml $CATALINA_HOME2/conf/server.xml 
sed -i.bak -e '22s/<Server port="8005" shutdown="SHUTDOWN">/<Server port="8006" shutdown="SHUTDOWN">/g' \
    -e 's/  <Connector port="8080" protocol="HTTP\/1.1"
/  <Connector port="8081" protocol="HTTP\/1.1"/g' \
    -e 's/ port="8009"/ port="8010" /g' \
    $CATALINA_HOME2/conf/server.xml || { echo "Tomcat configuration failed."; exit 1; }

# 更新start.sh和shutdown.sh文件
for tomcat_instance in 1 2; do
    cat <<EOF > ${tomcat_base_dir}/tomcat${tomcat_instance}/bin/start.sh
export CATALINA_BASE=\${CATALINA_BASE${tomcat_instance}}
export CATALINA_HOME=\${CATALINA_HOME${tomcat_instance}}
export TOMCAT_HOME=\${TOMCAT_HOME${tomcat_instance}}
EOF

    cat <<EOF > ${tomcat_base_dir}/tomcat${tomcat_instance}/bin/shutdown.sh
export CATALINA_BASE=\${CATALINA_BASE${tomcat_instance}}
export CATALINA_HOME=\${CATALINA_HOME${tomcat_instance}}
export TOMCAT_HOME=\${TOMCAT_HOME${tomcat_instance}}
EOF
done
mkdir -p $CATALINA_HOME1/webapps/kgc
mkdir -p $CATALINA_HOME2/webapps/benet
echo "This is kgc page\!" >$CATALINA_HOME1/webapps/kgc/index.jsp
echo "This is benet page\!" >$CATALINA_HOME2/webapps/benet/index.jsp

# 启动Tomcat实例
${tomcat_base_dir}/tomcat1/bin/startup.sh || { echo "Tomcat 1 startup failed."; exit 1; }
${tomcat_base_dir}/tomcat2/bin/startup.sh || { echo "Tomcat 2 startup failed."; exit 1; }

echo "Tomcat 1 and Tomcat 2 started successfully."


```