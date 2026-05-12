+++
date = '2024-04-24T14:20:25+08:00'
draft = false
title = 'nas 常用 docker 服务合集'
slug = "z5kn0wbanv"
description = "nas 常用 docker 服务合集"
categories = ["📒docker"]
tags = ["docker"]
summary = "nas 常用 docker 服务合集"
+++
# docker-compose.yml
```shell
version: '3.7'
services:
# docker监控工具
  portainer:
    image: outlovecn/portainer-cn
    container_name: portainer
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
  # 影音库
  jellyfin:
    image: nyanmisaka/jellyfin
    container_name: jellyfin
    ports:
      - 8096:8096
    volumes:
      - /data/media:/media #影音文件位置，冒号前自定义
    restart: always
  # 影音库
  plex:
    image: plexinc/pms-docker
    container_name: plex
    environment:
      - PLEX_CLAIM=claim-AmmJrgtS8xR3_f6zzLAD #替换为自己的CLAIM,访问 https://www.plex.tv/zh/claim 获取
      - TZ=Asia/Shanghai
    volumes:
      - /data/media:/data #影音文件位置，冒号前自定义
    restart: always
    network_mode: host
  # 影音库
  emby:
    image: emby/embyserver
    container_name: emby
    volumes:
      - /data/emby/config:/config # Configuration directory
      - /data/media:/mnt/share # Media directory
    ports:
      - 18096:8096 # HTTP port
    devices:
      - /dev/dri:/dev/dri # VAAPI/NVDEC/NVENC render nodes
    restart: always
  # bt/pt下载工具
  qbittorrent:
    image: linuxserver/qbittorrent:4.5.2
    container_name: qbittorrent
    environment:
      - WEBUI_PORT=8081 #web-ui端口号，默认用户名:admin，默认密码：adminadmin
      - TZ=Asia/Shanghai
    volumes:
      - /data/qbittorrent/config:/config
      - /data/qbittorrent/downloads:/downloads #下载文件放置位置
    restart: always
    network_mode: host
  # bt/pt下载工具
  transmission:
    # image: chisbread/transmission # 快校版镜像
    image: linuxserver/transmission # 官方版镜像，自行选择用哪种
    container_name: transmission
    environment:
      - TZ=Asia/Shanghai
      - USER=liangge # 默认账号
      - PASS=password  # 默认密码
    volumes:
      - /data/transmission/config:/config
      - /data/transmission/watch:/watch
      - /data/transmission/downloads:/downloads #下载文件放置位置
      - /data/qbittorrent/downloads:/QBdownloads # qbittorrent的下载目录，用于转种
    restart: always
    network_mode: host #默认web端口9091
  # pt自动化
  nas-tools:
    image: nastool/nas-tools #默认账号密码：admin/password
    container_name: nas-tools
    ports:
      - 13000:3000        # webui端口
    volumes:
      - /data:/data #媒体库和下载目录的上级目录
    environment: 
      - NASTOOL_AUTO_UPDATE=false
    restart: always
  # pt自动化
  IYUUPlus:
    image: iyuucn/iyuuplus
    container_name: IYUUPlus
    ports:
      - 8787:8787
    volumes:
      - /data/IYUU/db:/IYUU/db
      - /data/transmission/config/torrents:/torrents #冒号左边是transmission的torrents文件夹在宿主机上的路径，如不使用tr，可删除本行
      - /data/qbittorrent/config/qBittorrent/BT_backup:/BT_backup  #冒号左边是qBittorrent的BT_backup文件夹在宿主机上的路径，如不使用qb，可删除本行
    restart: always
  # 网盘管理
  alist:
    image: 'xhofe/alist'
    container_name: alist
    volumes:
        - '/data/alist:/opt/alist/data'
    ports:
        - '5244:5244'
    restart: always
  # 网盘下载器
  Aria2-Pro:
    container_name: aria2-pro
    image: p3terx/aria2-pro
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /data/aria2:/downloads
    ports:
     - 6800:6800
     - 6888:6888
     - 6888:6888/udp
    restart: always
    logging:
      driver: json-file
      options:
        max-size: 1m
  # 网盘下载器的UI界面
  AriaNg:
    container_name: ariang
    image: p3terx/ariang
    ports:
     - 6880:6880
    restart: always
    logging:
      driver: json-file
      options:
        max-size: 1m
  # 个人网盘
  nextcloud:
    container_name: nextcloud
    image: nextcloud
    ports:
      - 8060:80
    environment:
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=root
      - MYSQL_PASSWORD=123456
      - MYSQL_HOST=mariadb
    volumes:
      - "/data/nextcloud:/var/www/html"
    restart: always
  # 音乐库
  navidrome:
    container_name: navidrome
    image: deluan/navidrome
    ports:
      - "14533:4533"
    volumes:
      - "/data/navidrome:/data"
      - "/data/media/music:/music:ro"
  # 电子书
  reader:
    image: hectorqin/reader
    container_name: reader
    ports:
      - 8082:8080
    volumes:
      - /data/reader/logs:/logs
      - /data/reader/storage:/storage
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - READER_APP_CACHECHAPTERCONTENT=false #是否开启缓存章节内容
      - READER_APP_REMOTEWEBVIEWAPI=http://remote-webview:8083 #开启远程webview
    restart: always
  remote-webview:
    image: hectorqin/remote-webview
    container_name: remote-webview
    restart: always
    ports:
      - 8083:8050
  # 电子书
  calibre-web:
    image: linuxserver/calibre-web
    container_name: calibre-web
    volumes:
      - /data/calibre-web/config:/config
      - /data/calibre-web/books:/books
    ports:
      - 8084:8083
    restart: always
  # 用来生成calibre-web需要的metadata.db文件
  calibre:
    image: linuxserver/calibre
    container_name: calibre
    volumes:
      - /data/calibre/config:/config
    ports:
      - 8087:8080
    restart: always
  # 照片墙
  photoprism:
    image: photoprism/photoprism
    container_name: photoprism
    # 设置重启策略
    restart: always
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
    ports:
      - 12342:12342
    environment:
      PHOTOPRISM_ADMIN_PASSWORD: "photoprism"        # 设置管理员密码，至少包含四个字符
      PHOTOPRISM_HTTP_PORT: 12342                     # 内部监听端口
      PHOTOPRISM_DATABASE_DRIVER: "mariadb"            # 使用 MariaDB (或 MySQL) 代替 SQLite 来提升性能
      PHOTOPRISM_DATABASE_SERVER: "mariadb"     # 设置 MariaDB 服务 (hostname:port)
      PHOTOPRISM_DATABASE_NAME: "photoprism"         # 设置 MariaDB 库名
      PHOTOPRISM_DATABASE_USER: "root"         # 指定 MariaDB 的用户
      PHOTOPRISM_DATABASE_PASSWORD: "123456"     # 设置 MariaDB 用户密码
      PHOTOPRISM_SITE_TITLE: "PhotoPrism"
      PHOTOPRISM_SITE_CAPTION: "Browse Your Life"
      PHOTOPRISM_SITE_DESCRIPTION: "照片墙"
      PHOTOPRISM_SITE_AUTHOR: "liangge"
    volumes:
      # 图片与视频的原生目录，文件上传后先存放在这里
      - "/data/photoprism/photo:/photoprism/originals"
      # 导入目录，如果该目录中存在文件会自动导入
      - "/data/media/photo:/photoprism/import"
  # 导航页
  heimdall:
    image: linuxserver/heimdall
    container_name: heimdall
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /data/heimdall/config:/config
    ports:
      - 8088:80
    restart: always
  # 监控面板
  grafana:
    container_name: grafana
    image: grafana/grafana
    ports:
      - "3000:3000" #默认账号admin/admin，初次登陆需重置密码
    restart: always
  prometheus:
    container_name: prometheus
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - /data/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    restart: always
  node-exporter:
    container_name: node-exporter
    image: prom/node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    restart: always
  qbittorrent-exporter:
    container_name: qbittorrent-exporter
    image: caseyscarborough/qbittorrent-exporter
    ports:
      - "9200:17871"
    environment:
      - TZ=Asia/Shanghai
    restart: always
  tinymediamanager:
    container_name: tinymediamanager
    image: dzhuang/tinymediamanager
    ports:
      - "15800:5800"
    volumes:
      - /data/media:/media:rw
    environment:
      - USER_ID=0
      - GROUP_ID=0
      - TZ=Asia/Shanghai
    extra_hosts: # 修改hosts文件,可能失效
        - "api.themoviedb.org:52.84.18.87"
        - "image.tmdb.org:84.17.46.53"
        - "www.themoviedb.org:52.84.125.129"
    restart: always
  # 智能家居
  homeassistant:
    container_name: homeassistant
    image: homeassistant/home-assistant
    environment:
      - TZ=Asia/Shanghai
    privileged: true
    network_mode: host #默认端口
    restart: always
  # 数据库，给nextcloud和photoprism使用
  mariadb:
    container_name: mariadb
    image: mariadb
    ports:
      - "3306:3306"
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: 123456
  # 软路由
  openwrt:
    image: piaoyizy/openwrt-x86  # 默认账号：root/password
    container_name: openwrt
    restart: always
    privileged: true
    networks:
      macnet: # 配置文件位置：/etc/config/network,一定要加上dns和网关
        ipv4_address: 192.168.18.62
# 虚拟网卡
networks:
  macnet:
    external: true
```

# docker-install.sh

```shell
#!/bin/bash
# 安装ssh
if command -v ssh >/dev/null 2>&1; then
    echo "ssh is installed."
else
    echo "ssh未安装,开始安装ssh..."
    apt update
    apt install openssh-server
fi
# 安装curl
if command -v curl >/dev/null 2>&1; then
    echo "curl is installed."
else
    echo "curl未安装,开始安装curl..."
    apt update
    apt install curl
fi
# 安装docker
if command -v docker >/dev/null 2>&1; then
    echo "Docker is installed."
else
    echo "Docker未安装,开始安装Docker..."
    curl -fsSL https://get.docker.com | bash -s docker
    systemctl start docker
    systemctl enable docker
    docker version
fi

# 安装docker-compose
if command -v docker-compose >/dev/null 2>&1; then
    echo "docker-compose is installed."
else
    echo "docker-compose未安装,开始安装docker-compose..."
    apt update
    apt install docker-compose
fi

# 设置合盖不休眠
if cat /etc/systemd/logind.conf | grep '#HandleLidSwitch=suspend'; then
    sed -i 's/#HandleLidSwitch=suspend/HandleLidSwitch=lock/g' /etc/systemd/logind.conf
    systemctl restart systemd-logind
else
    echo '已设置合盖不休眠'
fi
```

# raid.sh

```shell
# 创建raid,默认两盘位，不是的话需修改此处
read -p "请输入第一块硬盘: " sd1
read -p "请输入第一块硬盘: " sd2
mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/$sd1 /dev/$sd2
# 挂载磁盘
mount /dev/md0 /data

# 开机自动挂载磁盘
touch /etc/rc.local
chmod 755 /etc/rc.local
echo '''#!/bin/bash''' >> /etc/rc.local
echo '''mount /dev/md0 /data''' >> /etc/rc.local
systemctl start rc-local
systemctl enable rc-local
init 6 # 重启
```

# router-install.sh

```shell
# 设置虚拟网卡
read -p "请输入你的网卡名称: " name
ip link set $name promisc on
read -p "请输入你的主路由ip: " ip
docker network create -d macvlan --subnet=$ip/24 --gateway=$ip -o parent=$name macnet
```

# share-install.sh

```shell
# 设置文件共享
mkdir -p /data/media/tv
mkdir -p /data/media/movie
mkdir -p /data/media/av

if command -v samba >/dev/null 2>&1; then
    echo "smb is installed."
else
    echo "smb未安装,开始安装smb..."
    read -p "请输入你要共享的目录: " path
    mkdir -p $path   # 共享目录，可修改。和smb.conf的path配置对应
    apt update
    apt install samba
    systemctl start smb
    systemctl enable smb
    read -p "请输入你的共享用户名称: " user
    cat >> /etc/samba/smb.conf <<EOF
[share]
    comment = share
    path = $path
    valid users = $user
    browseable = yes
    read only = yes
EOF
    smbpasswd -a $user   # 共享用户名
    systemctl restart samba
fi
```