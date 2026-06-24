+++
date = '2026-06-24T09:01:38+08:00'
draft = false
title = 'Linux环境下 Oracle备份'
slug = "6jx5ffo1jw"
description = "Linux环境下 Oracle备份"
summary = "Linux环境下 Oracle备份"
categories = ["📒数据库"]
tags = ["Oracle"]
+++

# Linux环境下 Oracle备份


## 第一章：Oracle 备份基础知识

### 1.1 逻辑备份 vs 物理备份

| 对比维度 | 逻辑备份 (expdp)                   | 物理备份 (RMAN)                |
| :------- | :--------------------------------- | :----------------------------- |
| 备份内容 | 数据库对象（表、索引、存储过程等） | 数据文件、控制文件、归档日志   |
| 恢复粒度 | 可恢复到表级                       | 恢复到库级/表空间级            |
| 恢复速度 | 慢（需要重新执行SQL）              | 快（直接替换文件块）           |
| 适用场景 | 跨版本迁移、小规模数据导出         | 灾难恢复、RPO<15分钟的生产环境 |
| 跨平台   | 支持                               | 不支持（需配合传输表空间）     |
| 增量支持 | 不支持真正的增量                   | 支持 Level 0/1 增量备份        |

**生产建议**：两者结合使用。RMAN 作为主备份方案（保障 RPO/RTO），expdp 作为补充（跨版本容灾、逻辑验证）。

### 1.2 RMAN 备份层级

```
Level 0（全量备份）
  └── 备份所有使用过的数据块
  └── 相当于一个"基准点"

Level 1（增量备份）
  ├── 差异增量（Differential）：备份自上次 Level 0 或 Level 1 以来变化的块（默认）
  └── 累积增量（Cumulative）：备份自上次 Level 0 以来所有变化的块
```

**典型调度策略**：

```
周日：Level 0 全量
周一 ~ 周六：Level 1 差异增量
每天：每隔 1-4 小时备份归档日志
每天：备份控制文件和参数文件
```

### 1.3 CDB/PDB 备份模型（12c+）

Oracle 12c 引入了多租户架构（Multitenant），一个 CDB（Container Database）下可以挂载多个 PDB（Pluggable Database）。

备份策略要点：

- CDB 级别备份

  ：备份整个容器，包括所有 PDB 的数据

- PDB 级别备份

  ：可以单独备份某个 PDB

- 归档日志

  ：CDB 级别统一管理，不分 PDB

- 控制文件

  ：CDB 只有一套控制文件

⚠️ 注意：12c 以上版本的 RMAN 默认连接的是 CDB$ROOT，备份会自动包含所有 PDB。如果只想备份单个 PDB，需要指定 `PLUGGABLE DATABASE` 子句。

### 1.4 备份保留策略

```
-- 保留最近 7 天的备份
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;

-- 或者：保留最近 3 份冗余备份
CONFIGURE RETENTION POLICY TO REDUNDANCY 3;

-- 查看过期备份
RMAN> REPORT OBSOLETE;

-- 删除过期备份
RMAN> DELETE OBSOLETE;
```

### 1.5 异地备份与 3-2-1 原则

**3-2-1 原则**：

- 至少 **3** 份数据副本
- 存放在 **2** 种不同介质上
- 至少 **1** 份存放在异地

本方案通过 FTP/SFTP 将备份文件传输到异地存储，满足 3-2-1 原则的异地要求。

## 第二章：环境准备

### 2.1 目录规划

```
# 备份相关目录
BACKUP_BASE=/backup/oracle
LOG_DIR=${BACKUP_BASE}/logs          # 日志目录
EXPDP_DIR=${BACKUP_BASE}/expdp       # 逻辑备份存放目录
RMAN_DIR=${BACKUP_BASE}/rman         # 物理备份存放目录
ARCH_DIR=${BACKUP_BASE}/arch         # 归档日志备份目录
SCRIPT_DIR=${BACKUP_BASE}/scripts    # 脚本存放目录

# 创建目录
mkdir -p ${LOG_DIR} ${EXPDP_DIR} ${RMAN_DIR} ${ARCH_DIR} ${SCRIPT_DIR}
chown -R oracle:oinstall /backup/oracle
chmod -R 755 /backup/oracle
```

### 2.2 Oracle 目录对象（逻辑备份用）

```
-- 以 sysdba 登录
sqlplus / as sysdba

-- 创建目录对象
CREATE OR REPLACE DIRECTORY BACKUP_DIR AS '/backup/oracle/expdp';
CREATE OR REPLACE DIRECTORY BACKUP_LOG AS '/backup/oracle/logs';

-- 授权
GRANT READ, WRITE ON DIRECTORY BACKUP_DIR TO system;
GRANT READ, WRITE ON DIRECTORY BACKUP_LOG TO system;
```

### 2.3 统一配置文件

所有脚本使用统一的配置文件，便于维护：

```
cat > /backup/oracle/scripts/oracle_backup.conf << 'EOF'
#=============================================
# Oracle 备份全局配置
#=============================================

# --- 基本参数 ---
ORACLE_SID=orcl
ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
ORACLE_BASE=/u01/app/oracle
export ORACLE_SID ORACLE_HOME ORACLE_BASE
export PATH=${ORACLE_HOME}/bin:${PATH}
export LD_LIBRARY_PATH=${ORACLE_HOME}/lib:${LD_LIBRARY_PATH}

# --- RAC 环境（单机环境留空）---
RAC_INSTANCES="orcl1 orcl2"    # RAC 多实例，空格分隔
IS_RAC=false                    # true=RAC, false=单机

# --- 12c+ 多租户 ---
IS_CDB=true                     # true=CDB模式, false=非CDB
CDB_SID=orcl                    # CDB 实例名
PDB_NAME=                       # 留空=备份整个CDB，填名=备份指定PDB

# --- 备份路径 ---
BACKUP_BASE=/backup/oracle
EXPDP_DIR=${BACKUP_BASE}/expdp
RMAN_DIR=${BACKUP_BASE}/rman
ARCH_DIR=${BACKUP_BASE}/arch
LOG_DIR=${BACKUP_BASE}/logs

# --- 保留天数 ---
RETENTION_DAYS=7

# --- 远程传输 ---
REMOTE_TYPE=sftp                # ftp 或 sftp
REMOTE_HOST=192.168.1.100
REMOTE_PORT=22
REMOTE_USER=backup_user
REMOTE_PASS=                    # 密码方式（不推荐）
REMOTE_KEY=/home/oracle/.ssh/id_rsa
REMOTE_DIR=/backup/oracle_remote
BANDWIDTH_LIMIT=0               # 限速KB/s，0=不限速

# --- 通知 ---
NOTIFY_EMAIL=dba@example.com
NOTIFY_ENABLE=true
EOF

chmod 600 /backup/oracle/scripts/oracle_backup.conf
chown oracle:oinstall /backup/oracle/scripts/oracle_backup.conf
```

⚠️ 配置文件权限设为 600，仅 oracle 用户可读，防止泄露密码。

## 第三章：逻辑备份 — expdp 全库导出 + 远程传输

### 3.1 expdp 核心参数速查

| 参数            | 说明           | 推荐值                          |
| :-------------- | :------------- | :------------------------------ |
| FULL            | 全库导出       | y                               |
| DIRECTORY       | 目录对象       | BACKUP_DIR                      |
| DUMPFILE        | 导出文件名     | full_%U.dmp（%U自动编号）       |
| PARALLEL        | 并行度         | 4（需Enterprise Edition）       |
| COMPRESSION     | 压缩           | ALL（体积最小）                 |
| FILESIZE        | 单文件最大大小 | 30G                             |
| FLASHBACK_TIME  | 一致性导出     | SYSTIMESTAMP                    |
| CONTENT         | 导出内容       | ALL / DATA_ONLY / METADATA_ONLY |
| REUSE_DUMPFILES | 覆盖同名文件   | y                               |

### 3.2 expdp 全库备份脚本

```
cat > /backup/oracle/scripts/expdp_full_backup.sh << 'SCRIPT'
#!/bin/bash
#=============================================================
# 脚本：expdp_full_backup.sh
# 功能：Oracle expdp 全库逻辑备份，支持单机/RAC/12c+ PDB
# 版本：10g / 11g / 12c / 19c / 21c
# 调度：每周执行 1-2 次（如周日、周三凌晨 2:00）
#=============================================================

set -euo pipefail

#--- 加载配置 ---
CONF_FILE="/backup/oracle/scripts/oracle_backup.conf"
[[ ! -f "${CONF_FILE}" ]] && { echo "[ERROR] 配置不存在"; exit 1; }
source "${CONF_FILE}"

#--- 基础变量 ---
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DATE_TAG=$(date +%Y%m%d)
LOG_FILE="${LOG_DIR}/expdp_full_${TIMESTAMP}.log"
LOCK_FILE="/tmp/expdp_full_backup.lock"

log_info()  { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" | tee -a "${LOG_FILE}"; }
log_error() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "${LOG_FILE}" >&2; }

#--- 防重复锁 ---
cleanup() { rm -f "${LOCK_FILE}"; }
trap cleanup EXIT

if [[ -f "${LOCK_FILE}" ]]; then
    PID=$(cat "${LOCK_FILE}")
    if kill -0 "${PID}" 2>/dev/null; then
        log_error "另一个实例正在运行 (PID: ${PID})"
        exit 1
    fi
    rm -f "${LOCK_FILE}"
fi
echo $$ > "${LOCK_FILE}"

#--- 磁盘空间检查 ---
check_disk_space() {
    local dir=$1 required_gb=$2
    local avail_gb=$(( $(df -k "${dir}" | awk 'NR==2{print $4}') / 1024 / 1024 ))
    (( avail_gb < required_gb )) && { log_error "磁盘不足: 需要${required_gb}G, 可用${avail_gb}G"; return 1; }
    log_info "磁盘检查通过: 可用${avail_gb}G"
}

check_disk_space "${EXPDP_DIR}" 20

#--- 确定参数 ---
DUMP_PREFIX="full_${DATE_TAG}"
LOG_NAME="expdp_full_${DATE_TAG}.log"

if [[ "${IS_CDB}" == "true" && -n "${PDB_NAME}" ]]; then
    log_info "模式：PDB 级别导出 PDB=${PDB_NAME}"
    DUMP_PREFIX="pdb_${PDB_NAME}_${DATE_TAG}"
    CONN_STR="system/${SYSTEM_PASS}@${CDB_SID}_${PDB_NAME}"
else
    log_info "模式：全库导出"
    CONN_STR="system/${SYSTEM_PASS} as sysdba"
fi

#--- 执行 expdp ---
log_info "执行 expdp 导出..."

expdp "${CONN_STR}" \
    FULL=y \
    DIRECTORY=BACKUP_DIR \
    DUMPFILE="${DUMP_PREFIX}_%U.dmp" \
    LOGFILE="${LOG_NAME}" \
    PARALLEL=4 \
    COMPRESSION=ALL \
    CONTENT=ALL \
    FILESIZE=30G \
    REUSE_DUMPFILES=y \
    FLASHBACK_TIME="SYSTIMESTAMP" \
    METRICS=yes \
    >> "${LOG_FILE}" 2>&1

[[ $? -ne 0 ]] && { log_error "expdp 导出失败"; exit 1; }
log_info "expdp 导出完成"

#--- 压缩打包 ---
DUMP_TOTAL_SIZE=$(du -sh "${EXPDP_DIR}/${DUMP_PREFIX}_"*.dmp | tail -1 | awk '{print $1}')
ARCHIVE_FILE="${EXPDP_DIR}/${DUMP_PREFIX}.tar.gz"
cd "${EXPDP_DIR}"
tar czf "${ARCHIVE_FILE}" "${DUMP_PREFIX}_"*.dmp "${LOG_NAME}" 2>/dev/null
ARCHIVE_SIZE=$(du -sh "${ARCHIVE_FILE}" | awk '{print $1}')
log_info "压缩包大小: ${ARCHIVE_SIZE}"

#--- SFTP 上传 ---
if [[ -n "${REMOTE_HOST}" && "${REMOTE_TYPE}" == "sftp" && -n "${REMOTE_KEY}" ]]; then
    log_info "上传到 ${REMOTE_HOST}..."
    SFTP_BATCH=$(mktemp)
    cat > "${SFTP_BATCH}" << SFTP_EOF
mkdir ${REMOTE_DIR}
cd ${REMOTE_DIR}
mkdir ${DATE_TAG}
cd ${DATE_TAG}
put "${ARCHIVE_FILE}"
put "${LOG_DIR}/${LOG_NAME}"
bye
SFTP_EOF

    sftp -o StrictHostKeyChecking=no -o ConnectTimeout=30 \
         -i "${REMOTE_KEY}" -P "${REMOTE_PORT}" \
         -b "${SFTP_BATCH}" \
         "${REMOTE_USER}@${REMOTE_HOST}" >> "${LOG_FILE}" 2>&1

    [[ $? -eq 0 ]] && log_info "上传成功" || log_error "上传失败"
    rm -f "${SFTP_BATCH}"
fi

#--- 清理过期 ---
find "${EXPDP_DIR}" -name "full_*.dmp" -mtime +${RETENTION_DAYS} -delete 2>/dev/null
find "${EXPDP_DIR}" -name "full_*.tar.gz" -mtime +${RETENTION_DAYS} -delete 2>/dev/null
find "${EXPDP_DIR}" -name "pdb_*.dmp" -mtime +${RETENTION_DAYS} -delete 2>/dev/null
log_info "过期备份清理完成"

#--- 邮件通知 ---
if [[ "${NOTIFY_ENABLE}" == "true" && -n "${NOTIFY_EMAIL}" ]]; then
    echo "expdp 全库备份完成 | 主机:$(hostname) | 大小:${ARCHIVE_SIZE} | 时间:$(date)" \
        | mail -s "[SUCCESS] Oracle expdp备份完成 - ${DATE_TAG}" "${NOTIFY_EMAIL}"
fi

log_info "expdp 全库备份完成"
SCRIPT

chmod +x /backup/oracle/scripts/expdp_full_backup.sh
chown oracle:oinstall /backup/oracle/scripts/expdp_full_backup.sh
```

### 3.3 脚本要点解析

**① 一致性保证** — `FLASHBACK_TIME="SYSTIMESTAMP"`

确保导出的数据在时间点上一致。没有这个参数，并发事务可能导致表间数据不一致。

**② 并行导出** — `PARALLEL=4` + `DUMPFILE=full_%U.dmp`

`%U` 自动编号为 01, 02, 03...，4 个进程同时导出。注意：需要 Enterprise Edition 许可。

**③ 压缩** — `COMPRESSION=ALL`

压缩元数据+数据，通常可节省 60%-80% 存储空间。

**④ 锁机制** — 防止多个备份实例同时运行，自动处理残留锁文件。

## 第四章：物理备份 — RMAN 全量 + 增量 + 归档

### 4.1 RMAN 核心概念

| 概念                   | 说明                                         |
| :--------------------- | :------------------------------------------- |
| 备份集（Backup Set）   | RMAN 原生格式，包含备份片，只能被 RMAN 读取  |
| 镜像副本（Image Copy） | 数据文件逐块拷贝，可被 Oracle 直接使用       |
| 差异增量               | 备份自上次同级或更高级别以来变化的块（默认） |
| 累加增量               | 备份自上次 Level 0 以来所有变化的块          |
| BCT                    | 变更跟踪文件，加速增量备份（必须启用）       |

**启用 BCT（强烈推荐）**：

```
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING
  USING FILE '/u01/app/oracle/change_tracking.f';
```

### 4.2 RMAN 全库备份脚本（Level 0 + Level 1）

```
cat > /backup/oracle/scripts/rman_full_backup.sh << 'SCRIPT'
#!/bin/bash
#=============================================================
# 脚本：rman_full_backup.sh
# 功能：RMAN 全库物理备份 Level0/Level1
#       支持单机/RAC，支持10g/12c+CDB/PDB
# 用法：./rman_full_backup.sh [level0|level1]
# 调度：Level0 每周日，Level1 周一至周六
#=============================================================

set -euo pipefail
source "/backup/oracle/scripts/oracle_backup.conf"

BACKUP_LEVEL="${1:-level1}"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DATE_TAG=$(date +%Y%m%d)
LOG_FILE="${LOG_DIR}/rman_${BACKUP_LEVEL}_${TIMESTAMP}.log"
LOCK_FILE="/tmp/rman_backup.lock"

log_info()  { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" | tee -a "${LOG_FILE}"; }
log_error() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "${LOG_FILE}" >&2; }

cleanup() { rm -f "${LOCK_FILE}"; }
trap cleanup EXIT

if [[ -f "${LOCK_FILE}" ]]; then
    PID=$(cat "${LOCK_FILE}")
    kill -0 "${PID}" 2>/dev/null && { log_error "备份运行中(PID:${PID})"; exit 1; }
    rm -f "${LOCK_FILE}"
fi
echo $$ > "${LOCK_FILE}"

# RAC 环境
if [[ "${IS_RAC}" == "true" ]]; then
    ORACLE_SID=$(echo "${RAC_INSTANCES}" | awk '{print $1}')
    export ORACLE_SID
    log_info "RAC 环境，使用实例: ${ORACLE_SID}"
fi

log_info "RMAN ${BACKUP_LEVEL} 备份开始"

#--- 生成 RMAN 命令 ---
RMAN_CMD=$(mktemp /tmp/rman_XXXXXX.rman)

cat > "${RMAN_CMD}" << RMAN_EOF
RUN {
    CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO COMPRESSED BACKUPSET;
    CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
    CONFIGURE CONTROLFILE AUTOBACKUP ON;
    CONFIGURE CONTROLFILE AUTOBACKUP FORMAT
        FOR DEVICE TYPE DISK TO '${RMAN_DIR}/ctl_%F';
    CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 2 TIMES TO DISK;
    CONFIGURE BACKUP OPTIMIZATION ON;

    ALLOCATE CHANNEL ch1 DEVICE TYPE DISK
        FORMAT '${RMAN_DIR}/bk_${BACKUP_LEVEL}_%d_%T_%U';
    ALLOCATE CHANNEL ch2 DEVICE TYPE DISK
        FORMAT '${RMAN_DIR}/bk_${BACKUP_LEVEL}_%d_%T_%U';
    ALLOCATE CHANNEL ch3 DEVICE TYPE DISK
        FORMAT '${RMAN_DIR}/bk_${BACKUP_LEVEL}_%d_%T_%U';
    ALLOCATE CHANNEL ch4 DEVICE TYPE DISK
        FORMAT '${RMAN_DIR}/bk_${BACKUP_LEVEL}_%d_%T_%U';
RMAN_EOF

# 根据级别追加
if [[ "${BACKUP_LEVEL}" == "level0" ]]; then
    cat >> "${RMAN_CMD}" << 'RMAN_EOF'
    BACKUP INCREMENTAL LEVEL 0 DATABASE
        TAG 'LEVEL0_&DATE_TAG'
        PLUS ARCHIVELOG TAG 'ARCH_&DATE_TAG';
RMAN_EOF
    sed -i "s/&DATE_TAG/${DATE_TAG}/g" "${RMAN_CMD}"
else
    cat >> "${RMAN_CMD}" << 'RMAN_EOF'
    BACKUP INCREMENTAL LEVEL 1 DATABASE
        TAG 'LEVEL1_&DATE_TAG'
        PLUS ARCHIVELOG TAG 'ARCH_&DATE_TAG';
RMAN_EOF
    sed -i "s/&DATE_TAG/${DATE_TAG}/g" "${RMAN_CMD}"
fi

# 12c+ PDB 特定备份
if [[ "${IS_CDB}" == "true" && -n "${PDB_NAME}" ]]; then
    cat >> "${RMAN_CMD}" << RMAN_EOF
    BACKUP PLUGGABLE DATABASE ${PDB_NAME}
        TAG 'PDB_${PDB_NAME}_${DATE_TAG}';
RMAN_EOF
fi

# 公共部分
cat >> "${RMAN_CMD}" << RMAN_EOF
    -- 备份控制文件和参数文件
    BACKUP CURRENT CONTROLFILE
        TAG 'CTRL_${DATE_TAG}'
        FORMAT '${RMAN_DIR}/ctl_%d_%T_%U';

    BACKUP SPFILE
        TAG 'SPFILE_${DATE_TAG}'
        FORMAT '${RMAN_DIR}/spf_%d_%T_%U';

    -- 独立归档备份（用于异地传输）
    BACKUP ARCHIVELOG ALL DELETE INPUT
        TAG 'ARCH_IND_${DATE_TAG}'
        FORMAT '${ARCH_DIR}/arch_%d_%T_%U';

    -- 清理过期
    DELETE NOPROMPT OBSOLETE;
    DELETE NOPROMPT EXPIRED BACKUP;
    CROSSCHECK BACKUP;
    CROSSCHECK ARCHIVELOG ALL;

    RELEASE CHANNEL ch1;
    RELEASE CHANNEL ch2;
    RELEASE CHANNEL ch3;
    RELEASE CHANNEL ch4;
}

LIST BACKUP SUMMARY;
REPORT SCHEMA;
RMAN_EOF

#--- 执行 RMAN ---
rman target / cmdfile="${RMAN_CMD}" >> "${LOG_FILE}" 2>&1
RMAN_EXIT=$?
rm -f "${RMAN_CMD}"

if [[ ${RMAN_EXIT} -ne 0 ]]; then
    log_error "RMAN 失败，退出码: ${RMAN_EXIT}"
    [[ "${NOTIFY_ENABLE}" == "true" ]] && \
        echo "RMAN ${BACKUP_LEVEL} 失败|日志:${LOG_FILE}" | \
        mail -s "[ALERT] RMAN备份失败 - $(hostname)" "${NOTIFY_EMAIL}"
    exit ${RMAN_EXIT}
fi

RMAN_SIZE=$(du -sh "${RMAN_DIR}" | awk '{print $1}')
ARCH_SIZE=$(du -sh "${ARCH_DIR}" | awk '{print $1}')
log_info "RMAN:${RMAN_SIZE} 归档:${ARCH_SIZE}"

#--- 上传归档到远程 ---
if [[ -n "${REMOTE_HOST}" && -n "${REMOTE_KEY}" ]]; then
    # 归档日志每次都上传
    ARCH_NEW=$(find "${ARCH_DIR}" -name "arch_*" -mtime -1 -type f)
    if [[ -n "${ARCH_NEW}" ]]; then
        SFTP_B=$(mktemp)
        echo "mkdir ${REMOTE_DIR}/arch" > "${SFTP_B}"
        echo "cd ${REMOTE_DIR}/arch" >> "${SFTP_B}"
        echo "${ARCH_NEW}" | while read f; do echo "put \"${f}\"" >> "${SFTP_B}"; done
        echo "bye" >> "${SFTP_B}"
        sftp -o StrictHostKeyChecking=no -i "${REMOTE_KEY}" \
             -P "${REMOTE_PORT}" -b "${SFTP_B}" \
             "${REMOTE_USER}@${REMOTE_HOST}" >> "${LOG_FILE}" 2>&1
        rm -f "${SFTP_B}"
    fi

    # Level 0 上传完整备份集
    if [[ "${BACKUP_LEVEL}" == "level0" ]]; then
        tar czf "${RMAN_DIR}/rman_l0_${DATE_TAG}.tar.gz" \
            -C "${RMAN_DIR}" $(ls bk_level0_* 2>/dev/null) 2>/dev/null || true
        [[ -f "${RMAN_DIR}/rman_l0_${DATE_TAG}.tar.gz" ]] && {
            SFTP_B=$(mktemp)
            cat > "${SFTP_B}" << SFTP_EOF
mkdir ${REMOTE_DIR}/rman
cd ${REMOTE_DIR}/rman
put ${RMAN_DIR}/rman_l0_${DATE_TAG}.tar.gz
bye
SFTP_EOF
            sftp -o StrictHostKeyChecking=no -i "${REMOTE_KEY}" \
                 -P "${REMOTE_PORT}" -b "${SFTP_B}" \
                 "${REMOTE_USER}@${REMOTE_HOST}" >> "${LOG_FILE}" 2>&1
            rm -f "${SFTP_B}"
        }
    fi
fi

#--- 清理 + 通知 ---
find "${ARCH_DIR}" -name "arch_*" -mtime +${RETENTION_DAYS} -delete 2>/dev/null

if [[ "${NOTIFY_ENABLE}" == "true" && -n "${NOTIFY_EMAIL}" ]]; then
    echo "RMAN ${BACKUP_LEVEL} 完成|主机:$(hostname)|RMAN:${RMAN_SIZE}|归档:${ARCH_SIZE}" | \
        mail -s "[SUCCESS] RMAN ${BACKUP_LEVEL} - $(hostname) - ${DATE_TAG}" "${NOTIFY_EMAIL}"
fi

log_info "RMAN ${BACKUP_LEVEL} 备份完成"
SCRIPT

chmod +x /backup/oracle/scripts/rman_full_backup.sh
chown oracle:oinstall /backup/oracle/scripts/rman_full_backup.sh
```

### 4.3 独立归档日志备份脚本

归档日志备份频率应高于数据库备份（建议每 1-4 小时一次），以最小化 RPO。

```
cat > /backup/oracle/scripts/rman_arch_backup.sh << 'SCRIPT'
#!/bin/bash
#=============================================================
# 脚本：rman_arch_backup.sh
# 功能：RMAN 归档日志独立备份
# 调度：每 1-4 小时
#=============================================================
set -euo pipefail
source "/backup/oracle/scripts/oracle_backup.conf"

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DATE_TAG=$(date +%Y%m%d)
LOG_FILE="${LOG_DIR}/rman_arch_${TIMESTAMP}.log"
LOCK_FILE="/tmp/rman_arch.lock"

log_info() { echo "[$(date '+%H:%M:%S')] [INFO] $*" | tee -a "${LOG_FILE}"; }

cleanup() { rm -f "${LOCK_FILE}"; }
trap cleanup EXIT
[[ -f "${LOCK_FILE}" ]] && exit 1
echo $$ > "${LOCK_FILE}"

[[ "${IS_RAC}" == "true" ]] && export ORACLE_SID=$(echo "${RAC_INSTANCES}" | awk '{print $1}')

log_info "归档日志备份开始"

rman target / cmdfile=/dev/stdin >> "${LOG_FILE}" 2>&1 << RMAN_EOF
RUN {
    ALLOCATE CHANNEL ch1 DEVICE TYPE DISK FORMAT '${ARCH_DIR}/arch_%d_%T_%U';
    ALLOCATE CHANNEL ch2 DEVICE TYPE DISK FORMAT '${ARCH_DIR}/arch_%d_%T_%U';

    BACKUP ARCHIVELOG ALL NOT BACKED UP 1 TIMES
        TAG 'ARCH_${DATE_TAG}';

    BACKUP CURRENT CONTROLFILE
        TAG 'CTRL_ARCH_${DATE_TAG}'
        FORMAT '${RMAN_DIR}/ctl_%d_%T_%U';

    DELETE NOPROMPT ARCHIVELOG ALL BACKED UP 2 TIMES TO DISK;
    CROSSCHECK ARCHIVELOG ALL;
    DELETE NOPROMPT EXPIRED ARCHIVELOG ALL;

    RELEASE CHANNEL ch1;
    RELEASE CHANNEL ch2;
}
RMAN_EOF

[[ $? -ne 0 ]] && { log_info "归档备份失败"; exit 1; }

find "${ARCH_DIR}" -name "arch_*" -mtime +${RETENTION_DAYS} -delete 2>/dev/null
log_info "归档备份完成"
SCRIPT

chmod +x /backup/oracle/scripts/rman_arch_backup.sh
chown oracle:oinstall /backup/oracle/scripts/rman_arch_backup.sh
```

### 4.4 RMAN 初始化配置（首次执行前）

```
# rman target /
RMAN> CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO COMPRESSED BACKUPSET;
RMAN> CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
RMAN> CONFIGURE CONTROLFILE AUTOBACKUP ON;
RMAN> CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/backup/oracle/rman/ctl_%F';
RMAN> CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 2 TIMES TO DISK;
RMAN> CONFIGURE BACKUP OPTIMIZATION ON;
RMAN> SHOW ALL;
```

## 第五章：crontab 定时任务配置

### 5.1 crontab 配置

```
# 以 oracle 用户编辑
crontab -e
#=============================================
# Oracle 备份定时任务（crontab 基础版）
#=============================================

# 环境变量（crontab 不继承 .bash_profile）
ORACLE_SID=orcl
ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
ORACLE_BASE=/u01/app/oracle
PATH=/usr/local/bin:/bin:/usr/bin:${ORACLE_HOME}/bin
LD_LIBRARY_PATH=${ORACLE_HOME}/lib

# 归档日志备份 - 每4小时
0 */4 * * * /backup/oracle/scripts/rman_arch_backup.sh >> /backup/oracle/logs/cron_arch.log 2>&1

# RMAN Level0 全量 - 每周日凌晨2:00
0 2 * * 0 /backup/oracle/scripts/rman_full_backup.sh level0 >> /backup/oracle/logs/cron_rman.log 2>&1

# RMAN Level1 增量 - 周一至周六凌晨2:00
0 2 * * 1-6 /backup/oracle/scripts/rman_full_backup.sh level1 >> /backup/oracle/logs/cron_rman.log 2>&1

# expdp 逻辑备份 - 每周三、周日凌晨4:00
0 4 * * 0,3 /backup/oracle/scripts/expdp_full_backup.sh >> /backup/oracle/logs/cron_expdp.log 2>&1

# 日志清理 - 每周日凌晨6:00
0 6 * * 0 find /backup/oracle/logs -name "*.log" -mtime +30 -delete 2>/dev/null
```

⚠️ crontab 注意事项：
① 环境变量必须显式设置，crontab 不加载 .bash_profile
② 路径使用绝对路径
③ 确保 oracle 用户对脚本和目录有执行权限

### 5.2 验证 crontab

```
crontab -l                                    # 查看当前任务
systemctl status crond                        # 检查 crond 服务
su - oracle -c "/backup/.../rman_full_backup.sh level1"  # 手动测试
tail -f /var/log/cron                         # 查看 crontab 日志
```

## 第六章：进阶版 — systemd 系统服务

crontab 的不足：无法设资源限制、不支持依赖管理、错过执行不会补跑、日志分散。systemd timer + service 是更现代的替代方案。

### 6.1 RMAN Level 0 服务 + 定时器

```
# /etc/systemd/system/oracle-rman-level0.service
cat > /etc/systemd/system/oracle-rman-level0.service << 'EOF'
[Unit]
Description=Oracle RMAN Level 0 Full Backup
After=network.target

[Service]
Type=oneshot
User=oracle
Group=oinstall
Environment=ORACLE_SID=orcl
Environment=ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
Environment=ORACLE_BASE=/u01/app/oracle
Environment=PATH=/usr/local/bin:/usr/bin:/bin:/u01/app/oracle/product/12.2.0/dbhome_1/bin
Environment=LD_LIBRARY_PATH=/u01/app/oracle/product/12.2.0/dbhome_1/lib
ExecStart=/backup/oracle/scripts/rman_full_backup.sh level0
LimitNOFILE=65536
TimeoutStartSec=7200
PrivateTmp=true
StandardOutput=journal
StandardError=journal
SyslogIdentifier=oracle-rman-level0
Restart=no
EOF

# /etc/systemd/system/oracle-rman-level0.timer
cat > /etc/systemd/system/oracle-rman-level0.timer << 'EOF'
[Unit]
Description=Oracle RMAN Level 0 Weekly Timer
Requires=oracle-rman-level0.service

[Timer]
OnCalendar=Sun *-*-* 02:00:00
Persistent=true
RandomizedDelaySec=300
AccuracySec=60

[Install]
WantedBy=timers.target
EOF
```

### 6.2 RMAN Level 1 服务 + 定时器

```
cat > /etc/systemd/system/oracle-rman-level1.service << 'EOF'
[Unit]
Description=Oracle RMAN Level 1 Incremental Backup
After=network.target

[Service]
Type=oneshot
User=oracle
Group=oinstall
Environment=ORACLE_SID=orcl
Environment=ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
Environment=ORACLE_BASE=/u01/app/oracle
Environment=PATH=/usr/local/bin:/usr/bin:/bin:/u01/app/oracle/product/12.2.0/dbhome_1/bin
Environment=LD_LIBRARY_PATH=/u01/app/oracle/product/12.2.0/dbhome_1/lib
ExecStart=/backup/oracle/scripts/rman_full_backup.sh level1
LimitNOFILE=65536
TimeoutStartSec=7200
PrivateTmp=true
StandardOutput=journal
StandardError=journal
SyslogIdentifier=oracle-rman-level1
Restart=no
EOF

cat > /etc/systemd/system/oracle-rman-level1.timer << 'EOF'
[Unit]
Description=Oracle RMAN Level 1 Daily Timer
Requires=oracle-rman-level1.service

[Timer]
OnCalendar=Mon,Tue,Wed,Thu,Fri,Sat *-*-* 02:00:00
Persistent=true
RandomizedDelaySec=300
AccuracySec=60

[Install]
WantedBy=timers.target
EOF
```

### 6.3 归档日志备份（每4小时）

```
cat > /etc/systemd/system/oracle-rman-arch.service << 'EOF'
[Unit]
Description=Oracle Archive Log Backup
After=network.target

[Service]
Type=oneshot
User=oracle
Group=oinstall
Environment=ORACLE_SID=orcl
Environment=ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
Environment=ORACLE_BASE=/u01/app/oracle
Environment=PATH=/usr/local/bin:/usr/bin:/bin:/u01/app/oracle/product/12.2.0/dbhome_1/bin
Environment=LD_LIBRARY_PATH=/u01/app/oracle/product/12.2.0/dbhome_1/lib
ExecStart=/backup/oracle/scripts/rman_arch_backup.sh
LimitNOFILE=65536
TimeoutStartSec=3600
PrivateTmp=true
StandardOutput=journal
StandardError=journal
SyslogIdentifier=oracle-rman-arch
Restart=no
EOF

cat > /etc/systemd/system/oracle-rman-arch.timer << 'EOF'
[Unit]
Description=Oracle Archive Log Backup Timer
Requires=oracle-rman-arch.service

[Timer]
OnCalendar=*-*-* 00,04,08,12,16,20:00:00
Persistent=true
RandomizedDelaySec=60
AccuracySec=30

[Install]
WantedBy=timers.target
EOF
```

### 6.4 expdp 逻辑备份（每周三、日）

```
cat > /etc/systemd/system/oracle-expdp-backup.service << 'EOF'
[Unit]
Description=Oracle expdp Logical Backup
After=network.target

[Service]
Type=oneshot
User=oracle
Group=oinstall
Environment=ORACLE_SID=orcl
Environment=ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
Environment=ORACLE_BASE=/u01/app/oracle
Environment=PATH=/usr/local/bin:/usr/bin:/bin:/u01/app/oracle/product/12.2.0/dbhome_1/bin
Environment=LD_LIBRARY_PATH=/u01/app/oracle/product/12.2.0/dbhome_1/lib
ExecStart=/backup/oracle/scripts/expdp_full_backup.sh
LimitNOFILE=65536
TimeoutStartSec=7200
PrivateTmp=true
StandardOutput=journal
StandardError=journal
SyslogIdentifier=oracle-expdp-backup
Restart=no
EOF

cat > /etc/systemd/system/oracle-expdp-backup.timer << 'EOF'
[Unit]
Description=Oracle expdp Backup Timer
Requires=oracle-expdp-backup.service

[Timer]
OnCalendar=Sun,Wed *-*-* 04:00:00
Persistent=true
RandomizedDelaySec=300
AccuracySec=60

[Install]
WantedBy=timers.target
EOF
```

### 6.5 启用和管理

```
# 重载配置
systemctl daemon-reload

# 启用 + 启动所有定时器
systemctl enable --now oracle-rman-level0.timer
systemctl enable --now oracle-rman-level1.timer
systemctl enable --now oracle-rman-arch.timer
systemctl enable --now oracle-expdp-backup.timer

# 查看定时器状态
systemctl list-timers | grep oracle

# 手动触发测试
systemctl start oracle-rman-level1.service

# 查看日志
journalctl -u oracle-rman-level0 -f
journalctl -u oracle-rman-level1 --since "today"
```

### 6.6 systemd vs crontab 对比

| 特性     | crontab      | systemd timer              |
| :------- | :----------- | :------------------------- |
| 依赖管理 | ❌ 不支持     | ✅ After/Requires           |
| 资源限制 | ❌ 不支持     | ✅ LimitCPU/Memory/NOFILE   |
| 错过执行 | ❌ 不执行     | ✅ Persistent=true 自动补跑 |
| 随机延迟 | ❌ 不支持     | ✅ RandomizedDelaySec       |
| 日志管理 | 分散在各文件 | ✅ journalctl 统一管理      |
| 故障处理 | ❌ 不支持     | ✅ Restart/OnFailure        |
| 查看任务 | crontab -l   | systemctl list-timers      |
| 安全加固 | ❌ 无         | ✅ PrivateTmp/ProtectSystem |

💡 **生产建议**：RAC 环境或对运维规范有要求的场景，强烈推荐 systemd 方案。

## 第七章：备份验证与恢复演练

> 备份不做验证，等于没备份。可以日常准备一套虚拟机，安装好基本数据库软件，做恢复环境应急（打好快照），两套，一套安装好实例（逻辑恢复），一套仅安装数据库（RMAN恢复），在生产环境，切勿轻易做操作，测试环境验证后再逐步生产，数据安全第一！！！

### 7.1 expdp 恢复

```
# 全库导入
impdp system/password \
    DIRECTORY=BACKUP_DIR \
    DUMPFILE=full_20240101_%U.dmp \
    FULL=y \
    LOGFILE=impdp_full_restore.log \
    PARALLEL=4

# 仅导入指定模式
impdp system/password \
    DIRECTORY=BACKUP_DIR \
    DUMPFILE=full_20240101_%U.dmp \
    SCHEMAS=hr,scott \
    LOGFILE=impdp_schema_restore.log

# 仅导入指定表
impdp system/password \
    DIRECTORY=BACKUP_DIR \
    DUMPFILE=full_20240101_%U.dmp \
    TABLES=hr.employees,scott.dept \
    LOGFILE=impdp_table_restore.log
```

### 7.2 RMAN 恢复

```
# 完全恢复
rman target /
RMAN> STARTUP MOUNT;
RMAN> RESTORE DATABASE;
RMAN> RECOVER DATABASE;
RMAN> ALTER DATABASE OPEN;

# 不完全恢复（到指定时间点）
rman target /
RMAN> STARTUP MOUNT;
RMAN> RUN {
    SET UNTIL TIME "TO_DATE('2024-01-01 12:00:00','YYYY-MM-DD HH24:MI:SS')";
    RESTORE DATABASE;
    RECOVER DATABASE;
}
RMAN> ALTER DATABASE OPEN RESETLOGS;

# 验证备份完整性（不实际恢复）
RMAN> RESTORE DATABASE VALIDATE;
RMAN> RESTORE ARCHIVELOG ALL VALIDATE;
```

### 7.3 备份验证脚本

```
cat > /backup/oracle/scripts/backup_verify.sh << 'VERIFY'
#!/bin/bash
source "/backup/oracle/scripts/oracle_backup.conf"
VERIFY_LOG="${LOG_DIR}/backup_verify_$(date +%Y%m%d).log"

echo "========== Oracle 备份验证报告 ==========" | tee "${VERIFY_LOG}"
echo "时间: $(date)" | tee -a "${VERIFY_LOG}"

echo -e "\n[1] 最近的 RMAN 备份:" | tee -a "${VERIFY_LOG}"
rman target / << RMAN_EOF >> "${VERIFY_LOG}" 2>&1
LIST BACKUP SUMMARY;
REPORT NEED BACKUP;
RMAN_EOF

echo -e "\n[2] 验证备份集完整性:" | tee -a "${VERIFY_LOG}"
rman target / << RMAN_EOF >> "${VERIFY_LOG}" 2>&1
VALIDATE BACKUPSET ALL;
RMAN_EOF

echo -e "\n[3] expdp 备份文件:" | tee -a "${VERIFY_LOG}"
ls -lh ${EXPDP_DIR}/full_*.dmp 2>/dev/null | tail -5 | tee -a "${VERIFY_LOG}"

echo -e "\n[4] 磁盘空间:" | tee -a "${VERIFY_LOG}"
df -h /backup/oracle | tee -a "${VERIFY_LOG}"

echo -e "\n[5] 远程存储连通性:" | tee -a "${VERIFY_LOG}"
sftp -o ConnectTimeout=10 -i "${REMOTE_KEY}" \
     -P "${REMOTE_PORT}" \
     "${REMOTE_USER}@${REMOTE_HOST}" <<< "bye" >> "${VERIFY_LOG}" 2>&1

echo -e "\n========== 验证完成 ==========" | tee -a "${VERIFY_LOG}"
VERIFY

chmod +x /backup/oracle/scripts/backup_verify.sh
```

## 第八章：常见问题与最佳实践

### 8.1 FAQ

**Q1: expdp 报错 ORA-39002 / ORA-39070 目录对象不存在？**

```
SELECT * FROM dba_directories WHERE directory_name = 'BACKUP_DIR';
GRANT READ, WRITE ON DIRECTORY BACKUP_DIR TO system;
```

**Q2: RMAN 备份空间不足怎么办？**

```
RMAN> DELETE OBSOLETE;
RMAN> CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 3 DAYS;
```

**Q3: RAC 环境备份应该连哪个实例？**

连任意一个实例即可。`BACKUP ARCHIVELOG ALL` 会自动备份所有实例的归档日志。建议配置 NFS 共享归档日志目录。

**Q4: 12c CDB 下如何只备份某个 PDB？**

```
# RMAN 方式
RMAN> BACKUP PLUGGABLE DATABASE pdb1;

# expdp 方式
expdp system/password@pdb1 FULL=y ...
```

**Q5: crontab 脚本不执行？**

检查清单：① crontab -l 确认任务存在 ② systemctl status crond ③ 脚本 chmod +x ④ 使用绝对路径 ⑤ crontab 中设置环境变量 ⑥ /var/log/cron 看执行记录

### 8.2 最佳实践清单

- 备份恢复演练

  ：每月至少一次恢复验证

- Block Change Tracking

  ：启用 BCT 加速增量备份

- 并行度

  ：PARALLEL 设为 CPU 核数的一半

- 压缩

  ：RMAN 和 expdp 都开启压缩

- 监控告警

  ：备份失败必须通知 DBA

- 异地存储

  ：至少一份备份在不同物理位置

- 磁盘空间监控

  ：剩余 <20% 告警

- 备份窗口

  ：RMAN 和 expdp 错开执行时间

- 日志保留

  ：保留至少 30 天

- 配置版本化

  ：备份脚本纳入 Git 管理

- 锁机制

  ：防止并发运行

- RAC 环境

  ：用 systemd 管理，避免 crontab 多节点重复
