+++
date = '2026-05-20T09:56:33+08:00'
draft = false
title = 'shell_Mysql_5.7'
slug = "19ef7n67f9"
description = "shell_Mysql_5.7"
summary = "shell_Mysql_5.7"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_Mysql_5.7

```bash
#!/usr/bin/env bash
#
# @sys:     centos-stream-8, rocky-8, alma-8 (适用系统)
# @date:    Tue Sep 23 19:15:10 PDT 2025 (更新日期)
# @author:  yangfan (由 Gemini 重构)
# @info:    通过源码安装 MySQL 5.7，脚本经过健壮性和可用性增强。
# @run:     ./install-mysql57.sh -p /data/mysql -P 'YourSecureP@ssw0rd' (运行示例)

# set -e: 如果命令以非零状态退出，则立即退出脚本。
# set -u: 当替换时，将未设置的变量视为错误。
# set -o pipefail: 管道命令的返回值是最后一个以非零状态退出的命令的退出状态。
set -euo pipefail

# --- 全局变量与颜色定义 ---
readonly C_RESET='\033[0m'
readonly C_RED='\033[0;31m'
readonly C_GREEN='\033[0;32m'
readonly C_YELLOW='\033[0;33m'

LOG_FILE="/tmp/mysql57_install_$(date +%s).log" # 日志文件路径
MODEL="online" # online (在线) 或 offline (离线)
INSTALL_PATH="/opt"
ROOT_PASSWORD="" # 将在参数中获取或生成
DEBUG_MODE=false # 默认关闭调试模式
VERSION_NAME="mysql57" # 用于目录命名
MYSQL_VERSION="5.7.30" # 具体的版本号

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
    echo "用法: $0 [-p <路径>] [-P <密码>] [-m <模式>] [-d] [-h]"
    echo "  -p  安装的基础路径 (默认: /opt)。"
    echo "  -P  MySQL root 用户的密码。如果未提供，将自动生成一个随机密码。"
    echo "  -m  安装模式: 'online' (在线, 默认) 或 'offline' (离线)。"
    echo "  -d  开启调试模式 (会显示 cmake 和 make 的详细输出)。"
    echo "  -h  显示此帮助信息。"
    exit 0
}

# 执行命令并检查其退出状态，同时记录输出用于调试
run_command() {
    local cmd_desc="$1"
    shift
    log "信息" "${cmd_desc}..."
    if [[ "${DEBUG_MODE}" = true ]]; then
        # 调试模式下，直接输出到屏幕并记录到日志
        "$@" 2>&1 | tee -a "${LOG_FILE}"
        local exit_code=${PIPESTATUS[0]}
    else
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
    fi

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

    # 如果未提供密码，则生成一个随机密码
    if [[ -z "${ROOT_PASSWORD}" ]]; then
        ROOT_PASSWORD=$(head /dev/urandom | tr -dc 'A-Za-z0-9' | head -c 16)
        log "警告" "未提供 root 密码，已生成一个随机密码。"
    fi
    
    log "信息" "开始安装 MySQL ${MYSQL_VERSION}..."
    log "信息" "安装模式: ${MODEL}, 安装路径: ${INSTALL_PATH}, 调试模式: ${DEBUG_MODE}"
    log "信息" "完整的安装日志位于: ${LOG_FILE}"

    # 1. 安装依赖
    run_command "安装基础构建依赖" yum install -y bison ncurses-devel libtirpc-devel openssl-devel cmake gcc gcc-c++
    
    # 在 CentOS 8+ 上编译需要 rpcsvc-proto
    local rpcsvc_tarball="rpcsvc-proto-1.4.tar.gz"
    if [[ "${MODEL}" == "online" ]]; then
        run_command "下载 rpcsvc-proto" wget -c "https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4/${rpcsvc_tarball}" -O "${rpcsvc_tarball}"
    elif [[ ! -f "${rpcsvc_tarball}" ]]; then
        log "错误" "离线模式: 未找到源码包 '${rpcsvc_tarball}'"
        exit 1
    fi
    run_command "解压 rpcsvc-proto" tar -xf "${rpcsvc_tarball}"
    (cd rpcsvc-proto-1.4 && run_command "配置 rpcsvc-proto" ./configure && run_command "编译 rpcsvc-proto" make && run_command "安装 rpcsvc-proto" make install)
    run_command "清理 rpcsvc-proto" rm -rf rpcsvc-proto-1.4 "${rpcsvc_tarball}"

    # 2. 下载并编译 MySQL
    local mysql_tarball="mysql-boost-${MYSQL_VERSION}.tar.gz"
    if [[ "${MODEL}" == "online" ]]; then
        run_command "下载 MySQL ${MYSQL_VERSION}" wget -c "https://dev.mysql.com/get/Downloads/MySQL-5.7/${mysql_tarball}" -O "${mysql_tarball}"
    elif [[ ! -f "${mysql_tarball}" ]]; then
        log "错误" "离线模式: 未找到源码包 '${mysql_tarball}'"
        exit 1
    fi

    run_command "解压 ${mysql_tarball}" tar xf "${mysql_tarball}"
    cd "mysql-${MYSQL_VERSION}"

    local mysql_home="${INSTALL_PATH}/${VERSION_NAME}"
    local mysql_stable_path="${INSTALL_PATH}/mysql" # 稳定的符号链接

    local cmake_args=(
        "-DCMAKE_INSTALL_PREFIX=${mysql_home}"
        "-DSYSCONFDIR=${mysql_home}/etc"
        "-DMYSQL_DATADIR=${mysql_home}/data"
        "-DMYSQL_TCP_PORT=3306"
        "-DDEFAULT_CHARSET=utf8mb4"
        "-DDEFAULT_COLLATION=utf8mb4_general_ci"
        "-DMYSQL_UNIX_ADDR=${mysql_home}/run/mysql.sock"
        "-DWITH_BOOST=./boost"
    )
    run_command "配置 MySQL ${MYSQL_VERSION} (cmake)" cmake "${cmake_args[@]}"

    local cpu_cores; cpu_cores=$(nproc)
    run_command "使用 ${cpu_cores} 核心编译 MySQL" make "-j${cpu_cores}"
    run_command "安装 MySQL" make install
    cd ..
    run_command "清理 MySQL 源码" rm -rf "mysql-${MYSQL_VERSION}" "${mysql_tarball}"

    # 3. 配置 MySQL
    run_command "创建稳定版符号链接于 ${mysql_stable_path}" ln -snf "${mysql_home}" "${mysql_stable_path}"
    
    add_user "mysql"
    mkdir -p "${mysql_stable_path}"/{etc,run,logs,data}

    cat > "${mysql_stable_path}/etc/my.cnf" << END
[mysqld]
user = mysql
port = 3306
bind-address = 0.0.0.0
basedir = ${mysql_stable_path}
datadir = ${mysql_stable_path}/data
socket = ${mysql_stable_path}/run/mysql.sock
pid-file = ${mysql_stable_path}/run/mysqld.pid
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
max_connections = 1024
log-error = ${mysql_stable_path}/logs/mysqld.log
default-time_zone = '+8:00'
lower_case_table_names=1
sql_mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

[client]
socket = ${mysql_stable_path}/run/mysql.sock
default-character-set = utf8mb4
END

    run_command "设置目录所有权" chown -R mysql:mysql "${mysql_home}" "${mysql_stable_path}/data"
    add_firewall_rule "3306/tcp"

    # 4. 初始化和启动
    run_command "初始化 MySQL (无密码)" "${mysql_stable_path}/bin/mysqld" --initialize-insecure --basedir="${mysql_stable_path}" --datadir="${mysql_stable_path}/data" --user=mysql

    cat > /etc/systemd/system/mysqld.service << END
[Unit]
Description=MySQL Server
After=network.target
Wants=network.target

[Service]
User=mysql
Group=mysql
Type=forking
PIDFile=${mysql_stable_path}/run/mysqld.pid
ExecStart=${mysql_stable_path}/bin/mysqld --daemonize --pid-file=${mysql_stable_path}/run/mysqld.pid
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
END

    run_command "重新加载 systemd 配置" systemctl daemon-reload
    run_command "设置开机自启并启动 MySQL 服务" systemctl enable --now mysqld
    
    # 5. 安全设置和路径配置
    log "信息" "正在设置 root 密码..."
    # 使用非交互式、更安全的方式设置密码
    "${mysql_stable_path}/bin/mysql" -u root --connect-expired-password -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '${ROOT_PASSWORD}';"
    # 为了方便，额外创建一个远程访问用户 mw/123 (可按需修改或删除此行)
    "${mysql_stable_path}/bin/mysql" -u root -p"${ROOT_PASSWORD}" -e "GRANT ALL PRIVILEGES ON *.* TO 'mw'@'%' IDENTIFIED BY '123' WITH GRANT OPTION; FLUSH PRIVILEGES;"
    log "信息" "密码设置成功。"

    # 将 MySQL 路径添加到系统 PATH
    cat > /etc/profile.d/mysql.sh << END
export PATH=\$PATH:${mysql_stable_path}/bin
END
    source /etc/profile.d/mysql.sh

    log "信息" "MySQL ${MYSQL_VERSION} 安装完成！"
    log "信息" "安装路径: ${mysql_home}"
    log "信息" "服务名称: mysqld.service"
    log "信息" "root 密码: ${ROOT_PASSWORD}"
    log "信息" "请运行 'source /etc/profile' 或重新登录以使 mysql 命令生效。"
}

# --- 参数解析 ---
while getopts "p:P:m:dh" opt; do
    case ${opt} in
        p) INSTALL_PATH=${OPTARG} ;;
        P) ROOT_PASSWORD=${OPTARG} ;;
        m) MODEL=${OPTARG} ;;
        d) DEBUG_MODE=true ;;
        h) usage ;;
        *) usage ;;
    esac
done

# --- 运行主函数 ---
main
```

