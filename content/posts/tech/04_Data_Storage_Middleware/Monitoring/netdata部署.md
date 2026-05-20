+++
date = '2026-05-20T10:36:55+08:00'
draft = false
title = 'Netdata部署'
slug = "3nbilpi2fp"
description = "Netdata部署"
summary = "Netdata部署"
categories = ["📒运维"]
tags = ["Netdata"]
comments = true
+++

# Netdata 性能实时监控

```bash
# 创建工作目录
mkdir -p /podman/podman-netdata/{cache,etc,lib}

# 拉起容器
podman run --privileged -d \
--name=netdata \
-p 19999:19999 \
--tz Asia/Shanghai \
--hostname=$(hostname) \
-v /podman/podman-netdata/etc:/etc/netdata:Z \
-v /podman/podman-netdata/lib:/var/lib/netdata:Z \
-v /podman/podman-netdata/cache:/var/cache/netdata:Z \
-v /etc/os-release:/host/etc/os-release:ro \
-v /etc/passwd:/host/etc/passwd:ro \
-v /etc/group:/host/etc/group:ro \
-v /proc:/host/proc:ro \
-v /sys:/host/sys:ro \
--cap-add SYS_PTRACE \
--security-opt apparmor=unconfined \
netdata/netdata

# 生成 service 文件
mkdir -p ~/.config/systemd/user
podman generate systemd --files --new --name netdata
mv container-netdata.service ~/.config/systemd/user
podman stop netdata && podman rm netdata

# 设置开启启动容器
systemctl --user daemon-reload
systemctl --user enable container-netdata.service --now
```
