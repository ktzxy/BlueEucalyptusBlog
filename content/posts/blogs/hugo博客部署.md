+++
date = '2026-01-04T11:02:13+08:00'
draft = false
title = 'hugo博客部署'
slug = "4163nwhh20"
description = "使用docker部署hugo，部署到云服务器"  
summary = "使用docker部署hugo，部署到云服务器"         
categories = ["📒博客部署"]                                      
tags = ["hugo"]    
comments = true                                           
+++
# **一、准备 SSH 密钥**

GitHub Actions 要通过 SSH 上传文件到服务器，需要 **一个公私钥对**：

1. **在本地生成 SSH 密钥**（推荐单独为部署创建，不覆盖现有 SSH）：

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/myblog_deploy_key
```

- `-f ~/.ssh/myblog_deploy_key` → 保存路径和文件名
- 会生成两个文件：

```
~/.ssh/myblog_deploy_key        # 私钥
~/.ssh/myblog_deploy_key.pub    # 公钥
```

- **注意不要设置 passphrase**，GitHub Actions 无法交互输入密码

1. **把公钥上传到阿里云服务器**

假设服务器 IP：`123.45.67.89`，用户名 `root` 或普通用户：

```
ssh root@123.45.67.89
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "<myblog_deploy_key.pub 内容>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

- 这样 GitHub Actions 用私钥就可以 SSH 登录服务器，无需密码。

1. **私钥内容放到 GitHub Secrets**

- 打开 GitHub 仓库 → `Settings → Secrets and variables → Actions → New repository secret`
- Secret 名称：`ALI_KEY`
- 内容就是 `~/.ssh/myblog_deploy_key` 文件里的完整内容（保持换行）

> 注意：不要在 public key 中放入 GitHub Secrets，只放私钥。

或者可以直接将本地的公钥上传到服务器，私钥放在github Secrets中

# **二、阿里云服务器初始化**

假设服务器全新（Ubuntu 22.04 举例）：

1. **更新系统**

```
sudo apt update && sudo apt upgrade -y
```

1. **安装必需工具**

```
sudo apt install -y git rsync curl
```

1. **创建博客目录**

```
sudo mkdir -p /var/www/myblog
sudo chown -R $USER:$USER /var/www/myblog
```

1. **允许防火墙端口 80/443**

```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

# **三、Hugo 构建部署准备**

1. 安装 Hugo Extended（支持 PaperMod SCSS）

```
wget https://github.com/gohugoio/hugo/releases/download/v0.115.0/hugo_extended_0.157.0_Linux-64bit.tar.gz
tar -xzf hugo_extended_0.157.0_Linux-64bit.tar.gz
sudo mv hugo /usr/local/bin/

hugo version
# 输出下列
hugo v0.157.0-7747abbb316b03c8f353fd3be62d5011fa883ee6+extended linux/amd64 BuildDate=2026-02-25T16:38:33Z VendorInfo=gohugoio
```

# 安装docker 

https://ktzxy.top/posts/tech/docker/ivzblwvqy0/

## 创建docker-compose

```shell
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: myblog_nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./public:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
```

# **四、配置 ssl**

## 安装 Nginx + Certbot

```
sudo apt install -y nginx certbot python3-certbot-nginx
```

## 申请 SSL 证书（Let’s Encrypt）

使用 certbot 自动签发并配置：

```
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

按照提示：

- 输入邮箱
- 同意条款
- 选择是否强制 HTTPS（选 2 强制跳转）

成功后会自动生成：

```
/etc/letsencrypt/live/your-domain.com/
```

------

## 自动续期（重要）

测试自动续期：

```
sudo certbot renew --dry-run
```

Certbot 会自动加入系统定时任务，无需手动操作。

## 创建 Nginx 站点配置

创建文件：

```
sudo vim /var/www/myblog/nginx.conf
```

写入下面完整配置：

```
# ==============================
# HTTP 自动跳转 HTTPS
# ==============================
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$host$request_uri;
}

# ==============================
# HTTPS 主站配置
# ==============================
server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    # ⚠ 关键修改
    root /usr/share/nginx/html;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header Referrer-Policy no-referrer-when-downgrade;

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_min_length 1024;

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|webp|woff2?)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ /\. {
        deny all;
    }

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;
}

```

------

## 启用站点

```
sudo ln -s /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 最终检查

查看状态：

```
sudo systemctl status nginx
```

访问：

```
https://your-domain.com
```

应该看到你的 Hugo 博客。

# hugo配置

## hugo.toml

```shell
baseURL = 'https://ktzxy.top/'
languageCode = "zh-cn"
timeZone = "Asia/Shanghai"
title = "🏘️Home"
copyright = "[©2024 Blue Eucalyptus's Blog](https://ktzxy.top/)"
theme = "PaperMod"
enableRobotsTXT = true             # 生成 robots.txt
buildDrafts = false                 # 是否构建草稿
buildFuture = false                 # 是否构建未来内容
buildExpired = false                # 是否构建过期内容
canonifyURLs = true
enableGitInfo = true


# 输出 JSON 用于搜索
[outputs]
  home = ["HTML", "RSS", "JSON"]

[permalinks]
  posts = "/posts/:slug/"

[params]
  author = "Atticus Wilde"           # 作者名称
  enableThemeToggle = true           # 夜间模式开关
  enableFontSize = true              # 字号切换
  showReadingTime = true             # 显示阅读时间
  showWordCount = true               # 显示文章字数
  disableIntegrity = true            # 关闭 SRI 验证
  disableLiveReload = true           # 构建时关闭 LiveReload
  enableSearch = true                # 启用搜索功能
  enableInlineShortcodes = true      # 允许短代码
  hasCJKLanguage = true              # 中文/日文/韩文字符支持
  ShowCodeCopyButtons = true          # 代码复制按钮
  ShowRssButtonInSectionTermList = true  # RSS按钮
  enableEmoji = true                  # 启用 emoji 表情
  pygmentsUseClasses = true           # 高亮使用 class 而不是 inline 样式
  #  标签和首页显示配置
  ShowBreadCrumbs = true               # 面包屑导航
  ShowPostNavLinks = true              # 文章上下页导航
  ShowShareButtons = true              # 分享按钮
  UseHugoToc = true                    # 启用 Hugo 内置目录
  ShowToc  = true                        # 页面显示目录
  TocOpen  = false                        # 目录默认折叠
  disableFingerprinting = false          #启用资源指纹识别
  comments = true                         #启用评论
  ShowAllPagesInArchive = true          # 文章归档显示
  defaultTheme = "auto"               # 自动适配系统暗黑模式。
  

  # Fuse.js 搜索选项
  [params.fuseOpts]
    isCaseSensitive = false
    shouldSort = true
    location = 0
    distance = 1000
    threshold = 0.4  # 0.0 意味着完全精确匹配
    minMatchCharLength = 2 # 原为 1，避免单字搜索刷屏
    includeMatches = true
    keys = ["title", "tags", "summary"]
  
  [params.cover]
    hidden = true          # 默认隐藏
    hiddenInList = true
    hiddenInSingle = true
  
  [params.giscus]
    repo = "ktzxy/BlueEucalyptusBlog"
    repoId = "R_kgDORa2bBQ"
    category = "General"
    categoryId = "DIC_kwDORa2bBc4C3yKa"
    mapping = "pathname"
    reactionsEnabled = "1"
    inputPosition = "top"
    theme = "preferred_color_scheme"
    lang = "zh-CN"
    loading = "lazy"

  # 🖼 头像模式配置（参考 RexBlog）
  [params.profileMode]
    enabled = true                     # 启用头像模式
    title = "蓝桉の博客"
    subtitle = "行至水穷处，坐看云起时；山海自远，心自从容。"
    imageUrl = "./images/profile.png"  # 头像路径
    imageWidth = 120
    imageHeight = 120
    imageTitle = "头像"
    buttons = [
      { name = "Posts", url = "/posts/" },
      { name = "Tags", url = "/tags/" },
      { name = "Archives", url = "/archives/" },
      { name = "About", url = "/about/" }
    ]

  # 📂 社交图标（可自由添加）
  [[params.socialIcons]]
    name = "github"
    url = "https://github.com/ktzxy"
  [[params.socialIcons]]
    name = "email"
    url = "mailto:kt_zxh@163.com"
  [[params.socialIcons]]
    name = "csdn"
    url = "https://blog.csdn.net/qq_46087070?type=blog"
  

# 菜单
[menu]

  [[menu.main]]
    identifier = "search"
    name = "Search"
    url = "/search/"
    weight = 2
  
# Markdown/数学公式支持
[markup]
  [markup.goldmark]
    [markup.goldmark.extensions]
      passthrough.delimiters.block = [["\\[","\\]"], ["$$","$$"]]
      passthrough.delimiters.inline = [["\\(","\\)"], ["$","$"]]
      passthrough.enable = true

  [markup.highlight]
    style = "github-dark"
    lineNos = true
    codeFences = true
    guessSyntax = true

# Sitemap优化
[sitemap]
  changefreq = "weekly"
  priority = 0.5
  filename = "sitemap.xml"

# 性能优化
[minify]
  disableXML = true
  minifyOutput = true
```

## 搜索

myblog\content\search.md

```shell
+++
draft = false
layout = "search"
title = 'Search'
summary = "search"
placeholder = "Search posts..."
type = "page"
+++

```

```shell
  # Fuse.js 搜索选项
  [params.fuseOpts]
    isCaseSensitive = false
    shouldSort = true
    location = 0
    distance = 1000
    threshold = 0.4  # 0.0 意味着完全精确匹配
    minMatchCharLength = 2 # 原为 1，避免单字搜索刷屏
    includeMatches = true
    keys = ["title", "tags", "summary"]
```

## 归档

myblog\content\archives.md

```shell
+++
draft = false
title = 'Archives'
layout = "archives"
url = "/archives/"
type = "page"
+++

```

```shell
/* ==========================================
   Hugo「留白诗行」归档样式
   风格：侘寂 · 诗意 · 静谧高贵
========================================== */

.archive {
    position: relative;
    max-width: 800px;
    /* 控制宽度，避免过宽失去聚焦感 */
    margin: 0 auto;
    padding: 80px 40px;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    color: #222;
    line-height: 1.6;
}

/* ==========================================
   年份标题 - 如画卷开篇
========================================== */
.archive-year-header {
    position: relative;
    display: inline-block;
    margin: 60px 0 40px 0;

    font-size: 2.2rem;
    font-weight: 700;
    letter-spacing: -0.5px;
    color: #111;
}

/* 年份旁的小计数 */
.archive-year-header span {
    font-size: 0.9rem;
    color: #999;
    font-weight: 400;
    margin-left: 8px;
    vertical-align: super;
    /* 上标效果 */
    opacity: 0.7;
}

/* ==========================================
   月份标题 - 如章节分隔
========================================== */
.archive-month-header {
    position: relative;
    display: inline-block;
    margin: 40px 0 20px 0;

    font-size: 1.3rem;
    font-weight: 600;
    color: #333;
}

/* 月份旁的小计数 */
.archive-month-header span {
    font-size: 0.8rem;
    color: #aaa;
    font-weight: 400;
    margin-left: 6px;
    vertical-align: super;
    opacity: 0.6;
}

/* ==========================================
   文章列表 - 诗行式排列
========================================== */
.archive-posts {
    display: flex;
    flex-direction: column;
    gap: 28px;
    /* 增大间距，营造呼吸感 */
    margin-bottom: 60px;
}

.archive-entry {
    position: relative;
    display: flex;
    flex-direction: column;
    gap: 6px;

    padding: 16px 0;
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);

    /* 初始状态：轻微透明，营造“未展开”的静谧感 */
    opacity: 0.9;
}

/* Hover 效果：完全显现 + 微移 + 墨迹晕染 */
.archive-entry:hover {
    opacity: 1;
    transform: translateX(-4px);
    /* 向左微移，靠近年份/月份 */
}

/* ==========================================
   标题部分 - 如诗句主体
========================================== */
.archive-entry-link {
    font-size: 1.1rem;
    font-weight: 500;
    color: #222;
    text-decoration: none;
    line-height: 1.4;

    /* 关键：禁止换行，超长省略 */
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;

    transition: color 0.2s ease;
}

.archive-entry:hover .archive-entry-link {
    color: #000;
}

/* ==========================================
   元信息部分 - 如题跋小字
========================================== */
.archive-entry-meta {
    font-size: 0.8rem;
    color: #999;
    font-weight: 400;
    line-height: 1.3;
    letter-spacing: 0.3px;
}

/* ==========================================
   「墨迹晕染」Hover 特效（可选）
   使用伪元素模拟毛笔蘸水扩散
========================================== */
.archive-entry::before {
    content: '';
    position: absolute;
    top: 50%;
    left: 0;
    width: 0;
    height: 0;
    background: radial-gradient(circle, rgba(100, 100, 100, 0.05) 0%, transparent 70%);
    border-radius: 50%;
    transform: translateY(-50%) scale(0);
    transition: all 0.5s cubic-bezier(0.25, 0.8, 0.25, 1);
    z-index: 0;
    pointer-events: none;
}

.archive-entry:hover::before {
    width: 200px;
    height: 200px;
    transform: translateY(-50%) scale(1);
}

/* ==========================================
   响应式优化
========================================== */

@media (max-width: 768px) {
    .archive {
        padding: 40px 20px;
    }

    .archive-year-header {
        font-size: 1.8rem;
        margin: 40px 0 30px 0;
    }

    .archive-month-header {
        font-size: 1.2rem;
        margin: 30px 0 15px 0;
    }

    .archive-posts {
        gap: 20px;
    }

    .archive-entry-link {
        font-size: 1rem;
        white-space: normal;
        /* 手机端允许换行 */
        overflow: visible;
        text-overflow: clip;
    }

    .archive-entry-meta {
        font-size: 0.75rem;
    }

    .archive-entry:hover {
        transform: none;
    }

    .archive-entry::before {
        display: none;
        /* 移动端关闭晕染特效 */
    }
}
```

## 标签

myblog\assets\css\extended\tags.css

css样式必须放在该路径下，自动加载不用引入

```shell
/* ==========================================
   Hugo 极简意境标签样式 - 紧凑版 (Compact & Zen)
   风格参考：PaperMod / 现代杂志风 + 智能填充
========================================== */

/* 标签容器 */
.terms-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    /* 缩小间距，更紧凑 */
    margin-top: 24px;
    padding: 0;
    list-style: none;

    /* 关键：允许子项自由伸缩，自动填充满行 */
    justify-content: flex-start;
}

/* 单个标签项 */
.terms-tags li {
    margin: 0;
    /* 不允许换行中断，由父容器控制 */
}

/* 标签主体链接 */
.terms-tags li a {
    position: relative;
    display: inline-flex;
    align-items: center;

    /* 核心改动：动态宽度 + 最小限制 */
    padding: 6px 14px;
    min-width: 50px;
    /* 防止过窄 */
    max-width: 100%;
    /* 防止溢出 */

    font-size: 0.85rem;
    font-weight: 500;
    color: #333;
    text-decoration: none;
    letter-spacing: 0.3px;

    border-radius: 99px;
    border: 1px solid #eaeaea;
    background-color: #fff;

    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);

    /* 🔥 关键：允许内容撑开，但不强制等宽 */
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

/* Hover 动效：反色填充 */
.terms-tags li a:hover {
    background-color: #222;
    border-color: #222;
    color: #fff;
    transform: translateY(-1px);
    box-shadow: 0 3px 8px rgba(0, 0, 0, 0.06);
}

/* ==========================================
   数量角标优化 (Badge Style)
========================================== */

.terms-tags li a sup {
    position: static;
    display: inline-flex;
    align-items: center;
    justify-content: center;

    margin-left: 6px;

    min-width: 16px;
    height: 16px;
    padding: 0 5px;

    font-size: 0.7rem;
    font-weight: 600;
    line-height: 1;
    color: #999;

    background-color: #f5f5f5;
    border-radius: 99px;

    transition: all 0.3s ease;
}

/* Hover 时角标也跟随反色 */
.terms-tags li a:hover sup {
    background-color: rgba(255, 255, 255, 0.2);
    color: #fff;
}

/* ==========================================
   响应式适配
========================================== */

@media (max-width: 768px) {
    .terms-tags {
        gap: 6px;
    }

    .terms-tags li a {
        padding: 5px 12px;
        font-size: 0.8rem;
        min-width: 45px;
    }

    .terms-tags li a sup {
        min-width: 14px;
        height: 14px;
        font-size: 0.65rem;
        margin-left: 4px;
        padding: 0 4px;
    }
}
```

## 首页

myblog\assets\css\extended\homepage.css

```shell
/* =========================
   Emperor Supreme Edition
========================= */

:root {
    --bg-main: #f5f5f7;
    --text-main: #1d1d1f;
    --text-sub: #6e6e73;
    --accent: #111;
}

html[data-theme="dark"] {
    --bg-main: #000;
    --text-main: #f5f5f7;
    --text-sub: #8e8e93;
    --accent: #fff;
}

/* 背景纯净 */
body.list {
    background: var(--bg-main);
}

/* Hero结构 */
.profile {
    min-height: 95vh;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 0 20px;
}

/* 去卡片 */
.profile_inner {
    background: none;
    box-shadow: none;
    padding: 0;
    max-width: 720px;
}

/* 头像 */
.profile img {
    width: 120px;
    height: 120px;
    border-radius: 50%;
    margin-bottom: 40px;
    filter: grayscale(10%);
    transition: transform .4s ease;
}

.profile img:hover {
    transform: scale(1.04);
}

/* 标题（稳重，不夸张） */
.profile h1 {
    font-size: clamp(36px, 4vw, 52px);
    font-weight: 600;
    letter-spacing: -0.5px;
    line-height: 1.2;
    color: var(--text-main);
    margin-bottom: 22px;
}

/* 副标题 */
.profile span {
    display: block;
    font-size: 18px;
    line-height: 1.8;
    color: var(--text-sub);
    max-width: 600px;
    margin: 0 auto;
}

/* 按钮区域 */
.buttons {
    margin-top: 40px;
    display: flex;
    justify-content: center;
    gap: 20px;
    flex-wrap: wrap;
}

/* 统一按钮 */
.button {
    min-width: 130px;
    text-align: center;
    padding: 12px 28px;
    font-size: 16px;
    font-weight: 500;
    border-radius: 999px;
    border: 1px solid var(--accent);
    background: transparent;
    color: var(--accent);
    transition: all .3s ease;
}

.button:hover {
    background: var(--accent);
    color: var(--bg-main);
    transform: scale(1.05);
}

/* 社交图标 */
.social-icons {
    margin-top: 40px;
    opacity: 0.65;
}

.social-icons a:hover {
    opacity: 1;
}
```

## 关于我

myblog\content\about.md

```shell
+++
draft = false
title = 'About'
description = "技术为术，道在其外；笃行不怠，静水深流。"
layout = "about"
showToc = false
+++
........
```

# 评论配置

## 安装 Giscus App

直接打开这个页面：

👉 https://github.com/apps/giscus

这是 **GitHub** 的 App 页面。

右上角会看到：

```
Install
```

点击：

```
Install
```

然后选择：

```
Only select repositories
```

选择你的博客仓库，例如：

```
myblog
```

最后点击：

```
Install
```

安装完成。

------

## 开启 GitHub Discussions

进入你的博客仓库：

```
Settings
```

找到：

```
Features
```

勾选：

```
Discussions
```

保存。

此时仓库顶部会出现：

```
Code | Issues | Pull requests | Actions | Discussions
```

------

## 生成 Giscus 配置

打开：

👉 https://giscus.app

填写：

```
Repository
你的用户名/仓库名
```

例如：

```
AtticusWilde/myblog
```

如果安装成功，会自动检测到仓库。

然后选择：

```
Discussion Category
→ General
```

映射方式推荐：

```
pathname
```

下面就会生成 **配置代码**。

## 把配置写入 Hugo PaperMod

config.toml 添加 Giscus

```shell
[params]
comments = true

[params.giscus]
repo = "你的用户名/仓库名"
repoId = "R_kgDOxxxx"
category = "General"
categoryId = "DIC_kwDOxxxx"
mapping = "pathname"
strict = "0"
reactionsEnabled = "1"
emitMetadata = "0"
inputPosition = "top"
theme = "preferred_color_scheme"
lang = "zh-CN"
```

myblog\layouts\partials\comments.html

```shell
{{ if .Site.Params.comments }}
<div class="comments">
  <!-- Giscus 评论 -->
  <script
    src="https://giscus.app/client.js"
    data-repo="{{ .Site.Params.giscus.repo }}"
    data-repo-id="{{ .Site.Params.giscus.repoId }}"
    data-category="{{ .Site.Params.giscus.category }}"
    data-category-id="{{ .Site.Params.giscus.categoryId }}"
    data-mapping="pathname"
    data-reactions-enabled="1"
    data-input-position="top"
    data-theme="preferred_color_scheme"
    data-lang="zh-CN"
    crossorigin="anonymous"
    async
  ></script>
</div>
{{ end }}
```

myblog\layouts\partials\post_footer.html

```shell
{{ partial "comments.html" . }}
```

myblog\assets\css\extended\comments.css

```shell
/* ================================
   Giscus 评论区高级美化（PaperMod 专用）
================================ */

/* 评论整体容器 */
.giscus {
    margin-top: 60px;
    padding: 25px;
    border-radius: 16px;
    background: linear-gradient(145deg,
            rgba(255, 255, 255, 0.02),
            rgba(255, 255, 255, 0.04));
    backdrop-filter: blur(10px);
    border: 1px solid var(--border);
    box-shadow:
        0 6px 20px rgba(0, 0, 0, 0.08),
        inset 0 1px 0 rgba(255, 255, 255, 0.05);
    transition: all 0.3s ease;
}

/* hover 微动 */
.giscus:hover {
    transform: translateY(-3px);
    box-shadow:
        0 12px 30px rgba(0, 0, 0, 0.12);
}

/* 评论标题 */
.giscus::before {
    content: "💬 评论区";
    display: block;
    font-size: 18px;
    font-weight: 600;
    margin-bottom: 15px;
    color: var(--primary);
}

/* 评论 iframe */
.giscus iframe {
    border-radius: 12px;
}

/* 评论区加载动画 */
.giscus:empty {
    height: 120px;
    display: flex;
    align-items: center;
    justify-content: center;
}

.giscus:empty::after {
    content: "评论加载中...";
    color: var(--secondary);
    font-size: 14px;
    animation: fade 1.2s infinite;
}

@keyframes fade {
    0% {
        opacity: .3
    }

    50% {
        opacity: 1
    }

    100% {
        opacity: .3
    }
}

/* 暗黑模式优化 */
.dark .giscus {
    background: linear-gradient(145deg,
            rgba(255, 255, 255, 0.02),
            rgba(255, 255, 255, 0.05));
    border: 1px solid rgba(255, 255, 255, 0.08);
}

/* 评论区间距优化 */
.post-content+.giscus {
    margin-top: 80px;
}
```

# 卸载主题

```shell
# 1. 从 .gitmodules 文件中移除记录
git submodule deinit -f themes/PaperMod-PE

# 2. 从 Git 缓存中移除该模块
git rm -f themes/PaperMod-PE

# 3. 删除 .git/modules 中残留的配置 (如果存在)
Remove-Item -Recurse -Force .git/modules/themes/PaperMod-PE -ErrorAction SilentlyContinue
```

# 添加文章版权声明

参考 [PaperMod 添加文章版权声明 | Tofuwine's Blog](https://tofuwine.github.io/posts/18b224b5/)

```shell
```

