+++
date = '2025-07-27T15:08:21+08:00'
draft = false
title = 'Day 02 java基础语法'
slug = "3qcgapmddw"
description = "Day 02 java基础语法"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 02 java基础语法"
comments = true  
+++
﻿
# Day-02-java基础语法

快捷键操作

psvm -->快速生成public static void main(String[] args) {}

sout -->快速生成System.out.println();

- 可能会遇到的问题

  1. 每个单词的大小不能出现问题，==Java是大小写敏感的==；
  2. 尽量使用英文；
  3. 文件名和类名必须保证一致，并且首字母大写；
  4. 符号使用的了中文。

- Java运行机制

  - 编译型
  - 解释型

### 注释：

Java中的注释有三种：

```java
注释：
    单行注释：
    //我是单行注释
    多行注释  
    /*我是多行注释*/
    文档注释
    /**
    *@description  HelloWrold
    *@Author 作者
    */
```

### 标识符：

关键字

Java所有的组成部分都需要名字。类名、变量名以及方法名都被称为标识符。

标识符注意点

所有的标识符都应该以**字母(A-Z或者a-z),美元符（$)、数字或者下划线(_)**开始
首字符之后可以是字母（A-Z或者a-z),美元符（$)、下划线(_)或数字的任何字符组合

**不能使用关键字作为变量名或方法名。**

**标识符是大小写敏感的**

合法标识符举例: age、$salary._value、__1_value

非法标识符举例:123abc、-salary、#abc

**遵照驼峰命名：**

类名：首字母大写，其余遵循驼峰命名

方法名，变量名：首字母小写，其余遵循驼峰命名

包名：全部小写，不遵循驼峰命名

- 关键字

| abstract  | boolean    | break        | byte       | case     |
| --------- | ---------- | ------------ | ---------- | -------- |
| catch     | char       | const        | class      | continue |
| default   | do         | double       | else       | extends  |
| final     | finally    | float        | for        | goto     |
| if        | implements | import       | instanceof | int      |
| interface | long       | native       | new        | package  |
| private   | protected  | public       | return     | short    |
| static    | strictfp   | super        | switch     | this     |
| throw     | throws     | transient    | try        | void     |
| volatile  | while      | synchronized |            |          |

### 数据类型

Java的数据类型分为两大类
	基本类型(primitive type)

​	引用类型(reference type)

![img](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241623163_9c2e67.webp)

![img](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241623591_4ac425.webp)

```java
//八大基本数据类型

        //整数

        int num1 = 10;
        byte num2 = 20;
        short num3 = 30;
        long num4 = 40L;  //long类型要在数字后面加上L

        //浮点数
        float num5 = 50.1F;  //float类型要在数字后面加上F，浮点数默认是double类型的。
        double num6 = 6.15234;
		double num7 = 314E-2;  //科学计数法

        //字符，java中无论：字母，数字，符号，中文都是字符类型的常量，都占用2个字节
        char name = 'A';

        //字符串,String 不是关键字，而是类
        String names = "张三";

        //布尔值
        boolean flag = true;
```

什么是字节

- 位（bit）：是计算机内部数据储存的最小单位，11001100是一个八位二进制数。
- 字节（byte）：是计算机中数据处理的基本单位，习惯上用大写B来表示，1B（byte字节）=8bit（位）。
- 字符：是指计算机中使用的字母、数字、字和符号。
  - 1bit表示1位
  - 1Byte表示一个字节1B=8b
  - 1024B=1KB
  - 1024KB=1M
  - 1024M=1G

```java
//整数拓展：  进制   二进制0b  十进制   八进制0     十六进制0x
        int i = 10;
        int i2 = 010; //八进制0
        int i3 = 0x10;   //十六进制   0~9 A~F 16

        //浮点数拓展
        //float 有限   离散   舍入误差   大约  接近但不等于
        //最好不要使用浮点数进行比较，因为他们字节不相同
        float f = 0.1f;  //0.1
        double d = 1.0/10; //0.1
        System.out.println(f==d);  //false
        System.out.println(f);
        System.out.println(d);

        //字符拓展
        char c1 = 'a';
        char c2 = '中';
        System.out.println(c1);
        System.out.println((int)c1);  //强制转换 97

        System.out.println(c2);
        System.out.println((int)c2);   //强制转换  20013

        //所有的字符本质还是数字


        //转义字符
        // \t  制表符
        // \n  换行
		// \b 退格
		// \r  将光标到本行开头：回车
        // \"  双引号
		// \'  单引号
		// \\ 反斜杠

        String a1 = new String("hello world");
        String b1 = new String("hello world");
        System.out.println(a1==b1);  //false

        String s1 = "hello world";
        String s2 = "hello world";
        System.out.println(s1==s2);  //true

        //布尔值扩展
        boolean flag = true;
        if (flag==true){}  //新手
        if (flag){};  //老手
=============================================
public static void main(String[] args){
    //定义整数类型的变量
    int num1 = 12; //默认情况下赋值就是十进制的情况
    int num1 = 012;  //前面加上0，这个值就是八进制的
    int num1 = 0x12;  //前面加上0x或者0X，这个值就是十六进制的
    int num1 = 0b12; //前面加上0b或者0B，这个值就是二进制的
}
```

### 类型转换

由于java是强类型语言，所以要进行有些运算的时候，需要用到类型转换

低------------------------------------------------------------------------>高

byte,short,char——>int——>long——>float——>double

```java
int i = 128;
        byte b = (byte)i; //内存溢出
        //强制转换  （类型）变量名  高---->低
        System.out.println(i);
        System.out.println(b);
        //自动转换  低---->高
        /*
        注意点：
        1.不能对布尔值进行转换
        2.不能把对象类型转换为不相干的类型
        3.在把高容量转换到低容量的时候，强制转换
        4.转换的时候可能存在内存溢出，或者精度问题
        * */
        System.out.println((int)23.7);
        System.out.println((int)-45.7f);

        //操作比较大的数的时候，注意溢出问题
        //JDK新特性，数字之间可以用下划线分割
        int money = 10_0000_0000;
        int year = 20;
        int total = money*year;  //-1474836480
        long total1 = money*year; //默认是int，转换之前就存在问题
        long total2 = money*(long)year; //先把一个数转换为long

		//在同一个表达式中，有多个数据类型的时候，应该如和处理：
		//多种数据类型参与运算的时候，整数类型，浮点类型，字符类型都可以参与运算，维度布尔类型不可以参与运算。
		//当一个表达式中有多种数据类型的时候，要找出当前表达式中级别最高的那个类型，然后其余的类型都转换为当前表达式中级别最高的类型进行计算。
		//	double a = 12+1294L+8.5F+3.81+'a';
					 = 12.0+1294.0+8.5+3.81+97.0
		//在进行运算的时候：
          左=右  ：直接赋值               
          左<右  ：强转
          左>右  ： 直接自动转换
       //一下情况属于特殊情形：对于byte，short，char类型来说，只要在他们的表述范围内，赋值的时候不需要进行
       //强转了直接复制即可。
        byte b = 12；
        byte b2 = (byte)280;                 
```

### 变量

Java是一种强类型语言，每个变量都必须声明其类型。
Java变量是程序中最基本的存储单元，其要素包括变量名，变量类型和作用域。
注意事项:
    每个变量都有类型，类型可以是基本类型，也可以是引用类型。
    变量名必须是合法的标识符。
    变量声明是一条完整的语句，因此每一个声明都必须以分号结束。

​	变量不可以重复定义

变量作用域：

​	类变量

​	实例变量

​	局部变量

作用范围就是离变量最近的{}

```java
//类变量  static
static double salary = 2500;

//属性：变量

//实例变量：从属于对象：如果不自行初始化，这个类型的默认值   0  0.0
//布尔值：默认是false
//除了基本类型，其余的默认值都是null
String name;
int age;

//main方法
public static void main(String[] args) {

    //局部变量，必须声明和初始化值
    int i = 10;
    System.out.println(i);

    //变量类型   变量名字=new Demo2();
    Demo2 demo2= new Demo2();
    System.out.println(demo2.age);
    System.out.println(demo2.name);

    //类变量  static
    System.out.println(salary);
}
//其他方法
public void add(){
    System.out.println();
}
```

### 常量：

常量(Constant):初始化(initialize)后不能再改变值!不会变动的值。
所谓常量可以理解成一种特殊的变量，它的值被设定后，在程序运行过程中不允许被改变。

常量名一般使用大写字符。

```java
//修饰符，不存在先后顺序  等同于final static double PI=3.14;
static final double PI=3.14;
public static void main(String[] args) {
    System.out.println(PI);
}
```

**变量的命名规范**
所有变量、方法、类名:见名知意
类成员变量:首字母小写和驼峰原则: monthSalary除了第一个单词以外，后面的单词首字母大写 lastname lastName
局部变量}首字母小写和驼峰原则
常量:大写字母和下划线:MAvALUE .V类名:首字母大写和驼峰原则: Man, 99M4a"146.56KBs

方法名:首字母小写和驼峰原则: run(), runRun()

### 运算符

```java
Java语言支持如下运算符:
二元运算符（+，-，*，/， %）  一元运算符（++，--）
- 算术运算符:+，-，*，/， %，++（自增），--（自减）
- 赋值运算符=
- 关系运算符:>，<，>=，<=，==，!= ,instanceof
- 逻辑运算符: &，|，^，&&，||，!
- 位运算符:&，|，^，~，>>，<<，>>>(了解! ! ! )
- 条件运算符?:
- 扩展赋值运算符:+=，-=，*=，/=
```

快捷键操作 Ctrl + D :复制当前行到下一行

```java
// ++ -- 自增  自减  一元运算符
//++单独使用的时候，无论放在前还是后，都是加1操作   a++;    ++a;
//将++参与到运算中，看++在前还是在后，如果++在前，先加1，后运算，如果++在后，先运算，后加1
/*
		int a = 5;
        System.out.println(a++);  //5
        System.out.println("此时的a为："+a);  //6
        System.out.println(++a);  //7
        System.out.println("此时的a为："+a);  //7
        System.out.println(a--);  //7
        System.out.println("此时的a为："+a);  //6
        System.out.println(a++ + ++a);  //14
        System.out.println("此时的a为："+a);  //8
        System.out.println(--a);  //7
        System.out.println("此时的a为："+a);  //7
        System.out.println(a--);  //7
        System.out.println("此时的a为："+a);  //6
        System.out.println(--a + a--);  //10
        System.out.println("此时的a为："+a);  //4   
*/
    int a = 3;
    int b = a++;  //执行完这行代码后，先给b赋值，再自增
    //a++ 意味着 a = a+1
    System.out.println(a);  //4
    //a = a + 1
    int c = ++a;  //执行完这行代码，先自增，再给b赋值

    System.out.println(a); //5
    System.out.println(b);  //3
    System.out.println(c);  //5

    //幂运算  2^3  2*2*2 = 8
     double pow = Math.pow(2,3);
    System.out.println(pow);
```

```java
//逻辑运算符
//逻辑与：& 规律：只要有一个操作数是false，那么结果一定是false
//短路与：&& 规律：效率高一些，只要第一个表达式是false，那么第二个表达式就不用计算了，结果一定是false
//逻辑或：| 规律：只要有一个操作数是true，那么结果一定是true
//短路或：|| 规律：效率高一些，只要第一个表达式是true，那么第二个表达式就不用计算了，结果一定是true
//逻辑非：！ 规律：相反结果
//逻辑异或：^ 规律：两个操作数相同，结果为false，不相同，结果为true
// 与（and）  或（or）  非（取反）
        boolean a = true;
        boolean b = false;

        System.out.println("a && b:"+(a&&b));  // false 逻辑与运算：俩个变量都为真，结果才为true
        System.out.println("a || b:"+(a||b));  // true  逻辑或运算：俩个变量有一个为真，则结果才为true
        System.out.println("! (a && b):"+!(a&&b));  // true 如果是真，则变为假，如果是假则变为真

        //短路运算
        int c = 5;
        boolean d = (c<4)&&(c++<4);
        System.out.println(d);  //false
        System.out.println(c);  //5
```

```java
/*
位运算符
A = 0011  1100
B = 0000  1101

A&B = 0000  1100
A|B = 0011  1101
A^B = 0011  0001
~B = 1111  0010

2*8 = 16   2*2*2*2
<< 左移
>> 右移
0000  0000   0
0000  0001   1
0000  0010   2
0000  0011   3
0000  0100   4
0000  1000   8
0001  0000   16
 */
System.out.println(2<<3);  //16
```

有符号左移

![image-20210203152638207](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241623612_7fc63f.webp)

有符号右移

![image-20210203153200629](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624606_d3768d.webp)

无符号右移

![image-20210203153307938](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624538_832f5d.webp)

&与

6&3=2

![image-20210203153602467](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624034_29b581.webp)

|或

6|3=7

![image-20210203153720828](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624110_158020.webp)

^异或

6^3=3

![image-20210203153859643](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624781_3be9a3.webp)

~反

~6=-7

![image-20210203154041652](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624518_b0ff7b.webp)

```java
拓展赋值运算符
		int a = 10;
        int b = 20;
        a+=b; //a = a + b
        a-=b; //a = a - b
        System.out.println(a);
        //字符串连接符  + ，String
        System.out.println(""+a+b); //1020 拼接
        System.out.println(a+b+""); //30
```

```java
a+=b 和 a=a+b区别:
(1)a+=b 可读性稍差  编译效率高  底层自动进行类型转换
(2)a=a+b 可读性好  编译效率低  手动进行类型转换
```

```java
条件运算符
//x ? y : z
        //如果x==true，则结果为y，否则结果为z
        int score = 80;
        String type = score >60 ? "不及格":"及格";
        System.out.println(type); //不及格
```

**运算符的优先级**

```java
赋值<三目<逻辑<关系<算术<单目
案例：
    5<6|'A'>'a'&&12*6<=45+23&&!true
   =5<6|'A'>'a'&&12*6<=45+23&&false
   =5<6|'A'>'a'&&72<=68&&false
   =true|false&&false&&false
   =true&&false&&false
   =false&&false
   =false
```



### 包机制

为了更好的组织类，java提供了包机制，用于区别类名的命名空间。

一般利用公司域名倒置作为包名；

为了能够使用某一个包的成员，我们需要在Java程序中明确导入该包，使用“import”语句可以达到此功能。

```java
import 包名.类名
    import 包名.*   //改包下的所有类
    在JDK中，不同功能的类都放在不同的包中，其中Java的核心类主要放在java包及其子包下，Java扩展的大部分类都放在javax包及其子包下。
	java.util:包含Java中大量工具类、集合类等，例如Arrays、List、Set等java.net:包含Java网络编程相关的类和接口。
	java.io:包含了Java输入、输出有关的类和接口。
	java.awt:包含用于构建图形界面(GUI）的相关类和接口。
    除了上面提到的常用包，JDK中还有很多其他的包，比如数据库编程的java.sql包、编写GUI的javax.swing包等，JDK中所有包中的类构成了Java类库。
```

### JavaDoc

javadoc命令是用来生成自己API文档的API
参数信息：

- @ author作者名
- @ version版本号
- @ since指明需要最早使用的jdk版本
- @ paran参数名
- @ return返回值情况
- @ throws异常抛出情况

### 练习题

```java
import java.util.Scanner;

public class Demo {
    public static void main(String[] args) {
        //求圆的周长和面积
        //一个变量被final修饰，这个变量就变成了一个常量，这个常量的值就不可变了
        final double pi = 3.14;
        Scanner sc = new Scanner(System.in);
        System.out.print("请录入一个半径：");
        int r = sc.nextInt();
        //圆的周长
        double c = 2*pi*r;
        System.out.println("圆的周长"+c);

        //圆的面积
        double s = pi*r*r;
        System.out.println("圆的面积"+s);
    }
}
```

### 练习题二

```java
public static void main(String[] args) {
    //实现：任意给出一个四位数，求出每位上的数字并输出
    Scanner sc = new Scanner(System.in);
    System.out.print("请输入一个四位数：");
    int num = sc.nextInt();
    int num1 = num%10;
    int num2 = num/10%10;
    int num3 = num/100%10;
    int num4 = num/1000;
    System.out.println("个位上的数字为"+num1);
    System.out.println("十位上的数字为"+num2);
    System.out.println("百位上的数字为"+num3);
    System.out.println("千位上的数字为"+num4);

}
```

