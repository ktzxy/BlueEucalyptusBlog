+++
date = '2026-01-16T09:16:03+08:00'
draft = false
title = 'Day 07 java异常'
slug = "i7xvcz8xnq"
description = "Day 07 java异常"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 07 java异常"
comments = true  
+++
﻿
# Day-07-java异常

什么是异常

实际工作中，遇到的情况不可能是非常完美的。比如:你写的某个模块，用户输入不一定符合你的要求、你的程序要打开某个文件，这个文件可能不存在或者文件格式不对，你要读取数据库的数据，数据可能是空的等。我们的程序再跑着，内存或硬盘可能满了。等等。

软件程序在运行过程中，非常可能遇到刚刚提到的这些异常问题，我们叫异常，英文是:Exception，意思是例外。这些，例外情况，或者叫异常，怎么让我们写的程序做出合理的处理。而不至于程序崩溃。

异常指程序运行中出现的不期而至的各种状况,如:文件找不到、网络连接失败、非法参数等。异常发生在程序运行期间,它影响了正常的程序执行流程。

### 简单分类

要理解Java异常处理是如何工作的，你需要掌握以下三种类型的异常:

检查性异常:最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。例如要打开一个不存在文件时，一个异常就发生了，这些异常在编译时不能被简单地忽略。

运行时异常:运行时异常是可能被程序员避免的异常。与检查性异常相反，运行时异常可以在编译时被忽略。

错误:错误不是异常，而是脱离程序员控制的问题。错误在代码中通常被忽略。例如，当栈溢出时，一个错误就发生了，它们在编译也检查不到的。

### 层次结构

![image-20210215204604180](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241628904_c97bb7.webp)

### 异常体系结构

Java把异常当作对象来处理，并定义一个基类java.lang.Throwable作为所有异常的超类。在Java API中已经定义了许多异常类，这些异常类分为两大类，错误Error和异常Exception.

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241628392_8c060a.webp)

### Error

Error类对象由Java虚拟机生成并抛出，大多数错误与代码编写者所执行的操作无关

Java虚拟机运行错误(Virtual MachineError)，当JVM不再有继续执行操作所需的内存资源时，将出现OutOfMemoryError。这些异常发生时，Java虚拟机(JVM)一般会选择线程终止;

还有发生在虚拟机试图执行应用时，如类定义错误(NoClassDefFoundError)、链接错误(LinkageError)。这些错误是不可查的，因为它们在应用程序的控制和处理能力之外，而且绝大多数是程序运行时不允许出现的状况。

### Exception

在Exception分支中有一个重要的子类RuntimeException(运行时异常)
		ArraylndexOutOfBoundsException(数组下标越界)
		NullPointerException(空指针异常)
		ArithmeticException(算术异常)
		MissingResourceException(丢失资源)
		ClassNotFoundException(找不到类）等异常，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。

这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生;

Error和Exception的区别: Error通常是灾难性的致命的错误，是程序无法控制和处理的，当出现这些异常时，Java虚拟机(JVM)一般会选择终止线程; Exception通常情况下是可以被程序处理的，并且在程序中应该尽可能的去处理这些异常。

### 异常处理机制

抛出异常
捕获异常

异常处理五个关键字
try、catch、finally、throw、throws

```java
public class demo1 {
    public static void main(String[] args) {
        int a = 1;
        int b = 0;

        //Ctrl + Alt + T
        try{
            System.out.println(a/b);
        }catch (ArithmeticException e){  //catch  捕获异常
            System.out.println("程序出现异常，变量b不能为0");
        }finally {  //处理善后工作
            System.out.println("finally");
        }

        //finally  可以不要finally ， 假设IO ，资源 ，关闭
    }
}
```

```java
public class demo2 {
    public static void main(String[] args) {
        try {
            new demo2().test(1,0);
        } catch (ArithmeticException e) {
            e.printStackTrace();
        }
    }

    //假设这方法中，处理不了这个异常。方法上抛出异常
    public void test(int a,int b) throws ArithmeticException{
        if (b==0){  //throw throws
            throw new ArithmeticException();  //主动抛出异常，一般在方法中使用
        }
    }
}
```

一：在什么情况下，try-catch后面的代码不执行?

​		1.throw抛出异常的情况

​		2.catch中没有正常的进行异常捕获

​		3.在try中遇到return

二：怎么样才可以将try-catch后面的代码必须执行?

​		只要将必须执行的代码放入finally中，那么这个代码无论如何一定执行。

三：return和finally执行顺序?

​		先执行finally最后执行return

四：什么代码会放在finally中呢?

关闭数据库资源，关闭IO流资源，关闭socket资源。

五：有一句话代码很厉害，它可以让finally中代码不执行!

​		System.exit(0);//终止当前的虚拟机执行

六：try中出现异常以后，将异常类型跟catch后面的类型依次比较，按照代码的顺序进行比对，执行第一个与异常类型匹配的catch语句

七：一旦执行其中一条catch语句之后，后面的catch语句就会被忽略了!

八：在安排catch语句的顺序的时候，一般会将特殊异常放在前面(并列)，一般化的异常放在后面。先写子类异常，再写父类异常。

九：在JDK1.7以后，异常新处理方式:可以并列用|符号连接:

十：throw和throws的区别:

​	(1)位置不同:

​		throw:方法内部

​		throws:方法的签名处，方法的声明处

​	(2)内容不同:

​		throw+异常对象(检查异常，运行时异常)

​		throws+异常的类型(可以多个类型，用，拼接)

​	(3)作用不同:
​		throw:异常出现的源头，制造异常。
​		throws:在方法的声明处，告诉方法的调用者，这个方法中可能会出现我声明的这些异常。然后调用者对这个异常进行处理:要么自己处理要么再继续向外抛出异常

十一：重载和重写的区别:
	重载:在同一个类中，当方法名相同，形参列表不同的时候多个方法构成了重载
	重写:在不同的类中，子类对父类提供的方法不满意的时候，要对父类的方法进行重写。

![image-20210215205724248](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241628837_d46b4d.webp)

### 自定义异常

使用Java内置的异常类可以描述在编程时出现的大部分异常情况。除此之外，用户还可以自定义异常。用户自定义异常类，只需继承Exception类即可。

在程序中使用自定义异常类，大体可分为以下几个步骤:
		1．创建自定义异常类。
		2．在方法中通过throw关键字抛出异常对象。
		3.如果在当前抛出异常的方法中处理异常，可以使用try-catch语句捕获并处理;否则在方法
的声明处通过throws关键字指明要抛出给方法调用者的异常，继续进行下一步操作。

​		4。在出现异常方法的调用者中捕获并处理异常。

```java
/自定义的异常类
public class MyException extends Exception{
    //传递数字>10;
    private int detail;

    public MyException(int a){
        this.detail = a;
    }
    //toString  Alt + Insert

    @Override   //异常的打印信息
    public String toString() {
        return "MyException{" +
                "detail=" + detail +
                '}';
    }
}
===============================
public class demo3 {
    static void test(int a) throws MyException{

        System.out.println("传递的参数为："+a);

        if (a>10){
            throw new MyException(a);
        }
        System.out.println("OK");
    }

    public static void main(String[] args) {
        try {
            test(11);
        } catch (MyException e) {
            System.out.println("MyException is "+e);
        }
    }
}
```

### 实际应用中的经验总结

处理运行时异常时，采用逻辑去合理规避同时辅助try-catch处理

在多重catch块后面，可以加一个catch (Exception)来处理可能会被遗漏的异常

对于不确定的代码，也可以加上try-catch，处理潜在的异常

尽量去处理异常，切忌只是简单地调用printStackTrace()去打印输出

具体如何处理异常，要根据不同的业务需求和异常类型去决定

尽量添加finally语句块去释放占用的资源

# 
