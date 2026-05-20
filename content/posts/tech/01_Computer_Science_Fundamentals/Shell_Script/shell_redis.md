+++
date = '2026-05-20T09:56:53+08:00'
draft = false
title = 'shell_Redis'
slug = "5zzituxcv1"
description = "shell_Redis"
summary = "shell_Redis"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_Redis

```bash
#!/usr/bin/env bash
#
# @sys:     centos-stream-8, rocky-8, alma-8 (适用系统)
# @date:    Tue Sep 23 19:32:45 PDT 2025 (更新日期)
# @author:  yangfan (由 Gemini 重构)
# @info:    通过源码安装 Redis，脚本经过健壮性和可用性增强。
# @run:     ./install-redis.sh -v 7.0.5 -p /data/redis -P 'YourSecureP@ssw0rd' (运行示例)

# set -e: 如果命令以非零状态退出，则立即退出脚本。
# set -u: 当替换时，将未设置的变量视为错误。
# set -o pipefail: 管道命令的返回值是最后一个以非零状态退出的命令的退出状态。
set -euo pipefail

# --- 全局变量与颜色定义 ---
readonly C_RESET='\033[0m'
readonly C_RED='\033[0;31m'
readonly C_GREEN='\033[0;32m'
readonly C_YELLOW='\033[0;33m'

LOG_FILE="/tmp/redis_install_$(date +%s).log" # 日志文件路径
MODEL="online" # online (在线) 或 offline (离线)
INSTALL_PATH="/opt"
REDIS_PASSWORD="" # 将在参数中获取或生成
REDIS_VERSION="7.0.5" # 默认安装的 Redis 版本

# --- 函数定义 ---

# 带颜色的日志输出函数
log() {
    local level="$1"
    local message="$2"
    local color="${C_RESET}"
    case "${level}" in
        "信息") color="${C_GREEN}" ;;
        "警告") color="${C_YELLOW}" ;;
        "错误") color="${C_RED}" ;;
    esac
    echo -e "${color}[${level}] ${message}${C_RESET}"
}

# 显示用法信息的函数
usage() {
    echo "用法: $0 [-v <版本>] [-p <路径>] [-P <密码>] [-m <模式>] [-h]"
    echo "  -v  要安装的 Redis 版本 (例如: 7.0.5)。"
    echo "  -p  安装的基础路径 (默认: /opt)。"
    echo "  -P  Redis 的访问密码 (requirepass)。如果未提供，将自动生成一个随机密码。"
    echo "  -m  安装模式: 'online' (在线, 默认) 或 'offline' (离线)。"
    echo "  -h  显示此帮助信息。"
    exit 0
}

# 执行命令并检查其退出状态，同时记录输出用于调试
run_command() {
    local cmd_desc="$1"
    shift
    log "信息" "${cmd_desc}..."
    # 非调试模式下，后台执行并显示动画
    "$@" &> "${LOG_FILE}" &
    local pid=$!
    local spinner="/|\\-"
    while kill -0 $pid 2>/dev/null; do
        for i in $(seq 0 3); do
            echo -ne "\r[执行中] ${spinner:$i:1} "
            sleep 0.1
        done
    done
    echo -ne "\r"
    wait $pid
    local exit_code=$?

    if [[ ${exit_code} -ne 0 ]]; then
        log "错误" "${cmd_desc} 失败。详情请查看日志: ${LOG_FILE}"
        echo "--- 日志文件最后15行 ---"
        tail -n 15 "${LOG_FILE}"
        echo "--------------------------"
        exit 1
    else
        log "信息" "${cmd_desc} --------> 成功"
    fi
}

# 添加系统用户（如果不存在）
add_user() {
    if id "$1" &>/dev/null; then
        log "信息" "用户 '$1' 已存在。"
    else
        run_command "添加系统用户 '$1'" useradd -r -M -s /sbin/nologin "$1"
    fi
}

# --- 主逻辑 ---
main() {
    # 检查 root 权限
    if [[ "$(id -u)" -ne 0 ]]; then
        log "错误" "此脚本必须以 root 用户身份运行。"
        exit 1
    fi

    # 如果未提供密码，则生成一个随机密码
    if [[ -z "${REDIS_PASSWORD}" ]]; then
        REDIS_PASSWORD=$(head /dev/urandom | tr -dc 'A-Za-z0-9' | head -c 24)
        log "警告" "未提供 Redis 密码，已生成一个随机密码。"
    fi
    
    log "信息" "开始安装 Redis ${REDIS_VERSION}..."
    log "信息" "安装模式: ${MODEL}, 安装路径: ${INSTALL_PATH}"
    log "信息" "完整的安装日志位于: ${LOG_FILE}"

    # 1. 安装依赖
    run_command "安装编译依赖 (gcc, make)" yum install -y gcc make wget

    # 2. 下载并编译 Redis
    local redis_tarball="redis-${REDIS_VERSION}.tar.gz"
    if [[ "${MODEL}" == "online" ]]; then
        run_command "下载 Redis ${REDIS_VERSION}" wget -c "https://download.redis.io/releases/${redis_tarball}" -O "${redis_tarball}"
    elif [[ ! -f "${redis_tarball}" ]]; then
        log "错误" "离线模式: 未找到源码包 '${redis_tarball}'"
        exit 1
    fi

    run_command "解压 ${redis_tarball}" tar xf "${redis_tarball}"
    cd "redis-${REDIS_VERSION}"

    local redis_home="${INSTALL_PATH}/redis-${REDIS_VERSION}"
    local redis_stable_path="${INSTALL_PATH}/redis" # 稳定的符号链接

    local cpu_cores; cpu_cores=$(nproc)
    run_command "使用 ${cpu_cores} 核心编译 Redis" make "-j${cpu_cores}"
    run_command "安装 Redis 到 ${redis_home}" make PREFIX="${redis_home}" install
    
    cd ..
    run_command "清理 Redis 源码" rm -rf "redis-${REDIS_VERSION}" "${redis_tarball}"

    # 3. 配置 Redis
    run_command "创建稳定版符号链接于 ${redis_stable_path}" ln -snf "${redis_home}" "${redis_stable_path}"
    
    add_user "redis"
    mkdir -p "${redis_stable_path}"/{etc,run,logs,data}

    # 使用 heredoc 生成配置文件，比 sed 更可靠、更清晰
    log "信息" "正在生成 redis.conf 配置文件..."
    cat > "${redis_stable_path}/etc/redis.conf" << END
# 绑定所有网络接口，允许远程连接
bind 0.0.0.0
# 保护模式关闭（因为我们设置了密码）
protected-mode no
port 6379
# 以守护进程模式运行
daemonize yes
# 由 systemd 进行监督
supervised systemd
# PID 文件路径
pidfile ${redis_stable_path}/run/redis.pid
# 日志级别
loglevel notice
# 日志文件路径
logfile ${redis_stable_path}/logs/redis.log
# 数据库数量
databases 16
# RDB 持久化目录
dir ${redis_stable_path}/data
# 设置访问密码
requirepass ${REDIS_PASSWORD}
END

    run_command "设置目录所有权为 'redis' 用户" chown -R redis:redis "${redis_home}" "${redis_stable_path}"/{etc,run,logs,data}
    if ! command -v firewall-cmd &> /dev/null || ! systemctl is-active --quiet firewalld; then
        log "警告" "firewall-cmd 命令不存在或 firewalld 服务未运行，跳过防火墙规则设置。"
    else
    firewall-cmd --query-port="6379/tcp" --quiet || (run_command "开放防火墙端口 6379/tcp" firewall-cmd --permanent --add-port="6379/tcp" && run_command "重新加载防火墙" firewall-cmd --reload)
    fi

    # 4. 创建并启动服务
    log "信息" "创建 systemd 服务文件..."
    cat > /etc/systemd/system/redis.service << END
[Unit]
Description=Redis In-Memory Data Store
After=network.target

[Service]
User=redis
Group=redis
Type=forking
ExecStart=${redis_stable_path}/bin/redis-server ${redis_stable_path}/etc/redis.conf
ExecStop=${redis_stable_path}/bin/redis-cli -a "${REDIS_PASSWORD}" shutdown
Restart=always
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
END

    run_command "重新加载 systemd 配置" systemctl daemon-reload
    run_command "设置开机自启并启动 Redis 服务" systemctl enable --now redis

    # 5. 配置环境变量
    cat > /etc/profile.d/redis.sh << END
export PATH=\$PATH:${redis_stable_path}/bin
END
    source /etc/profile.d/redis.sh

    log "信息" "Redis ${REDIS_VERSION} 安装完成！"
    log "信息" "安装路径: ${redis_home}"
    log "信息" "服务名称: redis.service"
    log "信息" "配置文件: ${redis_stable_path}/etc/redis.conf"
    log "信息" "Redis 密码: ${REDIS_PASSWORD}"
    log "信息" "请运行 'source /etc/profile' 或重新登录以使 redis-cli 命令生效。"
}

# --- 参数解析 ---
while getopts "v:p:P:m:h" opt; do
    case ${opt} in
        v) REDIS_VERSION=${OPTARG} ;;
        p) INSTALL_PATH=${OPTARG} ;;
        P) REDIS_PASSWORD=${OPTARG} ;;
        m) MODEL=${OPTARG} ;;
        h) usage ;;
        *) usage ;;
    esac
done

# --- 运行主函数 ---
main
```

