+++
date = '2025-09-14T11:35:37+08:00'
draft = false
title = 'IDEA简单学习'
slug = "xsjar30pqe"
description = "IDEA简单学习"
categories = ["📒tools"]
tags = ["idea"]
summary = "IDEA简单学习"
comments = true  
+++
﻿# Day-0-IDEA简单学习

### 1.设置主题

![image-20210203155447990](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621425_6463f8.webp)

### 2.编辑区的字体变大或者变小：（ctrl+鼠标滚轮）

![image-20210203155701923](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/32481c032626fa423c0fa32011790a1f_d641cd.webp)

### 3.鼠标悬浮在代码上有提示：

![image-20210203155807587](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621228_418dcd.webp)

### 4.自动导包和优化多余的包：

手动导包：快捷键：alt+enter
自动导包和优化多余的包：

![image-20210203155935111](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/0efe9b1f914b651c666dfd85b43453d0_fde84d.webp)

### 5.显示行号 ，  方法和方法间的分隔符：

![image-20210203160134360](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621910_81d340.webp)

### 6.忽略大小写，进行提示：

![image-20210203160331324](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621601_d43fcd.webp)

### 7.修改代码中注释的字体颜色：

![image-20210203160649569](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241621788_f794d3.webp)

8.修改类头的文档注释信息：注意：对新建的类才有效
/**

* @Auther: XXX
* @Date: ${DATE} - ${MONTH} - ${DAY} - ${TIME} 
* @Description: ${PACKAGE_NAME}
* @version: 1.0
*/

![image-20210203161132148](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622146_7968bb.webp)

### 8.自动编译：

![image-20210203161338692](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622228_7e3dca.webp)

### 9.常用快捷键

【1】创建内容：alt+insert

【2】main方法：psvm

【3】输出语句：sout

【4】复制行：ctrl+d

【5】删除行：ctrl+y

【6】代码向上/下移动：Ctrl + Shift + Up / Down

【7】搜索类：  ctrl+n

【8】生成代码  ：alt + Insert（如构造函数等，getter,setter,hashCode,equals,toString）

【9】百能快捷键 : alt + Enter （导包，生成变量等）

【10】单行注释或多行注释 ：  Ctrl + / 或 Ctrl + Shift + /

【11】重命名 shift+f6

【12】for循环  直接 ：fori   回车即可

【13】代码块包围：try-catch,if,while等  ctrl+alt+t

【14】显示代码结构  : alt + 7

【15】显示导航栏： alt +1 

【16】撤回：ctrl+z

【17】缩进：tab  取消缩进： shift+tab

### 10.常用的代码模板

【1】模板1： main方法：

​	main  或者 psvm

【2】模板2：输出语句：

​	sout   或者   .sout

一些变型：

​	soutp:打印方法的形参
​	soutm:打印方法的名字
​	soutv:打印变量

【3】模板3： 循环

​	普通for循环：   fori（正向）   或者   .fori （正向）   . forr(逆向)
​	增强for循环：  iter  或者  .for
​	（可以用于数组的遍历，集合的遍历）

【4】模板4： 条件判断

​	ifn 或者  .null ：判断是否为null  （if null）
​	inn 或者 .nn ：判断不等于null   (if not null)

【5】模板5： 属性修饰符：

​	prsf : private static final
​	psf  :public static final

### 11.常用断点调试快捷键

调试在开发中大量应用：
【1】Debug的优化设置：更加节省内存空间：
设置Debug连接方式，默认是Socket。 Shared memory是Windows 特有的一个属性，一般在Windows系统下建议使用此设置，
内存占用相对较少。

![image-20210203162123277](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622587_8c617b.webp)

【2】常用断点调试快捷键：

![image-20210203162236689](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622541_f92136.webp)一步一步的向下运行代码，不会走入任何方法中。
![image-20210203162310424](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622014_bc61f8.webp)一步一步的向下运行代码，不会走入系统类库的方法中，但是会走入自定义的方法中。
![image-20210203162323520](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622129_df6e74.webp)一步一步的向下运行代码，会走入系统类库的方法中，也会走入自定义的方法中。
![image-20210203162335089](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622889_93559a.webp)跳出方法
![image-20210203162358567](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241622944_55a559.webp)结束程序
![image-20210203162417023](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241623226_af6d8a.webp)进入到下一个断点，如果没有下一个断点了，就直接运行到程序结束。

![image-20210203162435472](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241623957_2f316b.webp) 在当前次取消未执行的断点。
