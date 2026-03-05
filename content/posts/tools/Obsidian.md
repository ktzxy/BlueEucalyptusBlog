+++
date = '2025-07-17T13:31:51+08:00'
draft = false
title = 'Obsidian'
slug = "c3b5uqabt2"
description = "Obsidian学习笔记"
categories = ["📒tools"]
tags = ["tools"]
summary = "Obsidian学习笔记"
+++


# 备份

使用gitee远程仓库进行笔记备份

# Obsidian简介

中文名：黑曜石

[官网]([Obsidian - Sharpen your thinking](https://obsidian.md/))

# 插件

Obsidian Git 自动提交

Image Auto Upload Plugin 配合PicGo+gitee/github 仓库 图片上传

Editor Syntax Highlight 代码高亮

Calendar 日历  依赖于 核心插件 -> 日记

Pandoc 导出word

File Explorer Note Count 笔记数量

Recent Files 最近文章

Mindmap 思维导图形成大纲

Tasks 管理待办事项

Obsidian memos 记录灵感

Excalidraw 绘制流程图

cMenu

Quick Explorer 笔记快速选择功能

DataView 用来创建基于元字段的数据查询

QuickAdd 快速添加

Auto Link Title 粘贴链接后显示为网页的标题

Advanced Tables 增强表格功能

Outliner 使用快捷键快速进行操作

Weread Plugin 同步微信读书的笔记到obsidian中

Copy Image and URL context menu 笔记中拷贝图片文件

Style Settings

Weread Plugin 微信读书笔记

Buttons 运行命令并通过单击打开链接或按钮

Templater 创建模板，可以将变量和函数结果插入到注释中

# Obsidian使用技巧

## 模板

开启日记功能（核心插件 -> 日记）

开启模板功能（核心插件 -> 模板）

运用了 Obsidian 的 YAML front matter 方法，它的好处是导出时不会出现在正文中

[官方文档-YAML front matter - Obsidian 中文帮助 - Obsidian Publish](https://publish.obsidian.md/help-zh/%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95/YAML+front+matter)

[一文掌握Obsidian模板](https://www.jianshu.com/p/ba63900433c7)

Total Commander 软件 资源管理器

# 双链

创建链接

- 链接到文章：[[1. Obsidian简介]]

- 链接到某个标题：[[1. Obsidian简介#3 Obsidian定位]]
- 链接到文本块：[[1. Obsidian简介#^e3f585]]
- 链接别名：[[1. Obsidian简介#^e3f585 | Obsidian中文名]]

查看链接

- 查看链接内容
- 核心插件 -> 页面预览
- 按住 Ctrl 预览
- 显示内容 ！

打开链接面板

- 核心插件 -> 反向链接
- 核心插件 -> 出链

# 在Obsidian中搜索

快捷键

- 搜索当前文档：Ctrl + F
- 搜索整个资料库：Ctrl / Cmd + Shfift + F

搜索技巧

- 直接搜索关键词

- 搜索包含多个关键词的文档（空格间隔）

- 搜索包含某一个关键词的文档（OR）

- 指定搜索范围

  - 搜索文件名 file:word

  - 搜索文本内容 content:word
  - 搜索标签 tag:#tag:word
  - 搜索同一行中的多个关键词 line:word1 word2
  - 搜索同一章节中的多个关键词 section:word1 word2
  - 搜索同一段落（块）中的多个关键词 block:word1 word2

搜索任务

- 搜索任务 task:""
- 搜索未完成任务 task-todo:""
- 搜索已完成任务 task-done:""

# Data View查询

定义：Obsidian资料库的查询工具/插件

查询依据：YAML数据/Meatainfo

YAML

​ 位于Markdown文件开头

​ 首尾三个 -

Obsidian支持的YMAL字段

- tags
- publish
- cssclass
- aliases

自定义字段

- category
- date
- time
- title
- rating

行内标记

- One Field:: Value
- 这份文档，可以打 [rating:: 5] 分

Obsidian文件属性

- file.name: 文件标题(字符串)
- file.folder: 文件所属文件夹路径
- file.path: 文件路径
- file.size: (in bytes) 文件大小
- file.ctime: 文件的创建时间（包含日期和时间）
- file.mtime: 文件的修改时间
- file.cday: 文件创建的日期
- file.mday: 文件修改的日期
- file.tags: 笔记中所有标签数组
- file.etags: 除去子标签的数组
- file.inlinks: 指向此文件的所有传入链接的数组
- file.outlinks: 此文件所有出站的链接数组
- file.aliases: 文件别名数组
- file.day：如果文件名中有日期，那么会以这个字段显示。比如文 件名中包含 yyyy-mm-dd（年-月-日，例如2021-03-21），那么 就会存在这个 metadata。

任务属性

- Task 会继承所在文件的所有字段，比如 Task 所在的页面中已经包 含了 rating 信息了，那么 task 也会有
- completed: 任务是否完成
- fullyCompleted: 任务以及所有的子任务是否完成
- text: 任务名
- line: task 所在行
- path: task 所在路径
- section: 连接到任务所在区块
- link: 连接到距离任务最近的可连接的区块
- subtasks: 子任务
- real: 如果为 true, 则是一个真正的任务，否则就是一个任务之前 或之后的元素列表
- completion: 任务完成的日期
- due: 任务到期时间
- created: 创建日期
- annotated: 如果任务有自定义标记则为 True，否则为 False

DataView

展示方式

- Table
- List
- Task

语法

~~~text
```dataview
TABLE|LIST|TASK <field> [AS "Column Name"], <field>, ..., <field> 
FROM <source> (like #tag or "folder")
WHERE <expression> (like 'field = value')
SORT <expression> [ASC/DESC] (like 'field ASC')
... other data commands
```

```dataview
TABLE|LIST|TASK <字段> [AS "表格列名"], <字段>, ..., <字段> 
FROM <源> (like #标签 or "文件夹")
WHERE <表达式> (like '字段 = 值')
SORT <表达式> [ASC/DESC] (like '字段 ASC')
... other data commands
```
~~~

- dataview

- list|table|task

- from

- where

- sort

  - asc，升序

  - desc，降序

- 查询方式

  - 文件夹

  - 标签

- 使用建议

  - 保存常用查询

  - 生成文件夹索引
