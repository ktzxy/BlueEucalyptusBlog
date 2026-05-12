+++
date = '2025-02-13T19:25:58+08:00'
draft = false
title = 'Jenkins'
slug = "7oz04i87az"
description = "Jenkins"
categories = ["📒运维"]
tags = ["Jenkins"]
summary = "Jenkins"
comments = true  
+++
# Jenkins

[TOC]

## 安装

### rpm安装

> 下载

下载地址：https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat/

下载版本：jenkins-2.356-1.1.noarch.rpm

高于这个版本的Jenkins不支持JDK8



> 安装

```shell
rpm -ivh jenkins-2.356-1.1.noarch.rpm
```



> 卸载

```shell
# rpm卸载
rpm -e jenkins
# rpm检查是否卸载成功
rpm -ql jenkins
# 删除残留文件
find / -iname jenkins | xargs -n 1000 rm -rf
```

#Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
#Environment="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el7_5.x86_64"

> 修改配置

新版本配置文件 `/usr/lib/systemd/system/jenkins.service`，旧版本配置文件 `/etc/sysconfig/jenkins`

```shell
# 添加JDK路径
vim /usr/lib/systemd/system/jenkins.service
# 可以搜索JAVA_HOME查找, 这个配置被注释了
Environment="JAVA_HOME=JDK路径"
# 如果不知自己的jdk路径可以使用如下命令
which java
# 修改Jenkins后台访问前缀, 搜索JENKINS_PREFIX查找, 自行添加需要的访问前缀, 如果需要配置nginx必须要配置http路径

# 换源; 将里面的url标签内容替换为 https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
# 换源下载插件速度会快一些
vim /var/lib/jenkins/hudson.model.UpdateCenter.xml
```



> 启动

```shell
# 如果启动报错可以参考：https://www.cnblogs.com/aisonsun/p/16576587.html
systemctl daemon-reload
systemctl start jenkins.service
```



> 常用命令

```shell
# 重新加载配置
systemctl daemon-reload
# 启动Jenkins
systemctl start jenkins.service
# 重启Jenkins
systemctl restart jenkins.service
# 停止Jenkins
systemctl stop jenkins.service
# 查看Jenkins当前状态, 是否可以启动
systemctl status jenkins.service
```



> 新手安装

输入访问地址 `http://ip:port/` 访问管理页面，如果配置了访问前缀记得加上，使用新手插件安装，自行设置管理员账号密码



> 解决Jenkins

配置Nginx反向代理，并在 location 块中添加 `proxy_set_header   X-Forwarded-Proto $scheme;`



## 自动化部署

> Git提交代码后自动部署对应项目

### 下载插件

> 搜索：Generic Webhook Trigger Plugin 进行安装

![image-20230215103949929](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152857538.webp)

### 创建任务

![image-20230215104125692](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152904765.webp)

> 选择项目

![image-20230215104229186](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152909772.webp)



### 配置Git

> 在Jenkins工作空间下自动拉取代码，如果你只是想要Jenkins去指定目录执行命令来部署项目，可以不用配置这个

- Repository URL：Git地址，推荐使用https地址
- Credentials：可以pull项目的Git账号
- Branches to build：拉取到工作空间的分支名称

![image-20230215104444621](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152914749.webp)



### 配置Webhook

> 配置Git钩子，Jenkins收到钩子通知后会自动进行构建
>
> 参考：https://help.aliyun.com/document_detail/306411.html#sectiondiv-eyf-d39-ozl

1. 选择 Generic Webhook Trigger，新增一个 Post content parameters
   
2. 配置 Post content parameters
   ![image-20230215105159680](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152920117.webp)

3. 配置 Token

   > Token可以随便填，但是不要重复，最好使用任务名称，保证不会重复

   ![image-20230215105237208](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152925390.webp)

4. 过滤内容

   > 只处理指定分支的推送事件

   ![image-20230215105448285](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152930748.webp)



### 配置构建步骤

也可以选择在构建后操作中进行配置
如果是在当前服务器进行构建，选择 `执行 shell` 选项

![image-20230215105620097](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152935866.webp)



### 配置 shell 脚本

一下命令也可以写成 sh 文件随 git 一起提交，配置 shell 命令时执行 sh 文件就行

```shell
#!/bin/bash
# 重新加载环境变量, Jenkins可能无法运行java、mvn等命令, 加这个命令可以解决; 或者使用全路径运行命令, 比如: /usr/bin/java
source /etc/profile
# 进入项目目录
cd /datanew/code/dev/xxx
# 拉取指定分支
git pull origin master
# maven打包
mvn clean package
# 终止进程
kill $(lsof -i:8196|awk '{print $2}' |sort|uniq|awk '{if($0!="PID") print ""$0" " }')
# 启动项目
nohup java -jar target/jarName.jar &
# 没有这个命令, nohup java -jar命令启动的进程将会被Jenkins杀死
sleep 1s
```

> SpringBoot自动部署Shell

```shell
#!/bin/bash
#------------------------------------------------------------------------------------------------
#程序部署目录, pom.xml所在目录, logs文件夹和脚本日志会生成在这个目录下 TODO 修改部署目录
_workDir=/datanew/code/dev/Application

#部署程序jar名称 TODO 修改部署jar包名称
_program=$_workDir/target/Application.jar

#部署程序jar springboot启动参数如：--server.port=9000 --spring.profiles.active=dev
#不指定参数时，可以为空
_program_param=''

#启动时指定日志输出文件
_default_log_file=$_workDir/nohup.out

# TODO 正式环境如果服务器是 8G 内存，需要改成 Xmx 4096M Xms 4096M -XX:MetaspaceSize=256m
#最大堆内存
_xmx=512m
#初始堆内存
_xms=512m
#最大元空间大小
_xxms=102m

#远程debug调试端口号
_debugPort=5008

#脚本日志输出位置
_log=load.log

#------------------------------------------------------------------------------------------------
_programId=0

# which查找可执行命令路径, Jenkins会在/usr/bin目录下找, 如果你的命令不在这个目录下, 可以创建一个软连接解决
cmd_java=$(which java)
cmd_mvn=$(which mvn)
cd $_workDir || exist

#启动项目
function start() {
  prnt "starting $_program"
  #判断jar是否存在
  if [ ! -f $_program ]; then
    prnt "文件不存在: $_program"
    exit 1
  fi
  setProgramId
  if [ -z $_programId ]; then
    _debugParam=""
    if [ "$1" = "debug" -o "$2" = "debug" ]; then
      _debugParam="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=$_debugPort -agentpath:/usr/local/java/jrebel/lib/libjrebel64.so  -Drebel.remoting_plugin=true -Xdebug"
      prnt "debug模式启动"
    fi
    echo "$1"" ""$2"
    nohup "$cmd_java" -Xmx$_xmx -Xms$_xms -XX:MetaspaceSize=$_xxms $_debugParam -verbose:gc -Xloggc:./logs/gc.log -XX:+PrintGCDetails -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=8 -XX:+UseCMSCompactAtFullCollection -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./logs/heapDump.hprof -XX:+CMSParallelRemarkEnabled -XX:+CMSClassUnloadingEnabled -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=80 -XX:+DisableExplicitGC -XX:+PrintGCTimeStamps -XX:+UseCompressedOops -XX:+DoEscapeAnalysis -XX:MaxTenuringThreshold=10 -Dfile.encoding=UTF-8 -jar $_program $_program_param >>$_default_log_file 2>&1 &
    setProgramId

    if [ -z $_programId ]; then
      prnt "程序启动失败, $_default_log_file 查看原因"
    else
      prnt "程序启动成功, PID:$_programId"
      sleep 2s
      if [ "$1" != "nlog" -a "$2" != "nlog" ]; then
        log
      fi
    fi
  else
    prnt "程序正在运行, PID:$_programId"
  fi
}

# stop project
function stop() {
  setProgramId
  if [ -z $_programId ]; then
    prnt "not running"
  else
    prnt "程序PID: $_programId"
    prnt "Stopping..."
    kill $_programId
    sleep 1
    prnt "程序已停止"
  fi
}

# 查看程序当前运行状态
function status() {
  setProgramId
  if [ -z $_programId ]; then
    prnt "程序已停止"
  else
    prnt "程序正在运行, PID:$_programId"
  fi
}

# 部署
function update() {
  prnt "git pull origin master"
  git pull origin master
  prnt "mvn clean package"
  $cmd_mvn clean package
  restart "$1" "$2"
}

#重启
function restart() {
    stop
    start "$1" "$2"
}

#设置pid到_programId
function setProgramId() {
  _programId=$(ps aux | grep $_program | grep -v grep | awk 'END{print $2}')
}

# 展示日志
function log(){
  tail -n 500 -f $_default_log_file
}

# 展示使用方式
function usage() {
  echo "使用: $0 [start|stop|status|restart|update|log] "
  echo "       <1> sh bin/load.sh start , 在后台启动程序。"
  echo "       <2> sh bin/load.sh stop , 停止程序"
  echo "       <3> sh bin/load.sh status , 显示程序运行状态。"
  echo "       <4> sh bin/load.sh restart , 重启程序"
  echo "       <5> sh bin/load.sh update , 部署项目"
  echo "       <6> sh bin/load.sh log , 查看日志"
  echo "       <7> 其他shell命令暂未支持"
  echo "[start|restart|update] [debug|nlog]"
  echo "       增加debug参数可以开启远程debug调试模式"
  echo "       增加nlog参数启动后不打开日志"
}

# 打印日志并输出到文件中
function prnt(){
  echo -e $(date +%Y-%m-%d" "%T)" $1" | tee -a $_log
}

# use tips
if [ $# -ge 1 ]; then
  # 打印一个换行, 与上一次执行分开
  echo -e "\r" | tee -a $_log
  # 打印java、maven使用的可执行命令全路径
  prnt "java命令路径: $cmd_java"
  prnt "mvn命令路径: $cmd_mvn"
  echo $#
  case $1 in
  start)
    start "$2" "$3"
    ;;
  stop)
    stop
    ;;
  status)
    status
    ;;
  restart)
    restart "$2" "$3"
    ;;
  update)
    update "$2" "$3"
    ;;
  log)
    log
    ;;
  *)
    usage
    ;;
  esac
else
  usage
fi
```



### Git代码库配置 Webhooks

> 云效为例

http://{你的Jenkins地址}/generic-webhook-trigger/invoke?token={Generic Webhook Trigger中配置的Token}

![image-20230215114307333](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152945520.webp)



### Jenkins 安装服务器与项目实际运行不是一台服务器的示例

> 下载插件：SSH Slaves plugin
> 允许使用SSH协议的Java实现通过SSH启动代理。

> 添加 SSH 配置

![image-20230215111402420](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152951206.webp)

> 找到 Publish over SSH 进行配置

![image-20230215111454285](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320152956352.webp)

> 填写相关信息

- Name：SSH 配置的名称，可以随便填
- Hostname：服务器 IP
- Username：登录用户名
- Remote Directory：这个最好填 `/`

![image-20230215111829142](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153001657.webp)

点击 `高级` 按钮，按下方图片示例填写密码，最后使用 `Test Configuration` 按钮进行测试

![image-20230215112106166](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153014880.webp)

> 配置构建步骤

选择 Send files or execute commands over SSH

![image-20230215111211593](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153022733.webp)

填写相关信息

- SSH Server：选择上面配置的 SSH
- Source files：位置基于Jenkins的工作空间 `/{JENKINS_HOME}/workspace/{任务名称}`，传输到远程的文件，填写文件路径，使用 `,` 分割
- Remove prefix：移除的文件路径的前缀，可选，如果填写，`Source files` 中填写的所有路径都必须带有这个前缀
- Remote directory：将文件移动到远程服务器上的哪个目录下，这个配置会使用系统配置中 `Publish over SSH` 配置的 `Remote directory` 作为前缀
- Exec command：远程服务器上执行的命令
- 注意：`Source files` 和 `Exec command` 必须填写一项

参考：方式2没试过

1. 如果想让源代码在远程服务器中，则只配置 `Exec command` 进行拉取代码、打包、运行就行
2. 如果只想让远程服务器保留打包后的文件，先配置Git，再在 `构建设置` 选择进行打包，然后在 `构建后操作` 选项将打包后的文件传输过去执行启动命令；

![image-20230215112541505](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153033262.webp)



## 使用手册

> 前言

控制台访问地址：https://api.qianhai12315.com/jenkins

`Jenkins` 在 zsb上使用的用户加入了 `www` 分组，并将 `Jenkins` 运行所需的文件夹拥有者修改为了这个用户

### 前端使用手册

任务对应源代码所在位置 `/datanew/program/jenkins/workspace/任务名`，可以在这里进行 Git 等操作

#### 新建任务

选择 `font` 视图，前端任务都在这里进行创建，`example` 为前端示例项目

![image-20230216144238972](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153042378.webp)

填写任务名称，填写示例项目名称创建新任务

![image-20230216144400617](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153046246.webp)



#### 修改Git

修改 `Repository URL`，选择 Git 用户

![image-20230216144552883](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153057816.webp)



#### 修改 Shell

修改 `dir` 变量，值为 `zsb` 上的部署目录，基于 `/datanew/www/dev` 路径，如果需要基于别的路径需要修改 `basedir` 变量

**<span style="color:red">`basedir + dir` 目录下的文件都会被删除，请谨慎修改 `basedir`</span>**

![image-20230216144851812](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153051567.webp)



#### 构建项目

点击任务进入详情页，立即构建按钮点击后进行构建，下方是历史构建日志

![image-20230216150228348](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153105356.webp)



### 后端使用手册

#### 创建项目

选择 `back` 视图，后端任务都在这里进行创建，`run-sh` 为后端示例项目

![image-20230216150711707](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153113066.webp)

填写任务名称，填写示例项目名称创建新任务

![image-20230216150815285](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153120562.webp)



#### 修改 Token

修改 Token，使用任务名称作为 Token，保证唯一性

![image-20230216150902640](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153117946.webp)



#### 钩子分支调整

如果你只想当 `master` 分支有新提交的时候才自动构建，那么你下图位置的内容修改为 `master`，这里默认就是 `master`

![image-20230216151005447](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153130972.webp)



#### 修改 Shell

自行调整 Shell

![image-20230216151155651](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260320153127713.webp)



#### 云效配置钩子

详情请见 `自动化部署`-`Git代码库配置 Webhooks`

