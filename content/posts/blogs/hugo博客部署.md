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
series = ["博客部署系列"]
showSeries= true     
weight = 1
outer = false                                    
+++
# **准备 SSH 密钥**

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

# **阿里云服务器初始化**

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

# **Hugo 构建部署准备**

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

# **配置 ssl**

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

创建 `copyright.html` 文件：

myblog\layouts\partials\copyright.html

```shell
<div class="pe-copyright">
    <hr>
    <blockquote>
    {{ if .Param "reposted" }}
        <p>本文为转载内容，原文信息如下：</p>
        <p>原文标题：{{- .Param "repostedTitle" -}}</p>
        <p>原文作者：{{- .Param "repostedAuthor" -}}</p>
        <p>原文链接：<a href="{{- .Param "repostedLink" -}}" target="_blank">{{- .Param "repostedLink" -}}</a></p>
        <p>如有侵权，请<a href="mailto://{{ .Param "contactEmail" }}">联系作者</a>删除。</p>
    {{ else }}
        <p>本文为原创内容，版权归作者所有。如需转载，请在文章中声明本文标题及链接。</p>
        <p>文章标题：{{ .Title }} —— {{ .Param "author" }}</p>
        <p>文章链接：<a href="{{ .Permalink }}" target="_blank">{{ .Permalink }}</a></p>
        <p>许可协议：<a href="{{- .Param "licenseLink" -}}" target="_blank">{{- .Param "licenseName" -}}</a></p>
    {{ end }}
    </blockquote>
</div>
```

`copyright.css` 文件：

myblog\assets\css\extended\copyright.css

```shell
.pe-copyright {
    margin-top: 20px;
    font-size: 14px;
}

.pe-copyright hr {
    border-style: dashed;
    color: #e26c56;
}

.pe-copyright blockquote {
    margin: 10px 0;
    padding: 0 10px;
    border-inline-start: 3px solid #e26c56;
}

.pe-copyright a {
    box-shadow: 0 1px;
    box-decoration-break: clone;
    -webkit-box-decoration-break: clone;
}
```

在 `footer` 节点上添加如下内容：

这里我是将主题single.html copy一份到myblog\layouts\_default\single.html

```shell
{{- define "main" }}

<article class="post-single">
  <header class="post-header">
    {{ partial "breadcrumbs.html" . }}
    <h1 class="post-title entry-hint-parent">
      {{ .Title }} {{- if .Draft }}
      <span class="entry-hint" title="Draft">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          height="35"
          viewBox="0 -960 960 960"
          fill="currentColor"
        >
          <path
            d="M160-410v-60h300v60H160Zm0-165v-60h470v60H160Zm0-165v-60h470v60H160Zm360 580v-123l221-220q9-9 20-13t22-4q12 0 23 4.5t20 13.5l37 37q9 9 13 20t4 22q0 11-4.5 22.5T862.09-380L643-160H520Zm300-263-37-37 37 37ZM580-220h38l121-122-18-19-19-18-122 121v38Zm141-141-19-18 37 37-18-19Z"
          />
        </svg>
      </span>
      {{- end }}
    </h1>
    {{- if .Description }}
    <div class="post-description">{{ .Description }}</div>
    {{- end }} {{- if not (.Param "hideMeta") }}
    <div class="post-meta">
      {{- partial "post_meta.html" . -}} {{- partial "translation_list.html" .
      -}} {{- partial "edit_post.html" . -}} {{- partial "post_canonical.html" .
      -}}
    </div>
    {{- end }}
  </header>
  {{- $isHidden := (.Param "cover.hiddenInSingle") | default (.Param
  "cover.hidden") | default false }} {{- partial "cover.html" (dict "cxt" .
  "IsSingle" true "isHidden" $isHidden) }} {{- if (.Param "ShowToc") }} {{-
  partial "toc.html" . }} {{- end }} {{- if .Content }}
  <div class="post-content">
    {{- if not (.Param "disableAnchoredHeadings") }} {{- partial
    "anchored_headings.html" .Content -}} {{- else }}{{ .Content }}{{ end }}
  </div>
  {{- end }}

  <!-- Copyright -->
  {{ if .Param "enableCopyright" }} {{ partial "copyright.html" . }} {{ end }}

  <footer class="post-footer">
    {{- $tags := .Language.Params.Taxonomies.tag | default "tags" }}
    <ul class="post-tags">
      {{- range ($.GetTerms $tags) }}
      <li><a href="{{ .Permalink }}">{{ .LinkTitle }}</a></li>
      {{- end }}
    </ul>
    {{- if (.Param "ShowPostNavLinks") }} {{- partial "post_nav_links.html" . }}
    {{- end }} {{- if (and site.Params.ShowShareButtons (ne .Params.disableShare
    true)) }} {{- partial "share_icons.html" . -}} {{- end }}
  </footer>

  {{- if (.Param "comments") }} {{- partial "comments.html" . }} {{- end }}
</article>

{{- end }}{{/* end main */}}

```

hugo.toml配置

```shell
[params]
 # copyright
  enableCopyright = true
  licenseLink = "https://creativecommons.org/licenses/by-nc/4.0/"
  licenseName = "CC BY-NC 4.0"
```

转载文章

```shell
+++
reposted: true
repostedTitle: "修改为原文章标题"
repostedAuthor: "修改为原文章作者名"
repostedLink: "修改为原文章链接"
contactEmail: your email
+++
```

# 添加赞赏

创建 `reward.html` 文件：

myblog\layouts\partials\reward.html

```shell
{{- $button := .Params.rewardButton | default "" }} {{- $subtitle :=
.Params.rewardSubtitle | default "如果本文对您有所帮助，欢迎打赏支持作者！" }}
{{- $title := .Params.rewardTitle | default "赞赏作者" }}

<div class="pe-reward-wrap">
  <div class="pe-reward-card">
    <!-- 标题 -->
    <h3 class="pe-reward-title">{{ $title }}</h3>

    <!-- 副标题 -->
    {{- if not .Params.hideRewardSubtitle }}
    <p class="pe-reward-subtitle">{{ $subtitle }}</p>
    {{ end }}

    <!-- 按钮区域：改为扁平风格 -->
    <div class="pe-reward-btn-container">
      <a
        href="javascript:void(0);"
        class="pe-reward-trigger"
        onclick="
          document
            .querySelector('.pe-reward-overlay')
            .classList.remove('hidden')
        "
      >
        {{ if eq $button "" }}
        <!-- 默认图标：缩小尺寸 -->
        <svg
          viewBox="0 0 1024 1024"
          version="1.1"
          xmlns="http://www.w3.org/2000/svg"
          width="18"
          height="18"
        >
          <path
            d="M835.52 337.824a159.04 159.04 0 0 1 124.544 59.52 155.904 155.904 0 0 1 30.4 134.08l-80.896 341.632a157.952 157.952 0 0 1-56.224 87.68 160.544 160.544 0 0 1-98.688 33.952H166.72c-75.712 0-137.44-60.96-137.44-136.256v-363.84c0-75.296 61.76-136.288 137.44-136.288h80.704c50.176 0 71.232-12.48 85.792-36.544 10.368-17.12 17.472-41.376 23.04-76 1.728-10.656 2.88-19.392 5.408-38.72 11.712-90.944 21.888-124.736 62.528-155.168 25.504-19.072 56.768-25.984 90.72-20.992 64.672 9.504 115.936 43.52 145.824 97.6 23.36 42.272 32.64 95.584 27.904 154.24-1.056 12.832-2.752 25.888-5.12 38.912-0.64 3.68-1.856 9.024-3.712 16.192h155.68z m-261.472 80l14.72-51.104c9.472-32.704 15.04-53.376 16.064-59.2 1.92-10.56 3.264-21.024 4.096-31.296 3.584-43.84-3.04-81.6-18.208-109.056-17.472-31.68-46.88-51.2-87.392-57.12-13.696-2.016-23.456 0.128-31.168 5.888-15.808 11.84-22.4 33.792-31.136 101.312-2.56 20.16-3.84 29.44-5.76 41.248-7.04 43.872-16.704 76.8-33.536 104.64-28.736 47.52-75.424 75.2-154.24 75.2H166.72c-31.744 0-57.44 25.376-57.44 56.224v363.872c0 30.88 25.696 56.256 57.44 56.256h587.904c17.824 0 35.424-6.08 49.344-16.96 13.856-10.848 23.68-26.24 27.712-43.104l80.896-341.6a75.904 75.904 0 0 0-14.944-65.6 79.04 79.04 0 0 0-62.144-29.6H574.08z m-212.8 205.984l97.28 72.64 242.24-209.28s16.224-13.888 30.4-3.008c4.256 3.264 9.12 12.512-1.856 27.008l-252.896 277.856s-19.392 24.864-42.4-0.288l-109.12-138.208s-12.96-18.688 3.264-29.92c5.44-3.744 17.888-9.6 33.12 3.2z"
            fill="currentColor"
          ></path>
        </svg>
        {{ else }} {{ $button | safeHTML }} {{ end }}

        <span class="pe-reward-btn-text">打赏作者</span>
      </a>
    </div>

    <!-- 遮罩层 (保持不变) -->
    <div class="pe-reward-overlay hidden">
      <div class="pe-reward-qr-wrap">
        <div class="pe-reward-img-group">
          <div class="pe-reward-item">
            <div class="pe-reward-qr-box">
              {{- $wechatImg := .Site.Params.WechatPay }} {{- if and $wechatImg
              (not (hasPrefix $wechatImg "http")) }} {{- $wechatImg =
              urls.JoinPath $.Site.BaseURL $wechatImg }} {{- end }}
              <img
                src="{{ $wechatImg }}"
                alt="WeChat Pay"
                onerror="this.style.display = 'none'"
              />
            </div>
            <p>微信</p>
          </div>
          <div class="pe-reward-item">
            <div class="pe-reward-qr-box">
              {{- $alipayImg := .Site.Params.Alipay }} {{- if and $alipayImg
              (not (hasPrefix $alipayImg "http")) }} {{- $alipayImg =
              urls.JoinPath $.Site.BaseURL $alipayImg }} {{- end }}
              <img
                src="{{ $alipayImg }}"
                alt="Alipay"
                onerror="this.style.display = 'none'"
              />
            </div>
            <p>支付宝</p>
          </div>
        </div>
        <span
          class="pe-reward-close"
          onclick="this.closest('.pe-reward-overlay').classList.add('hidden')"
          >&times;</span
        >
      </div>
    </div>
  </div>
</div>

<script>
  document
    .querySelector(".pe-reward-overlay")
    .addEventListener("click", function (evt) {
      if (
        evt.target === this ||
        evt.target.classList.contains("pe-reward-qr-wrap")
      ) {
        this.classList.add("hidden");
      }
    });
</script>

```

添加赞赏按钮及其遮罩层样式。创建 `reward.css` 文件：

myblog\assets\css\extended\reward.css

```shell
/* 外层包裹 */
.pe-reward-wrap {
    margin: 2rem 0;
    /* 减小上下间距 */
    width: 100%;
}

/* 卡片容器 */
.pe-reward-card {
    background-color: var(--theme);
    border-radius: 12px;
    padding: 2rem 1.5rem;
    /* 减小内边距 */
    text-align: center;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
    position: relative;
}

/* 标题 */
.pe-reward-title {
    font-size: 1.2rem;
    /* 稍微调小标题 */
    font-weight: 700;
    margin-bottom: 0.5rem;
    color: var(--body-font-color);
}

/* 副标题 */
.pe-reward-subtitle {
    font-size: 0.9rem;
    color: var(--secondary-font-color);
    margin-bottom: 1.5rem;
    line-height: 1.5;
}

/* 按钮容器 */
.pe-reward-btn-container {
    display: flex;
    justify-content: center;
    align-items: center;
}

/* --- 核心修改：扁平长方形按钮 --- */
.pe-reward-trigger {
    display: inline-flex;
    /* 改为行内弹性布局，让图标和文字横向排列 */
    flex-direction: row;
    /* 横向排列 */
    align-items: center;
    justify-content: center;

    padding: 0.5rem 1.2rem;
    /* 扁平的内边距 */
    height: auto;
    /* 高度自适应 */
    min-width: 120px;
    /* 最小宽度 */

    background-color: #ff5e5e;
    /* 纯色背景，去掉渐变 */
    color: #fff;
    border-radius: 50px;
    /* 胶囊形状 (长方形圆角) */

    font-size: 0.9rem;
    /* 字体调小 */
    font-weight: 500;
    text-decoration: none;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 8px rgba(255, 94, 94, 0.3);
    /* 轻微阴影 */
    border: 1px solid transparent;
}

/* 悬停效果 */
.pe-reward-trigger:hover {
    background-color: #ff4040;
    transform: translateY(-2px);
    /* 轻微上浮 */
    box-shadow: 0 4px 12px rgba(255, 94, 94, 0.4);
}

.pe-reward-trigger svg {
    width: 16px;
    /* 图标缩小 */
    height: 16px;
    fill: currentColor;
    margin-right: 6px;
    /* 图标和文字之间的间距 */
    margin-bottom: 0;
    /* 移除之前的底部间距 */
}

.pe-reward-btn-text {
    line-height: 1;
    white-space: nowrap;
    /* 防止文字换行 */
}

/* --- 遮罩层与二维码 (保持简洁) --- */
.pe-reward-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.6);
    z-index: 9999;
    display: flex;
    justify-content: center;
    align-items: center;
}

.pe-reward-overlay.hidden {
    display: none;
}

.pe-reward-qr-wrap {
    background-color: var(--theme);
    padding: 2rem;
    border-radius: 12px;
    position: relative;
    max-width: 90%;
    width: 450px;
    /* 稍微调窄一点 */
    text-align: center;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.pe-reward-img-group {
    display: flex;
    justify-content: center;
    gap: 1.5rem;
    flex-wrap: wrap;
}

.pe-reward-item {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.pe-reward-qr-box {
    width: 140px;
    height: 140px;
    border: 1px solid #eee;
    padding: 8px;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #fff;
}

.pe-reward-qr-box img {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
    display: block;
}

.pe-reward-item p {
    margin-top: 0.6rem;
    font-size: 0.85rem;
    color: var(--body-font-color);
}

.pe-reward-close {
    position: absolute;
    top: 8px;
    right: 12px;
    font-size: 1.8rem;
    line-height: 1;
    color: #999;
    cursor: pointer;
}

.pe-reward-close:hover {
    color: #333;
}

/* 移动端适配 */
@media screen and (max-width: 568px) {
    .pe-reward-card {
        padding: 1.5rem 1rem;
    }

    .pe-reward-qr-wrap {
        width: 85%;
        padding: 1.5rem;
    }

    .pe-reward-qr-box {
        width: 110px;
        height: 110px;
    }
}
```

## 将赞赏按钮添加到文章章末

在 `footer` 节点上添加如下内容：

```shell
{{- define "main" }}

<article class="post-single">
  <header class="post-header">
    {{ partial "breadcrumbs.html" . }}
    <h1 class="post-title entry-hint-parent">
      {{ .Title }} {{- if .Draft }}
      <span class="entry-hint" title="Draft">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          height="35"
          viewBox="0 -960 960 960"
          fill="currentColor"
        >
          <path
            d="M160-410v-60h300v60H160Zm0-165v-60h470v60H160Zm0-165v-60h470v60H160Zm360 580v-123l221-220q9-9 20-13t22-4q12 0 23 4.5t20 13.5l37 37q9 9 13 20t4 22q0 11-4.5 22.5T862.09-380L643-160H520Zm300-263-37-37 37 37ZM580-220h38l121-122-18-19-19-18-122 121v38Zm141-141-19-18 37 37-18-19Z"
          />
        </svg>
      </span>
      {{- end }}
    </h1>
    {{- if .Description }}
    <div class="post-description">{{ .Description }}</div>
    {{- end }} {{- if not (.Param "hideMeta") }}
    <div class="post-meta">
      {{- partial "post_meta.html" . -}} {{- partial "translation_list.html" .
      -}} {{- partial "edit_post.html" . -}} {{- partial "post_canonical.html" .
      -}}
    </div>
    {{- end }}
  </header>
  {{- $isHidden := (.Param "cover.hiddenInSingle") | default (.Param
  "cover.hidden") | default false }} {{- partial "cover.html" (dict "cxt" .
  "IsSingle" true "isHidden" $isHidden) }} {{- if (.Param "ShowToc") }} {{-
  partial "toc.html" . }} {{- end }} {{- if .Content }}
  <div class="post-content">
    {{- if not (.Param "disableAnchoredHeadings") }} {{- partial
    "anchored_headings.html" .Content -}} {{- else }}{{ .Content }}{{ end }}
  </div>
  {{- end }}

  <!-- Copyright -->
  {{ if .Param "enableCopyright" }} {{ partial "copyright.html" . }} {{ end }}

  <!-- Reward -->
  {{ if .Param "enableReward" }} {{ partial "reward.html" . }} {{ end }}

  <footer class="post-footer">
    {{- $tags := .Language.Params.Taxonomies.tag | default "tags" }}
    <ul class="post-tags">
      {{- range ($.GetTerms $tags) }}
      <li><a href="{{ .Permalink }}">{{ .LinkTitle }}</a></li>
      {{- end }}
    </ul>
    {{- if (.Param "ShowPostNavLinks") }} {{- partial "post_nav_links.html" . }}
    {{- end }} {{- if (and site.Params.ShowShareButtons (ne .Params.disableShare
    true)) }} {{- partial "share_icons.html" . -}} {{- end }}
  </footer>

  {{- if (.Param "comments") }} {{- partial "comments.html" . }} {{- end }}
</article>

{{- end }}{{/* end main */}}

```

## 启用赞赏

在 hugo 配置文件中添加以下配置：

```shell
[params]
 # 启用赞赏功能
  enableReward = true
  # 赞赏按钮显示字符 (可在文章 `frontmatter` 中设置)，默认为赞赏图标
  # rewardButton = 赞赏
  # 赞赏描述 (可在文章 `frontmatter` 中设置)
  rewardDescription = "如果本文对你有所帮助，可以点击上方按钮请作者喝杯咖啡！"
  # 设置微信收款码图片 (路径相对于博客工程的 static 目录。如下文件在 `static/images/wechat_pay.jpg`)
  WechatPay = "images/wechat_pay.webp"
  # 设置支付宝收款码图片 (路径相对于博客工程的 static 目录。如下文件在 `static/images/alipay.jpg`)
  Alipay = "images/alipay.webp"
```

# 文章页展示所属系列的文章列表

[PaperMod 文章页展示所属系列的文章列表 | loyayz](https://loyayz.com/website/220612-hugo-papermodx-series-in-post-page/)

新建模板

myblog\layouts\partials\series-posts.html

```shell
{{- if .Params.series }}
<div class="x-post-series">
  {{- range .Params.series }} {{- $seriesName := . }} {{- /*
  获取当前系列下的所有页面 */}} {{- $pages := where $.Site.RegularPages
  "Params.series" "intersect" (slice $seriesName) }} {{- if $pages }}
  <details open>
    <summary>
      <strong>📚 系列文章：{{ $seriesName }}</strong>
      <span style="font-size: 0.8em; color: var(--secondary)"
        >({{ len $pages }}篇)</span
      >
    </summary>
    <ol>
      {{- /* 核心逻辑：自定义排序 1. 提取标题开头的数字部分 2.
      转换为整数进行排序 3. 如果没数字，则排到最后 */}} {{- $sortedPages := sort
      $pages "Params.weight" }} {{- /* 如果没有设置 weight
      参数，我们尝试按标题数字排序 */}} {{- /* 注意：Hugo 原生 sort
      很难直接解析字符串中的数字，这里用一种变通方法：*/}} {{- /*
      我们假设用户会在 front matter 中设置 weight，或者我们手动处理 */}} {{- /*
      【更简单的方案】：利用 Hugo 的 "ByParam" 或手动构建排序列表 由于 Hugo
      模板语言限制，最稳健的方法是要求用户在 Front Matter 中设置 weight
      或者我们这里写一段稍微复杂的逻辑来提取数字。
      下面这段代码尝试提取标题第一个连续数字作为排序依据 */}} {{- /*
      初始化一个空切片用于存储带排序键的对象 */}} {{- $listWithSortKey := slice
      }} {{- range $pages }} {{- $title := .LinkTitle }} {{- $sortKey := 9999 }}
      {{- /* 默认最大值，如果没有数字排最后 */}} {{- /* 尝试提取标题开头的数字
      (支持 "1 ", "1.", "01 ", "01." 等格式) */}} {{- $matched := findRE
      "^\\s*(\\d+)" $title }} {{- if $matched }} {{- $numStr := index $matched 0
      }} {{- /* 去掉可能存在的非数字字符并转为 int */}} {{- $cleanNum :=
      replaceRE "[^0-9]" "" $numStr }} {{- if $cleanNum }} {{- $sortKey = int
      $cleanNum }} {{- end }} {{- end }} {{- /* 将页面和排序键打包 */}} {{-
      $item := dict "Page" . "SortKey" $sortKey }} {{- $listWithSortKey =
      $listWithSortKey | append $item }} {{- end }} {{- /* 对打包后的列表按
      SortKey 排序 */}} {{- $sortedList := sort $listWithSortKey "SortKey" }}
      {{- /* 遍历排序后的列表渲染 */}} {{- range $sortedList }} {{- $p := .Page
      }}
      <li style="margin-bottom: 8px">
        <a
          href="{{ $p.Permalink }}"
          style="text-decoration: none; color: var(--primary)"
        >
          {{- /* 正则替换：去掉开头的 "数字 + 可选点号 + 可选空格" */}} {{-
          $cleanTitle := replaceRE "^\\s*\\d+\\.?\\s*" "" $p.LinkTitle }} {{
          $cleanTitle }}
        </a>
        {{- if eq $p $ }}
        <span style="color: var(--accent); font-weight: bold">
          &lt;-- 当前阅读</span
        >
        {{- end }}
        <sub
          style="
            display: block;
            font-size: 0.75em;
            color: var(--secondary);
            margin-top: 2px;
          "
        >
          {{ $p.Date.Format "2006-01-02" }}
        </sub>
      </li>
      {{- end }}
    </ol>
  </details>
  {{- end }} {{- end }}
</div>
{{- end }}
```

添加样式

myblog\assets\css\extended\series-posts.css

```shell
.x-post-series {
    padding: 16px 0;
}
```

编辑文章模板

myblog\layouts\_default\single.html

```shell
<footer class="post-footer">
    <!--  添加下面这2行  -->
    {{- if and site.Params.ShowSeriesInPost (ne .Params.showSeries false ) }}
    {{- partial "series-posts.html" . -}} {{- end }} 
    
    
    {{- $tags :=
    .Language.Params.Taxonomies.tag | default "tags" }}
```

配置

```shell
params:
# 是否在文章页显示所属系列的文章列表
  ShowSeriesInPost = true
```

设置文章所属系列

```shell
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
series = ["博客部署美化系列"]
showSeries= true                                         
+++
```

# 搜索页展示系列列表

myblog\layouts\_default\search.html

```shell
{{- define "main" }}
.......
<div id="searchbox">
    <input id="searchInput" autofocus placeholder="{{ .Params.placeholder | default (printf "%s ↵" .Title) }}"
        aria-label="search" type="search" autocomplete="off" maxlength="64">
    <ul id="searchResults" aria-label="search results"></ul>
</div>
<!-- 搜索页加系列文章 -->
{{- if not (.Param "hideSeries") }}
  {{- /* 1. 安全获取系列分类数据 */}}
  {{- $taxonomies := .Site.Taxonomies.series }}
  
  {{- /* 2. 增加防御性判断：确保 $taxonomies 不是 nil 且有内容 */}}
  {{- if and $taxonomies (gt (len $taxonomies) 0) }}
    <h2 style="margin-top: 32px">{{- (.Param "seriesTitle") | default "📚 系列专栏" }}</h2>
    <ul class="terms-tags">
      {{- range $name, $value := $taxonomies }}
        {{- $count := .Count }}
        {{- /* 尝试获取系列页面，如果不存在则使用默认链接 */}}
        {{- $seriesPage := site.GetPage (printf "/series/%s" $name) }}
        {{- $href := cond $seriesPage $seriesPage.Permalink (printf "/series/%s" $name) }}
        {{- $displayName := cond $seriesPage $seriesPage.Name $name }}
        
        <li>
          <a href="{{ $href }}">
            {{ $displayName }} 
            <sup><strong>{{ $count }}</strong></sup>
          </a>
        </li>
      {{- end }}
    </ul>
  {{- end }}
{{- end }} #添加上面一段代码

{{- end }}{{/* end main */}}

```

hugo.toml

```shell
[taxonomies]
    tag = "tags"
    series = "series"
```

# 搜索页展示标签列表

myblog\layouts\_default\search.html

```shell
{{- define "main" }}

.......

<!-- 搜索页加标签列表 -->
{{- if not (.Param "hideTags") }}
  {{- /* 1. 安全获取标签分类数据 */}}
  {{- $taxonomies := .Site.Taxonomies.tags }}
  
  {{- /* 2. 防御性判断：确保数据存在且不为空 */}}
  {{- if and $taxonomies (gt (len $taxonomies) 0) }}
    <h2 style="margin-top: 32px">{{- (.Param "tagsTitle") | default "🏷️ 热门标签" }}</h2>
    <ul class="terms-tags">
      {{- range $name, $value := $taxonomies }}
        {{- $count := .Count }}
        {{- /* 尝试获取标签页面，如果不存在则使用默认链接 */}}
        {{- $tagPage := site.GetPage (printf "/tags/%s" $name) }}
        {{- $href := cond $tagPage $tagPage.Permalink (printf "/tags/%s" $name) }}
        {{- $displayName := cond $tagPage $tagPage.Name $name }}
        
        <li>
          <a href="{{ $href }}">
            {{ $displayName }} 
            <sup><strong>{{ $count }}</strong></sup>
          </a>
        </li>
      {{- end }}
    </ul>
  {{- end }}
{{- end }}

{{- end }}{{/* end main */}}

```

# 文章页底部添加面包屑导航

myblog\layouts\_default\single.html

```shell
<footer class="post-footer">
    <!--  添加下面这行  -->
    {{ partial "breadcrumbs.html" . }}

    {{- if (.Param "ShowPostNavLinks") }}
    {{- partial "post_nav_links.html" . }}
    {{- end }}
</footer>
```

```shell
# 随便哪个样式页面加入
/* 文章页面底部的面包屑居右 */
.post-footer > .breadcrumbs {
  justify-content: flex-end;
}
```

# 列表文章标题后添加标识

自动为列表中的文章添加对应的标识：`[置顶]`、`[转载]`

myblog\layouts\_default\list.html

替换整个 `<header class="entry-header">` 块：

```shell
<header class="entry-header">
  <h2 class="entry-hint-parent">
    {{- .Title }}

    {{- /* 1. 置顶标识 (权重为 1) */}}
    {{- if eq .Weight 1 }}
      <sup>
        <span class="x-entry-istop" style="font-size: 0.6em; font-weight: bold; color: var(--accent); margin-left: 6px;">[置顶]</span>
      </sup>
    {{- end }}

    {{- /* 2. 转载标识 (front matter 中 outer: true) */}}
    {{- if .Param "outer" }}
      <sup>
        <span class="x-entry-isouter" style="font-size: 0.6em; font-weight: bold; color: var(--secondary); margin-left: 6px;">[转载]</span>
      </sup>
    {{- end }}

    {{- /* 3. 草稿标识 (保留原有图标逻辑) */}}
    {{- if .Draft }}
      <span class="entry-hint" title="Draft" style="margin-left: 6px; vertical-align: middle;">
        <svg xmlns="http://www.w3.org/2000/svg" height="18" viewBox="0 -960 960 960" fill="currentColor">
          <path d="M160-410v-60h300v60H160Zm0-165v-60h470v60H160Zm0-165v-60h470v60H160Zm360 580v-123l221-220q9-9 20-13t22-4q12 0 23 4.5t20 13.5l37 37q9 9 13 20t4 22q0 11-4.5 22.5T862.09-380L643-160H520Zm300-263-37-37 37 37ZM580-220h38l121-122-18-19-19-18-122 121v38Zm141-141-19-18 37 37-18-19Z" />
        </svg>
      </span>
    {{- end }}
  </h2>
</header>
```

```shell
+++
title = "测试"
date = 2026-03-03
draft = false
# 1 表示置顶
weight = 1
# true 表示是转载
outer = true
+++
```

# 文章目录级别设置

hugo.toml

```shell
[markup.tableOfContents]
    startLevel = 1  # 从 h1 开始
    endLevel = 3    # 到 h3 结束 (可根据需要调整，比如 4, 5, 6)
    ordered = false # 是否使用有序列表 (1. 2. 3.)，false 为无序列表
```

# 添加瞬间

### 创建数据文件 (数据源)

在项目根目录下创建 `data` 文件夹（如果不存在），并在其中创建 `moments.yml`

```shell
# 格式说明：
# - date: 时间 (ISO 8601 格式)
#   tags: [标签列表]
#   content: | (多行内容，支持 Markdown)

- date: 2024-03-19T16:30:00+08:00
  tags: ["日常", "心情"]
  content: |
    这是第一条瞬间！🎉
    采用单文件模式，所有数据都在这一个 yaml 文件里。
    
    支持代码块：
    ```go
    println("Hello World")
```

- date: 2024-03-18T10:00:00+08:00
  tags: ["技术", "Hugo"]
  content: "只需复制粘贴一段 YAML 配置，就能新增一条瞬间，太方便了！☕️"

- date: 2024-03-15T09:20:00+08:00
  content: "没有标签的瞬间也是可以的。"
```

### 创建页面入口配置

创建瞬间页面的配置文件，定义标题和布局参数。

**路径：** `content/moments/_index.md`

```shell
+++
title = "🌟 瞬间"
description = "记录生活中的点滴灵感与碎片化思考"
date = 2024-01-01T00:00:00+08:00
draft = false

[build]
render = "always"

[params]
ShowToc = false
ShowShareButtons = false
DateFormat = "2006-01-02 15:04"
+++
```

### 创建页面模板

创建渲染逻辑，读取 YAML 数据并生成 HTML。

**路径：** `layouts/moments/list.html`

```shell
{{- define "main" }}
{{- $dateFormat := .Params.DateFormat | default site.Params.DateFormat | default "2006-01-02 15:04" }}
{{- $allMoments := site.Data.moments }}
{{- /* 按时间倒序排序：最新的在前面 */}}
{{- $sortedMoments := sort $allMoments "date" "desc" }}

<article class="post-single">
  <header class="page-header">
    <h1>{{ .Title }}</h1>
    {{- if .Description }}
    <div class="post-description">
      {{ .Description }}
    </div>
    {{- end }}
  </header>

  <div class="post-content">
    {{- if not $sortedMoments }}
      <div style="text-align: center; padding: 4rem 0; color: var(--secondary);">
        <p>暂无瞬间，快去 <code>data/moments.yml</code> 添加第一条吧！✨</p>
      </div>
    {{- else }}
    <ul class="pe-moments">
      {{- range $moment := $sortedMoments }}
      <li class="pe-moment">
                <!-- 1. 头像区域 (直接读取 hugo.toml 配置) -->
        <div class="pe-moment-avatar">
          {{- $avatarUrl := "" }}
          
          {{- /* 第一步：直接尝试从 hugo.toml (site.Params) 读取 */}}
          {{- if site.Params.avatar }}
            {{- $avatarUrl = site.Params.avatar }}
          {{- /* 第二步：如果 tom l没配，再尝试从 content/_index.md 读取 (兼容旧模式) */}}
          {{- else }}
            {{- with site.GetPage "/_index.md" }}
              {{- if .Params.avatar }}
                {{- $avatarUrl = .Params.avatar }}
              {{- end }}
            {{- end }}
          {{- end }}

          {{- /* 渲染结果 */}}
          {{- if $avatarUrl }}
            <img src="{{ $avatarUrl | relURL }}" alt="Avatar">
          {{- else }}
            <div class="pe-moment-avatar-placeholder">
              {{- with site.Params.author }}{{ substr . 0 1 | upper }}{{ else }}M{{ end }}
            </div>
          {{- end }}
        </div>

        <!-- 2. 内容主体 -->
        <div class="pe-moment-body">
          <div class="pe-moment-content">
            {{- /* 渲染 Markdown 内容 */}}
            {{- $moment.content | markdownify }}
          </div>

          <!-- 3. 标签 -->
          {{- if $moment.tags }}
          <div class="pe-moment-tags">
            {{- range $tag := $moment.tags }}
            <a href="{{ "/tags/" | relURL }}{{ $tag | urlize }}/" class="pe-moment-tag">{{ $tag }}</a>
            {{- end }}
          </div>
          {{- end }}

          <!-- 4. 时间 -->
          <div class="pe-moment-meta">
            <span class="pe-moment-time">
              {{- time.Format $dateFormat (time $moment.date) }}
            </span>
          </div>
        </div>
      </li>
      {{- end }}
    </ul>
    {{- end }}
  </div>
</article>
{{- end }}
```

### 添加样式 (CSS)

让瞬间页面拥有朋友圈/便签风格的样式，并适配深色模式。

**路径：** `assets/css/extended/moments.css`

```shell
/* 容器 */
.pe-moments {
  list-style: none;
  padding: 0;
  margin: 0;
}

/* 单条瞬间卡片 */
.pe-moment {
  display: flex;
  gap: 1.2rem;
  padding: 1.5rem 0;
  border-bottom: 1px solid var(--border-color, #eaeaea);
  align-items: flex-start;
  animation: fadeIn 0.5s ease;
}

.pe-moments li:last-child {
  border-bottom: none;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 头像 */
.pe-moment-avatar img {
  width: 3.2rem;
  height: 3.2rem;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid var(--border-color, #eaeaea);
  background-color: var(--code-bg, #f5f5f5);
}

.pe-moment-avatar-placeholder {
  width: 3.2rem;
  height: 3.2rem;
  border-radius: 50%;
  background-color: var(--primary, #333);
  color: var(--theme, #fff);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 1.2rem;
  border: 2px solid var(--border-color, #eaeaea);
}

/* 内容区 */
.pe-moment-body {
  flex: 1;
  min-width: 0;
}

.pe-moment-content {
  font-size: 1rem;
  line-height: 1.7;
  color: var(--content-color, #333);
  margin-bottom: 0.8rem;
  word-wrap: break-word;
}

/* 标签 */
.pe-moment-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 0.8rem;
}

.pe-moment-tag {
  font-size: 0.75rem;
  padding: 0.2rem 0.6rem;
  background-color: var(--code-bg, #f5f5f5);
  color: var(--secondary, #666);
  border-radius: 4px;
  text-decoration: none;
  transition: all 0.2s;
}

.pe-moment-tag:hover {
  background-color: var(--primary, #333);
  color: var(--theme, #fff);
}

/* 时间 */
.pe-moment-meta {
  font-size: 0.85rem;
  color: var(--secondary, #999);
  font-style: italic;
}

/* 移动端适配 */
@media screen and (max-width: 768px) {
  .pe-moment {
    gap: 0.8rem;
    padding: 1.2rem 0;
  }
  .pe-moment-avatar img,
  .pe-moment-avatar-placeholder {
    width: 2.5rem;
    height: 2.5rem;
    font-size: 1rem;
  }
}
```

### 添加到导航菜单 (可选)

为了让访客能方便地找到“瞬间”页面，建议在导航栏添加链接。

编辑  `hugo.toml`)：

```shell
[[menu.main]]
name = "瞬间"
url = "/moments/"
weight = 5  # 调整权重以控制显示顺序
```

头像配置

hugo.toml

```shell
[params]
  # 其他配置...
  
  # 添加作者名字（可选，用于显示首字母 fallback）
  author = "YourName"
  
  # 【关键】添加头像路径
  # 注意：这里写的是相对路径，去掉 "static" 前缀
  avatar = "images/profile.png"
```

# 目录导航右侧显示

修改 single.html 布局，把 TOC 从文章开头移到右侧悬浮位置

```shell
 # 删除35行左右的下列代码
 	{{- if (.Param "ShowToc") }}
     {{- partial "toc.html" . }}
   {{- end }}

#在文章末尾添加 TOC 容器（在第 66 行 </article> 之后，{{- end }} 之前）
   {{- if (.Param "ShowToc") }}
   <aside class="toc-sidebar">
     {{- partial "toc.html" . }}
   </aside>
   {{- end }}
   
```

创建 CSS 样式文件

myblog\assets\css\extended\toc-sidebar.css

```shell
/* 右侧悬浮目录容器 */
.toc-sidebar {
    position: fixed;
    top: calc(var(--header-height) + var(--gap));
    right: 5px;
    width: 300px;
    max-height: calc(100vh - var(--header-height) - var(--gap) * 2 - var(--footer-height));
    overflow-y: auto;
    z-index: 10;
    padding: var(--gap);
}

.toc-sidebar .toc {
    margin-bottom: 0;
    border: 1px solid var(--border);
    background: var(--entry);
    border-radius: var(--radius);
    padding: 0.4em;
    position: sticky;
    top: 0;
}

[data-theme="dark"] .toc-sidebar .toc {
    background: var(--entry);
}

.toc-sidebar .toc details summary {
    cursor: zoom-in;
    margin-inline-start: 10px;
    user-select: none;
    font-weight: 600;
    font-size: 14px;
}

.toc-sidebar .toc details[open] summary {
    cursor: zoom-out;
}

.toc-sidebar .toc .details {
    display: inline;
    font-weight: 600;
    color: var(--primary);
}

.toc-sidebar .toc .inner {
    margin: 10px 15px;
    padding: 0 5px;
    opacity: 0.9;
}

.toc-sidebar .toc li ul {
    margin-inline-start: var(--gap);
}

.toc-sidebar .toc a {
    display: block;
    padding: 4px 0;
    color: var(--secondary);
    font-size: 13px;
    line-height: 1.4;
    text-decoration: none;
    transition: color 0.2s ease;
}

.toc-sidebar .toc a:hover {
    color: var(--primary);
    text-decoration: underline;
}

.toc-sidebar::-webkit-scrollbar {
    width: 4px;
}

.toc-sidebar::-webkit-scrollbar-thumb {
    background: var(--tertiary);
    border-radius: 2px;
}

.toc-sidebar::-webkit-scrollbar-track {
    background: transparent;
}

/* 屏幕宽度小于 1200px 时，取消悬浮，放在文章顶部 */
@media screen and (max-width: 1200px) {
    .toc-sidebar {
        position: static;
        width: 100%;
        max-width: 720px;
        margin: 0 auto var(--content-gap) auto;
        padding: 0;
    }

    .toc-sidebar .toc {
        position: static;
    }
}

/* 移动端隐藏目录 */
@media screen and (max-width: 768px) {
    .toc-sidebar {
        display: none;
    }
}

/* 当前激活的目录项高亮 */
.toc-sidebar .toc a.active {
    color: var(--primary);
    font-weight: 600;
    position: relative;
}

/* 添加箭头指示 */
.toc-sidebar .toc a.active::before {
    content: '➤';
    position: absolute;
    left: -12px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 14px;
    font-weight: bold;
    color: var(--primary);
}

/* 悬停时也显示箭头（可选） */
.toc-sidebar .toc a:hover::before {
    content: '→';
    position: absolute;
    left: -12px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 14px;
    color: var(--secondary);
}

/* 确保有足够空间显示箭头 */
.toc-sidebar .toc a {
    padding-left: 15px;
}
```

创建 JavaScript 文件实现滚动监听

myblog\layouts\partials\toc-highlight.html

```shell
<script>
    function initTocHighlight() {
        const headings = document.querySelectorAll('.post-content h1, .post-content h2, .post-content h3, .post-content h4, .post-content h5, .post-content h6');
        const tocLinks = document.querySelectorAll('.toc-sidebar .toc a');

        if (headings.length === 0 || tocLinks.length === 0) return;

        const observerOptions = {
            root: null,
            rootMargin: '-20% 0px -80% 0px',
            threshold: 0
        };

        const observer = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    const id = entry.target.getAttribute('id');
                    if (id) {
                        tocLinks.forEach(link => {
                            link.classList.remove('active');
                            if (link.getAttribute('href') === '#' + id) {
                                link.classList.add('active');
                                link.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
                            }
                        });
                    }
                }
            });
        }, observerOptions);

        headings.forEach(heading => observer.observe(heading));
    }

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', initTocHighlight);
    } else {
        initTocHighlight();
    }
</script>
```

复制主题下 toc.html 到 layouts/partials/
myblog\layouts\partials\toc.html

```shell
........
</div>
<script>initTocHighlight()</script>   # 在最后添加该行
{{- end }}
```

修改配置，改为true，点击文章自动展开目录导航

```shell
TocOpen  = true                        # 目录默认折叠
```

