+++
date = '2026-05-20T09:55:54+08:00'
draft = false
title = 'shell_生成证书'
slug = "5lxb6hpiay"
description = "shell_生成证书"
summary = "shell_生成证书"
categories = ["📒shell"]
tags = ["shell"]
comments = true
+++

# shell_生成证书

### ./generate_cert.sh

```bash
#!/bin/bash
#
# ===================================================================================
# Generic Self-Signed Certificate Generator
#
# 功能:
#   - 生成包含 SAN (Subject Alternative Name) 的自签名SSL证书。
#   - 支持自定义多个域名和IP地址。
#   - 自动生成文件名，并提供覆盖前确认。
#   - 提供清晰、带颜色的日志输出。
#
# ===================================================================================
# 遇到任何错误则立即退出
set -e

# ========================= 1. 配置区域 =========================

# --- 证书基本信息 ---
# 证书的主域名 (Common Name)
PRIMARY_DOMAIN="registry.example.com"

# 证书包含的其他域名或IP地址 (Subject Alternative Names)
# 如果没有，请保留括号为空: ALT_NAMES=()
ALT_NAMES=(
    "www.registry.example.com"
    "dev.registry.example.com"
    "192.168.1.100"
)

# 生成的证书和私钥存放目录
CERT_DIR="./certs"
# 证书有效期 (天)
DAYS_VALID=3650
# 私钥位数
KEY_SIZE=2048

# --- 证书所有者信息 (可按需修改) ---
COUNTRY="CN"
STATE="Shanxi"
CITY="Xi'an"
ORGANIZATION="My Example Corp"
ORG_UNIT="IT Department"


# ========================= 2. 初始化和函数定义 =========================

# --- 颜色定义 和 日志函数 ---
C_RESET='\033[0m'; C_GREEN='\033[32m'; C_YELLOW='\033[33m'; C_RED='\033[31m'; C_BLUE='\033[34m'
log_info()  { echo -e "${C_GREEN}[INFO] $1${C_RESET}"; }
log_warn()  { echo -e "${C_YELLOW}[WARN] $1${C_RESET}"; }
log_error() { echo -e "${C_RED}[ERROR] $1${C_RESET}"; }
log_header(){ echo -e "${C_BLUE}==================== $1 ====================${C_RESET}"; }
abort()     { log_error "$1"; exit 1; }

# 临时文件清理函数
TMP_FILES=()
cleanup() {
    if [ ${#TMP_FILES[@]} -gt 0 ]; then
        log_info "正在清理临时文件..."
        rm -f "${TMP_FILES[@]}"
    fi
}
trap cleanup EXIT INT TERM


# ========================= 3. 主程序 =========================

log_header "启动自签名证书生成任务"

# --- 步骤 0: 检查依赖和环境 ---
if ! command -v openssl &> /dev/null; then
    abort "依赖命令 'openssl' 未找到，请先安装。"
fi

KEY_FILE="${CERT_DIR}/${PRIMARY_DOMAIN}.key"
CERT_FILE="${CERT_DIR}/${PRIMARY_DOMAIN}.crt"

if [ -f "$KEY_FILE" ] || [ -f "$CERT_FILE" ]; then
    log_warn "在目标目录 '${CERT_DIR}' 中已发现同名证书文件。"
    read -p "您确定要覆盖吗? (y/N): " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        log_info "操作已取消。"
        exit 0
    fi
fi

# --- 步骤 1: 准备工作 ---
log_info "准备工作：创建证书目录..."
mkdir -p "$CERT_DIR"
log_info "证书将保存在: $(realpath "$CERT_DIR")"

# --- 步骤 2: 生成私钥 ---
log_header "步骤 1: 生成私钥"
openssl genrsa -out "${KEY_FILE}" "${KEY_SIZE}" || abort "私钥生成失败！"
log_info "私钥已生成: ${KEY_FILE}"

# --- 步骤 3: 动态创建CSR配置文件 ---
log_header "步骤 2: 创建证书签名请求 (CSR) 配置文件"
# 使用 mktemp 创建安全的临时文件
TMP_CNF=$(mktemp)
TMP_FILES+=("$TMP_CNF")

cat > "$TMP_CNF" <<EOF
[req]
default_bits = ${KEY_SIZE}
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = v3_req

[dn]
C = ${COUNTRY}
ST = ${STATE}
L = ${CITY}
O = ${ORGANIZATION}
OU = ${ORG_UNIT}
CN = ${PRIMARY_DOMAIN}

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = ${PRIMARY_DOMAIN}
EOF

# 动态添加 SANs
dns_count=2
ip_count=1
for name in "${ALT_NAMES[@]}"; do
    if [[ $name =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
        echo "IP.${ip_count} = ${name}" >> "$TMP_CNF"
        ((ip_count++))
    else
        echo "DNS.${dns_count} = ${name}" >> "$TMP_CNF"
        ((dns_count++))
    fi
done
log_info "CSR 配置文件已动态创建。"

# --- 步骤 4: 生成证书签名请求 (CSR) ---
log_header "步骤 3: 生成证书签名请求 (CSR)"
TMP_CSR=$(mktemp)
TMP_FILES+=("$TMP_CSR")
openssl req -new -key "${KEY_FILE}" -out "${TMP_CSR}" -config "${TMP_CNF}" || abort "CSR 文件生成失败！"
log_info "CSR 文件已生成。"

# --- 步骤 5: 签署证书 ---
log_header "步骤 4: 签署并生成自签名证书"
openssl x509 -req -days "${DAYS_VALID}" -in "${TMP_CSR}" -signkey "${KEY_FILE}" -out "${CERT_FILE}" -extensions v3_req -extfile "${TMP_CNF}" || abort "证书签署失败！"
log_info "自签名证书已生成: ${CERT_FILE}"

# --- 步骤 6: 验证证书 ---
log_header "步骤 5: 验证生成的证书信息"
openssl x509 -in "${CERT_FILE}" -text -noout

log_header "任务完成"
log_info "证书生成完毕！以下文件已保存在 '${CERT_DIR}' 目录中:"
log_info "  - 私钥: $(basename "${KEY_FILE}")"
log_info "  - 证书: $(basename "${CERT_FILE}")"
#  trap 会自动执行 cleanup 函数来删除临时文件
exit 0
```
