+++
date = '2025-11-11T19:03:16+08:00'
draft = false
title = 'Day 04 java方法详解'
slug = "q6trfkl2e6"
description = "Day 04 java方法详解"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 04 java方法详解"
comments = true  
+++
﻿
# Day-04-java方法详解

### 何谓方法？

​	System.out.println(),那么它是什么呢?

​	调用系统类里的标准输出对象out中的方法println

​	Java方法是语句的集合，它们在一起执行一个功能。

​	方法是解决一类问题的步骤的有序组合

​	方法包含于类或对象中，方法和方法是并列的关系，所以我们定义的方法不能写到main方法中

​	方法在程序中被创建，在其他地方被引用

​	设计方法的原则:方法的本意是功能块，就是实现某个功能的语句块的集合。我们设计方法的时候，最好保持方法的原子性，**就是一个方法只完成1个功能**，这样利于我们后期的扩展。

### 方法的定义

Java的方法类似争其它语言的函数,是一段**用来完成特定功能的代码片段**，一股情况卜，定义一个方法包含以下语法:

**方法包含一个方法头和一个方法体。** 下面是一个方法的所有部分:

**修饰符**:修饰符，这是可选的，告诉编译器如何调用该方法。定义了该方法的访问类型。

**返回值类型∶**方法可能会返回值。returnValueType 是方法返回值的数据类型。有些方法执行所需的操作，但没有返回值。在这种情况下，returnValueType是关键字void。

**方法名:** 是方法的实际名称。方法名和参数表共同构成方法签名。

**参数类型:** 参数像是一个占位符。当方法被调用时，传递值给参数。这个值被称为实参或变量。参数列表是指方法的参数类型、顺序和参数的个数。参数是可选的，方法可以不包含任何参数。

​		形式参数:在方法被调用时用于接收外界输入的数据。

​		实参:调用方法时实际传给方法的数据。
**方法体:** 方法体包含具体的语句，定义该方法的功能。

![在这里插入图片描述](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241625160_e706d2.webp)


```java
//main 方法
public static void main(String[] args) {
    //实际参数：实际调用传递给它的参数
    int sum = add(1,2);
    System.out.println(sum);
}
//形式参数：用来定义作用的
public static int add(int a,int b){
    return a+b;
}
```

总结方法定义的格式：

1）修饰符：暂时使用public static

2）方法返回值类型：方法的返回值对应的数据类型

数据类型：可以是基本数据类型（byte，short，int，long，float，double，char，boolean）也可以是引用数据类型

3）方法名：见名之意，首字母小写，其余遵循驼峰命名

4）形参列表：方法定义的时候需要的形式参数：int a，int b 相当于告诉方法的调用者：需要传入几个参数，需要传入的参数的类型

​	实际参数：方法调用的时候传入的具体的参数

5）方法体：具体的业务逻辑代码

6）return方法返回值：

​	方法如果有返回值的话：return +方法返回值，将返回值返回到方法的调用处

​	方法没有返回值的话：return可以省略不写，并且方法的返回值类型为：void

### 方法调用

调用方法:对象名.方法名(实参列表)

Java支持两种调用方法的方式，根据方法是否返回值来选择。

当方法返回一个值的时候，方法调用通常被当做一个值。例如:

**方法的作用：提高代码的复用性**

```java
int larger = max(10，20);
```

如果方法返回值是void，方法调用一定是一条语句。

```java
System.out.println( "HelloWorld!");
```

### 方法的重载

重载就是在一个类中，有相同的函数名称，但形参不同的函数。

方法的重载的规则:
		方法名称必须相同。

​		参数列表必须不同（个数不同、或类型不同、参数排列顺序不同等)。

​		方法的返回类型可以相同也可以不相同。
​		仅仅返回类型不同不足以成为方法的重载。
实现理论:
​		方法名称相同时，编译器会根据调用方法的参数个数、参数类型等去逐个匹配,
以选择对应的方法，如果匹配失败，则编译器报错。

### 命令行传参

有时候你希望运行一个程序时候再传递给它消息。这要靠传递命令行参数给main()函数实现。

```java
public class CommandLine {
	public static void main(String args[]){
		for(int i=0; i<args.length; i++){
			system.out.println( "args[" + i + "]: " + args[i]);
        }
	}
}
```

### 可变参数

JDK 1.5开始，Java支持传递同类型的可变参数给一个方法。

在方法声明中，在指定参数类型后加一个省略号(.….)。

一个方法中只能指定一个可变参数，它必须是方法的最后一个参数。任何普通的参数必须在它之前声明。

```java
public static void main(String[] args) {
        //调用可变参数的方法
        printMax(1,2,3,5,46,45);
        printMax(new double[]{1,2,3});
    }
    public static void printMax( double... numbers){
        if ( numbers.length == 0) {
            System.out.println( "No argument passed" );
            return;
        }
        double result = numbers[0];

        //排序!
        for (int i = 1; i <numbers.length; i++){
            if ( numbers[i] >result) {
                result = numbers[i];
            }
        }
        System.out.println( "The max value is " + result);
    }
```

### 递归

A方法调用B方法，我们很容易理解!

递归就是: A方法调用A方法!就是自己调用自己

利用递归可以用简单的程序来解决一些复杂的问题。它通常把一个大型复杂的问题层层转化为一个与原问题相似的规模较小的问题来求

解，递归策略只需少量的程序就可描述出解题过程所需要的多次重复计算，大大地减少了程序的代码量。递归的能力在于用有限的语句来

定义对象的无限集合。

递归结构包括两个部分:

​		递归头:什么时候不调用自身方法。如果没有头，将陷入死循环。

​		递归体:什么时候需要调用自身方法。

```java
//5! 5的阶乘
public static void main(String[] args) {
    System.out.println(f(5));
}
public static int f(int n){
    if (n==1){
        return 1;
    }else {
        return n*f(n-1);
    }
}
```

### 面试题分析：

```java
public class demo4 {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;
        System.out.println("输出交换前的两个数："+a+"---"+b);
        changeNum(a,b);
        System.out.println("输出交换后的两个数："+a+"---"+b);

    }
    public static void changeNum(int num1,int num2){
        int t;
        t = num1;
        num1 = num2;
        num2 = t;
    }
}
```

![image-20210207185642165](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241625403_7a920f.webp)

# 
