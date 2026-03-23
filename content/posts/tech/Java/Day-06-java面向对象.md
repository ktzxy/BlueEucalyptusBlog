+++
date = '2025-03-30T13:17:21+08:00'
draft = false
title = 'Day 06 java面向对象'
slug = "4qvv7e4syw"
description = "Day 06 java面向对象"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 06 java面向对象"
comments = true  
+++
﻿
# Day-06-java面向对象

### 什么是面向对象

面向对象编程(Object-Oriented Programming, OOP)

面向对象编程的本质就是:==以类的方式组织代码，以对象的组织(封装)数据。==

抽象

三大特性:

​		**封装**

​		**继承**

​		**多态**
从认识论角度考虑是先有对象后有类。对象，是具体的事物。类，是抽象的，是对对象的抽象

从代码运行角度考虑是先有类后有对象。类是对象的模板。

### 回顾方法及加深

**方法的定义**

**修饰符**

**返回类型**

==**break: 跳出switch，结束循环和return的区别方法名**==

**参数列表**

**异常抛出**

**方法的调用**
		**静态方法**

​		**非静态方法**

​		**形参和实参**

​		**值传递和引用传递**

​		**this关键字**

```java
//demo1  类
public class demo1 {

    //main方法
    public static void main(String[] args) {

    }
    /*
        修饰符  返回值类型   方法名（...）{
            //方法体
            return 返回值；
        }
     */
    public String sayHello(){
        return "Helloworld!";
    }

    public void printhell(){
        return;
    }

    public int max(int a,int b){
        return a>b ? a :b;  //三元运算符
    }
}
```

```java
package com.oop.Demo01;

public class demo3 {
        public static void say(){
            System.out.println("HelloWorld!");
        }
}

===================================================
package com.oop.Demo01;

public class demo2 {
    //调用
    public static void main(String[] args) {
        //静态方法  static
        demo3.say();
    }
}
```

```java
package com.oop.Demo01;

public class demo3 {
        public  void say(){
            System.out.println("HelloWorld!");
        }
}

=================================================
package com.oop.Demo01;

public class demo2 {
    public static void main(String[] args) {
        //非静态方法
        //实例化这个类  new
        //对象类型  对象名  =  对象值；
        demo3 a = new demo3();
        //调用
        a.say();
    }
}
```

```java
public class demo4 {
    public static void main(String[] args) {
        //实际参数和形式参数的类型要对应！
        int c = demo4.add(1,5);
        System.out.println(c);
    }
    public static int add(int a,int b){
        return a+b;
    }
}
```

```java
//值传递
public class demo5 {
    public static void main(String[] args) {
        int a =1;
        System.out.println(a);  //1

        demo5.change(a);
        System.out.println(a);  //1
    }

    //返回值为空
    public static void change(int a){
        a =10;
    }
}
```

```java
//引用传递：对象，本质还是值传递
public class demo6 {
    public static void main(String[] args) {
        Person person = new Person();
        System.out.println(person.name);  //null

        demo6.change(person);
        System.out.println(person.name);
    }
    public static void change(Person person){
        //person 是一个对象：指向的 -----> Person person = new Person();  这是一个具体的人，可以改变属性
        person.name = "张三";
    }
}
//定义一个Person类，有一个属性：name
class Person{
    String name;  //null
}
```

### 类与对象的关系

万事万物皆对象

类是一种抽象的数据类型,它是对某一类事物整体描述/定义,但是并不能代表某一个具体的事物.

对象是抽象概念的具体实例

​		张三就是人的一个具体实例,张三家里的旺财就是狗的一个具体实例。
​		能够体现出特点,展现出功能的是具体的实例,而不是一个抽象的概念。

### 面向对象的三个阶段

【1】 面向对象分析OOA -- Object Oriented Analysis

对象：张三，王五，。。

抽取一个类：人类

类里面有什么:

动词——》动态特性——》方法

名词——》静态特性——》属性

【2】面向对象设计OOD -- Object Oriented Design

先有类，再有对象：

类：人类：Person

对象：张三，李四

【3】面向对象编程OOP -- Object Oriented Programming

### 局部变量和成员变量的关系

区别1：代码中位置不同

​				成员变量：类中方法外定义的变量

​				局部变量：方法中定义的变量      代码块中定义的变量

区别2：代码的作用范围

​				成员变量：当前类的很多方法

​				局部变量：当前一个方法（当前代码块）

区别3：是否有默认值

​				成员变量：有

​				局部变量：没有

```java
基本类型						默认值
    boolean						Flase
    char						'\u0000'(null)
    byte						(byte)0
    short						(short)0
    int							0
    long						0L	
    float						0.0f
    double						0.0d
```



区别4：是否要初始化

​				成员变量：不需要

​				局部变量：一定需要

区别5：内存中位置不同

​				成员变量：堆内存

​				局部变量：栈内存

区别6：作用时间不同

​				成员变量：当前对象从创建到销毁

​				局部变量：当前方法从开始执行到执行完毕		

### 创建与初始化对象

**使用new关键字创建对象**

使用new关键字创建的时候，除了分配内存空间之外,还会给创建好的对象进行默认的初始化
以及对类中构造器的调用。

类中的构造器也称为构造方法, 是在进行创建对象的时候必须要调用的。并且构造器有以下俩个特点:

​		1.必须和类的名字相同

​		2.必须没有返回类型,也不能写void

**构造器必须要掌握**

```java
//学生类
public class Person {
    //属性：字段
    String name;
    int age;
    double height;
    double weight;
}
	public void eat(){
        int num = 10; //局部变量
        System.out.println("我喜欢吃饭");
    }
	public void sleep(String address){
        System.out.println("我在"+address+"睡觉");
    }
	public String introduce(){
        return "我的名字是"+name+",我的年龄是"+age+",我的身高是"+height+",我的体重是"+weight;
    }
```

### 构造器详解

```java
//java -->class
public class Person {
    //一个类即使什么都不写，它也会存在一个方法
    //显示的定义一个构造器

    String name;

    //实例化初始值
    //1.使用new关键字 ，本质是在调用构造器
    //2.用来初始化值
    public Person(){
        //this.name = "张三";
    }

    //有参构造:一旦定义了有参构造，就必须显示定义
    public Person(String name){
        this.name = name;
    }
}


=========================================
public class Application {

    public static void main(String[] args) {

        //new 实例化了一个对象
        Person person = new Person("zhangsan");
        System.out.println(person.name);
    }
}


alt+insert 快捷键生成有参和无参
/*
	创建对象的过程：
	1.第一次遇到一个类的时候，进行类的加载（只加载一次）
	2.创建对象，为这个对象在堆中开辟空间
	3.为对象进行属性的初始化动作，属性赋值都是默认的初始值
	4.new关键字调用构造器，执行构造方法，在构造器中对属性重新进行赋值
	
	new关键字实际上是在调用一个方法，这个方法叫做构造方法（构造器）
	调用构造器的时候，如果该类中没有写构造器，那么系统会默认给你分配一个构造器，只是我们看不到罢了
	可以自己显示 的将构造器编写出来：
	构造器的格式：
	[修饰符] 构造器的名字(){
	
	}
	构造器和方法的区别：
	1.没有方法的返回值类型
	2.方法体内部不能有return语句
	3.构造器的名字很特殊，必须跟类名一样
	
	构造器的作用：不是为了创建对象，因为在调用构造器之前，这个对象就已经创建好了，并且属性有默认的初始化的值。
	调用构造器的目的是给属性进行赋值操作的。
	
	注意：我们一般不会在空构造器中进行初始化操作，因为那样的话，每个对象的属性就一样了。
	实际上，我们只要保证空构造器的存在就可以了，里面的东西不用写
*/
```

### 构造器重载

```java
public class Person {

    //属性
    String name;
    int age;
    double height;

    //空构造器
    public Person(){

    }
    public Person(String name,int age,double height){  //构造器的重载

        //当形参名字和属性名字重名的时候，会出现就近原则
        //在要表示对象的属性前加上this.来修饰，因为this代表的就是你创建的那个对象
        this.name = name;
        this.age = age;
        this.height = height;
    }
    //方法：
    public void eat(){
        System.out.println("我喜欢吃饭");
    }
}
=====================================================
public class test {
    public static void main(String[] args) {
        /*
        1.一般保证空构造器的存在，空构造器中一般不会进行属性的赋值操作
        2.一般我们会重载构造器，在重载的构造器中进行属性赋值操作
        3.在重载构造器以后，假如空构造器忘写了，系统也不会给你分配默认的空构造器了，那么你要调用的话就会出错了。
        4.当形参名字和属性名字重名的时候，会出现就近原则
        在要表示对象的属性前加上this.来修饰，因为this代表的就是你创建的那个对象
         */
        Person p = new Person("张三",10,175.5);
        System.out.println(p.name);
        System.out.println(p.age);
        System.out.println(p.height);

    }
}
```

### 内存分析

![image-20210209174424353](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241626762_a6d829.webp)

![image-20210209175152547](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241626090_d8c875.webp)

![image-20210209175931106](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241626206_043b2a.webp)

### this的使用

this指代的就是当前对象

this关键字 用法：

（1）this可以修饰属性：

当属性名字和形参发生重名的时候，或者 属性名字和局部变量重名的时候，都会发生就近原则，所以如果我要是直接使用变量名字的话就指的是离的近的那个形参或者局部变量，这时候如果我想要表示属性的话，在前面要加上：this.修饰

如果不发生重名问题的话，实际上你要访问属性也可以省略this.

（2）this修饰方法

在同一个类中，方法可以互相调用，this.可以省略不写

(3)this可以修饰构造器：

同一个类中的构造器可以相互用this调用，注意：this修饰构造器必须放在第一行

```java
String name;
int age;
double height;
public Person(String name,int age,double height){
    this(name,age);
    this.height = height;
}
public Person(String name,int age){
    this(name);
    this.age = age;
}
public Person(String name){
    this.age = age;
}
```

### static关键字

static可以修饰：属性，方法，代码块，内部类

（一）

static修饰属性：

```java
public class demo1 {
    //属性：
    int id;
    static int sid;

    public static void main(String[] args) {
        //创建具体对象
        demo1 t1 = new demo1();
        t1.id = 10;
        t1.sid = 10;

        demo1 t2 = new demo1();
        t2.id = 20;
        t2.sid = 20;

        demo1 t3 = new demo1();
        t3.id = 30;
        t3.sid = 30;

        //读取属性的值：
        System.out.println(t1.id); //10
        System.out.println(t2.id);  //20
        System.out.println(t3.id);  //30

        System.out.println(t1.sid);  //30
        System.out.println(t2.sid);  //30
        System.out.println(t3.sid);  //30
    }
}
```

![image-20210209203005814](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241626808_7cd556.webp)

一般官方的推荐访问方式：可以通过类名.属性名的方式去访问：

![image-20210209203256877](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241626351_d1d6d8.webp)

总结：

（1）在类加载的时候一起加载入方法区中的静态域中

（2）先于对象存在

（3）访问方式：对象名.属性名    类名.属性名（推荐）

（4）应用场景：某些特定的数据想要在内存中共享，只有一块 ，这个情况下，就可以用static修饰的属性

属性：

静态属性：（类变量）

非静态属性：（实例变量）

static修饰方法：

```java
public class Demo2 {
    int id;
    static int sid;
    public void a(){
        System.out.println(id);
        System.out.println(sid);
        System.out.println("--------a");
    }
    //1.static和public都是修饰符，并列，没有先后顺序，
    public static void b(){
//        System.out.println(this.id); 4.在静态方法中不能使用this关键字
//        a(); 3.在静态方法中不能访问非静态的方法
//        System.out.println(id);  2.在静态方法中不能访问非静态的属性
        System.out.println(sid);
        System.out.println("-------b");
    }

    public static void main(String[] args) {
        //5.非静态的方法可以用对象名.方法名去调用
        Demo2 d = new Demo2();
        d.a();
        //6.静态的方法可以用   对象名.方法名去调用
        Demo2.b();
        d.b();

        //在同一个类中可以直接调用
        b();
    }

}
```

（二）

```java
//static
public class Student {
    private static int age;  //静态的变量   多线程
    private double score;  //非静态的变量

    public static void main(String[] args) {
        Student s1 = new Student();

        System.out.println(Student.age);
        System.out.println(s1.age);
        System.out.println(s1.score);
    }
}
```

```java
public class Person {
    //第二个执行  赋初始值
    {
        System.out.println("匿名代码块");
    }
    //第一个执行  只执行一次
    static {
        System.out.println("静态代码块");
    }
    //第三个执行
    public Person(){
        System.out.println("构造方法");
    }

    public static void main(String[] args) {
        Person person1 = new Person();
        System.out.println("===============");
        Person person2 = new Person();
    }
}
```

```java
//静态导入包
import static java.lang.Math.random;

public class Student{
    public void test(){
        System.out.println(random());
    }
}
```

### 代码块

【1】类的组成：属性，方法，构造器，代码块，内部类

【2】代码块分类：普通块，构造块，静态块，同步块（多线程）

【3】代码

```java
public class Demo3 {
    //属性
    int a;
    static int b;

    public Demo3() {  //空构造器
        System.out.println("这是空构造器");
    }

    //方法
    public void a(){
        System.out.println("-----a");
        {
            System.out.println("这是普通块");
            //普通块限制了局部变量的作用范围
            System.out.println("你好");
            int num = 10;
            System.out.println(num);
        }
    }
    public static void b(){
        System.out.println("-----b");
    }
    //构造块
    {
        System.out.println("这是构造块");
    }

    //静态块
    static {
        System.out.println("这是静态块");
        //在静态块中只能访问：静态属性，静态方法
        System.out.println(b);
        b();
    }

    //构造器
    public Demo3(int a){
        this.a = a;
    }

    public static void main(String[] args) {
        Demo3 d = new Demo3();
        d.a();
    }
}
```

总结：

代码块执行顺序：

最先执行静态块，只在类加载的时候执行一次，所以一般以后实战写项目：创建工厂，数据库的初始化信息都放入静态块。

一般用于执行一些全局性的初始化操作。

再执行构造块，（不常用）

再执行构造器

再执行方法中的普通块。

### 包

包名定义：

（1）名字全部小写

（2）中间用.隔开

（3）一般都是公司域名倒着写

（4）加上模块名字

（5）不能使用系统中的关键字

（6）包声明的位置一般都在非注释性代码的第一行：

（7）使用不同包下的类需要导包

（8）在java.lang包下的类，可以直接使用无需导包。

### 封装

我们程序设计要追求“高内聚，低耦合”。高内聚就是类的内部数据细节自己完成，不允许外部干涉；低耦合：仅暴露少量的方法给外部使用。

封装(数据的隐藏)
通常，应禁止直接访问一个对象中数据的实际表示，而应通过操作接口来访问，这称为信息隐藏。

```java
/*
封装的好处
    1.提高程序的安全性,保护数据
    2.隐藏代码的实现细节
    3.统一接口
    4.系统可维护增加了
*/
public class Application {
    public static void main(String[] args) {
        Student s1 = new Student();
        s1.setName("张三");
        System.out.println(s1.getName());
        s1.setId(1535453);
        System.out.println(s1.getId());
    }
}

======================================
public class Student {

    //属性私有
    private String name;  //名字
    private int id;  //学号
    private char sex;   // 性别

    //提供一些可以操作这个属性的方法!1/提供一些public 的get、 set方法
    //get  获得这个数据
    public String getName(){
        return this.name;
    }
    //set  给这个数据设置值
    public void setName(String name){
        this.name = name;
    }
    //alt + insert 直接获取get和set方法

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public char getSex() {
        return sex;
    }

    public void setSex(char sex) {
        this.sex = sex;
    }
}
```

进行封装：

（1）将属性私有化，被private修饰---加人权限修饰符

一旦加入了权限修饰符，其他人就不可以随意地获取这个属性

（2）提供public修饰的方法让别人来访问/使用

（3）即使外界可以通过方法来访问属性了，但是也不能随意访问，因为咱们在方法中可以加入限制条件。

### 继承

**类是对对象的抽象**

**继承是对类的抽象**

继承的本质是对某一批类的抽象，从而实现对现实世界更好的建模。

extends的意思是“扩展”。子类是父类的扩展。

JAVA中类只有单继承，没有多继承!

继承是类和类之间的一种关系。除此之外,类和类之间的关系还有依赖、组合、聚合等。

继承关系的俩个类，一个为子类(派生类).一个为父类(基类)。子类继承父类,使用关键字extends来表示。

子类和父类之间,从意义上讲应该具有"is a"的关系.

object类

super

方法重写

```java
//在Java中，所有的类。都默认直接或者间接继承Object
//父类
public class Person {
    public void say(){
        System.out.println("你好");
    }
}
===========================================
//派生类  子类
//子类继承了父类，就会拥有父类的全部方法!
public class Student extends Person {
}
===========================================
//派生类  子类
public class Teacher extends Person {
}
============================================
//测试类    
public class Application {
    public static void main(String[] args) {
        Student student = new Student();
        student.say();  #输出你好
    }
}
```

继承的好处：提供代码的复用性

父类定义的内容，子类可以直接拿过来用就可以了，不用代码上反复定义了

需要注意的点：

父类private修饰的内容，子类实际上也继承，只是因为封装的特性阻碍了直接调用，但是提供了间接调用的方式，可以间接调用。

总结：

(1)继承关系：

父类/基类/超类

子类/派生类

子类继承父类一定在合理的范围进行继承的   子类extends父类

（2）继承的好处：

1.提高了代码的复用性，父类定义的内容，子类可以直接拿过来用就可以了，不用代码上反复定义了

2.便于代码的扩展

3.为了以后多态的使用。是多态的前提。

（3）父类private修饰的内容，子类也继承过来了。

（4）一个父类可以有多个子类

（5）一个子类只能由一个直接父类。

但是可以间接的继承自其它类

（6）继承具有传递性：

Object类是所有类的根基父类。

所有的类都是直接或者间接的继承自Object

### **super**

指的是父类的

super可以修饰属性，可以修饰方法；

在子类的方法中，可以通过super.属性   super.方法 的方式，显示的去调用父类提供的属性，方法。在通常情况下，super.可以省略不写：

在特殊情况下，当子类和父类的属性重名时，你要想使用父类的属性，必须加上修饰符super.,只能通过super.属性来调用。

在特殊情况下，当子类和父类的方法重名时，你要想使用父类的方法，必须加上修饰符super.,只能通过super.方法来调用。

在这种情况下，super.就不可以省略。

super修饰构造器：

其实我们平时写的构造器的第一行都有：super（）----->作用：调用父类的空构造器，只是我们一般都省略不写。（所有构造器的第一行默认情况下都有super（），但是一旦你的构造器中显示的使用super调用了父类构造器，那么这个super（）就不会给你默认分配了。如果构造器中没有显示的调用父类构造器的话，那么第一行都有super（），可以省略不写）

如果构造器中已经显示的调用super父类构造器，那么它的第一行就没有默认分配的super（）；

在构造器中，super调用父类构造器和this调用子类构造器只能存在一个，两者不能共存：因为super修饰构造器要放在第一行，this修饰构造器也要放在第一行；

以后写代码构造器的生成可以直接使用IDEA提供的快捷键：alt+insert

```java
//父类
public class Person {
    protected String name = "张三";
}
==================================
//子类
public class Student extends Person {
    public String name = "李四";
    public void test(String name){
        System.out.println(name);  //王五
        System.out.println(this.name);  //李四
        System.out.println(super.name);  //张三
    }
}
===================================
//测试类
public class Application {
    public static void main(String[] args) {
        Student student = new Student();
        student.test("王五");
    }
}
```

```java
//父类
public class Person {
    public void print(){
        System.out.println("张三");
    }
}
==================================
//子类
public class Student extends Person {
    public void print(){
        System.out.println("李四");
    }
    public void test1(){
        print();  //李四
        this.print();  //李四
        super.print();  //张三
    }
}
===================================
//测试类
public class Application {
    public static void main(String[] args) {
        Student student = new Student();
        student.test1();
    }
}
```

```java
//父类
public class Person {
    public Person(){
        System.out.println("Person无参执行了");
    }
}
//子类
public class Student extends Person {
    public Student() {
        //隐藏代码：调用了父类的无参构造
        super();  //调用父类的构造器，必须要在子类构造器的第一行
        System.out.println("studnet无参实行了");
    }
}
//测试类
public class Application {
    public static void main(String[] args) {
        Student student = new Student();
    }
}
```

```txt
super注意点:
    1. super调用父类的构造方法，必须在构造方法的第一个
    2. super 必须只能出现在子类的方法或者构造方法中!
    3. super和 this不能同时调用构造方法!

vs this:
    代表的对象不同:
        this:本身调用者这个对象
        super:代表父类对象的应用
    前提
        this:没有继承也可以使用
        super:只能在继承条件才可以使用
    构造方法
        this();本类的构造
        super():父类的构造!
```

### 权限修饰符

```java
				同一个类	同一个包	子类		所有类
private			*
default			*			*
protected		*			*			*
public			*			*			*		*
```

**private**

![image-20210210092650471](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241627686_22e2be.webp)

![image-20210210092821315](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241627191_fb3fd7.webp)

**default：缺省修饰符**

**protected：**在不同改包下的子类中依然可以访问

### 方法重写

发生在子类和父类中，当子类对父类提供的方法不满意的时候，要对父类的方法进行重写。

重写有严格的格式要求：

子类的方法名字和父类必须一致，参数列表（个数，类型，顺序）也要和父类一致。

重载和重写的区别：

重载：在同一个类中，当方法名相同，形参列表不同的时候，多个方法构成了重载

重写：在不同的类中。子类对父类提供的方法不满意的时候，要对父类的方法进行重写

![image-20210210094952885](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241627056_31f195.webp)

```java
//父类
//重写都是方法的重写,和属性无关
public class B {
    public static void test(){
        System.out.println("B-->test()");
    }
}
======================================
//子类
//继承
public class A extends B {
    public static void test(){
        System.out.println("A-->test()");
    }
}
========================================
//测试类
public class Application {
    public static void main(String[] args) {
        //静态方法：   方法的调用只和定义的数据类型有关
        A a = new A();
        a.test();  //A

        //父类的引用指向了子类
        B b = new A();
        b.test(); //B
    }
}
//输出：
    A-->test()
	B-->test()
```

```java
//父类
//重写都是方法的重写,和属性无关
public class B {
    public static void test(){
        System.out.println("B-->test()");
    }
}
======================================
//子类
//继承
public class A extends B {
    //Override  重写
    @Override  //注解：有功能的注释！
    public void test() {
        System.out.println("A-->test()");
    }
}
========================================
//测试类
public class Application {
    //静态的方法和非静态的方法区别很大！
    //静态方法：  //方法的调用只和定义的数据类型有关
    //非静态：重写
    public static void main(String[] args) {

        A a = new A();
        a.test();  //A

        //父类的引用指向了子类
        B b = new A();
        b.test(); //B
    }
}
//输出：
    A-->test()
	A-->test()
```

```txt
重写:需要有继承关系，子类重写父类的方法!
    1．方法名必须相同
    2．参数列表必须相同
    3. 修饰符:范围可以扩大但不能缩小:  public>Protected>Default>private
    4．抛出的异常:范围，可以被缩小，但不能扩大; ClassNotFoundException --> Exception(大)
重写，子类的方法和父类必要一致;方法体不同!

为什么需要重写:
	1．父类的功能,子类不一定需要，或者不一定满足!
	Alt + Insert ; override;
```

### Object类

所有类都直接或间接的继承自Object类，Object类是所有Java类的根基类。

也就意味着所有的Java对象都拥有Object类的属性和方法。

如果在类的声明中未使用extends关键字指明其父类，则默认继承Object类。

equals作用：这个方法提供了对对象的内容是否相等的一个比较方式，对象的内容指的就是属性。

父类Object提供的equals就是在比较==地址，没有实际意义，我们一般不会直接使用父类提供的方法，而是在子类中对这个方法进行重写。

equals可以使用快捷键快速生成

### 类和类之间的关系

（1）将一个类作为另一个类中的方法的形参

（2）将一个类作为另一个类的属性

总结：

**一、继承关系**   继承指的是一个类（称为子类、子接口）继承另外的一个类（称为父类、父接口）的功能，并可以增加它自己的新功能的能力。在Java中继承关系通过关键字extends明确标识，在设计时一般没有争议性。在UML类图设计中，继承用一条带空心三角箭头的实线表示，从子类指向父类，或者子接口指向父接口。 

![img](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241627882_aff696.webp)
**二、实现关系**   实现指的是一个class类实现interface接口（可以是多个)的功能，实现是类与接口之间最常见的关系。在Java中此类关系通过关键字implements明确标识，在设计时一般没有争议性。在UML类图设计中，实现用一条带空心三角箭头的虚线表示，从类指向实现的接口。 

![img](assets/202310241627735_273513.jpeg)


**三、依赖关系**   简单的理解，依赖就是一个类A使用到了另一个类B，而这种使用关系是具有偶然性的、临时性的、非常弱的，但是类B的变化会影响到类A。比如某人要过河，需要借用一条船，此时人与船之间的关系就是依赖。表现在代码层面，为类B作为参数被类A在某个method方法中使用。在UML类图设计中，依赖关系用由类A指向类B的带箭头虚线表示。 

![img](assets/202310241627691_58a0d4.jpeg)
**四、关联关系** 关联体现的是两个类之间语义级别的一种强依赖关系，比如我和我的朋友，这种关系比依赖更强、不存在依赖关系的偶然性、关系也不是临时性的，一般是长期性的，而且双方的关系一般是平等的。关联可以是单向、双向的。表现在代码层面，为被关联类B以类的属性形式出现在关联类A中，也可能是关联类A引用了一个类型为被关联类B的全局变量。在UML类图设计中，关联关系用由关联类A指向被关联类B的带箭头实线表示，在关联的两端可以标注关联双方的角色和多重性标记。 

![img](assets/202310241627800_dd62d9.jpeg)
**五、聚合关系**   聚合是关联关系的一种特例，它体现的是整体与部分的关系，即has-a的关系。此时整体与部分之间是可分离的，它们可以具有各自的生命周期，部分可以属于多个整体对象，也可以为多个整体对象共享。比如计算机与CPU、公司与员工的关系等，比如一个航母编队包括海空母舰、驱护舰艇、舰载飞机及核动力攻击潜艇等。表现在代码层面，和关联关系是一致的，只能从语义级别来区分。在UML类图设计中，聚合关系以空心菱形加实线箭头表示。 

![img](assets/202310241627742_311700.jpeg)
**六、组合关系**   组合也是关联关系的一种特例，它体现的是一种contains-a的关系，这种关系比聚合更强，也称为强聚合。它同样体现整体与部分间的关系，但此时整体与部分是不可分的，整体的生命周期结束也就意味着部分的生命周期结束，比如人和人的大脑。表现在代码层面，和关联关系是一致的，只能从语义级别来区分。在UML类图设计中，组合关系以实心菱形加实线箭头表示。 

![在这里插入图片描述](assets/202310241627158_35f061.jpeg)

**七、总结**   对于继承、实现这两种关系没多少疑问，它们体现的是一种类和类、或者类与接口间的纵向关系。其他的四种关系体现的是类和类、或者类与接口间的引用、横向关系，是比较难区分的，有很多事物间的关系要想准确定位是很难的。前面也提到，这四种关系都是语义级别的，所以从代码层面并不能完全区分各种关系，但总的来说，后几种关系所表现的强弱程度依次为：组合>聚合>关联>依赖。

### 多态

即同一方法可以根据发送对象的不同而采用多种不同的行为方式。

一个对象的实际类型是确定的，但可以指向对象的引用的类型有很多

多态存在的条件

​		有继承关系

​		子类重写父类方法

​		父类引用指向子类对象

**注意:** **多态是方法的多态，属性没有多态性。**

instanceof  (类型转换)  引用类型

（1）先有父类，再有子类：--》继承   先有子类，再抽取父类----》泛化

（2）什么是多态：

​	多态就是多种状态：同一个行为，不同的子类表现出来不同的形态。

​	多态指的就是同一个方法调用，然后由于对象不同会产生不同的行为。

（3）多态的好处：

​	为了提高代码的扩展性，符合面向对象的设计原则：开闭原则。

​	开闭原则：指的就是扩展是 开放的，修改是关闭的。

​	注意：多态可以提高扩展性，但是扩展性没有达到最好，以后会学习反射

（4）多态的要素：

一：继承

二：重写

三：父类引用指向子类对象。

应用场合：父类当方法的形参，然后传入的是具体的子类的对象，然后调用同一个方法，根据传入的子类的不同展现出来的效果也不同，构成了多态。

```java
//父类
public class Person {
    public void run(){
        System.out.println("run");
    }
}
=================================
//子类
public class Student extends Person {
    @Override
    public void run() {
        System.out.println("run");
    }
    public void eat() {
        System.out.println("eat");
    }
}
===================================
//测试类
public class Application {
    public static void main(String[] args) {
        //可以指向的引用类型就不确定了：父类的引用指向子类

        //Student 能调用的方法都是自己的或者继承父类的！
        Student s1 = new Student();
        //Person 父类型，可以指向子类，但是不能调用子类独有的方法
        Person s2 = new Student();
        Object s3 = new Student();

        s2.run();  //子类重写了父类的方法，执行子类的方法
        s1.run();
        //对象能执行哪些方法，主要看对象左边的类型，和右边关系不大！
        ((Student) s2).eat();  //子类重写了父类的方法，执行子类的方法
        s1.eat();
    }
}

多态注意事项:
1.多态是方法的多态，属性没有多态
2．父类和子类，有联系 类型转换异常! ClassCastException !
3．存在条件:继承关系，方法需要重写，父类引用指向子类对象!    Father f1 = new Son( );

    1. static方法,属于类，它不属于实例
    2.final常量;
    3. private方法;
```

### instanceof  

```java
//Object >Person >Student
//Object > person >Teacher
//Object > String
//System.out.println(X instanceof Y); //能不能编译通过

Object object =  new Student();
System.out.println(object instanceof Student);  //true
System.out.println(object instanceof Person);  //true
System.out.println(object instanceof Teacher);  //false
System.out.println(object instanceof Object);  //true
System.out.println(object instanceof String);  //false

System.out.println("========================");
Person person =  new Student();
System.out.println(person instanceof Student);  //true
System.out.println(person instanceof Person);  //true
System.out.println(person instanceof Teacher);  //false
System.out.println(person instanceof Object);  //true
//System.out.println(person instanceof String);  //编译报错

System.out.println("========================");
Student student =  new Student();
System.out.println(student instanceof Student);  //true
System.out.println(student instanceof Person);  //true
//System.out.println(student instanceof Teacher);  //编译报错
System.out.println(student instanceof Object);  //true
//System.out.println(student instanceof String);  //编译报错
```

### final修饰符

【1】修饰变量

```java
public static void main(String[] args) {
        //第一种情况：
        //final修饰一个变量，变量的值不可以改变，这个变量也变成了一个字符常量，约定俗称的规定：名字大写
        final int A = 10;  //final 修饰基本数据类型
//        A = 20; //报错：不可以修改值
        //第二种情况:
        final Dog d = new Dog();  //final修饰引用数据类型，难么地址值就不可以改变
//        d = new Dog(); 地址值不可以更改
        //d对象的属性依然可以改变：
        d.age = 10;
        d.weight = 12.5;

        //第三种情况：
        final Dog d2 = new Dog();
        a(d2);
        //第四种情况：
        b(d2);

    }
    public static void a(Dog d){
        d = new Dog();
    }
    public static void b(final Dog d){ //b被final修饰，指向不可以改变
//        d = new Dog();
    }
```

【2】修饰方法

final修饰方法，那么这个方法不可以被该类的子类重写：

【3】修饰类

final修饰类，代表没有子类，该类不可以被继承：

一旦一个类被final修饰，那么里面的方法也没有必要用final修饰了（final省略不写）

### 类型转化

```
public class Application {
    public static void main(String[] args) {
        //类型之间的转化：父   子
        //高          低
        Person a = new Student();
        ((Student) a).go();
    }
}

/*
    1.父类引用指向子类的对象
    2.把子类转换为父类，向上转型；
    3.把父类转换为子类，向下转型； 强制转换
    4.方便方法的调用，减少重复的代码！简洁
 */
```

### 抽象类

**abstract**修饰符可以用来修饰方法也可以修饰类,如果修饰方法,那么该方法就是抽象方法;如果修饰类,那么该类就是抽象类。

抽象类中可以没有抽象方法,但是有抽象方法的类一定要声明为抽象类。

抽象类,不能使用new关键字来创建对象,它是用来让子类继承的。

抽象方法,只有方法的声明,没有方法的实现,它是用来让子类实现的。

子类继承抽象类,那么就必须要实现抽象类没有实现的抽象方法,否则该子类也要声明为抽象类。

抽象类作用：

​	在抽象类中定义抽象方法，目的是为了对子类提供一个通用的模板，子类可以在模板的基础上进行开发，先重写父类的抽象方法，然后可以扩展子类自己的内容。抽象类涉及避免了子类设计的随意性，通过抽象类，子类的设计变得更加严格，进行某些程度上的限制。

```java
//abstract 抽象类 :类 extends：单继承   （接口可以多继承）
public abstract class Action {
    //abstract ，抽象方法，只有方法名字，没有方法的实现
    public abstract void doSomething();

    //1.不能new这个抽象类，只能靠子类去实现它； 约束！
    //2.抽象类中可以写普通的方法
    //3.抽象方法必须在抽象类中
}
/*
1.在一个类中，会有一类方法，子类对这个方法非常满意，无需重写，直接使用
2.在一个类中，会有一类方法，子类对这个方法非常不满意，会对这个方法进行重写
3.一个方法的方法体去掉，然后被abstract修饰，那么这个方法就变成了一个抽象方法
4.一个类中如果有方法是抽象方法，那么这个类也要变成一个抽象类。
5.一个抽象类中可以有0——n个抽象方法
6.抽象类可以被其它类继承
7.一个类继承一个抽象类，那么这个类可以变成抽象类
8.一般子类不会加abstract修饰，一般会让子类重写父类中的抽象方法
9.子类继承抽象类，就必须重写全部的抽象方法
10.子类如果没有重写父类全部的抽象方法，那么子类也可以变成一个抽象类
11.创建抽象类的对象：--->抽象类不可以创建对象
12.创建子类对象
13.多态的写法：父类只想引用子类对象:
*/
```

（1）抽象类不能创建对象，那么抽象类中是否有构造器？

抽象类中一定有构造器。构造器的作用  给子类初始化对象的时候要先super调用父类的构造器。

（2）抽象类是否可以被final修饰？

不能被final修饰，因为抽象类设计的初衷就是给子类继承用的。要是被final修饰了这个抽象类了，就不存在继承了，就没有子类。

### 接口

普通类:只有具体实现

抽象类:具体实现和规范(抽象方法)都有!

接口:只有规范!  约束和实现分离：面对接口编程

接口就是规范，定义的是一组规则

声明类的关键字是class,声明接口的关键字是interface

```java
/*
1.类是类，接口是接口，他们是同一层次的概念。
2.接口中没有构造器
3.接口如何声明：interface
4.在JDK1.8之前，接口中只有两部分内容：
	常量：固定修饰符：public static final
	抽象方法：固定修饰符：public abstract
注意：修饰符可以省略不写，IDE会自动补全
5.类和接口的关系是什么？实现关系  类实现接口
6.一旦实现一个接口，那么实现类要重写接口中的全部抽象方法
7.如果没有全部重写抽象方法，那么这个类可以变成一个抽象类
8.java只有单继承，java还有多实现
一个类继承其它类，只能直接继承一个父类
但是实现类实现接口的话，可以实现多个接口
9.先继承，后实现
10.接口不能创建对象
11.接口如何访问？ 接口.常量名  实现类.常量名  创建实现类的对象
12.在JDK1.8之后，新增非抽象方法：
	被public default修饰的非抽象方法：
		注意：default修饰符必须要写
			实现类中要是想重写接口中的非抽象方法，那么default修饰符必须不能加，否则出错。
	静态方法：
		注意：static不可以省略不写
			静态方法不能重写
疑问:为什么要在接口中加入非抽象方法???
如果接口中只能定义抽象方法的话，那么我要是修改接口中的内容，那么对实现类的影响太大了，所有实现类都会受到影响现在在接口中加入非抽象方法，对实现类没有影响，想调用就去调用即可。			
*/
//interface  定义的关键字，接口都需要有实现类
public interface uerService {
    //接口中的所有定义其实都是抽象的  public abstract
    void add(String name);
    void delete(String name);
}

========================================
public interface demo {
}
========================================
//抽象类 ：extends
//类  可以实现接口  implements  接口
//实现了接口的类，就需要重写接口中的方法

//多继承  利用接口可以实现多继承
public class userServicempl implements uerService,demo{
    @Override
    public void add(String name) {

    }

    @Override
    public void delete(String name) {

    }
}

/*
作用:
定义规则，只是跟抽象类不同地方在哪？他是接口不是类。
接口定义好规则之后，实现类负责实现即可。

继承：子类对父类的继承
实现：实现类对接口的实现
继承：手机  extends 照相机   “is a” 的关系，手机是一个照相机
实现：手机 implements 拍照功能  “has-a”的关系，手机具备照相的能力

多态的应用场合:
(1)父类当做方法的形参，传入具体的子类的对象
(2)父类当做方法的返回值，返回的是具体的子类的对象
(3)接口当做方法的形参，传入具体的实现类的对象
(4)接口当做方法的返回值，返回的是具体的实现类的对象

接口和抽象类的区别：
1.默认方法：抽象类可以有默认的方法实现，接口中不存在方法的实现，接口是完全抽象的。
2.实现方式：子类使用extends关键字来继承抽象类，如果子类不是抽象类，子类需要提供抽象类中所声明的所有方法的实现。而接口的子类使用implements来实现接口，需要提供接口中声明的所有方法实现。
3.构造函数：抽象类中可以有构造函数，接口中不能。
4.和正常类区别：抽象类不能被实例化，接口则是完全不同的类型。
5.访问修饰符：抽象方法可以有public,protected和default等修饰，接口默认是public，不能使用其他修饰符。
6.多继承：一个子类只能继承在一个抽象类，而一个子类可以实现多个接口。
7.添加新方法：想在抽象类中添加新方法，可以提供默认的实现，因此可以不修改子类现有的代码。如果往接口中添加新方法，则子类中需要实现该方法。

 */
```

### 内部类

内部类就是在一个类的内部在定义一个类，比如，A类中定义一个B类，那么B类相对A类来说就称为内部类，而A类相对B类来说就是外部类了。

1.类的组成:属性，方法，构造器，代码块（普通块，静态块，构造块，同步块），内部类

2.一个类的内部的类叫内部类，内部类:Inner外部类:Outer

3.内部类:成员内部类（静态的，非静态的)和﹑局部内部类（位置:方法内，块内，构造器内）

4.成员内部类:
	里面属性，方法，构造器等
	修饰符: private，default，protect，public，final, abstract

5.内部类可以访问外部类的内容

6.静态内部类中只能访问外部类中被static修饰的内容

7.外部类想要访问内部类的东西，需要创建内部类的对象然后进行调用

8.在局部内部类中访问到的变量必须是被 final修饰的



```java
public class Outer {
    private int id=10;
    public void out(){
        System.out.println("这是外部类的方法");
    }
    public  class Inner{
        public void in(){
            System.out.println("这是内部类的方法");
        }

        //获得外部类的私有属性和私有方法
        public void getID(){
            System.out.println(id);
        }
    }
}
=================================
//测试
public class Application {
    public static void main(String[] args) {
        Outer outer = new Outer();
        //通过这个外部类来实例化内部类
        Outer.Inner inner = outer.new Inner();
        inner.in();
        inner.getID();

    }
}
```

局部内部类

```java
public void method(){
    final int num = 10;
    class A{
        public void a(){
            System.out.println(num);
        }
    }
}
//2.如果类B在整个项目中只使用一次，那么就没有必要单独创建一个B类，使用内部类就可以了
public Comparable method2(){
    class B implements Comparable{
        @Override
        public int compareTo(Object o) {
            return 100;
        }
    }
    return new B();
}
public Comparable method3(){
    //匿名内部类
    return new Comparable() {
        @Override
        public int compareTo(Object o) {
            return 200;
        }
    };
}
public void eat(){
    Comparable com = new Comparable(){
        @Override
        public int compareTo(Object o) {
            return 300;
        }
    };
    System.out.println(com.compareTo("abc"));
}
```

### 项目

![image-20210215180014291](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241627965_42be68.webp)

```java
package com.zy.test1;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 * 父类：Pissa类
 */
public class Pizza {
    //属性
    private String name;
    private int size;
    private int price;
    //方法

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    //展示披萨信息：
    public String showPizza(){
        return "披萨的名字是："+name+"\n披萨的大小是："+size+"寸\n披萨的价格是："+price+"元";
    }

    //构造器

    public Pizza() {
    }

    public Pizza(String name, int size, int price) {
        this.name = name;
        this.size = size;
        this.price = price;
    }

}
```

```java
package com.zy.test1;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class FruitsPizza extends Pizza {
    //属性
    private String burdening;

    public String getBurdening() {
        return burdening;
    }

    public void setBurdening(String burdening) {
        this.burdening = burdening;
    }
    //构造器

    public FruitsPizza() {
    }

    public FruitsPizza(String name, int size, int price, String burdening) {
        super(name, size, price);
        this.burdening = burdening;
    }
    //重写父类showPizza方法：

    @Override
    public String showPizza() {
        return super.showPizza()+"你要加入的水果是："+burdening;
    }
}
```

```java
package com.zy.test1;

import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 * 工厂类的提取
 */
public class PizzaStore {
    public static Pizza getPizza(int choice){
        Scanner sc = new Scanner(System.in);
        Pizza p = null;
        switch (choice) {
            case 1: {
                System.out.println("请录入培根的克数：");
                int weight = sc.nextInt();
                System.out.println("请录入培根的大小：");
                int size = sc.nextInt();
                System.out.println("请录入培根的价格：");
                int price = sc.nextInt();
                //将录入的信息封装为培根披萨的对象
                BaconPizza bp = new BaconPizza("培根披萨", size, price, weight);
                p = bp;
            }
            break;
            case 2: {
                System.out.println("请录入你想要加入的水果：");
                String burdening = sc.next();
                System.out.println("请录入培根的大小：");
                int size = sc.nextInt();
                System.out.println("请录入培根的价格：");
                int price = sc.nextInt();
                //将录入的信息封装为水果披萨的对象
                FruitsPizza fp = new FruitsPizza("水果披萨", size, price, burdening);
                p = fp;
            }
            break;
        }
        return p;
    }
}
```

```java
package com.zy.test1;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class BaconPizza extends Pizza {
    //属性
    private int weight;

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }
    //构造器

    public BaconPizza() {
    }

    public BaconPizza(String name, int size, int price, int weight) {
        super(name, size, price);
        this.weight = weight;
    }
    //重写父类showPizza方法：

    @Override
    public String showPizza() {
        return super.showPizza()+"\n培根的克数："+weight+"克";
    }
}
```

```java
package com.zy.test1;

import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class Test {
    public static void main(String[] args) {
        //选择购买披萨
        Scanner sc = new Scanner(System.in);
        System.out.println("请选择你想要购买的披萨（1.培根披萨  2.水果披萨）：");
        int choice = sc.nextInt();
        //通过工厂获取披萨：
        Pizza pizza = PizzaStore.getPizza(choice);
        System.out.println(pizza.showPizza());
    }
}
```

# 
