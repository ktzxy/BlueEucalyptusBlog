+++
date = '2026-05-12T17:52:11+08:00'
draft = false
title = 'OpenClaw'
slug = "4r7p6wmto3"
description = "OpenClaw 部署与内网访问实战指南 "
summary = "OpenClaw 部署与内网访问实战指南 "
categories = ["📒AI"]
tags = ["OpenClaw"]
comments = true
+++

# OpenClaw 部署与内网访问实战指南 (AlmaLinux 9.x)

## 一、 环境准备与编译安装

### 1. 基础环境

-   **操作系统:** AlmaLinux 9.x
-   **Node.js 环境:** v24.14.0

### 2. 安装构建依赖

通过 Yum 安装必要的开发工具链及基础组件：

```bash
yum groupinstall "Development Tools" -y
yum install -y cmake lsof
```

### 3. 源码拉取与编译

克隆官方仓库并切换至指定版本标签进行构建：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
git checkout v2026.2.23

# 安装依赖并编译
pnpm install
pnpm ui:build
pnpm build
```

### 4. 服务初始化

执行 onboarding 流程并安装后台守护进程：

```bash
pnpm openclaw onboard --install-daemon
```

## 二、 内网访问与 Nginx 反向代理

由于 OpenClaw 自带的 configure 工具在处理非本地 IP 绑定时存在严格的校验限制，在生产环境中，推荐使用 **Nginx 反向代理** 方案来接管内网流量。

### 1. Nginx 代理配置

创建或修改 Nginx 配置文件 `/etc/nginx/conf.d/openclaw.conf`：

```bash
server {
    listen 80;
    server_name 192.168.122.7; # 替换为宿主机的实际内网 IP

    location / {
        proxy_pass http://127.0.0.1:18789;

        # 核心配置：传递真实 IP 与 Host，防止触发 Origin 校验拦截
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 核心配置：启用 WebSocket 支持 (OpenClaw 前后端通信刚需)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 延长超时时间，避免 AI 长任务或长连接异常中断
        proxy_read_timeout 300s;
    }
}
```

### 2. 操作系统安全设置 (SELinux)

在 AlmaLinux 9 中，若 Nginx 访问本地端口报错 `502 Bad Gateway` 且 `error.log` 显示 `Permission Denied`，通常是被 SELinux 拦截了 Nginx 的网络代理请求。

**推荐方案 (生产环境最佳实践)：** 保持 SELinux 开启，仅放行 HTTPD（Nginx）的网络连接权限：

```bash
setsebool -P httpd_can_network_connect 1
```

**备用方案 (仅限本地调试)：** 临时关闭或永久禁用 SELinux：

```bash
# 临时宽容模式
setenforce 0

# 永久修改 (需重启)
# vi /etc/selinux/config
# 修改 SELINUX=disabled 或 SELINUX=permissive
```

------

## 三、 常见限制与故障排除

### 3.1 跨域访问限制 (Origin Not Allowed)

-   **现象:** 访问 Web UI 提示 `origin not allowed (open the Control UI from the gateway host or allow it in gateway.controlUi.allowedOrigins)`。

-   **解决方案:** 修改配置文件 `~/.openclaw/openclaw.json`，将实际访问的内网 URL 完整写入白名单（建议避免使用 `*` 通配符）。

    ```json
    "gateway": {
      "port": 18789,
      "mode": "local",
      "bind": "loopback",
      "controlUi": {
        "allowedOrigins": ["http://192.168.122.7:18789"] 
      }
    }
    ```

### 3.2 非安全环境/设备验证限制

-   **现象:** 提示 `control ui requires device identity (use HTTPS or localhost secure context)`。浏览器在纯 HTTP 内网环境下会限制安全特性。

-   **解决方案:** 在 `openclaw.json` 中强制禁用安全与设备认证检查：

    ```json
    "gateway": {
      "controlUi": {
        "allowedOrigins": ["http://192.168.122.7:18789"],
        "allowInsecureAuth": true,
        "dangerouslyDisableDeviceAuth": true
      }
    }
    ```

### 3.3 设备身份认证要求 (Device Identity Required)

-   **现象:** 界面拦截并提示 `device identity required`。
-   **解决方案:** 首次登录需携带服务端签发的 Token。
    1.  在服务器执行 `pnpm openclaw dashboard` 获取输出日志中的 `token=xxxxxx`。
    2.  拼接 URL 访问：
        -   首选: `http://192.168.122.7/chat?session=main&token=你的TOKEN`
        -   备选: `http://192.168.122.7/#token=你的TOKEN`

### 3.4 模型上下文窗口过小

-   **现象:** 触发报错 `Agent failed before reply: Model context window too small (4096 tokens). Minimum is 16000.`

-   **解决方案:** 调整 `openclaw.json` 中 `models` 节点的容量参数：

    ```json
    "contextWindow": 16000,
    "maxTokens": 16000
    ```

### 3.5 模型调用返回 400 状态码

-   **现象:** 控制台提示 `400 status code (no body)`。
-   **解决方案:** 检查 LLM 接口配置。以阿里云百炼平台为例，需确保以下参数准确：
    -   `baseUrl`: `https://dashscope.aliyuncs.com/compatible-mode/v1`
    -   `api`: `openai-completions`
    -   `models.id`: 填写具体模型名称（如 `qwen-plus`）。

## 四、 大语言模型 (LLM) 配置与接入

OpenClaw 默认可能指向 Anthropic 等受限或需额外付费的模型。建议接入兼容 OpenAI 接口格式的 DeepSeek 或其他高性价比模型。

### 4.1 注册并配置 Provider

编辑 `~/.openclaw/openclaw.json`，在 `models.providers` 节点下添加引擎配置：

**方案 A：DeepSeek 官方 API (需充值)**

```json
"models": {
  "providers": {
    "deepseek": {
      "api": "openai-completions",
      "baseUrl": "https://api.deepseek.com/v1",
      "apiKey": "sk-你的真实APIKey",
      "models": [
        {
          "id": "deepseek-chat",
          "name": "DeepSeek V3",
          "contextWindow": 64000,
          "maxTokens": 8192
        }
      ]
    }
  }
}
```

**方案 B：使用硅基流动 (SiliconFlow) 获取免费测试额度** 若需免费测试，可使用聚合平台提供的模型：

```JSON
"models": {
  "providers": {
    "siliconflow": {
      "api": "openai-completions",
      "baseUrl": "https://api.siliconflow.cn/v1",
      "apiKey": "sk-你的硅基流动APIKey",
      "models": [
        {
          "id": "deepseek-ai/DeepSeek-V3",
          "name": "DeepSeek V3 (SF)"
        }
      ]
    }
  }
}
```

### 4.2 强制绑定全局默认 Agent 模型 (核心)

配置完 Provider 后，**必须**在 `agents.defaults` 节点下显式声明全局默认模型，否则 OpenClaw 在启动新会话时仍会报错寻找默认的 Anthropic 密钥。

在 `openclaw.json` 中找到或创建 `agents` 节点：

```JSON
"agents": {
  "defaults": {
    "model": "deepseek/deepseek-chat",  // 格式为：Provider名称/模型ID
    "workspace": "/root/.openclaw/workspace",
    "compaction": {
      "mode": "safeguard"
    },
    "maxConcurrent": 4
  }
}
```

*(注：如果使用方案 B 硅基流动，此处应填写 `"model": "siliconflow/deepseek-ai/DeepSeek-V3"`)*

### 4.3 清理旧会话缓存并重启

如果你曾因为模型配置错误导致 Agent 启动失败，旧的会话环境会被污染并锁定在错误配置上。修改配置后，务必清理旧缓存：

```bash
# 1. 停止网关服务
pnpm openclaw gateway stop

# 2. 物理删除报错的旧会话缓存（如名为 main 的默认会话）
rm -rf /root/.openclaw/agents/main

# 3. 重新启动服务
pnpm openclaw gateway start
```

重新访问 Web UI，系统将使用最新配置的模型重新初始化环境。

