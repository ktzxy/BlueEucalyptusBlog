+++
date = '2026-05-20T10:10:30+08:00'
draft = false
title = 'GitLab 代码管理平台'
slug = "7c4gsvu2f6"
description = "GitLab 代码管理平台"
summary = "GitLab 代码管理平台"
categories = ["CI_CD"]
tags = ["GitLab"]
comments = true
+++
# GitLab 代码管理平台

## 容器部署 GitLab

### 1. 修改 sshd 服务默认端口

```bash
sed -i 's/^#Port.*/Port 65522/' /etc/ssh/sshd_config
semanage port -a -t ssh_port_t -p tcp 65522
systemctl restart sshd.service
```

### 2. 创建 gitlab 所需目录

```bash
mkdir -p /podman/podman-gitlab/{config,logs,data}
```

### 3. 拉起容器

```bash
podman run -d \
--name gitlab-ce \
--hostname gitlab.local.com \
-p 443:443 -p 80:80 -p 22:22 \
--tz Asia/Shanghai \
-v /podman/podman-gitlab/config:/etc/gitlab:Z \
-v /podman/podman-gitlab/logs:/var/log/gitlab:Z \
-v /podman/podman-gitlab/data:/var/opt/gitlab:Z \
--shm-size 256m \
gitlab/gitlab-ce:17.1.1-ce.0
```

### 4. 生成 service 文件

```bash
mkdir -p ~/.config/systemd/user
podman generate systemd --files --new --name gitlab-ce
mv container-gitlab-ce.service ~/.config/systemd/user
podman stop gitlab-ce && podman rm gitlab-ce
```

### 5. 设置开启启动容器

```bash
loginctl enable-linger
systemctl --user daemon-reload
systemctl --user enable container-gitlab-ce.service --now
```

### 6. 放行防火墙

```bash
firewall-cmd --add-port=80/tcp --add-port=443/tcp --add-port=65522/tcp
firewall-cmd --add-port=80/tcp --add-port=443/tcp --add-port=65522/tcp --per
```

### 7. 查看默认用户默认的密码

```bash
grep 'Password:' /podman/podman-gitlab/config/initial_root_password
```

### 8. gitlab-runner

```bash
# 安装 podman-docker 包。
# 这个包提供了一个兼容层，它会创建一个 /var/run/docker.sock 文件，
# 并将所有发往这个文件的请求重定向到 Podman 的 API socket。
# 这使得需要 Docker 的应用程序（如 GitLab Runner）可以无缝地使用 Podman。
yum install podman-docker.noarch -y

# 启用并立即启动 podman.socket 服务。
systemctl enable --now podman.socket

# 在宿主机上创建一个目录，用于存放 GitLab Runner 的配置文件 (config.toml)。
mkdir -p /opt/gitlab-runner/config

# 使用 podman 运行 GitLab Runner 容器。
podman run -d --name gitlab-runner     \
  -v /opt/gitlab-runner/config:/etc/gitlab-runner:Z  \
  -v /run/podman/podman.sock:/var/run/docker.sock:Z   \
  --add-host gitlab.example.com:192.168.122.80 \
  docker.io/gitlab/gitlab-runner:latest

# 进入正在运行的 gitlab-runner 容器，并启动一个交互式的 bash shell。
podman exec -it gitlab-runner bash

# 在容器内部执行注册命令。
# gitlab-runner register: 启动注册向导。
# --url http://gitlab.example.com: 指定你的 GitLab 实例的 URL。
# --token glrt-9Jy_vgWtMPvQ7rqAFW8R: 指定从 GitLab CI/CD 设置页面获取的注册令牌 (token)。
# 这个令牌是一次性的，注册成功后就会失效。
gitlab-runner register  --url http://gitlab.example.com  --token glrt-9Jy_vgWtMPvQ7rqAFW8R
```

## yum 部署 GitLab

### 1. 软件下载及安装

```bash
wget https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/8/gitlab-ce-14.7.2-ce.0.el8.x86_64.rpm/download.rpm
yum install gitlab-ce-14.7.2-ce.0.el8.x86_64.rpm
```

### 2. 放行防火墙

```bash
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```

### 3. 更改配置文件

```bash
sed -i "s/^external_url.*/external_url 'http:\/\/gitlab.moonpac.com'/" /etc/gitlab/gitlab.rb
sed -i "/webhook_timeout/a gitlab_rails['webhook_timeout'] = 90" /etc/gitlab/gitlab.rb
sed -i "/^# gitlab_rails\['backup_path'\]/a gitlab_rails['backup_path'] = \"/gitlab/backups\"" /etc/gitlab/gitlab.rb
sed -i "/^### Gitaly settings/i git_data_dirs({\n  \"default\" => {\n    \"path\" => \"\/gitlab\/git-data\"\n  }\n})\n" /etc/gitlab/gitlab.rb
```

### 4. 配置 gitlab

```bash
gitlab-ctl reconfigure
```

### 5. 重启 gitlab

```bash
gitlab-ctl restart
```

## 重置密码

```bash
gitlab-rails console -e production

# 查看所有用户
User.all

# 重置密码
user=User.where(id:1).first
user.password='2wsxzaq1'
user.password_confirmation='2wsxzaq1'

# 保存密码
user.save!
```

## 备份 gialab

```bash
# 创建备份目录
mkdir /gitlab/backups

# 备份配置文件
cp /etc/gitlab/{gitlab.rb,gitlab-secrets.json} /gitlab/backups

# 开始备份. 默认得备份目录是在 /var/opt/gitlab/backups，更改后的目录在 /data/gitlab/backups/
/usr/bin/gitlab-rake gitlab:backup:create

# 备份到远程节点
scp /gitlab/backups/* remote_node:/mpdata/gitlab/backups/
```

## 还原 gitlab

```bash
# 还原操作会重置 gitlab 的 postgresql 数据库相关的东西, 需要启动 postgresql
chmod +x /data/gitlab/backups/1645469009_2022_02_22_11.8.3_gitlab_backup.tar
gitlab-rake gitlab:backup:restore BACKUP=1645469009_2022_02_22_11.8.3

# 启动 gitlab
gitlab-ctl start
```

## 服务异常排查解决

### 1. 常见异常 1

#### 1.1 问题描述

```bash
Whoops, something went wrong on our end.
gitlab正常显示登录页面 登录用户（或创建删除project）报错报500错误 “Whoops, something went wrong on our end.”

# 查看日志
/var/log/gitlab/gitlab-rails/production.log
```

#### 1.2 解决办法

```bash
# 进入DB控制台
gitlab-rails dbconsole

# 重置CI/CD变量,检查ci_组变量和ci_变量表,删除变量
SELECT * FROM public."ci_group_variables";
SELECT * FROM public."ci_variables";

# 重置运行程序注册令牌,清除项目、组和整个实例的所有标记
UPDATE projects SET runners_token = null, runners_token_encrypted = null;
UPDATE namespaces SET runners_token = null, runners_token_encrypted = null;
UPDATE application_settings SET runners_registration_token_encrypted = null;
UPDATE ci_runners SET token = null, token_encrypted = null;

# 执行完后不用重启服务，刷新页面恢复
```

