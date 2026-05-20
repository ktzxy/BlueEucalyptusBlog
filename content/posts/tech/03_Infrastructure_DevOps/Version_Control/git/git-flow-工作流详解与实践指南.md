+++
date = '2026-05-20T09:42:26+08:00'
draft = false
title = 'Git Flow 工作流详解与实践指南'
slug = "0okho9os5d"
description = "Git Flow 工作流详解与实践指南"
summary = "Git Flow 工作流详解与实践指南"
categories = ["📒版本控制"]
tags = ["Git"]
+++

# Git Flow 工作流详解与实践指南

本指南旨在详细介绍由 Vincent Driessen 提出的、业界广泛采用的 Git 分支管理模型——**Git Flow**。内容从其核心概念和分支职责入手，通过流程图和简化的手动命令进行实践，并最终介绍如何使用 `git-flow` 工具链实现自动化。

## 1. 核心概念：分支的职责划分

Git Flow 的精髓在于为不同类型的开发活动定义了专门的、有明确生命周期的分支，主要分为两类主分支和三类辅助分支。

### 1.1 两条主分支 (Long-lived)

**`master` (或 `main`)**: **生产分支**。

- 永远处于**可发布**状态，其上的每一个 `commit` 都应对应一个正式的发布版本，并使用版本号 `tag` 标记。
- **绝对禁止**直接在该分支上进行开发。

**`develop`**: **主开发分支**。

- 作为所有新功能开发的**集成分支**，包含了所有已完成、待发布的功能。
- 它是新功能分支的起点，也是新功能合并的终点。

### 1.2 三类辅助分支 (Short-lived)

**`feature/*`**: **功能分支**。

- 用于开发新的、非紧急的功能。
- **起点**: 从 `develop` 分支切出。
- **终点**: 开发完成后，合并回 `develop` 分支。
- **命名**: 建议 `feature/user-login`, `feature/shopping-cart`。

**`release/*`**: **发布分支**。

- 用于准备一次新的正式发布。在此分支上，只进行 Bug 修复、文档生成等与发布相关的任务。
- **起点**: 从 `develop` 分支切出。
- **终点**: 发布完成后，**必须同时合并回 `master` 和 `develop` 分支**，并在 `master` 上打上版本标签。

**`hotfix/*`**: **热修复分支**。

- 用于紧急修复**线上 `master` 分支**出现的 Bug。
- **起点**: 从 `master` 分支切出。
- **终点**: 修复完成后，**必须同时合并回 `master` 和 `develop` 分支**，并在 `master` 上打上修复后的版本标签。

## 2. Git Flow 流程图

以下是 Git Flow 工作模式的流程图，您可以在支持 Mermaid 语法的 Markdown 编辑器中查看。

```sequence
participant master
participant develop
participant feature
participant release
participant hotfix

master->>develop: 初始化, 创建 develop 分支
develop->>feature: 切出功能分支 (e.g., feature/login)
feature-->>feature: 开发新功能...
feature->>develop: 功能完成, 合并回 develop
    
develop->>release: 准备发布, 切出 release 分支 (e.g., release/v1.0)
release-->>release: Bug 修复, 版本号更新...
    
release->>master: 发布完成, 合并到 master
master->>master: (Tag: v1.0)
release->>develop: 同时合并回 develop
    
master->>hotfix: 线上出现紧急 Bug, 切出 hotfix 分支 (e.g., hotfix/v1.0.1)
hotfix-->>hotfix: 紧急修复...
    
hotfix->>master: 修复完成, 合并到 master
master->>master: (Tag: v1.0.1)
hotfix->>develop: 同时合并回 develop
```

## 3. 手动实践 Git Flow (单仓库)

以下是在一个仓库内，通过基础 Git 命令模拟 Git Flow 流程。

### 3.1 初始设置

```bash
# 初始化项目并首次提交
git init
echo "Initial Project" > README.md
git add .
git commit -m "Initial commit"
git tag v0.1

# 创建 develop 分支
git branch develop
```

### 3.2 开发新功能 (Feature)

```bash
# 1. 从 develop 切出功能分支
git checkout -b feature/user-auth develop

# 2. 在功能分支上进行开发...
echo "User login logic" > login.php
git add .
git commit -m "feat: Add user login feature"

# 3. 开发完成，合并回 develop 分支
git checkout develop
git merge --no-ff feature/user-auth
# --no-ff (no fast-forward) 会创建一个新的合并 commit，保留分支历史

# 4. 删除已合并的功能分支
git branch -d feature/user-auth
```

### 3.3 准备发布 (Release)

```bash
# 1. 从 develop 切出发布分支
git checkout -b release/v1.0 develop

# 2. 在发布分支上进行最后的 Bug 修复和版本准备...
# (例如修改版本号文件，修复小 Bug)

# 3. 发布完成，将 release 分支合并到 master 和 develop
git checkout master
git merge --no-ff release/v1.0
git tag -a v1.0 -m "Release version 1.0"

git checkout develop
git merge --no-ff release/v1.0

# 4. 删除已合并的发布分支
git branch -d release/v1.0
```

### 3.4 紧急热修复 (Hotfix)

```bash
# 1. 线上 master 版本发现紧急 Bug，从 master 切出 hotfix 分支
git checkout -b hotfix/v1.0.1 master

# 2. 在 hotfix 分支上修复 Bug...
echo "Critical security patch" >> patch.php
git add .
git commit -m "fix: Address critical security issue"

# 3. 修复完成，将 hotfix 分支合并到 master 和 develop
git checkout master
git merge --no-ff hotfix/v1.0.1
git tag -a v1.0.1 -m "Hotfix for version 1.0.1"

git checkout develop
git merge --no-ff hotfix/v1.0.1

# 4. 删除已合并的热修复分支
git branch -d hotfix/v1.0.1
```

## 4. 使用 `git-flow` 自动化工具 (推荐)

手动执行上述流程容易出错，社区为此提供了 `git-flow` 命令行扩展工具，可以极大地简化操作。

### 4.1 安装

```bash
# RHEL/CentOS
yum install git-flow

# Debian/Ubuntu
apt install git-flow
```

### 4.2 使用流程

```bash
# 初始化 (会在仓库中配置好 master 和 develop 分支)
git flow init

# 开始一个新功能
git flow feature start user-profile

# ... 在 feature/user-profile 分支上开发 ...

# 完成一个功能 (会自动合并回 develop 并删除 feature 分支)
git flow feature finish user-profile

# 开始一个发布
git flow release start v1.1

# ... 在 release/v1.1 分支上进行发布准备 ...

# 完成一个发布 (会自动合并到 master 和 develop，打标签，并删除 release 分支)
git flow release finish v1.1
```

> `git flow hotfix` 的用法与此类似。使用此工具可以安全、规范地执行 Git Flow 模型。
