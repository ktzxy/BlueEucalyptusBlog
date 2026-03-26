+++
date = '2025-07-12T00:55:58+08:00'
draft = false
title = 'Typora到CSDN博客图片上传问题'
slug = "fxmzy316ih"
description = "Typora到CSDN博客图片上传问题"
categories = ["📒成长"]
tags = ["问题集锦"]
summary = "Typora到CSDN博客图片上传问题"
+++
﻿# Typora到CSDN博客图片上传问题

### 问题描述：

​		通常在Typora中写笔记，上传到CSDN博客中去，那么就会遇到图片上传问题，一个一个上传？又或者是图片还存留到本地？

### 问题解决：

​		在Github或者Gitee中注册一个仓库，用来储存图片，那么我们就可以随时随地访问，写笔记再也不用担心图片了。

### 前期准备：

1. 注册一个Gitee账号

2. 下载nodejs，并安装

   https://nodejs.org/en/

3. 下载PicGo，并安装

   https://molunerfinn.com/PicGo/

4. **Typora**

### 创建仓库，并设置令牌

![image-20210124221038187](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241656226.webp)

![image-20210124221144543](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241656066.webp)

然后点击设置，创建令牌

![image-20210124221557945](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241656159.webp)

然后会要求你输入账户密码，然后进入

![image-20210124221948910](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241656334.webp)

==注意：将令牌记住，后面有用，万一忘记可重新点回此界面，进行修改重新生成令牌或者删除重头来过。==

### PicGo图床设置

![image-20210124222515210](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241656377.webp)

### 配置Typora

![image-20210124222942752](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241657817.webp)

### 测验

总结：

1. 然后利用Typora写笔记就会发现图片地址变成Gitee仓库储存图片地址
2. 以前的笔记需要上传CSDN，需要重新复制，新建Typora文件，将之前图片地址转换成Gitee仓库储存图片地址
3. 某些博客对于gitee仓库并不友好，所有最好采用github仓库，然后通过cdn/fastly改变前缀

![image-20260326084440317](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/image-20260326084440317.webp)

```bash
https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main

https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main
```

