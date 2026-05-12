+++
date = '2025-05-02T06:52:56+08:00'
draft = false
title = 'Day 10 Junit&注解&枚举'
slug = "mzza080f4s"
description = "Day 10 Junit&注解&枚举"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 10 Junit&注解&枚举"
comments = true  
+++
﻿# Day-09-Junit&注解&枚举

【1】软件测试的目的:
软件测试的目的是在规定的条件下对程序进行操作,以发现程序错误,衡量软件质量,并对其是否能满足设计要求进行评估的过程

【2】测试分类:
	(1)黑盒测试:
软件的黑盒测试意味着测试要在软件的接口处进行。这种方法是把测试对象看做一个黑盒子,测试人员完全不考虑程序内部的逻辑结构和内部特性只依据程序的需求规格说明书,检查程序的功能是否符合它的功能说明。因此黑盒测试又叫功能测试。
	(2)白盒测试:
软件的白盒测试是对软件的过程性细节做细致的检查。这种方法是把测试对象看做一个打开的盒子,它允许测试人员利用程序内部的逻辑结构及有关信息设计或选择测试用例对程序的所有逻辑路径进行测试、通过在不同点检查程序状态.确定实际状态是否与预期的状态一致。因此白盒测试又称为结构测试。

![image-20210220215302010](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631119_969bbc.webp)

在没有使用Junit的时候，缺点:

​	(1）测试一定走main方法，是程序的入口，main方法的格式必须不能写错。

​	(2）要是在同一个main方法中测试的话，那么不需要测试的东西必须注释掉。

​	(3)测试逻辑如果分开的话，需要定义多个测试类，麻烦。

​	(4)业务逻辑和测试代码，都混淆了。

### Junit的使用

【1】一般测试和业务做一个分离，分离为不同的包：

【2】测试类的名字：XXXTest

【3】测试方法的定义 --》这个方法可以独立运行，不依托于main方法

建议：

​	参数：无参

​	返回值：void

【4】测试方法定义完以后，不能直接就独立运行了，必须要在方法前加入一个注解：@Test

【5】导入Junit依赖的环境：

![image-20210221101624630](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631071_980ec8.webp)

【6】代码：

```java
package com.zy.test1;


import com.zy.calculator.Calculator;
import org.junit.Test;

/**
 * @Auther: 赵羽
 * @Description: com.zy.test1
 * @version: 1.0
 */
public class CalculatorTest {
    //测试add方法
   @Test
    public void testAdd(){
       System.out.println("测试Add方法");
       Calculator cal = new Calculator();
       int add = cal.add(10, 20);
       System.out.println(add);
   }
    //测试sub方法
   @Test
    public void testSub(){
        System.out.println("测试sub方法");
       Calculator cal = new Calculator();
       int sub = cal.sub(20, 30);
       System.out.println(sub);
   }
}
```

【7】判定结果：

绿色：正常结果

红色：出现异常

即使出现绿色效果，也有可能会发生代码中逻辑出现问题，则需要加入断言

```java
//加入断言：预测结果与实际结果进行匹对
Assert.assertEquals(30,add); //第一个参数为预测结果 第二个结果为实际结果
```

![image-20210221102817187](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631822_c66bf6.webp)

【8】@Before&@After

@Before：某一个方法中，加入了@Before注解以后，那么这个方法中的功能会在测试方法执行前先执行。

​	一般会在@Before修饰的那个方法中加入：加入一些申请资源的代码：申请数据库资源，申请IO资源，申请网络资源。。

@After：某一个方法中，加入了@After注解以后，那么这个方法中的功能会在测试方法执行后先执行。

​	一般会在@After修饰的那个方法中加入：加入一些释放资源的代码：释放数据库资源，释放IO资源，释放网络资源。。

```java
@Before
public void init(){
    System.out.println("测试开始了");
}
@After
public void close(){
    System.out.println("测试结束了");
}
```

### 注解

##### 注解初识

JDK5.0新增---注解(Annotation),也叫元数据

注解其实就是代码里的==特殊标记==，这些标记可以在编译,类加载运行时被读取.并执行相应的处理。==通过使用注解程序员可以在不改变原有逻辑的情况下，在源文件中嵌入一些补充信息。==代码分析工具、开发工具和部署工具可以通过这些补充信息进行验证或者进行部署。
使用注解时要在其前面增加@符号;并把该注解当成一个修饰符使用。用于修饰它支持的程序元素。

Annotation可以像修饰符一样被使用，可用于修饰包，类，构造器,方法，成员变量,参数，局部变量的声明，这些信息被保存在Annotation的"name=value"对中。在JavaSE中，注解的使用目的比较简单，例如标记过时的功能，忽略警告等。在JavaEE/Arldroid中注解占据了更重要的角色，例如用来配置应用程序的任何切面，==代替JavaE旧版中所遗留的繁冗代码和XML配置等。==未来的开发模式都是基于注解
的，JPA(java的持久化API)是基于注解的，Spring2.5以.E都是基于注解的，Hibernate3.x以后也是基于注解的，现在的Struts2有一部分也是基于注解的了，注解是一种趋势，一定程度上可以说︰==框架=注解+反射+设计模式==。

##### 文档相关注解

说明注释允许你在程序中嵌入关于程序的信息。你可以使用javadoc 工具软件来生成信息，并输出到HTML文件中。

说明注释，使你更加方便的记录你的程序信息。
文档注解我们一般使用在文档注释中，配合javadoc工具javadoc工具软件识别以下标签:

| 标签          | 描述                                                   |                             示例                             |
| :------------ | :----------------------------------------------------- | :----------------------------------------------------------: |
| @author       | 标识一个类的作者                                       |                     @author description                      |
| @deprecated   | 指名一个过期的类或成员                                 |                   @deprecated description                    |
| {@docRoot}    | 指明当前文档根目录的路径                               |                        Directory Path                        |
| @exception    | 标志一个类抛出的异常                                   |            @exception exception-name explanation             |
| {@inheritDoc} | 从直接父类继承的注释                                   |      Inherits a comment from the immediate surperclass.      |
| {@link}       | 插入一个到另一个主题的链接                             |                      {@link name text}                       |
| {@linkplain}  | 插入一个到另一个主题的链接，但是该链接显示纯文本字体   |          Inserts an in-line link to another topic.           |
| @param        | 说明一个方法的参数                                     |              @param parameter-name explanation               |
| @return       | 说明返回值类型                                         |                     @return explanation                      |
| @see          | 指定一个到另一个主题的链接                             |                         @see anchor                          |
| @serial       | 说明一个序列化属性                                     |                     @serial description                      |
| @serialData   | 说明通过writeObject( ) 和 writeExternal( )方法写的数据 |                   @serialData description                    |
| @serialField  | 说明一个ObjectStreamField组件                          |              @serialField name type description              |
| @since        | 标记当引入一个特定的变化时                             |                        @since release                        |
| @throws       | 和 @exception标签一样.                                 | The @throws tag has the same meaning as the @exception tag.  |
| {@value}      | 显示常量的值，该常量必须是static属性。                 | Displays the value of a constant, which must be a static field. |
| @version      | 指定类的版本                                           |                        @version info                         |

**需要注意这些标记的使用是有位置限制的：**

- 可以出现在类或者接口文档注释中的标记有：@see、@deprecated、@author、@version等。
- 可以出现在方法或者构造器文档注释中的标记有：@see、@deprecated、@param、@return、@throws、@exception等。
- 可以出现在文档注释中的标记有：@see、@deprecated等。

其中注意:

@param @return和@exception这三个标记都是只用于方法的。

@param的格式要求:@param形参名形参类型形参说明

@return的格式要求:@return返回值类型返回值说明，如果方法的返回值类型是void就不能写

@exception的格式要求:@exception异常类型异常说明

@param和@exception可以并列多个

IDEA中javadoc使用：

![image-20210221105600043](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631025_201f06.webp)

![image-20210221110133326](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631753_3e59b0.webp)

##### JDK内置的三个注解

@Override:限定重写父类方法，该注解只能用于方法

@Deprecated;用于表示所修饰的元素(类方法，构造器，属性等)已过时。通常是因为所修饰的结构危险或存在更好的选择

@SuppressWarnings:抑制编译器警告

##### 自定义注解

![image-20210221111144408](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631459_fcba11.webp)

注解内部：看上去是无参数方法，实际上理解为一个成员变量，一个属性无参数方法名字--》成员变量的名字
无参数方法的返回值--》成员变量的类型

这个参数叫做 配置参数

无参数方法的类型:基本数据类型（八种)，String，枚举，注解类型，还可以是以上类型对应的数组。

PS:注意:如果只有一个成员变量的话，名字尽量叫value。

**使用注解：**

（1）使用注解的话，如果你定义了配置参数，就必须给赋值操作:

```java
@MyAnnotation(value = {"abc","def"})
public class Person {
}
```

（2）如果只有一个参数，并且这个参数的名字为value的话，那么value=可以省略不写。

```java
@MyAnnotation({"abc","def"})
public class Person {
}
```

（3）如果配置参数已经设置默认的值，那么使用的时候可以无需传值:

```java
public @interface MyAnnotation {
    String  value() default "abc";
}
```

```java
@MyAnnotation
public class Person {
}
```

（4）一个注解的内部是可以不定义配置参数的:

```java
public @interface MyAnnotation {
}
```

内部没有定义配置参数的注解--》可以叫做标记

内部定义配置参数的注解--》元数据

##### 元注解

元注解是用于修饰其它注解的注解。

JDK5.0提供了四种元注解: Retention,Target,Documented, Inherited

【1】Retention

@Retention:用于修饰注解，用于指定修饰的那个注解的生命周期，@Rentention包含一个RetentionPolicy枚举类型的成员变量使用@Rentention时必须为该value成员变量指定值:

>RetentionPolicy.SOURCE:在源文件中有效(即源文件保留)编译器直接丢弃这种策略的注释，在.class文件中不会保留注解信息
>RetenlionPolicy.CLASS:在class文件中有效(即class保留)，保留在.class文件中，但是当运行Java程序时，他就不会继续加载了，不会保留在内存中，JVM不会保留注解。==如果注释没有加Retention元注解，那么相当于默认的注解就是这种状态。==
>RetentionPolicy.RUNTIME.在运行时有效(即运行时保留)当运行.Java程序时，JVM会保留注释，加载在内存中了，那么程序可以通过反射获取该注释。

【2】Target

用于修饰注解的注解，用于指定被修饰的注解能用于修饰哪些程序元素。@Target也包含一个名为value的成员变量。

```java
package com.zy.anno;

import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;

/**
 * @Auther: 赵羽
 * @Description: com.zy.anno
 * @version: 1.0
 */
@Target({TYPE,CONSTRUCTOR,METHOD})
public @interface MyAnnotation2 {
}
```

![image-20210221114909481](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631090_5abbd3.webp)

【3】Documented

用于指定被该元注解修饰的注解类将被javadoc工具提取成文档。默认情况下，javadoc是不包括注解的，但是加上了这个注解生成的文档中就会带着注解了。

案例：

如果:Documented注解修饰了Deprecated注解，

![image-20210221115415872](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241631068_c6d892.webp)

那么Deprecated注解就会在javadoc提取的时候，提取到API中:

![image-20210221115437142](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241632128_21b7a2.webp)

【4】Inherited

被它修饰的Annotation将具有继承性。如果某个类使用了被Inherited修饰的Annotation,则其子类将自动具有该注解。

案例：

如果MyAnnotation2注解使用了@Inherited之后，就具备了继承性，那么相当于子类Student也使用了这个MyAnnotation2

```java
package com.zy.anno;

import java.lang.annotation.Inherited;

@Inherited
public @interface MyAnnotation2 {
}
```

父类：

```java
package com.zy.anno;

import com.zy.anno2.MyAnnotation;

@MyAnnotation
public class Person {
}
```

子类：

```java
package com.zy.anno;

public class Student extends Person {
}
```

### 枚举

【1】在java中，类的对象是有限个，确定的。这个类我们可以定义为枚举类。

【2】自定义枚举类：

```java
package com.zy.enum01;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum01
 * @version: 1.0
 * 定义枚举类：季节
 */
public class Season {
    //属性：
    private final String seasonName; //季节名字
    private final String seasonDesc; //季节描述

    //利用构造器对属性进行赋值操作：
    //构造器私有化，外界不能调用这个构造器，只能Season内部自己使用
    private Season(String seasonName,String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //提供枚举类的有限的 确定的对象：
    public static final Season SPRING = new Season("春天","春暖花开");
    public static final Season SUMMER = new Season("夏天","烈日炎炎");
    public static final Season AUTUMN = new Season("秋天","硕果累累");
    public static final Season WINTER = new Season("冬天","冰天雪地");

    //额外因素：

    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    //toString():

    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }
}
```

测试类：

```java
public class TestSeason {
    public static void main(String[] args) {
        Season autumn = Season.AUTUMN;
        System.out.println(autumn);
        System.out.println(autumn.getSeasonName());
    }
}
```

【3】JDK1.5后使用enum关键字定义枚举类：

```java
package com.zy.enum02;


/**
 * @Auther: 赵羽
 * @Description: com.zy.enum01
 * @version: 1.0
 * 定义枚举类：季节
 */
public enum  Season {
    //提供枚举类的有限的 确定的对象： --->enum枚举要求对象（常量）必须放在最开始位置
    //多个对象之间用,进行连接，最后一个对象后面用;结束
    SPRING ("春天","春暖花开"),
    SUMMER ("夏天","烈日炎炎"),
    AUTUMN("秋天","硕果累累"),
    WINTER("冬天","冰天雪地");
    //属性：
    private final String seasonName; //季节名字
    private final String seasonDesc; //季节描述

    //利用构造器对属性进行赋值操作：
    //构造器私有化，外界不能调用这个构造器，只能Season内部自己使用
    private Season(String seasonName, String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }



    //额外因素：

    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    //toString():

    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }
}
```

使用枚举类：

```java
public class Test {
    public static void main(String[] args) {
        Season autumn = Season.AUTUMN;
        System.out.println(autumn);
        //enum关键字对应的枚举类的上层父类是: java.lang.Enum
        //但是我们自定义的枚举类的上层父类:Object
        System.out.println(Season.class.getSuperclass().getName()); //java.lang.Enum
    }
}
```

源码中也有人这样写枚举类：

```java
public enum  Season {
    SPRING,
    SUMMER,
    AUTUMN,
    WINTER;
}
```

这个枚举类底层没有属性，属性，构造器，toString, get方法都删掉不写
看到的形态就剩:常量名(对象名)

【4】enum类常用方法：

```java
//用enum关键字创建的Season枚举类上面的父类是: java.Lang.Enum,常用方法子类Season可以直接拿过来使用
//toString();--->获取对象的名字
Season autumn = Season.AUTUMN;
System.out.println(autumn); //AUTUMN

//values:返回枚举类对象的数组
Season[] values = Season.values();
for (Season s:values){
    System.out.println(s);
}

//valueOf:通过对象名字获取这个枚举对象
//注意：对象的名字必须传正确
Season autumn1 = Season.valueOf("AUTUMN");
System.out.println(autumn1);
```

【5】枚举类实现接口：

```java
package com.zy.enum04;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum03
 * @version: 1.0
 */
public enum Season implements TestInterface {
    SPRING,
    SUMMER,
    AUTUMN,
    WINTER;

    @Override
    public void show() {
        System.out.println("你好");
    }
}
```

```java
package com.zy.enum04;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum04
 * @version: 1.0
 */
public interface TestInterface {
    void show();
}
```

```java
package com.zy.enum04;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum04
 * @version: 1.0
 */
public class Test {
    public static void main(String[] args) {
        Season autumn = Season.AUTUMN;
        autumn.show(); //你好
        Season spring = Season.SPRING;
        spring.show(); //你好
    }
}
```

==上面发现所有的枚举对象，调用这个show方法的时候走的都是同一个方法，结果都一样。==

**实现：不同的对象调用的show方法也不同。**

```java
package com.zy.enum04;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum03
 * @version: 1.0
 */
public enum Season implements TestInterface {
    SPRING{
        @Override
        public void show() {
            System.out.println("这是春天。。");
        }
    },
    SUMMER{
        @Override
        public void show() {
            System.out.println("这是夏天。。");
        }
    },
    AUTUMN{
        @Override
        public void show() {
            System.out.println("这是秋天。。");
        }
    },
    WINTER{
        @Override
        public void show() {
            System.out.println("这是冬天。。");
        }
    };
}
```

【6】枚举的应用：

```java
package com.zy.enum05;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum05
 * @version: 1.0
 */
public class Person {
    //属性
    private int age;
    private String name;
    private Gender sex;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Gender getSex() {
        return sex;
    }

    public void setSex(Gender sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}
```

```java
package com.zy.enum05;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum05
 * @version: 1.0
 */
public enum Gender {
    男,
    女;
}
```

```java
package com.zy.enum05;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum05
 * @version: 1.0
 */
public class Test {
    public static void main(String[] args) {
        Person p = new Person();
        p.setName("张三");
        p.setAge(18);
        p.setSex(Gender.男);
        System.out.println(p);

    }
}
```

枚举和switch结合：

```java
package com.zy.enum05;

/**
 * @Auther: 赵羽
 * @Description: com.zy.enum05
 * @version: 1.0
 */
public class Test02 {
    public static void main(String[] args) {
        Gender sex = Gender.男;
        //switch后面的（）中可以传入枚举类型
        //switch后面的（）：int,short,byte,char,String,枚举
        switch (sex){
            case 女:
                System.out.println("这是个女孩。。");
                break;
            case 男:
                System.out.println("这是个男孩。。");
                break;
        }
    }
}
```

# 
