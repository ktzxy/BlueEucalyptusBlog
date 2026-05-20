+++
date = '2026-05-20T09:12:27+08:00'
draft = false
title = 'Podman 搭建内网容器仓库（私有 Registry）'
slug = "50y2s9719o"
description = "Podman 搭建内网容器仓库（私有 Registry）"
summary = "Podman 搭建内网容器仓库（私有 Registry）"
categories = ["📒tools"]
tags = ["Podman"]
comments = true
series = ["Podman 系列教程"]
showSeries = true
+++

# Podman 搭建内网容器仓库（私有 Registry）

## 1. 服务端搭建 Registry 服务

### 1.1 拉取官方镜像

```bash
podman pull registry:3.0.0
```

### 1.2 创建数据目录并启动服务

```bash
mkdir -p /podman/podman-registry
podman run --privileged -d \
    -v /podman/podman-registry:/var/lib/registry:Z \
    -p 5000:5000 \
    --name registry \
    registry:3.0.0
```

### 1.3 测试仓库是否正常工作

```bash
curl http://localhost:5000/v2/
```

### 1.4 添加 HTTPS 支持（使用自签名证书）

#### 1.4.1 创建证书目录并生成密钥和证书请求

```bash
mkdir -p /podman/podman-registry/certs
cd /podman/podman-registry/certs

openssl genrsa 2048 > cert.key

cat > csr.cnf <<EOF
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[dn]
C = CN
ST = shanxi
L = xi'an
O = example
OU = example
CN = localhost
EOF

openssl req -new -key cert.key -config csr.cnf -out cert.csr
```

#### 1.4.2 签发自签名证书

```bash
openssl x509 -req -days 3650 -in cert.csr -signkey cert.key -out cert.crt
```

#### 1.4.3 查看证书信息（可选）

```bash
openssl x509 -in /podman/podman-registry/certs/cert.crt -text -noout
```

### 1.5 配置认证（htpasswd）

```bash
yum install -y httpd-tools
mkdir -p /podman/podman-registry/auth

htpasswd -Bbn fanyang Fhl3y_Jsdwj > /podman/podman-registry/auth/htpasswd
```

### 1.7 配置文件

```yaml
mkdir -p /podman/podman-registry/etc
cat > /podman/podman-registry/etc/config.yml << 'END'
version: 0.1
log:
  level: info
storage:
  delete:
    enabled: true
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  secret: "Fs4Pl+rZazJr7HaBvTIOQ/crN0PUGRA06kvzJG/OmHA="
  tls:
    certificate: /var/lib/registry/certs/cert.crt
    key: /var/lib/registry/certs/cert.key
  headers:
    Access-Control-Allow-Origin: ['https://registry.example.com:5000']
    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']
    Access-Control-Allow-Headers: ['Authorization', 'Accept', 'Cache-Control']
    Access-Control-Allow-Credentials: ['true']
    Access-Control-Expose-Headers: ['Docker-Content-Digest']
auth:
  htpasswd:
    realm: Registry Realm
    path: /var/lib/registry/auth/htpasswd
END
```

### 1.7 重新运行支持 TLS 和认证的 Registry 容器

```bash
podman rm -f registry

podman run --privileged -d -p 5000:5000 \
    -v /podman/podman-registry:/var/lib/registry:Z \
    -v /podman/podman-registry/etc/config.yml:/etc/distribution/config.yml:Z \
    -e OTEL_TRACES_EXPORTER=none \
    --name registry \
    registry:3.0.0
```

### 1.8 配置 systemd 管理

```bash
podman generate systemd --files --new --name registry
mv container-registry.service ~/.config/systemd/user/
restorecon -RvF ~/.config/systemd/user/

# 设置用户级别服务自动启动
systemctl --user enable --now container-registry.service
```

## 2. 客户端配置

### 2.1 配置镜像仓库

```bash
# 备份并修改配置文件
cp -a /etc/containers/registries.conf{,.bak}

cat > /etc/containers/registries.conf << 'EOF'
unqualified-search-registries = ["registry.hflxhn.com:50000", "docker.io"]

[[registry]]
prefix = "docker.io"
location = "fiyc0dbc.mirror.aliyuncs.com"

[[registry]]
prefix = "registry.hflxhn.com:50000"
location = "registry.hflxhn.com:50000"
insecure = true
EOF
```

### 2.2 添加 `/etc/hosts` 解析

```bash
cat >> /etc/hosts << EOF
172.24.0.248 registry.hflxhn.com
EOF
```

### 2.3 打标签

```bash
podman tag docker.io/library/nginx registry.hflxhn.com:50000/nginx:latest
podman tag docker.io/library/registry registry.hflxhn.com:50000/registry:latest
```

### 2.4 登陆仓库

```bash
podman login --username fanyang --password Fhl3y_Jsdwj registry.hflxhn.com:50000
```

### 2.5 推送镜像

```bash
podman push registry.hflxhn.com:50000/nginx:latest
podman push registry.hflxhn.com:50000/registry:latest
```

## 3. 仓库管理操作

### 3.1 查看仓库中有哪些镜像

```bash
curl -u fanyang:Fhl3y_Jsdwj -X GET https://registry.hflxhn.com:50000/v2/_catalog -k
```

### 3.2 查看某个镜像的 tag 列表

```bash
curl -u fanyang:Fhl3y_Jsdwj -k https://registry.hflxhn.com:50000/v2/php/tags/list
```

### 3.3 删除某个镜像 tag（需开启删除功能）

```bash
# 获取镜像 Manifest 的 digest：
curl -I -X GET -k \
  --header "Accept:application/vnd.docker.distribution.manifest.v2+json" \
  -u fanyang:Fhl3y_Jsdwj \
  https://registry.hflxhn.com:50000/v2/php/manifests/fpm
  
# 执行删除操作：
curl -k -I -X DELETE -u fanyang:Fhl3y_Jsdwj \
  https://registry.hflxhn.com:50000/v2/php/manifests/sha256:<digest>

# 删除功能默认关闭，需在 config.yml 中开启 delete.enabled=true
```

### 3.4 Registry 垃圾回收（释放磁盘空间）

```bash
# 在删除镜像后，需要手动进行垃圾回收才能释放磁盘空间。
podman exec -it registry registry garbage-collect /etc/distribution/config.yml
```
