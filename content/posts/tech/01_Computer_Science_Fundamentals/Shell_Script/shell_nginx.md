+++
date = '2026-05-20T09:56:45+08:00'
draft = false
title = 'shell_Nginx'
slug = "n4gk91ud2a"
description = "shell_Nginx"
summary = "shell_Nginx"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_Nginx

### install-nginx.sh

```bash
#!/usr/bin/env bash
#
# @sys:     centos-stream-8, rocky-8, alma-8 (适用系统)
# @date:    Tue Sep 23 19:10:41 PDT 2025 (更新日期)
# @author:  yangfan (由 Gemini 重构)
# @info:    通过源码安装 Nginx，脚本经过健壮性和可用性增强。
# @run:     ./install-nginx.sh -v 1.22.1 -p /data/nginx (运行示例)

# set -e: 如果命令以非零状态退出，则立即退出脚本。
# set -u: 当替换时，将未设置的变量视为错误。
# set -o pipefail: 管道命令的返回值是最后一个以非零状态退出的命令的退出状态。
set -euo pipefail

# --- 全局变量与颜色定义 ---
readonly C_RESET='\033[0m'
readonly C_RED='\033[0;31m'
readonly C_GREEN='\033[0;32m'
readonly C_YELLOW='\033[0;33m'

LOG_FILE="/tmp/nginx_install_$(date +%s).log" # 日志文件路径
MODEL="online" # online (在线) 或 offline (离线)
NGINX_VERSION="1.22.1"
INSTALL_PATH="/opt"

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
    echo "用法: $0 [-m <模式>] [-v <版本>] [-p <路径>] [-h]"
    echo "  -m  安装模式: 'online' (在线, 默认) 或 'offline' (离线)。"
    echo "  -v  需要安装的 Nginx 版本 (例如: 1.22.1)。"
    echo "  -p  安装的基础路径 (默认: /opt)。"
    echo "  -h  显示此帮助信息。"
    exit 0
}

# 执行命令并检查其退出状态，同时记录输出用于调试
run_command() {
    local cmd_desc="$1"
    shift
    log "信息" "${cmd_desc}..."
    # 执行命令，将标准输出/错误重定向到日志文件，并在屏幕上显示一个 spinner 动画
    "$@" &> "${LOG_FILE}" &
    local pid=$!
    local spinner="/|\\-"
    while kill -0 $pid 2>/dev/null; do
        for i in $(seq 0 3); do
            echo -ne "\r[执行中] ${spinner:$i:1} "
            sleep 0.1
        done
    done
    echo -ne "\r" # 清除 spinner 动画行

    # 检查命令的实际退出码
    if ! wait $pid; then
        log "错误" "${cmd_desc} 失败。详情请查看日志: ${LOG_FILE}"
        # 打印日志的最后10行以便快速诊断
        echo "--- 日志文件最后10行 ---"
        tail -n 10 "${LOG_FILE}"
        echo "--------------------------"
        exit 1
    else
        log "信息" "${cmd_desc} --------> 成功"
    fi
}

# 使用 YUM/DNF 安装软件包
install_deps() {
    run_command "安装基础依赖" yum install -y gcc gcc-c++ make cmake git wget tar zip unzip bzip2
    run_command "安装 Nginx 构建依赖" yum install -y pcre-devel zlib-devel openssl-devel
}

# 添加系统用户（如果不存在）
add_user() {
    if id "$1" &>/dev/null; then
        log "信息" "用户 '$1' 已存在。"
    else
        run_command "添加用户 '$1'" useradd -r -M -s /sbin/nologin "$1"
    fi
}

# 添加防火墙端口规则（如果尚未添加）
add_firewall_rule() {
    if ! command -v firewall-cmd &> /dev/null || ! systemctl is-active --quiet firewalld; then
        log "警告" "firewall-cmd 命令不存在或 firewalld 服务未运行，跳过防火墙规则设置。"
        return
    fi
    if firewall-cmd --query-port="$1" --quiet; then
        log "信息" "防火墙端口 '$1' 已经开放。"
    else
        run_command "开放防火墙端口 '$1'" firewall-cmd --permanent --add-port="$1"
        run_command "重新加载防火墙" firewall-cmd --reload
    fi
}

# --- 主逻辑 ---
main() {
    # 检查 root 权限
    if [[ "$(id -u)" -ne 0 ]]; then
        log "错误" "此脚本必须以 root 用户身份运行。"
        exit 1
    fi

    log "信息" "开始安装 Nginx..."
    log "信息" "完整的安装日志位于: ${LOG_FILE}"

    install_deps

    # 下载 Nginx
    local nginx_tarball="nginx-${NGINX_VERSION}.tar.gz"
    if [[ "${MODEL}" == "online" ]]; then
        run_command "下载 ${nginx_tarball}" wget -c "https://nginx.org/download/${nginx_tarball}" -O "${nginx_tarball}"
    elif [[ ! -f "${nginx_tarball}" ]]; then
        log "错误" "离线模式: 未找到源码包 '${nginx_tarball}'。"
        exit 1
    fi

    # 解压并构建 Nginx
    run_command "解压 ${nginx_tarball}" tar xf "${nginx_tarball}"
    cd "nginx-${NGINX_VERSION}"

    local nginx_home="${INSTALL_PATH}/nginx-${NGINX_VERSION}"
    local nginx_stable_path="${INSTALL_PATH}/nginx" # 稳定的符号链接
    run_command "配置 Nginx ${NGINX_VERSION}" ./configure \
        --prefix="${nginx_home}" \
        --user=nginx \
        --group=nginx \
        --with-http_stub_status_module \
        --with-http_ssl_module \
        --with-pcre

    local cpu_cores
    cpu_cores=$(nproc)
    run_command "使用 ${cpu_cores} 核心编译 Nginx" make "-j${cpu_cores}"
    run_command "安装 Nginx" make install
    
    cd ..
    run_command "清理源码文件" rm -rf "nginx-${NGINX_VERSION}" "${nginx_tarball}"

    # 创建稳定的符号链接，方便管理
    run_command "创建稳定版符号链接于 ${nginx_stable_path}" ln -snf "${nginx_home}" "${nginx_stable_path}"

    add_user "nginx"
    add_firewall_rule "80/tcp"
    add_firewall_rule "443/tcp"

    # 使用稳定路径创建 systemd 服务文件
    log "信息" "创建 systemd 服务文件..."
    cat > /etc/systemd/system/nginx.service << END
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=${nginx_stable_path}/logs/nginx.pid
ExecStart=${nginx_stable_path}/sbin/nginx -c ${nginx_stable_path}/conf/nginx.conf
ExecReload=/bin/kill -s HUP \$MAINPID
ExecStop=/bin/kill -s QUIT \$MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
END

    # 设置权限和所有权
    run_command "为 ${nginx_home} 设置所有权" chown -R nginx:nginx "${nginx_home}"

    # 启动并设置开机自启服务
    run_command "重新加载 systemd 配置" systemctl daemon-reload
    run_command "设置开机自启并启动 Nginx 服务" systemctl enable --now nginx

    log "信息" "Nginx ${NGINX_VERSION} 安装完成！"
    log "信息" "安装路径: ${nginx_home}"
    log "信息" "服务名称: nginx.service"
    log "信息" "运行状态: $(systemctl is-active nginx)"
}

# --- 参数解析 ---
while getopts "m:v:p:h" opt; do
    case ${opt} in
        m) MODEL=${OPTARG} ;;
        v) NGINX_VERSION=${OPTARG} ;;
        p) INSTALL_PATH=${OPTARG} ;;
        h) usage ;;
        *) usage ;;
    esac
done

# 运行主函数
main
```

