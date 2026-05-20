+++
date = '2026-05-20T09:45:13+08:00'
draft = false
title = 'sonarqube'
slug = "43vies7ql2"
description = "一款开源的、多语言的代码质量与安全管理平台"
summary = "一款开源的、多语言的代码质量与安全管理平台"
categories = ["📒虚拟化"]
tags = ["sonarqube"]
comments = true
+++

# sonarqube Service

## 准备 postgres

```bash
# 创建目录结构
mkdir -p /podman/podman-sonar/postgres/{postgresql,data}

# 拉起容器
podman run --privileged -d \
--name postgres \
-p 5432:5432 \
--tz Asia/Shanghai \
-v /podman/podman-sonar/postgres/postgresql:/var/lib/postgresql:Z \
-v /podman/podman-sonar/postgres/data:/var/lib/postgresql/data:Z \
-e POSTGRES_USER=sonar \
-e POSTGRES_PASSWORD=sonar \
-e POSTGRES_DB=sonar \
postgres:16.2

# 生成 service 文件
mkdir -p ~/.config/systemd/user
podman generate systemd --files --new --name postgres
mv container-postgres.service ~/.config/systemd/user
podman stop postgres && podman rm postgres

# 设置开启启动容器
systemctl --user daemon-reload
systemctl --user enable container-postgres.service --now
```

## 拉起 sonarqube

```bash
# 创建目录结构
mkdir -p /podman/podman-sonar/sonarqube/{logs,conf,data,extensions}
chmod -R 777 /podman/podman-sonar/sonarqube/

# 修改系统参数
echo "vm.max_map_count=262144" > /etc/sysctl.conf
sysctl -p

# 拉起容器
podman run -d \
--name sonarqube \
-p 9000:9000 \
--tz Asia/Shanghai \
-e SONAR_JDBC_URL=jdbc:postgresql://10.173.28.206:5432/sonar \
-e SONAR_JDBC_USERNAME=sonar \
-e SONAR_JDBC_PASSWORD=sonar \
-v /podman/podman-sonar/sonarqube/logs:/opt/sonarqube/logs:Z \
-v /podman/podman-sonar/sonarqube/conf:/opt/sonarqube/conf:Z \
-v /podman/podman-sonar/sonarqube/data:/opt/sonarqube/data:Z \
-v /podman/podman-sonar/sonarqube/extensions:/opt/sonarqube/extensions:Z \
sonarqube:9.9.4-community

# 生成 service 文件
mkdir -p ~/.config/systemd/user
podman generate systemd --files --new --name sonarqube
mv container-sonarqube.service ~/.config/systemd/user
podman stop sonarqube && podman rm sonarqube

# 设置开启启动容器
systemctl --user daemon-reload
systemctl --user enable container-sonarqube.service --now
```
