+++
date = '2026-05-20T09:55:30+08:00'
draft = false
title = 'shell_阿里云DDNS'
slug = "7c0hrvu2f6"
description = "shell_阿里云DDNS"
summary = "shell_阿里云DDNS"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_阿里云DDNS

### ./ddns-aliyun.sh

```bash
#!/bin/bash
#
# ===================================================================================
# Aliyun DDNS Master Script (Generic Version)
#
# 功能:
#   - 自动更新阿里云DNS记录，使其指向目标节点当前的公网IP地址。
#   - 支持同时更新多个子域名。
#   - 支持IPv4(A)和IPv6(AAAA)记录。
#   - 能够自动清理重复的DNS记录。
#   - 启动时自动检查依赖项，提供彩色日志和最终执行摘要。
#
# 用法 (Cron):
#   */5 * * * * /path/to/this_script.sh > /dev/null 2>&1
#
# ===================================================================================

# ========================= 1. 配置区域 =========================

# --- 域名配置 ---
DOMAIN="example.com"
SUBDOMAINS=(
    "data"
    "gitlab"
    "registry"
)

# --- IP 配置 ---
IP_TYPE="AAAA" # IP类型: A (IPv4) 或 AAAA (IPv6)
# 当IP_TYPE为AAAA时，用于获取IPv6地址的网卡名称
IPV6_INTERFACE="enp1s0"

# ========================= 2. 初始化和函数定义 =========================

# 为cron等最小化环境设置一个可靠的PATH
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# --- 颜色定义 ---
C_RESET='\033[0m'; C_GREEN='\033[32m'; C_YELLOW='\033[33m'; C_RED='\033[31m'; C_BLUE='\033[34m'

# --- 日志函数 ---
log_info()  { echo -e "${C_GREEN}[$(date "+%Y-%m-%d %H:%M:%S")] [INFO] $1${C_RESET}"; }
log_warn()  { echo -e "${C_YELLOW}[$(date "+%Y-%m-%d %H:%M:%S")] [WARN] $1${C_RESET}"; }
log_error() { echo -e "${C_RED}[$(date "+%Y-%m-%d %H:%M:%S")] [ERROR] $1${C_RESET}"; }
log_header(){ echo -e "${C_BLUE}==================== $1 ====================${C_RESET}"; }
abort()     { log_error "$1"; exit 1; }

# --- 依赖项检查 ---
check_dependencies() {
    log_info "正在检查依赖项..."
    local dependencies=("aliyun" "jq" "ip" "curl")
    for cmd in "${dependencies[@]}"; do
        if ! command -v "$cmd" &> /dev/null; then
            abort "依赖命令 '$cmd' 未找到，请先安装。"
        fi
    done
    ALIYUN_CMD=$(command -v aliyun)
}

# --- IP 获取函数 ---
get_current_ip() {
    if [ "$IP_TYPE" == "A" ]; then
        # 获取 IPv4 地址
        curl -s http://v4.ipv6-test.com/api/myip.php
    else
        # 获取 IPv6 地址
        ip -6 addr show dev "${IPV6_INTERFACE}" scope global | grep 'inet6' | awk '{print $2}' | cut -d'/' -f1 | head -n 1
    fi
}

# --- 核心更新函数 ---
update_record() {
    local subdomain=$1
    local ip_to_set=$2
    local ddns_domain="${subdomain}.${DOMAIN}"
    local ttl=600

    log_info "正在处理子域名: ${ddns_domain}"

    local records_json
    records_json=$(sudo ${ALIYUN_CMD} alidns DescribeDomainRecords --DomainName "${DOMAIN}" --RRKeyWord "${subdomain}" --Type "${IP_TYPE}")

    if ! echo "${records_json}" | jq . > /dev/null 2>&1; then
        log_error "从阿里云获取 '${ddns_domain}' 记录失败，请检查网络或配置。"
        return 2 # 返回失败状态
    fi

    local record_count
    record_count=$(echo "${records_json}" | jq -r '.DomainRecords.Record | length')

    # 如果发现重复记录，则全部删除
    if [[ ${record_count} -gt 1 ]]; then
        log_warn "发现 '${ddns_domain}' 存在 ${record_count} 条重复的 ${IP_TYPE} 记录，将全部删除后重新添加。"
        local record_ids
        record_ids=$(echo "${records_json}" | jq -r '.DomainRecords.Record[].RecordId')
        local cleanup_failed=false
        for record_id in $record_ids; do
            sudo ${ALIYUN_CMD} alidns DeleteDomainRecord --RecordId "${record_id}"
            if [[ $? -ne 0 ]]; then cleanup_failed=true; fi
        done

        if [[ "$cleanup_failed" == "true" ]]; then
            log_error "清理 '${ddns_domain}' 的重复记录时发生错误，中止该任务。"
            return 2
        else
            log_info "所有重复记录已成功清理。"
            record_count=0 # 重置计数，强制执行添加逻辑
        fi
    fi

    # 根据记录数决定是“添加”还是“更新”
    if [[ ${record_count} -eq 0 ]]; then
        log_info "记录不存在或已清理，准备添加..."
        sudo ${ALIYUN_CMD} alidns AddDomainRecord --DomainName "${DOMAIN}" --RR "${subdomain}" --Type "${IP_TYPE}" --Value "${ip_to_set}" --TTL "${ttl}"
        if [ $? -eq 0 ]; then log_info "添加 '${ddns_domain}' 成功 -> ${ip_to_set}"; return 1; else log_error "添加 '${ddns_domain}' 失败！"; return 2; fi
    else
        local current_ip record_id
        current_ip=$(echo "${records_json}" | jq -r '.DomainRecords.Record[0].Value')
        record_id=$(echo "${records_json}" | jq -r '.DomainRecords.Record[0].RecordId')

        if [ "$current_ip" == "$ip_to_set" ]; then
            log_info "IP地址一致 (${current_ip})，无需更新。"
            return 0 # 返回跳过状态
        else
            log_info "IP不一致。当前: ${current_ip} -> 目标: ${ip_to_set}。准备更新..."
            sudo ${ALIYUN_CMD} alidns UpdateDomainRecord --RecordId "${record_id}" --RR "${subdomain}" --Type "${IP_TYPE}" --Value "${ip_to_set}" --TTL "${ttl}"
            if [ $? -eq 0 ]; then log_info "更新 '${ddns_domain}' 成功"; return 1; else log_error "更新 '${ddns_domain}' 失败！"; return 2; fi
        fi
    fi
}

# ========================= 3. 主程序 =========================

log_header "DDNS 任务启动"
check_dependencies

# --- 步骤1: 获取目标IP ---
log_info "正在获取目标IP地址 (${IP_TYPE})..."
current_ip=$(get_current_ip)
if [ -z "$current_ip" ]; then
    abort "获取目标IP地址失败，脚本终止。"
fi
log_info "获取到目标IP: ${current_ip}"

# --- 步骤2: 遍历子域名并执行更新 ---
updated_count=0
skipped_count=0
failed_count=0

for sub in "${SUBDOMAINS[@]}"; do
    update_record "${sub}" "${current_ip}"
    case $? in
        0) skipped_count=$((skipped_count + 1));;
        1) updated_count=$((updated_count + 1));;
        2) failed_count=$((failed_count + 1));;
    esac
done

# --- 步骤3: 打印最终摘要 ---
log_header "任务摘要"
log_info "更新/添加: ${updated_count}"
log_info "无需更新: ${skipped_count}"
if [ ${failed_count} -gt 0 ]; then
    log_error "失败: ${failed_count}"
else
    log_info "失败: ${failed_count}"
fi
log_header "DDNS 任务结束"

exit ${failed_count}

```

### ./setup_aliyun_cli.sh

```bash
#!/bin/bash
#
# ===================================================================================
# Aliyun CLI Installer & Verifier
#
# 功能:
#   - 检查系统中是否存在 'aliyun' 命令行工具。
#   - 如果不存在，则自动下载并执行官方安装脚本进行安装。
#   - 本脚本为一次性环境准备工具，用于为其他脚本（如DDNS）配置好依赖。
#
# 用法:
#   bash setup_aliyun_cli.sh
# ===================================================================================

# --- 颜色定义 和 日志函数 ---
C_RESET='\033[0m'
C_GREEN='\033[32m'
C_YELLOW='\033[33m'
C_RED='\033[31m'

log_info()  { echo -e "${C_GREEN}[INFO] $1${C_RESET}"; }
log_warn()  { echo -e "${C_YELLOW}[WARN] $1${C_RESET}"; }
log_error() { echo -e "${C_RED}[ERROR] $1${C_RESET}"; }
abort()     { log_error "$1"; exit 1; }

# ========================= 主程序 =========================

log_info "开始检查 Aliyun CLI 环境..."

# 1. 检查 'aliyun' 命令是否已安装
if command -v aliyun &> /dev/null; then
    log_info "检测成功: Aliyun CLI 已安装在 $(command -v aliyun)"
    log_info "您现在可以运行 'sudo $(command -v aliyun) configure' 来配置它 (如果尚未配置)。"
    exit 0
fi

# 2. 如果未安装，则准备安装
log_warn "Aliyun CLI 命令未找到，准备开始自动安装..."

# 检查依赖: curl
if ! command -v curl &> /dev/null; then
    abort "安装前置依赖检查失败: 'curl' 命令未找到，请先安装它 (例如: sudo yum install curl)。"
fi

# 检查依赖: sudo
if ! command -v sudo &> /dev/null; then
    abort "安装前置依赖检查失败: 'sudo' 命令未找到。安装过程需要sudo权限。"
fi

log_info "正在使用官方脚本下载并安装 (此过程可能需要您输入sudo密码)..."

# 3. 执行安装
# 使用 sudo bash -c '...' 来确保整个下载和执行过程在 root 权限下
sudo bash -c "$(curl -fsSL https://aliyuncli.alicdn.com/install.sh)"

# 4. 再次检查，验证安装结果
if command -v aliyun &> /dev/null; then
    log_info "安装成功！"
    log_info "Aliyun CLI 已安装到: $(command -v aliyun)"
    log_warn "下一步关键操作:"
    log_warn "  请立即执行以下命令来配置您的 AccessKey:"
    echo # 增加空行以突出显示命令
    echo "    sudo $(command -v aliyun) configure"
    echo
else
    log_error "安装失败！"
    log_error "脚本执行完毕后未能找到 'aliyun' 命令。"
    log_error "请尝试访问官网手动安装: https://github.com/aliyun/aliyun-cli"
    exit 1
fi

exit 0
```
