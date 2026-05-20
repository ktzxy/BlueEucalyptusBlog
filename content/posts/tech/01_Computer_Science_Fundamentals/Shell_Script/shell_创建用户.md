+++
date = '2026-05-20T09:55:43+08:00'
draft = false
title = 'shell_创建用户'
slug = "ldlzexvtvc"
description = "shell_创建用户"
summary = "shell_创建用户"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_创建用户

```bash
#!/usr/bin/env bash
# shellcheck disable=SC2155,SC2034,SC2086
# Description: Batch User Creation Script (Fixed Compatibility)

set -euo pipefail

# ==================== 1. 静态配置 ====================
_user_file="/tmp/users.txt"
_backup_base="/var/backup/batch-create-user"
_timestamp=$(date +"%Y%m%d_%H%M%S")
_today=$(date +%Y%m%d)
_password_prefix="Cxmt@"

_group_meta=(
    "sa.adm:6000:sa.:6001:true"
    "svc.adm:6001:svc.:8001:true"
    "app.adm:6002:app.:9001:false"
)

declare -A _group_alias=(
    ["sa"]="sa.adm"
    ["svc"]="svc.adm"
    ["app"]="app.adm"
)

C_RESET='\033[0m'; C_RED='\033[31m'; C_GREEN='\033[32m'; C_YELLOW='\033[33m'; 
C_BLUE='\033[34m'; C_BOLD='\033[1m'; C_CYAN='\033[36m';

# ==================== 2. 基础工具函数 ====================
log_header() {
    local multi_line=false
    [[ "${1:-}" == "--multi" ]] && { multi_line=true; shift; }
    local title="$*"
    if [[ "$multi_line" == true ]]; then
        echo -e "\n${C_BOLD}${C_BLUE}# $(printf '=%.0s' {1..40})${C_RESET}"
        echo -e "${C_BOLD}${C_BLUE}# ${title}${C_RESET}"
        echo -e "${C_BOLD}${C_BLUE}# $(printf '=%.0s' {1..40})${C_RESET}"
    else
        echo -e "${C_BOLD}${C_CYAN}# ${title} ${C_RESET}"
    fi
}

log_info() { echo -e "${C_YELLOW}[INFO ]${C_RESET} $*"; }
log_error() { echo -e "${C_RED}[ERROR]${C_RESET} $*" >&2; }

run_check() {
    local desc="$1"; shift
    if "$@" >/dev/null 2>&1; then
        echo -e "${C_GREEN}[PASS ]${C_RESET} ${desc} -> ${C_GREEN}OK${C_RESET}"
        return 0
    else
        echo -e "${C_RED}[FAIL ]${C_RESET} ${desc} -> ${C_RED}FAILED${C_RESET}"
        return 1
    fi
}

# ==================== 3. 核心业务函数 ====================
get_max_uid_for_prefix() {
    local prefix="$1"
    awk -F: -v pre="$prefix" '
        length($1) >= length(pre) && substr($1, 1, length(pre)) == pre {
            uid = $3 + 0
            if (uid > max) max = uid
        }
        END { print max+0 }
    ' /etc/passwd
}

get_next_available_uid() {
    local uid=$1
    # 兼容性修复：使用扩展正则表达式获取精确匹配
    local used_uids=$(getent passwd | cut -d: -f3)
    while echo "$used_uids" | grep -qEx "$uid"; do
        ((uid++))
    done
    echo "$uid"
}

append_to_sudo_file() {
    local full_user="$1"
    local prefix="$2"
    local sudo_file=""
    case "$prefix" in
        "sa.")  sudo_file="/etc/sudoers.d/sa_users" ;;
        "svc.") sudo_file="/etc/sudoers.d/svc_users" ;;
        "app.") sudo_file="/etc/sudoers.d/app_users" ;;
        *)      return ;;
    esac
    if [[ ! -f "$sudo_file" ]]; then
        touch "$sudo_file"
        chmod 0440 "$sudo_file"
    fi
    if ! grep -q "^${full_user} " "$sudo_file" 2>/dev/null; then
        echo "${full_user} ALL=(ALL:ALL) ALL" >> "$sudo_file"
    fi
}

# ==================== 4. 初始化 ====================
_backup_dir="$_backup_base/$_timestamp"
mkdir -p "$_backup_dir" && chmod 700 "$_backup_dir"
exec > >(tee -a "$_backup_dir/run.log") 2>&1

log_header --multi "开始批量创建用户任务 (时间: $_timestamp)"

if command -v openssl >/dev/null; then
    _hashed_pass=$(openssl passwd -6 "${_password_prefix}${_today}")
elif command -v mkpasswd >/dev/null; then
    _hashed_pass=$(mkpasswd -m sha-512 "${_password_prefix}${_today}")
else
    log_error "无法生成 SHA512 哈希"
    exit 1
fi

[[ -f "$_user_file" ]] || { log_error "用户文件不存在: $_user_file"; exit 1; }
cp "$_user_file" "$_backup_dir/users.txt.bak"
mapfile -t _raw_users < <(grep -v '^[[:space:]]*$' "$_user_file" | tr '[:upper:]' '[:lower:]')

# ==================== 5. 策略解析 ====================
log_header "1. 解析目标组策略"
_input_group_list="${TARGET_GROUPS:-app,svc}"
IFS=',' read -r -a _input_groups_arr <<< "$_input_group_list"
declare -A _target_group_map
for g in "${_input_groups_arr[@]}"; do
    g="${g// /}"
    if [[ -n "${_group_alias[$g]+x}" ]]; then _target_group_map["${_group_alias[$g]}"]=1
    elif [[ " ${_group_meta[*]} " == *"${g}:"* ]]; then _target_group_map["$g"]=1
    else log_error "无效组: '$g'"; exit 1; fi
done

# ==================== 6. 环境预热 ====================
log_header "2. 准备运行环境"
declare -A _current_uids
for meta in "${_group_meta[@]}"; do
    IFS=':' read -r gname gid prefix start_uid sudo_flag <<< "$meta"
    [[ -z "${_target_group_map[$gname]+x}" ]] && continue

    if ! getent group "$gname" >/dev/null; then
        # 尝试创建组，如果 GID 冲突则提示
        if ! groupadd -g "$gid" "$gname" 2>/dev/null; then
            log_error "无法创建组 $gname (可能 GID $gid 已被占用)。"
            # 如果组创建失败，从目标地图中移除，防止后面用户创建报错
            unset _target_group_map["$gname"]
            continue
        fi
        log_info "成功创建用户组 $gname (GID: $gid)"
    fi

    _max_in_passwd=$(get_max_uid_for_prefix "$prefix")
    _current_uids["$gname"]=$(( _max_in_passwd == 0 ? start_uid : _max_in_passwd + 1 ))
done

# ==================== 7. 核心执行 ====================
log_header "3. 执行用户批量创建"
for user in "${_raw_users[@]}"; do
    for meta in "${_group_meta[@]}"; do
        IFS=':' read -r gname gid prefix start_uid sudo_flag <<< "$meta"
        [[ -z "${_target_group_map[$gname]+x}" ]] && continue
        
        full_user="${prefix}${user}"
        if id "$full_user" &>/dev/null; then
            log_info "用户 $full_user 已经存在，跳过。"
        else
            next_uid=$(get_next_available_uid "${_current_uids["$gname"]}")
            if useradd -m -u "$next_uid" -g "$gname" -s /bin/bash "$full_user" 2>/dev/null; then
                echo "$full_user:$_hashed_pass" | chpasswd -e
                echo -e "${C_GREEN}[PASS ]${C_RESET} 创建用户 $full_user (UID: $next_uid)"
                _current_uids["$gname"]=$((next_uid + 1))
            else
                log_error "用户 $full_user 创建失败（检查组 $gname 是否存在）。"
                continue
            fi
        fi
        [[ "$sudo_flag" == "true" ]] && append_to_sudo_file "$full_user" "$prefix"
    done
done

log_header "4. 任务完成"
log_info "备份目录: $_backup_dir"
```

