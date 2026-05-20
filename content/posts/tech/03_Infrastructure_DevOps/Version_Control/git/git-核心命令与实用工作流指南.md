+++
date = '2026-05-20T09:42:54+08:00'
draft = false
title = 'Git 核心命令与实用工作流指南'
slug = "qrr4hxa36w"
description = "Git 核心命令与实用工作流指南"
summary = "Git 核心命令与实用工作流指南"
categories = ["📒版本控制"]
tags = ["Git"]
+++

# Git 核心命令与实用工作流指南

本指南旨在为开发和运维人员提供一份关于 Git 的全面、实用的命令参考和工作流说明。内容从 Git 的首次配置开始，贯穿本地操作、远程协作、版本历史管理及分支策略等核心环节。

## 1. Git 核心概念：三区模型

要理解 Git，首先要理解其核心的“三区”模型：

- **工作区 (Working Directory)**：您在电脑上能看到的、正在进行编辑的目录和文件。
- **暂存区 (Staging Area / Index)**：一个临时的存储区域，用于存放您希望在下一次提交中包含的变更。`git add` 命令就是将工作区的修改添加到暂存区。
- **本地仓库 (Local Repository)**：一个位于您项目目录下的 `.git` 目录，它包含了项目的所有版本历史。`git commit` 命令就是将暂存区的内容永久性地快照到本地仓库中。

**基本流程**: **工作区修改 → `git add` → 暂存区 → `git commit` → 本地仓库**

## 2. 初始配置与仓库创建

### 2.1 首次配置 (自报家门)

在首次使用 Git 时，必须设置您的用户名和邮箱，这会作为您提交代码的身份标识。

```bash
# 设置全局用户名
git config --global user.name "Your Name"

# 设置全局邮箱
git config --global user.email "your_email@example.com"

# 查看所有全局配置
git config --global --list
```

### 2.2 创建仓库

**方法一：初始化一个新仓库**

```bash
# 在当前目录下创建一个新的 .git 仓库
git init
```

**方法二：克隆一个远程仓库**

```bash
# 从远程地址克隆一个完整的项目仓库到本地
git clone <repository_url>
```

## 3. 本地工作流：修改、暂存与提交

这是日常开发中最频繁的操作循环。

### 3.1 查看文件状态

```bash
# 查看当前工作区和暂存区的状态
git status
```

### 3.2 暂存变更 (`git add`)

将工作区的修改添加到暂存区。

| 命令               | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| `git add <文件名>` | 将指定文件的修改添加到暂存区。                               |
| `git add .`        | 将**所有**新建和已修改的文件添加到暂存区（**不包括**被删除的文件）。 |
| `git add -u`       | 将**所有**已跟踪的、被修改和被删除的文件添加到暂存区（**不包括**新建的文件）。 |
| `git add -A`       | **(最常用)** 将**所有**变更（新建、修改、删除）都添加到暂存区。 |

### 3.3 提交变更 (`git commit`)

将暂存区的所有内容创建一个新的版本快照，并永久保存在本地仓库中。

```bash
# -m 参数用于直接在命令行中添加提交信息
git commit -m "feat: Add user registration feature"
```

### 3.4 文件重命名与删除

```bash
# 重命名文件 (相当于 mv file target; git add file; git rm target)
git mv old-filename new-filename

# 从工作区和暂存区同时删除文件
git rm filename
```

### 3.5 撤销本地修改

> **警告**：以下命令会**永久丢弃**您未提交的本地工作，请谨慎操作。

```bash
# 丢弃工作区中所有文件的修改，恢复到上一次提交的状态
git checkout .

# 丢弃工作区修改，并删除所有未被跟踪的新建文件和目录
git checkout . && git clean -xdf
```

## 4. 远程仓库协作

### 4.1 配置远程访问 (SSH 公钥)

为了通过 SSH 协议安全地与远程仓库（如 GitHub, GitLab）通信，需要配置 SSH 密钥。

```bash
# 1. 生成 SSH 密钥对 (-t 指定类型, -C 添加注释)
ssh-keygen -t rsa -C "your_email@example.com"

# 2. 按提示操作，会在 ~/.ssh/ 目录下生成私钥 (id_rsa) 和公钥 (id_rsa.pub)
# 3. 将公钥 (id_rsa.pub) 的内容，复制并添加到您的远程仓库平台的 SSH Key 设置中。
```

### 4.2 管理远程仓库别名 (`git remote`)

Git 允许您为远程仓库的 URL 设置一个更简短、易记的别名（默认克隆的仓库别名为 `origin`）。

```bash
# 查看所有远程仓库别名及其 URL
git remote -v

# 添加一个新的远程仓库别名
git remote add <alias_name> <repository_url>

# 重命名一个别名
git remote rename <old_alias> <new_alias>

# 删除一个别名
git remote remove <alias_name>
```

### 4.3 同步变更 (`push` & `pull`)

```bash
# 推送：将本地仓库的某个分支推送到远程仓库
# git push <远程别名> <本地分支名>
git push origin main

# 拉取：从远程仓库的某个分支拉取最新代码，并与本地分支合并
# git pull <远程别名> <远程分支名>
git pull origin main
```

> **提示**：`git pull` 实际上是 `git fetch` (获取远程更新) 和 `git merge` (合并更新) 两个命令的组合。

## 5. 版本历史与回退

### 5.1 查看提交历史 (`git log`)

```bash
# 查看详细的提交历史
git log

# 以单行简洁格式显示历史
git log --pretty=oneline

# 查看所有操作记录，包括被 reset 删除的提交记录 (救命稻草)
git reflog
```

### 5.2 版本回退 (`git reset`)

> **❗ 高风险警告**：`git reset --hard` 是一个**破坏性**操作，它会**永久丢弃**指定版本之后的所有提交和本地修改。在执行前回滚前，请务必确认您不再需要这些变更。

```bash
# 回退到上一个版本 (HEAD^ 表示上一个，HEAD~n 表示前 n 个)
git reset --hard HEAD^

# 回退到 git reflog 中查到的任意一个版本
git reset --hard <commit_id>
```

> **安全替代方案**：在共享分支上，推荐使用 `git revert <commit_id>`，它会创建一个**新的提交**来撤销某个旧的提交，而不会修改项目历史。

## 6. 分支管理 (`git branch`)

分支是 Git 的核心功能之一，它允许您在不影响主线（`main`/`master`）的情况下，独立地开发新功能或修复 Bug。

```bash
# 查看所有本地分支 (* 表示当前所在分支)
git branch

# 创建一个新分支
git branch <branch_name>

# 切换到指定分支
git checkout <branch_name>
# 创建并立即切换到一个新分支 (常用)
# git checkout -b <branch_name>

# 合并分支：先切换回主分支，再将 feature-branch 的代码合并进来
git checkout main
git merge <feature-branch>

# 删除一个已合并的分支
git branch -d <branch_name>
# 强制删除一个未合并的分支
# git branch -D <branch_name>

# 重命名分支
git branch -m <old_name> <new_name>
```
