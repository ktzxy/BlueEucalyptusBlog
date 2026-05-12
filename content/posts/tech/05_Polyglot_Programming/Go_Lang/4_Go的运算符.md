+++
date = '2024-06-24T06:04:59+08:00'
draft = false
title = '4 Go的运算符'
slug = "znzw94r6va"
description = "4 Go的运算符"
categories = ["📒编程"]
tags = ["Go"]
summary = "4 Go的运算符"
comments = true  
series = ["Go语言系列"]
showSeries= true
+++
# Go的运算符

## 算数运算符

- +：相加
- -：相减
- *：相乘
- /：相除
- %：求余

在golang中， ++ 和 -- 只能单独使用，错误的写法如下

```go
var i int = 8
var a int
a = i++  // 错误，i++只能单独使用
a = i--  // 错误，i--只能单独使用
```

同时在golang中，没有 ++i这样的操作

```go
var i int = 1
++i  // 错误
```

正确的写法

```go
var i int = 1
i++ //正确
```

