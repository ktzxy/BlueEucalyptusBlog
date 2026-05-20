+++
date = '2026-05-20T09:56:15+08:00'
draft = false
title = 'shell_Mysql_8.x'
slug = "6elqpfh59l"
description = "shell_Mysql_8.x"
summary = "shell_Mysql_8.x"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_Mysql_8.x

```bash
#!/usr/bin/env bash
#
# @sys:      RHEL 8+, CentOS 8+, Rocky 8+, Alma 8+, Debian 10+, Ubuntu 20.04+
# @date:     2025-12-17 (Fix: ID_LIKE unbound variable)
# @author:   yangfan (Modified by Gemini)
# @info:     通过源码安装 MySQL 8.0，自动适配 RHEL 和 Debian 系系统。
# @run:      ./install-mysql8.sh -v 8.0.34 -p /data/mysql -P 'YourPass'

set -euo pipefail

# --- 全局变量与颜色定义 ---
readonly C_RESET='\033[0m'
readonly C_RED='\033[0;31m'
readonly C_GREEN='\033[0;32m'
readonly C_YELLOW='\033[0;33m'

LOG_FILE="/tmp/mysql8_install_$(date +%s).log"
MODEL="online"
INSTALL_PATH="/opt"
ROOT_PASSWORD=""
DEBUG_MODE=false
VERSION_NAME="mysql8"
MYSQL_VERSION="8.0.34"

# 操作系统变量
OS_FAMILY=""  # rhel 或 debian
OS_ID=""      # centos, rhel, ubuntu, debian

# --- 函数定义 ---

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

usage() {
    echo "用法: $0 [-v <版本>] [-p <路径>] [-P <密码>] [-m <模式>] [-d] [-h]"
    echo "  -v  要安装的 MySQL 版本 (例如: 8.0.34)。"
    echo "  -p  安装的基础路径 (默认: /opt)。"
    echo "  -P  MySQL root 用户的密码。"
    echo "  -m  安装模式: 'online' (在线) 或 'offline' (离线)。"
    echo "  -d  开启调试模式。"
    echo "  -h  显示此帮助信息。"
    exit 0
}

# 检测操作系统
check_os() {
    # 初始化局部变量
    local id_like=""
    
    if [ -f /etc/os-release ]; then
        . /etc/os-release
        
        # 使用 :- 防止变量未定义导致脚本退出 (set -u)
        OS_ID="${ID:-unknown}"
        id_like="${ID_LIKE:-}"
        
        # 逻辑判断
        if [[ "$OS_ID" =~ (centos|rhel|rocky|alma) ]] || [[ "$id_like" =~ (centos|rhel|fedora) ]]; then
            OS_FAMILY="rhel"
        elif [[ "$OS_ID" =~ (debian|ubuntu|kali) ]] || [[ "$id_like" =~ (debian|ubuntu) ]]; then
            OS_FAMILY="debian"
        else
            log "错误" "不支持的操作系统: $OS_ID"
            exit 1
        fi
    else
        log "错误" "无法检测操作系统类型 (/etc/os-release 缺失)"
        exit 1
    fi
    log "信息" "检测到操作系统: $OS_ID (家族: $OS_FAMILY)"
}

run_command() {
    local cmd_desc="$1"
    shift
    log "信息" "${cmd_desc}..."
    local exit_code=0
    
    if [[ "${DEBUG_MODE}" = true ]]; then
        "$@" 2>&1 | tee -a "${LOG_FILE}"
        exit_code=${PIPESTATUS[0]}
    else
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
        exit_code=$?
    fi

    if [[ ${exit_code} -ne 0 ]]; then
        log "错误" "${cmd_desc} 失败。日志: ${LOG_FILE}"
        echo "--- 日志最后15行 ---"
        tail -n 15 "${LOG_FILE}"
        echo "----------------------"
        exit 1
    else
        log "信息" "${cmd_desc} --------> 成功"
    fi
}

# 包装器：处理 RHEL SCL 编译器环境与其他系统的差异
run_build_command() {
    local desc="$1"
    shift
    if [[ "$OS_FAMILY" == "rhel" ]]; then
        # RHEL 8 需要 devtoolset/gcc-toolset
        run_command "$desc" scl enable gcc-toolset-10 -- "$@"
    else
        # Debian/Ubuntu 直接运行
        run_command "$desc" "$@"
    fi
}

# --- 主逻辑 ---
main() {
    check_os

    if [[ "$(id -u)" -ne 0 ]]; then
        log "错误" "必须以 root 身份运行。"
        exit 1
    fi

    if [[ -z "${ROOT_PASSWORD}" ]]; then
        ROOT_PASSWORD=$(head /dev/urandom | tr -dc 'A-Za-z0-9!@#%^&*' | head -c 20)
        log "警告" "已生成随机 Root 密码。"
    fi
    
    log "信息" "开始安装 MySQL ${MYSQL_VERSION}..."
    log "信息" "模式: ${MODEL}, 路径: ${INSTALL_PATH}"

    # 1. 安装依赖 (根据系统区分)
    if [[ "$OS_FAMILY" == "rhel" ]]; then
        run_command "安装基础依赖 (RHEL)" yum install -y bison ncurses-devel libtirpc-devel openssl-devel cmake wget diffutils
        run_command "安装 GCC 10 工具集" yum install -y gcc-toolset-10
    elif [[ "$OS_FAMILY" == "debian" ]]; then
        export DEBIAN_FRONTEND=noninteractive
        run_command "更新 apt 缓存" apt-get update
        # Debian 对应的包名
        run_command "安装基础依赖 (Debian)" apt-get install -y build-essential bison libncurses-dev libtirpc-dev libssl-dev cmake wget pkg-config
    fi

    # rpcsvc-proto 处理
    local rpcsvc_tarball="rpcsvc-proto-1.4.tar.gz"
    if [[ "${MODEL}" == "online" ]]; then
        run_command "下载 rpcsvc-proto" wget -c "https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4/${rpcsvc_tarball}" -O "${rpcsvc_tarball}"
    elif [[ ! -f "${rpcsvc_tarball}" ]]; then
        log "错误" "离线模式: 缺少 '${rpcsvc_tarball}'"
        exit 1
    fi
    run_command "解压 rpcsvc-proto" tar -xf "${rpcsvc_tarball}"
    (cd rpcsvc-proto-1.4 && run_command "配置 rpcsvc" ./configure && run_command "编译 rpcsvc" make && run_command "安装 rpcsvc" make install)
    rm -rf rpcsvc-proto-1.4 "${rpcsvc_tarball}"

    # 2. 下载并编译 MySQL
    local mysql_tarball="mysql-boost-${MYSQL_VERSION}.tar.gz"
    if [[ "${MODEL}" == "online" ]]; then
        run_command "下载 MySQL" wget -c "https://dev.mysql.com/get/Downloads/MySQL-8.0/${mysql_tarball}" -O "${mysql_tarball}"
    elif [[ ! -f "${mysql_tarball}" ]]; then
        log "错误" "离线模式: 缺少 '${mysql_tarball}'"
        exit 1
    fi

    run_command "解压 MySQL" tar xf "${mysql_tarball}"
    cd "mysql-${MYSQL_VERSION}"

    local mysql_home="${INSTALL_PATH}/${VERSION_NAME}"
    local mysql_stable_path="${INSTALL_PATH}/mysql"

    local cmake_args=(
        "-DCMAKE_INSTALL_PREFIX=${mysql_home}"
        "-DSYSCONFDIR=${mysql_home}/etc"
        "-DMYSQL_DATADIR=${mysql_home}/data"
        "-DMYSQL_TCP_PORT=3306"
        "-DDEFAULT_CHARSET=utf8mb4"
        "-DDEFAULT_COLLATION=utf8mb4_general_ci"
        "-DMYSQL_UNIX_ADDR=${mysql_home}/run/mysql.sock"
        "-DWITH_BOOST=./boost"
        "-DFORCE_INSOURCE_BUILD=1"
        "-DWITH_SSL=system"
    )

    # 使用封装的构建函数
    run_build_command "配置 MySQL (cmake)" cmake "${cmake_args[@]}"

    local cpu_cores; cpu_cores=$(nproc)
    run_build_command "编译 MySQL (-j${cpu_cores})" make "-j${cpu_cores}"
    run_build_command "安装 MySQL" make install
    
    cd ..
    rm -rf "mysql-${MYSQL_VERSION}" "${mysql_tarball}"

    # 3. 配置与用户
    run_command "创建软链接" ln -snf "${mysql_home}" "${mysql_stable_path}"
    
    id mysql &>/dev/null || run_command "创建 mysql 用户" useradd -r -M -s /sbin/nologin mysql
    mkdir -p "${mysql_stable_path}"/{etc,run,logs,data}

    # my.cnf (通用)
    cat > "${mysql_stable_path}/etc/my.cnf" << END
[mysqld]
user = mysql
port = 3306
bind-address = 0.0.0.0
basedir = ${mysql_stable_path}
datadir = ${mysql_stable_path}/data
socket = ${mysql_stable_path}/run/mysql.sock
pid-file = ${mysql_stable_path}/run/mysqld.pid
log-error = ${mysql_stable_path}/logs/mysqld.log
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
default_authentication_plugin=caching_sha2_password
max_connections = 1024
default-time_zone = '+8:00'
lower_case_table_names=1

[client]
socket = ${mysql_stable_path}/run/mysql.sock
default-character-set = utf8mb4
END

    run_command "设置权限" chown -R mysql:mysql "${mysql_home}" "${mysql_stable_path}/data"

    # 防火墙处理
    handle_firewall() {
        local port="$1"
        if command -v firewall-cmd &> /dev/null && systemctl is-active --quiet firewalld; then
             run_command "Firewalld 放行端口 $port" firewall-cmd --permanent --add-port="${port}/tcp"
             firewall-cmd --reload >/dev/null 2>&1
        elif command -v ufw &> /dev/null && ufw status | grep -q "Status: active"; then
             run_command "UFW 放行端口 $port" ufw allow "${port}/tcp"
        else
             log "警告" "未检测到活跃的防火墙 (firewalld/ufw)，跳过规则配置。"
        fi
    }
    handle_firewall "3306"

    # 4. 初始化与启动
    run_command "初始化数据库" "${mysql_stable_path}/bin/mysqld" --initialize-insecure --basedir="${mysql_stable_path}" --datadir="${mysql_stable_path}/data" --user=mysql

    cat > /etc/systemd/system/mysqld.service << END
[Unit]
Description=MySQL 8.0 Server
After=network.target

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

    run_command "重载 Systemd" systemctl daemon-reload
    run_command "启动 MySQL" systemctl enable --now mysqld
    
    # 5. 安全设置
    log "信息" "设置 Root 密码..."
    "${mysql_stable_path}/bin/mysql" -u root --connect-expired-password -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '${ROOT_PASSWORD}';"
    
    # 额外的远程用户
    "${mysql_stable_path}/bin/mysql" -u root -p"${ROOT_PASSWORD}" -e "CREATE USER IF NOT EXISTS 'mw'@'%' IDENTIFIED WITH caching_sha2_password BY '123'; GRANT ALL PRIVILEGES ON *.* TO 'mw'@'%' WITH GRANT OPTION; FLUSH PRIVILEGES;"

    cat > /etc/profile.d/mysql.sh << END
export PATH=\$PATH:${mysql_stable_path}/bin
END
    
    log "信息" "MySQL 安装完成！"
    log "信息" "Root 密码: ${ROOT_PASSWORD}"
    log "信息" "请执行: source /etc/profile.d/mysql.sh"
}

# --- 参数解析 ---
while getopts "v:p:P:m:dh" opt; do
    case ${opt} in
        v) MYSQL_VERSION=${OPTARG} ;;
        p) INSTALL_PATH=${OPTARG} ;;
        P) ROOT_PASSWORD=${OPTARG} ;;
        m) MODEL=${OPTARG} ;;
        d) DEBUG_MODE=true ;;
        h) usage ;;
        *) usage ;;
    esac
done

main
```

