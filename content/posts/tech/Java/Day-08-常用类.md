+++
date = '2025-02-14T13:10:44+08:00'
draft = false
title = 'Day 08 常用类'
slug = "7mrznf9d6j"
description = "Day 08 常用类"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 08 常用类"
comments = true  
+++
﻿# Day-08-常用类

**什么是包装类:**
以前定义变量，经常使用基本数据类型，
对于基本数据类型来说，它就是一个数，加点属性，加点方法，加点构造器,将基本数据类型对应进行了一个封装，产生了一个新的类，---》包装类。int,byte.....--->基本数据类型
包装类--->引用数据类型

**对应关系**

![image-20210215213419566](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241628076_73e319.webp)

**已经有基本数据类型了，为什么要封装为包装类?**

(1) java语言面向对象的语言，最擅长的操作各种各样的类。

(2)以前学习装数据的---》数组，int[] String[] double[]Student[]以后学习的装数据的--

->集合，有一个特点，只能装引用数据类型的数据

**是不是有了包装类以后就不用基本数据类型了?**

不是。

### Integer

【1】直接使用，无需导包

![image-20210217161003738](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241628108_20337a.webp)

【2】类的继承关系

![image-20210217161043650](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629677_4cfc0d.webp)

【3】实现接口

![image-20210217161204797](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629917_42960c.webp)

【4】这个类被final修饰，那么这个类不能有子类，不能被继承

![image-20210217161340952](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629236_bc9280.webp)

【5】包装类是对基本数据类型的封装：对int类型封装产生了Integer

![image-20210217161515176](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/fd3eb3e14972561b1750cc77b3e7065b_222512.webp)

【6】属性

```java
//属性
System.out.println(Integer.MAX_VALUE);
System.out.println(Integer.MIN_VALUE);
//“物极必反”
System.out.println(Integer.MAX_VALUE+1);
System.out.println(Integer.MIN_VALUE-1);
```

【7】构造器（发现没有空构造器）

（1）int类型作为构造器的参数：

```java
Integer i1 = new Integer(10);
System.out.println(i1.toString()); //10
```

![image-20210217163602384](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629191_a0d97d.webp)

（2）String类型作为构造器的参数

```java
Integer i2 = new Integer("12");
System.out.println(i2); //12
Integer i3 = new Integer("abc");
System.out.println(i3);  //NumberFormatException: For input string: "abc"
```

![image-20210217163829934](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629408_23be7b.webp)

【8】包装类特有的机制：自动装箱  自动拆箱

```java
//自动装箱：int--->Integer
Integer i = 10;
System.out.println(i);
//自动拆箱：Integer--->int
Integer i1 = new Integer(10);
int num = i1;
System.out.println(num);
```

自动装箱 自动拆箱 是从JDK1.5以后新出的特性

自动装箱 自动拆箱 ：将基本数据类型和包装类进行快速的类型转换。

【9】常用方法：

valueOf方法的底层：

![image-20210217171307108](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629585_e40513.webp)

```java
//comparable:只返回三个值：0  -1   1
Integer i1 = new Integer(12);
Integer i2 = new Integer(13);
System.out.println(i1.compareTo(i2));  //-1   return (x < y) ? -1 : ((x == y) ? 0 : 1)

//equals：Integer对Object中equals方法进行了重写，比较的是底层封装的那个value值。
//Integer对象是通过new关键自创建的对象：
Integer i3 = new Integer(12);
Integer i4 = new Integer(12);
System.out.println(i3 == i4); //false 因为==比较的是两个对象的地址
boolean flag = i3.equals(i4);
System.out.println(flag);  //true

//Integer对象通过自动装箱来完成：
Integer i5 = 130;
Integer i6 = 130;
System.out.println(i5.equals(i6));  //true
System.out.println(i5 == i6);  //flase
/*
如果自动装箱值在-128~127之间，那么比较的就是具体的数值
否则，比较的就是对象的地址
 */
```

```java
//intValue() :作用将Integer---->int
Integer i7 = 130;
int i = i7.intValue();
System.out.println(i);  //130

//parseInt(String s) :String --->int
int i2 = Integer.parseInt("12");
System.out.println(i2);  //12

//toString:Integer ---->String
Integer i9 = 130;
System.out.println(i9.toString());  //130
```

### 日期相关类

##### java.util.Date

```java
//java.util.Date:
Date d = new Date();
System.out.println(d); //Wed Feb 17 17:35:52 CST 2021
System.out.println(d.toString()); //Wed Feb 17 17:35:52 CST 2021
System.out.println(d.toLocaleString()); //方法过时 2021-2-17 17:35:52
System.out.println(d.toGMTString()); //17 Feb 2021 09:35:52 GMT

System.out.println(d.getYear());  //121+1900=2021
System.out.println(d.getMonth()); //1 :返回的值在0~11之间，0表示1月
```

##### java.sql.Date

```java
//java.sql.Date:
Date d = new Date(1613555029609L);
System.out.println(d);
/*
(1)java.sql.Date和java.util.Date的区别：
java.util.Date：年月日  时分秒
java.sql.Date ：年月日
（2）java.sql.Date和java.util.Date的联系：
java.sql.Date(子类) extends java.util.Date(父类)
 */
//java.sql.Date和java.util.Date相互转换
//util-->sql:
java.util.Date date = new java.util.Date();  //创建util.Date对象
//方式一：向下转型
Date date1 = (Date) date;
//方式二：利用构造器
Date date2 = new Date(date.getTime());


//sql--->util
java.util.Date date3 = d;

//String-->sql.Date
Date date4 = Date.valueOf("2021-1-1");
```

##### SimpleDateFormat

【1】日期转换：

```java
//日期转换：
//SimpleDateFormat(子类) extends DateFormat(父类)
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//String--->Date
try {
    Date d = df.parse("2020-1-1 12:12:24");
    System.out.println(d);
} catch (ParseException e) {
    e.printStackTrace();
}

//Date ---->String
String format = df.format(new Date());
System.out.println(format);
```

【2】日期格式：

![image-20210217180806443](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629394_5f85ff.webp)

##### Calendar

```java
//Calendar是一个抽象类，不可以直接创建对象
//GregorianCalendar（子类）extends Calendar(父类)父类是一个抽象类
//创建对象方式一
Calendar cal = new GregorianCalendar();
//创建对象方式二
Calendar cal2 = Calendar.getInstance();
System.out.println(cal);

//常用的方法:get方法，传入参数：Calendar中定义的常量
System.out.println(cal.get(Calendar.YEAR));
System.out.println(cal.get(Calendar.MONTH));
System.out.println(cal.get(Calendar.DATE));
System.out.println(cal.get(Calendar.DAY_OF_WEEK));
System.out.println(cal.getActualMaximum(Calendar.DATE)); //当前月份最大值
System.out.println(cal.getActualMinimum(Calendar.DATE)); //当前月份最小值

//set方法：可以改变Calendar中的内容
cal.set(Calendar.YEAR,2000);
cal.set(Calendar.MONTH,2);
cal.set(Calendar.DATE,15);
System.out.println(cal);
System.out.println(cal.get(Calendar.YEAR));

//String----->Calendar:
java.sql.Date date = java.sql.Date.valueOf("2021-1-1");
cal.setTime(date);
System.out.println(cal);
```

**练习**

需求：

![image-20210218100602247](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629517_ec37ac.webp)

```java
package com.zy.test1;

import java.util.Calendar;
import java.util.Scanner;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class Demo5 {
    public static void main(String[] args) {
        //录入日期的String
        Scanner sc = new Scanner(System.in);
        System.out.print("请输入你想要查看的日期：（提示：请按照例如2019-3-7的格式）");
        String strDate = sc.next();

        //String ---->Calendar
        java.sql.Date date = java.sql.Date.valueOf(strDate);
        Calendar cal = Calendar.getInstance();
        cal.setTime(date);

        //星期提示
        System.out.println("日\t一\t二\t三\t四\t五\t六\t");

        //获取本月的最大天数：
        int maxDay = cal.getActualMaximum(Calendar.DATE);
        //获取当前日期中的日
        int nowDay = cal.get(Calendar.DATE);
        //将日期凋萎本月的1号：
        cal.set(Calendar.DATE,1);
        //获取这个月一号是本周的第几天：
        int num= cal.get(Calendar.DAY_OF_WEEK);
        //前面空出来的天数为：
        int day = num - 1;

        //引入一个计时器：
        int count = 0;
        //打印日期前的空格
        for (int i = 1; i <=day ; i++) {
            System.out.print("\t");
        }
        //空出来的天数也要放入计数器
        count = count + day;
        //遍历：从1号开始到maxDay号
        for (int i = 1; i <= maxDay; i++) {
            if (i==nowDay){  //如果遍历的i和当前日子一样的话，后面加一个*
                System.out.print(i+"*"+"\t");
            }else {
                System.out.print(i+"\t");
            }
            count++;
            if (count%7==0){  //当计数器的个数是7的倍数的时候，就换行操作
                System.out.println();
            }
        }

    }
}
```

##### JDK1.8新增日期类

**LocalDate/LocalTime/LocalDateTime使用**

```java
package com.zy.test1;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class Demo6 {
    public static void main(String[] args) {
        //1.实例化
        //方法1：now() -- 获取当前的日期，时间，日期+时间
        LocalDate localDate = LocalDate.now();
        System.out.println(localDate);
        LocalTime localTime = LocalTime.now();
        System.out.println(localTime);
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime);

        //方法2：of()  --设置指定的日期，时间，日期和时间
        LocalDate of1 = localDate.of(2020,1,1);
        System.out.println(of1);
        LocalTime of2 = localTime.of(10,30,30);
        System.out.println(of2);
        LocalDateTime of3 = localDateTime.of(2020,1,1,10,30,30);
        System.out.println(of3);

        //get方法 主要用LocalDateTime讲解：
        System.out.println(localDateTime.getYear()); //2021
        System.out.println(localDateTime.getMonth()); //FEBRUARY
        System.out.println(localDateTime.getMonthValue());  //2
        System.out.println(localDateTime.getDayOfWeek()); //THURSDAY
        System.out.println(localDateTime.getHour()); //10
        System.out.println(localDateTime.getMinute()); //54
        System.out.println(localDateTime.getSecond()); //48

        //with方法  相当于之前的set方法
        //不可变性
        LocalDateTime localDateTime1 = localDateTime.withMonth(5);
        System.out.println(localDateTime1);  //2021-05-18T10:59:12.298
        System.out.println(localDateTime);  //2021-02-18T10:59:12.298

        //加减操作
        //加操作
        LocalDateTime localDateTime2 = localDateTime.plusMonths(5);
        System.out.println(localDateTime); //2021-02-18T11:01:14.230
        System.out.println(localDateTime2); //2021-07-18T11:01:14.230

        //减操作
        LocalDateTime localDateTime3 = localDateTime.minusMonths(6);
        System.out.println(localDateTime); //2021-02-18T11:03:26.694
        System.out.println(localDateTime3); //2020-08-18T11:03:26.694
    }
}
```

##### DateTimeFormatter

```java
//格式化类：DateTimeFormatter

//方式一：预定义的标准格式。如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME
DateTimeFormatter df1 = DateTimeFormatter.ISO_LOCAL_DATE_TIME;

//df1就可以帮我们完成LocalDateTime和String之间的相互转换：
//LocalDateTime ----》String
LocalDateTime now = LocalDateTime.now();
String str = df1.format(now);
System.out.println(str); //2021-02-18T11:22:46.796
//String--->LocalDateTime
TemporalAccessor parse = df1.parse("2021-02-18T11:22:46.796");
System.out.println(parse); //{},ISO resolved to 2021-02-18T11:22:46.796

//方式二：本地化相关的格式。如ofLocalizedDateTime()
//参数：
// FormatStyle.LONG  2021年2月18日 下午02时48分27秒
// FormatStyle.MEDIUM  2021-2-18 14:49:49
// FormatStyle.SHORT  21-2-18 下午2:50
DateTimeFormatter df2 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT);

//LocalDateTime ----》String
LocalDateTime now1 = LocalDateTime.now();
String str2 = df2.format(now1);
System.out.println(str2); //2021年2月18日 下午02时48分27秒

//String--->LocalDateTime
TemporalAccessor parse1 = df2.parse("21-2-18 下午2:50");
System.out.println(parse1);  //{},ISO resolved to 2021-02-18T14:50

//方式三：自定义的格式，如：ofPattern("yyyy-MM-dd hh:mm:ss");
DateTimeFormatter df3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
//LocalDateTime ----》String
LocalDateTime now2 = LocalDateTime.now();
String format = df3.format(now2);
System.out.println(format); //2021-02-18 02:54:57
//String--->LocalDateTime
TemporalAccessor parse2 = df3.parse("2021-02-18 02:54:57");
System.out.println(parse2);
```

### Math类

【1】直接使用，无需导包

![image-20210218165357133](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/4a267e8785b8fb80e112c42eebaed80c_83e63f.webp)

【2】这个类被final修饰，那么这个类不能有子类，不能被继承

![image-20210218165445501](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629729_0ac314.webp)

【3】构造器私有化，不能创建Math类的对象：

![image-20210218165622174](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629263_5c8901.webp)

![image-20210218165716202](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/ad28902700b4b1646fac4c8e23345897_eb8e75.webp)

【4】Math内部的所有的属性，方法都被static修饰：类名.直接调用，无需创建对象。

【5】常用方法

```java
//常用属性：
System.out.println(Math.PI);
//常用方法：
System.out.println("随机数"+Math.random());
System.out.println("绝对值"+Math.abs(-10));
System.out.println("向上取值"+Math.ceil(9.4));
System.out.println("向下取值"+Math.floor(9.4));
System.out.println("四舍五入"+Math.round(2.6));
System.out.println("取大的那个值"+Math.max(12, 46));
System.out.println("取小的那个值"+Math.min(12, 46));
```

【6】静态导入

```java
package com.zy.test1;

import static java.lang.Math.*;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class Demo8 {
    public static void main(String[] args) {
        //常用属性：
        System.out.println(PI);
        //常用方法：
        System.out.println("随机数"+random());
        System.out.println("绝对值"+abs(-10));
        System.out.println("向上取值"+ceil(9.4));
        System.out.println("向下取值"+floor(9.4));
        System.out.println("四舍五入"+round(2.6));
        System.out.println("取大的那个值"+max(12, 46));
        System.out.println("取小的那个值"+min(12, 46));
    }
    //如果跟Math中方法重复了，那么会优先走本类中的方法
    public static int random(){
        return 200;
    }
}
```

### Random类

```java
//(1)利用带参数的构造器创建对象：
Random r1 = new Random(System.currentTimeMillis());
int i = r1.nextInt();
System.out.println(i);

//(2)利用空参构造器创建对象：
Random r2 = new Random(); //表面上是在调用无参构造器，实际上底层还是调用了带参构造器
System.out.println(r2.nextInt(10)); //在 0（包括）和指定值（不包括）之间均匀分布的 int 值。
System.out.println(r2.nextDouble()); //在 0.0 和 1.0 之间均匀分布的 double 值。
```

### String类

【1】直接使用，无需导包

【2】这个类被final修饰，不可以被继承，不能有子类。

【3】String底层是一个char类型的数组

![image-20210218172741163](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241629544_270263.webp)

【4】构造器：底层就是给对象底层的value数组进行赋值操作

```java
//通过构造器来创建对象：
String s1 = new String();
String s2 = new String("abc");
String s3 = new String(new char[]{'a', 'b', 'c'});
```

【5】常用方法：

```java
String s4 = "abc";
System.out.println("字符串的长度为："+s4.length());

String s5 = new String("abc");
System.out.println("字符串是否为空："+s5.isEmpty());

System.out.println("获取字符串的下标对应的字符为："+s5.charAt(1));String s4 = "abc";
        System.out.println("字符串的长度为："+s4.length());

        String s5 = new String("abc");
        System.out.println("字符串是否为空："+s5.isEmpty());

        System.out.println("获取字符串的下标对应的字符为："+s5.charAt(1));
```

【6】equals：

```java
String s6 = new String("abc");
String s7 = new String("abc");
System.out.println(s6.equals(s7));
```

![image-20210218181048503](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630402_85dcaf.webp)

【7】String类实现了Comparable，里面有一个抽象方法叫compareTo，所以String中一定要对这个方法重写：

```java
String s8 = new String("abc");
String s9 = new String("abcdef");
System.out.println(s8.compareTo(s9));
```

![image-20210218183239196](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630737_1bb9d3.webp)

【8】常用方法：

```java
//字符串的截取
String s10 = new String("abcdefgki");
System.out.println(s10.substring(4));
System.out.println(s10.substring(4, 6));

//字符串的拼接
System.out.println(s10.concat("aaaaaa"));
//字符串的替换
String s11 = "abanagavah";
System.out.println(s11.replace('a', 'w'));
//字符串的切割
String s12 = "a-b-c-d-e-f";
String[] strs = s12.split("-");
System.out.println(Arrays.toString(strs));

//转换大小写
String s13 = "abc";
System.out.println(s13.toUpperCase());
System.out.println(s13.toUpperCase().toLowerCase());

//去除首尾空格
String s14 = "   a  b  c   ";
System.out.println(s14.trim());

//toString()
String s15 = "abc";
System.out.println(s15.toString());
//转换为String类型：
System.out.println(String.valueOf(false));
```

### String内存分析

【1】字符串拼接

```java
public class Demo3 {
    public static void main(String[] args) {
        String s1 = "a"+"b"+"c";
        String s2 = "ab"+"c";
        String s3 = "a"+"bc";
        String s4= "abc";
        String s5 = "abc"+"";
        
    }
}
```

上面的字符串，会进行编译器优化，直接合并成为完整的字符串，我们可以反编译验证：

![image-20210218190215340](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630304_d446a5.webp)

然后在常量池中，常量池的特点是第一次如果没有这个字符串，就放进去，如果有这个字符串，就直接从常量池中取。

内存分析：

![image-20210218185304098](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630019_5691f1.webp)

【2】new关键字创建对象：

```java
String s6 = new String("abc");
```

内存：开辟两个空间（1.字符串常量池中的字符串 2. 堆中的开辟的空间）

![image-20210218190531709](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630642_e50289.webp)

【3】有变量参与的字符串拼接：

```java
String a = "abc";
String b = a + "def";
System.out.println(b);
```

a变量在编译的时候不知道a是“abc”字符串，所以不会进行编译期优化，不会直接合并为“abcdef"
反汇编过程:为了更好的帮我分析字节码文件是如何进行解析的:

![image-20210218191722202](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630722_0ddecc.webp)

![image-20210218191642430](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630371_7abcf9.webp)

### StringBuilder类

【1】字符串的分类:
		(1)不可变字符串:String
		(2)可变字符串: StringBuilder，StringBuffer

【2】StringBuilder底层：非常重要的两个属性。

![image-20210220211444870](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241630781_351a8a.webp)

```java
//创建StringBuilder的对象：
StringBuilder sb3 = new StringBuilder();
//表面上调用StringBuilder的空构造器，实际底层是对value数组进行初始化，长度为16
StringBuilder sb2 = new StringBuilder(3);
//表面上调用StringBuilder的有参构造器，传入一个int类型的数，实际底层是对value数组进行初始化，长度为你传入的数字
StringBuilder sb = new StringBuilder("abc");
sb.append("def").append("aaaa").append("bbb");
```

【3】StringBuilder和StringBuffer常用方法相似

```java
StringBuilder sb = new StringBuilder("abcdefghijklmno");
//增
sb.append("@@@");
System.out.println(sb); 
//删
sb.delete(1,3); //删除位置在[1,3)上的字符
System.out.println(sb);
sb.deleteCharAt(8); //删除位置在8上的字符
System.out.println(sb);
//改---》插入
StringBuilder sb2 = new StringBuilder("你好，我喜欢学习");
sb2.insert(3,","); //在下标为3的位置上插入,
System.out.println(sb2);

//改---》替换
sb2.replace(3,5,"唉唉");
System.out.println(sb2);
sb2.setCharAt(4,'!');
System.out.println(sb2);

//查
for (int i = 0; i < sb2.length(); i++) {
    System.out.println(sb2.charAt(i) + "\t");
}
System.out.println(sb2);

//截取
String str = sb2.substring(2, 4);
System.out.println(sb2);
```

String、StringBuffer、StringBuilder区别于联系

1.String类是不可变类，即一旦一个String对象被创建后，包含在这个对象中的字符序列是不可改变的，直至这个对象销毁。

2.StringBuffer类则代表一个字符序列可变的字符串，可以通过append、insert、reverse、setChartAt、setLength等方法改变其内容。一旦生成了最终的字符串，调用toString方法将其转变为String

3.JDK1.5新增了一个StringBuilder类，与StringBuffer相似，构造方法和方法基本相同。不同是StringBuffer是线程安全的，而StringBuilder是线程不安全的，所以性能略高。通常情况下，创建一个内容可变的字符串，应该优先考虑使用StringBuilder

StringBuilder:JDK1.5开始效率高线程不安全
StringBuffer:JDK1.0开始效率低线程安全

