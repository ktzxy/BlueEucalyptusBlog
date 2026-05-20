+++
date = '2026-05-20T10:37:28+08:00'
draft = false
title = 'Prometheus部署'
slug = "e2qq2jdwyf"
description = "Prometheus部署"
summary = "Prometheus部署"
categories = ["📒运维"]
tags = ["Prometheus"]
comments = true
series = ["Prometheus系列"]
showSeries = true
+++

# Prometheus 监控系统

## 1. 部署 Prometheus

```bash
# 创建工作目录
mkdir -p /podman/podman-prometheus/{etc,data}
chown -R 65534:65534 /podman/podman-prometheus/data

# 生成 prometheus 配置文件
cat > /podman/podman-prometheus/etc/prometheus.yml << 'END'
global:
  scrape_interval: 15s
  evaluation_interval: 15s
alerting:
  alertmanagers:
    - static_configs:
        - targets:
rule_files:
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
END

# 拉起容器
podman run -dt \
--name prometheus \
-p 9090:9090 \
--tz Asia/Shanghai \
-v /podman/podman-prometheus/etc:/etc/prometheus:Z \
-v /podman/podman-prometheus/data:/prometheus:Z \
prom/prometheus

# 注册系统服务
loginctl enable-linger
mkdir -p ~/.config/systemd/user && cd ~/.config/systemd/user
podman generate systemd --files --new --name prometheus
podman stop prometheus && podman rm prometheus

# 设置开启启动容器
systemctl --user daemon-reload
systemctl --user enable container-prometheus.service --now

# 防火墙放行
firewall-cmd --add-port=9090/tcp
firewall-cmd --add-port=9090/tcp --per

# 浏览器访问
http://10.173.21.48:9090/
```
