+++
date = '2026-05-20T09:57:09+08:00'
draft = false
title = 'shell_repo仓库生成'
slug = "igx6sfm4wj"
description = "shell_repo仓库生成"
summary = "shell_repo仓库生成"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_repo仓库生成

## make_local_repo_final.sh

```bash
#!/bin/bash
# -----------------------------------------------------------------------------
# 脚本名称: make_local_repo_final.sh
# 功能: 清洗缓存目录名，按系统版本构建仓库，并生成对应 repo 文件
# -----------------------------------------------------------------------------

set -e

# =================配置区域=================
# 1. 动态获取系统版本 (例如: RHEL-8.9)
if [ -f /etc/os-release ]; then
    source /etc/os-release
    OS_NAME=${ID^^}       # 转大写，如 RHEL
    OS_VER=${VERSION_ID}  # 如 8.9
    DISTRO_NAME="${OS_NAME}-${OS_VER}"
else
    DISTRO_NAME="Generic-Linux"
fi

# DNF 缓存源目录
CACHE_SRC="/var/cache/dnf"

# 目标仓库根目录 (增加版本号层级)
REPO_BASE="/opt/local_repo"
REPO_DEST="$REPO_BASE/$DISTRO_NAME"

# Repo 配置文件存放位置
LOCAL_REPO_FILE="$REPO_DEST/local-${DISTRO_NAME}.repo"
SYSTEM_REPO_DIR="/etc/yum.repos.d"

# =========================================

# 2. 检查 createrepo
if ! command -v createrepo &> /dev/null; then
    echo "错误: 未找到 createrepo 命令。"
    echo "请先安装: dnf install createrepo -y"
    exit 1
fi

# 3. 创建目标根目录
if [ ! -d "$REPO_DEST" ]; then
    echo ">> 创建版本化仓库目录: $REPO_DEST"
    mkdir -p "$REPO_DEST"
fi

echo ">> 开始构建仓库 (来源: $CACHE_SRC)..."
echo "------------------------------------------------"

# 初始化 Repo 文件头
echo "# Local Repo for $DISTRO_NAME" > "$LOCAL_REPO_FILE"

# 4. 遍历缓存目录
for dir in "$CACHE_SRC"/*; do
    if [ -d "$dir" ]; then
        # 获取原始目录名
        base_name=$(basename "$dir")
        
        # [你的逻辑] 清洗目录名：去除后面的哈希值
        clean_name=$(echo "$base_name" | sed -E 's/-[0-9a-f]{16,}.*//')
        if [ -z "$clean_name" ]; then clean_name="$base_name"; fi

        # 检查是否有 RPM 包
        rpm_count=$(find "$dir" -name "*.rpm" | wc -l)

        if [ "$rpm_count" -gt 0 ]; then
            # 定义具体的子仓库路径
            repo_subdir="$REPO_DEST/$clean_name"
            pkg_dir="$repo_subdir/packages"

            echo "正在处理: $clean_name (发现 $rpm_count 个包)"

            # 创建 packages 目录
            mkdir -p "$pkg_dir"

            # 复制 RPM (使用 -n 跳过已存在的)
            find "$dir" -name "*.rpm" -exec cp -n {} "$pkg_dir/" \;

            # 生成/更新 repodata (针对 repo_subdir)
            echo "  -> 生成 repodata..."
            createrepo --update "$repo_subdir" > /dev/null

            # =========================================
            # [新增] 动态追加内容到 .repo 文件
            # =========================================
            cat >> "$LOCAL_REPO_FILE" <<EOF

[Local-${clean_name}]
name=Local ${DISTRO_NAME} ${clean_name}
baseurl=file://${repo_subdir}
enabled=1
gpgcheck=0
skip_if_unavailable=1
EOF
            echo "  -> 已添加到配置文件"
            echo "------------------------------------------------"
        fi
    fi
done

echo "================================================"
echo "所有任务完成！"
echo "1. 仓库根路径: $REPO_DEST"
echo "2. 生成的Repo: $LOCAL_REPO_FILE"
echo "3. 系统已应用: $SYSTEM_REPO_DIR/local-${DISTRO_NAME}.repo"
echo "================================================"
```

