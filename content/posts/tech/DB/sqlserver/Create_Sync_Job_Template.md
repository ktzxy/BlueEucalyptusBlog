+++
date = '2026-03-15T12:47:37+08:00'
draft = false
title = 'Create_Sync_Job_Template'
slug = "xl3qr9l97w"
description = "Create_Sync_Job_Template"
categories = ["📒数据库"]
tags = ["sqlserver"]
summary = "Create_Sync_Job_Template 脚本用于在 SQL Server 中创建一个同步作业模板，支持自定义频率、重试机制和超时控制。通过修改配置区域内的变量值，可以快速定制化适用于不同场景的定时任务。此脚本特别适合于高频数据同步需求，如每10分钟执行一次"
+++
# Create_Sync_Job_Template

```sql
/*
================================================================================
  脚本名称：SQL Server Agent 作业创建模板 (高频同步专用)
  功能描述：创建一个定时执行存储过程的作业，支持自定义频率、重试机制和超时控制。
  适用场景：数据同步、定时报表、批量处理等。
  
  【修改指南】
  下次使用时，只需修改脚本顶部的 "【配置区域】" 中的变量值即可。
  无需修改下方的逻辑代码，除非需要改变作业的高级行为。
================================================================================
*/

USE msdb;
GO

-- =============================================
-- 【配置区域】 (下次修改请只改这里)
-- =============================================
DECLARE 
    -- 1. 作业名称 (建议包含业务含义和频率，如：Job_Sync_Sale_10Min)
    @JobName            NVARCHAR(128) = N'Job_Sync_Sale_Every10Min',
    
    -- 2. 目标数据库名称 (存储过程所在的数据库)
    @DatabaseName       NVARCHAR(128) = N'zyscm_dherp',
    
    -- 3. 要执行的存储过程名称 (可带参数，如: 'dbo.sync_sale_to_middle @param=1')
    @StoredProcedure    NVARCHAR(200) = N'dbo.sync_sale_to_middle',
    
    -- 4. 作业所有者 (通常留空使用当前登录用户，或指定为 'sa')
    @OwnerLogin         NVARCHAR(128) = N'', 
    
    -- 5. 调度频率设置
    @FreqType           INT           = 4,   -- 4=每天, 3=每周, 2=一次性
    @FreqSubdayType     INT           = 4,   -- 4=分钟, 8=小时, 1=一次
    @FreqSubdayInterval INT           = 10,  -- 间隔数值 (例如：每10分钟，每2小时)
    
    -- 6. 开始运行时间 (格式：HHMMSS，例如 080000 = 早上8点)
    @ActiveStartTime    INT           = 000000, 
    
    -- 7. 失败重试设置
    @RetryAttempts      INT           = 1,   -- 失败后重试次数
    @RetryInterval      INT           = 1;   -- 重试间隔 (分钟)

-- =============================================
-- 【逻辑区域】 (通常不需要修改)
-- =============================================

DECLARE @ReturnCode INT;
DECLARE @JobId UNIQUEIDENTIFIER;
DECLARE @StepName NVARCHAR(128) = N'Execute Sync Step';
DECLARE @ScheduleName NVARCHAR(128) = N'Schedule_' + CAST(@FreqSubdayInterval AS VARCHAR) + '_Mins';
DECLARE @Command NVARCHAR(MAX);
DECLARE @FinalOwner NVARCHAR(128);

-- 自动处理所有者：如果未指定，则使用当前用户
IF ISNULL(@OwnerLogin, '') = '' SET @FinalOwner = SUSER_SNAME();
ELSE SET @FinalOwner = @OwnerLogin;

-- 构建执行命令
SET @Command = N'USE [' + @DatabaseName + N']; EXEC ' + @StoredProcedure + N';';

PRINT '正在配置作业: ' + @JobName;
PRINT '目标数据库: ' + @DatabaseName;
PRINT '执行命令: ' + @Command;

-- 1. 删除已存在的同名作业 (防止冲突)
IF EXISTS (SELECT job_id FROM msdb.dbo.sysjobs WHERE name = @JobName)
BEGIN
    EXEC msdb.dbo.sp_delete_job @job_name = @JobName, @delete_unused_schedule = 1;
    PRINT '>>> 已删除旧作业: ' + @JobName;
END

-- 2. 创建新作业
EXEC @ReturnCode = msdb.dbo.sp_add_job 
    @job_name = @JobName,
    @enabled = 1,                   -- 1=启用, 0=禁用
    @notify_level_eventlog = 2,     -- 2=失败时记录到Windows事件日志
    @notify_level_email = 0,        -- 0=不发送邮件 (如需邮件通知需配置数据库邮件)
    @notify_level_netsend = 0,
    @notify_level_page = 0,
    @delete_level = 0,              -- 0=作业完成后不删除
    @description = N'自动生成的同步作业 - 频率:' + CAST(@FreqSubdayInterval AS VARCHAR) + '分钟',
    @category_name = N'[Uncategorized (Local)]',
    @owner_login_name = @FinalOwner,
    @job_id = @JobId OUTPUT;

IF (@@ERROR <> 0 OR @ReturnCode <> 0) 
BEGIN
    PRINT '!!! 创建作业失败';
    GOTO QuitWithRollback;
END

-- 3. 添加作业步骤
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep 
    @job_id = @JobId,
    @step_name = @StepName,
    @step_id = 1,
    @cmdexec_success_code = 0,
    @on_success_action = 1,  -- 1=结束作业 (成功)
    @on_success_step_id = 0,
    @on_fail_action = 2,     -- 2=结束作业 (失败) - 如果需要重试，主要靠下面的重试参数
    @on_fail_step_id = 0,
    @retry_attempts = @RetryAttempts,
    @retry_interval = @RetryInterval,
    @os_run_priority = 0,
    @subsystem = N'TSQL',
    @command = @Command,
    @database_name = @DatabaseName, -- 关键：确保上下文在正确的数据库
    @flags = 0;

IF (@@ERROR <> 0 OR @ReturnCode <> 0) 
BEGIN
    PRINT '!!! 创建作业步骤失败';
    GOTO QuitWithRollback;
END

-- 4. 添加调度计划
EXEC @ReturnCode = msdb.dbo.sp_add_jobschedule 
    @job_id = @JobId,
    @name = @ScheduleName,
    @enabled = 1,
    @freq_type = @FreqType,
    @freq_interval = 1,
    @freq_subday_type = @FreqSubdayType,
    @freq_subday_interval = @FreqSubdayInterval,
    @freq_relative_interval = 0,
    @freq_recurrence_factor = 1,
    @active_start_date = CONVERT(VARCHAR(8), GETDATE(), 112), -- 从今天开始
    @active_end_date = 99991231,
    @active_start_time = @ActiveStartTime,
    @active_end_time = 235959;

IF (@@ERROR <> 0 OR @ReturnCode <> 0) 
BEGIN
    PRINT '!!! 创建调度计划失败';
    GOTO QuitWithRollback;
END

-- 5. 将作业关联到本地服务器
EXEC @ReturnCode = msdb.dbo.sp_add_jobserver 
    @job_id = @JobId, 
    @server_name = N'(local)';

IF (@@ERROR <> 0 OR @ReturnCode <> 0) 
BEGIN
    PRINT '!!! 关联服务器失败';
    GOTO QuitWithRollback;
END

PRINT '=========================================';
PRINT '>>> 作业创建成功!';
PRINT '>>> 名称: ' + @JobName;
PRINT '>>> 频率: 每 ' + CAST(@FreqSubdayInterval AS VARCHAR) + ' 分钟';
PRINT '>>> 下次运行时间请查看 SQL Agent 作业历史记录';
PRINT '=========================================';

GOTO EndRoutine;

QuitWithRollback:
    IF (@JobId IS NOT NULL) 
    BEGIN
        EXEC msdb.dbo.sp_delete_job @job_id = @JobId;
        PRINT '>>> 已回滚并删除部分创建的作业';
    END

EndRoutine:
GO
```

# 作业清理某表日志

```shell
/*
===============================================================================
脚本功能：创建 SQL Server Agent 作业，自动清理 wtps_middledb..sync_log 的历史日志
清理策略：保留最近 7 天数据，每天凌晨 02:00 执行
===============================================================================
*/

USE msdb;
GO

-- ================= 配置区域 (请在此处修改) =================
DECLARE @dbName       NVARCHAR(128) = N'wtps_middledb';
DECLARE @tableName    NVARCHAR(128) = N'sync_log';
-- 【重要】请将 'log_time' 修改为你表中真实的时间字段名！
DECLARE @timeColumn   NVARCHAR(128) = N'end_time'; 
DECLARE @retainDays   INT           = 7; -- 保留天数
DECLARE @jobName      NVARCHAR(128) = N'Job_Cleanup_SyncLog';
DECLARE @scheduleName NVARCHAR(128) = N'Schedule_3Daily_2AM';
-- ===========================================================

BEGIN TRY
	DECLARE @jobDescription NVARCHAR(512);
	SET @jobDescription = N'自动清理 ' + @dbName + '.' + @tableName + N' 中 ' + CAST(@retainDays AS NVARCHAR(10)) + N' 天前的日志';
    
	-- 1. 如果作业已存在，先删除以避免冲突
    IF EXISTS (SELECT 1 FROM msdb.dbo.sysjobs WHERE name = @jobName)
    BEGIN
        EXEC msdb.dbo.sp_delete_job @job_name = @jobName;
        PRINT N'✓ 已删除存在的旧作业：' + @jobName;
    END

    -- 2. 创建新作业
    DECLARE @jobId UNIQUEIDENTIFIER;
    EXEC msdb.dbo.sp_add_job
        @job_name = @jobName,
        @enabled = 1,
        @description = @jobDescription,
        @start_step_id = 1,
        @job_id = @jobId OUTPUT;
    
    PRINT N'✓ 作业创建成功：' + @jobName;

    -- 3. 构建删除逻辑的 T-SQL 命令
    DECLARE @sqlCommand NVARCHAR(MAX);
    SET @sqlCommand = N'
USE [' + @dbName + N'];
SET NOCOUNT ON;

DECLARE @CutoffDate DATETIME = DATEADD(DAY, -' + CAST(@retainDays AS NVARCHAR) + N', GETDATE());
DECLARE @DeletedCount INT = 0;

-- 执行删除
DELETE FROM [' + @tableName + N']
WHERE [' + @timeColumn + N'] < @CutoffDate;

SET @DeletedCount = @@ROWCOUNT;

-- 输出信息到作业历史日志
PRINT N''[清理完成] 表：' + @tableName + N' | 截止期限：'' + CONVERT(NVARCHAR(19), @CutoffDate, 120) + N'' | 删除行数：'' + CAST(@DeletedCount AS NVARCHAR(20));
';

    -- 4. 添加作业步骤
    EXEC msdb.dbo.sp_add_jobstep
        @job_id = @jobId,
        @step_name = N'Delete Old Logs Step',
        @subsystem = N'TSQL',
        @command = @sqlCommand,
        @database_name = @dbName,
        @on_fail_action = 2; -- 失败时停止并报告

    PRINT N'✓ 作业步骤已配置';

    -- 5. 创建调度计划 (每天 02:00:00)
    -- 如果计划已存在则复用，否则创建
    DECLARE @scheduleId INT;
    
    IF EXISTS (SELECT 1 FROM msdb.dbo.sysschedules WHERE name = @scheduleName)
    BEGIN
        SELECT @scheduleId = schedule_id FROM msdb.dbo.sysschedules WHERE name = @scheduleName;
    END
    ELSE
    BEGIN
        EXEC msdb.dbo.sp_add_schedule
            @schedule_name = @scheduleName,
            @freq_type = 4,      -- 每天
            @freq_interval = 3,  -- 每隔 3 天
            @active_start_time = 20000, -- 02:00:00 (HHMMSS)
            @schedule_id = @scheduleId OUTPUT;
    END

    -- 6. 关联作业与计划
    EXEC msdb.dbo.sp_attach_schedule
        @job_id = @jobId,
        @schedule_id = @scheduleId;

    -- 7. 关联作业到当前服务器
    EXEC msdb.dbo.sp_add_jobserver
        @job_id = @jobId,
        @server_name = @@SERVERNAME;

    PRINT N'========================================';
    PRINT N'🎉 作业部署成功！';
    PRINT N'作业名称：' + @jobName;
    PRINT N'目标对象：' + @dbName + '..' + @tableName;
    PRINT N'时间字段：' + @timeColumn;
    PRINT N'保留策略：最近 ' + CAST(@retainDays AS NVARCHAR) + N' 天';
    PRINT N'执行频率：每天凌晨 02:00';
    PRINT N'========================================';
    PRINT N'⚠️ 下一步操作建议：';
    PRINT N'请确保字段 [' + @timeColumn + N'] 上有索引，否则删除大量数据时会锁表！';
    PRINT N'执行以下 SQL 创建索引（如果尚未存在）：';
    PRINT N'CREATE INDEX IX_' + @tableName + '_Time ON ' + @dbName + '.dbo.' + @tableName + '(' + @timeColumn + ');';
    PRINT N'========================================';

END TRY
BEGIN CATCH
    PRINT N'❌ 发生错误：' + ERROR_MESSAGE();
    PRINT N'错误号：' + CAST(ERROR_NUMBER() AS NVARCHAR);
    -- 如果出错，尝试回滚可能创建的半成品的作业
    IF EXISTS (SELECT 1 FROM msdb.dbo.sysjobs WHERE name = @jobName)
        EXEC msdb.dbo.sp_delete_job @job_name = @jobName;
END CATCH
GO
```

