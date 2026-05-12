+++
date = '2024-01-18T10:50:48+08:00'
draft = false
title = 'Day 13 多线程'
slug = "pjh3g4qobi"
description = "Day 13 多线程"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 13 多线程"
comments = true  
+++
﻿# Day-12-多线程

### 简介

线程：在多任务操作系统中，每个运行的程序都是一个进程，用来执行不同的任务，而在一个进程中还可以有多个执行单元同时运行，来同时完成一个或多个程序任务，这些执行单元可以看做程序执行的一条条线索，被称为线程。

注意︰操作系统中的每一个进程中都至少存在一个线程，当一个Java程序启动时，就会产生一个进程，该进程中会默认创建一个线程，在这个线程上会运行main()方法中的代码。

多线程可以充分利用CPU资源,进一步提升程序执行效率。

总结：

线程就是独立的执行路径;

在程序运行时，即使没有自己创建线程，后台也会有多个线程，如主线程，gc线程;main()称之为主线程，为系统的入口，用于执行整个程序;

在一个进程中，如果开辟了多个线程，线程的运行由调度器安排调度，调度器是与操作系统紧密相关的，先后顺序是不能认为的干预的。

对同一份资源操作时，会存在资源抢夺的问题，需要加入并发控制;线程会带来额外的开销，如CPU调度时间，并发控制开销。

每个线程在自己的工作内存交互，内存控制不当会造成数据不一致

### 创建

方式一：继承Thread类，重写run()方法

```java
package com.zy.thread_num;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num
 * @version: 1.0
 */
//创建线程方式一:继承Thread类，重写run()方法，调用start开启线程
//总结:注意,线程开启不一定立即执行,由CPU调度执行
public class TestThread01 extends Thread {
    @Override
    public void run() {
        //run方法线程体
        for (int i = 0; i < 20; i++) {
            System.out.println("我喜欢学java");
        }
    }
	
    //main线程，主线程
    public static void main(String[] args) {
        

        //创建一个线程对象
        TestThread01 testThread01 = new TestThread01();

        //调用start()开启线程
        testThread01.start();

        for (int i = 0; i < 100; i++) {
            System.out.println("你好");
        }
    }
}
```

**练习Thread，实现多线程同步下载图片**

准备：commons-io-2.8.0.jar包

![image-20210306160457937](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241637584_4d0f23.webp)

![image-20210306160626061](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241637346_bd4e42.webp)

```java
package com.zy.thread_num;

import org.apache.commons.io.FileUtils;

import java.io.File;
import java.io.IOException;
import java.net.URL;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num
 * @version: 1.0
 */
//练习Thread，实现多线程同步下载图片
public class TestThread02 extends Thread{

    private String url;  //网络图片地址
    private String name;  //保存得文件名

    public TestThread02(String url,String name) { //构造器
        this.url = url;
        this.name = name;
    }

    //下载图片线程得执行体
    @Override
    public void run() {
        WebDownloader webDownloader = new WebDownloader();
        webDownloader.downloader(url,name);
        System.out.println("下载了文件名为："+name);;
    }

    public static void main(String[] args) {
        TestThread02 t1 = new TestThread02("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-004.jpg","2.jpg");
        TestThread02 t2 = new TestThread02("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-005.jpg","3.jpg");
        TestThread02 t3 = new TestThread02("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-006.jpg","4.jpg");
        TestThread02 t4 = new TestThread02("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-007.jpg","5.jpg");

        t1.start();
        t2.start();
        t3.start();
        t4.start();

    }

}

//下载器
class WebDownloader{
    //下载方法
    public void downloader(String url,String name){
        try {
            FileUtils.copyURLToFile(new URL(url),new File(name));
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("io异常，downloader方法出现问题");
        }
    }
}
```

方式二：实现Runnable接口，重写run()方法

```java
package com.zy.thread_num;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num
 * @version: 1.0
 */
//创建线程方式二﹔实现runnable接口,重写run方法，执行线程需要丢入runnable接口实现类，调用start方法。
public class TestThread03 implements Runnable{
    @Override
    public void run() {
        //run方法线程体
        for (int i = 0; i < 20; i++) {
            System.out.println("我喜欢学java");
        }
    }

    public static void main(String[] args) {
        //创建runnable接口的实现类对象
        TestThread03 testThread03 = new TestThread03();

        //创建线程对象，通过线程对象来开启线程

        new Thread(testThread03).start();

        for (int i = 0; i < 100; i++) {
            System.out.println("你好");
        }
    }
}
```

乌龟赛跑

```java
package com.zy.thread_num;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num
 * @version: 1.0
 */
//模拟乌龟赛跑
public class TestThread05 implements Runnable{

    //胜利者
    private static String winner;

    @Override
    public void run() {
        for (int i = 0; i <=100; i++) {

            //模拟兔子休息
            if (Thread.currentThread().getName().equals("兔子")&&i%10==0){
                try {
                    Thread.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            //判断比赛是否结束
            boolean flag = gameover(i);
            //如果比赛结束了，就停止程序
            if (flag){
                break;
            }
            System.out.println(Thread.currentThread().getName()+"--->跑了"+i+"步");
        }
    }

    //判断是否完成比赛
    private boolean gameover(int steps){
        //判断是否有胜利者
        if (winner!=null){ //已经存在胜利者
            return true;
        }{
            if (steps>=100){
                winner = Thread.currentThread().getName();
                System.out.println("胜利者是："+winner);
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        TestThread05 race = new TestThread05();
        new Thread(race,"兔子").start();
        new Thread(race,"乌龟").start();

    }
}
```

方式三：实现Callable接口，重写call)方法，并使用Future来获取call()方法的返回结果

```java
package com.zy.thread_num;

import org.apache.commons.io.FileUtils;

import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.util.concurrent.*;

/**
 * @Auther: 赵羽
 * @Date: 2021/3/6 - 03 - 06 - 16:56
 * @Description: com.zy.thread_num
 * @version: 1.0
 */
//创建线程方式三：实现Callable接口
//从JDK 5开始,Java提供了一个新的Callable接口，来满足这种既能创建多线程又可以有返回值的需求。
public class TestThread06 implements Callable<Boolean> {
    private String url;  //网络图片地址
    private String name;  //保存得文件名

    public TestThread06(String url,String name) { //构造器
        this.url = url;
        this.name = name;
    }

    //下载图片线程得执行体
    @Override
    public Boolean call() {
        WebDownloader webDownloader = new WebDownloader();
        webDownloader.downloader(url,name);
        System.out.println("下载了文件名为："+name);
        return true;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        TestThread06 t1 = new TestThread06("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-004.jpg","2.jpg");
        TestThread06 t2 = new TestThread06("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-005.jpg","3.jpg");
        TestThread06 t3 = new TestThread06("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-006.jpg","4.jpg");
        TestThread06 t4 = new TestThread06("https://img.ivsky.com/img/tupian/t/202102/26/sunyunzhu_baotunqun-007.jpg","5.jpg");

       //创建执行服务：
        ExecutorService ser = Executors.newFixedThreadPool(4);

        //提交执行
        Future<Boolean> submit1 = ser.submit(t1);
        Future<Boolean> submit2 = ser.submit(t2);
        Future<Boolean> submit3 = ser.submit(t3);
        Future<Boolean> submit4 = ser.submit(t4);

        //获取结果
        Boolean r1 = submit1.get();
        Boolean r2 = submit2.get();
        Boolean r3 = submit3.get();
        Boolean r4 = submit4.get();

        //关闭服务
        ser.shutdownNow();
    }

}

//下载器
class WebDownloaders{
    //下载方法
    public void downloader(String url,String name){
        try {
            FileUtils.copyURLToFile(new URL(url),new File(name));
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("io异常，downloader方法出现问题");
        }
    }
}
```

### 对比

通过实现Runnable接口(或者Callable接口）相对于继承Thread类实现多线程来说，有如下显著的好处︰

- 适合多个线程去处理同一个共享资源的情况。
- 可以避免Java单继承带来的局限性。

### 静态代理

```java
package com.zy.thread_num;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num
 * @version: 1.0
 */
/*
* 静态代理模式总结：
* 真实对象和代理对象都要实现同一个接口
* 代理对象要代理真实角色
* 好处：
*   代理对象可以做很多真实对象做不了的事请
*   真实对象专注做自己的事请
* */
public class StaticProxy {
    public static void main(String[] args) {

        You you = new You();
        new Thread(()->System.out.println("我爱你")).start();
        new WeddingCompany(new You()).HappyMarry();

        /*WeddingCompany weddingCompany = new WeddingCompany(you);
        weddingCompany.HappyMarry();*/
    }
}

interface Marry{
    void HappyMarry();
}
//真实角色 结婚本人
class You implements Marry{
    @Override
    public void HappyMarry() {
        System.out.println("我要结婚了，很开心");
    }
}

//代理角色， 婚庆公司 帮助本人结婚
class WeddingCompany implements Marry{

    private Marry target;

    public WeddingCompany(Marry target) {
        this.target = target;
    }

    @Override
    public void HappyMarry() {
        before();
        this.target.HappyMarry();
        after();
    }

    private void after() {
        System.out.println("结婚之后，收尾款");
    }

    private void before() {
        System.out.println("结婚之前，布置现场");
    }

}
```

### Lamda表达式

入希腊字母表中排序第十一位的字母，英语名称为Lambda

避免匿名内部类定义过多

其实质属于函数式编程的概念

理解Functional lnterface(函数式接口）是学习Java8 lambda表达式的关键所在。

函数式接口的定义:

​	任何接口，如果只包含唯一一个抽象方法，那么它就是一个函数式接口。

​	对于函数式接口，我们可以通过lambda表达式来创建该接口的对象。

```java
package com.zy.thread_num2;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num2
 * @version: 1.0
 */
/*
推导lambda表达式
*/
public class Demo1 {

    //3.静态内部类
    static class Like2 implements ILike{
        @Override
        public void lambda() {
            System.out.println("I like lambda2");
        }
    }



    public static void main(String[] args) {
        Like like = new Like();
        like.lambda();

        Like2 like2 = new Like2();
        like2.lambda();


        //4.局部内部类
        class Like3 implements ILike{
            @Override
            public void lambda() {
                System.out.println("I like lambda3");
            }
        }

        Like3 like3 = new Like3();
        like3.lambda();

        //5.匿名内部类，没有类的名称，必须借助接口或者父类

        ILike ilike = new ILike() {
            @Override
            public void lambda() {
                System.out.println("I like lambda4");
            }
        };
        ilike.lambda();

        //6.用lambda简化
        ILike like1 = ()-> {
            System.out.println("I like lambda5");
        };
        like1.lambda();



    }
}
//1.定义一个函数式接口
interface ILike{
    void lambda();
}

//2.实现类
class Like implements ILike{
    @Override
    public void lambda() {
        System.out.println("I like lambda");
    }
}
```

```java
package com.zy.thread_num2;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num2
 * @version: 1.0
 */
/*
* 总结:
lambda表达式只能有一行代码的情况下才能简化成为一行，如果有多行，那么就用代码块包裹。
* 前提是接口为函数式接
* 多个参数也可以去掉参数类型，要去掉就都去掉，必须加上括号
* */
public class Demo2 {
    public static void main(String[] args) {

        //1.lambda表达示简化
        ILove love =(int a)-> {
            System.out.println("i love you-->"+a);
        };

        //简化1.参数类型
        love = (a)-> {
            System.out.println("i love you-->"+a);
        };

        //简化2.简化括号
        love = a-> {
            System.out.println("i love you-->"+a);
        };

        //简化3.去掉花括号
        love = a->System.out.println("i love you-->"+a);


        love.love(520);
    }

}
interface ILove{
    void love(int a);
}
```

### 线程状态

![image-20210307142626501](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241638368_4fd120.webp)

![image-20210307200157860](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241638373_4db6be.webp)

线程状态。线程可以处于下列状态之一： 

- [`NEW`](../../java/lang/Thread.State.html#NEW)
  至今尚未启动的线程处于这种状态。 
- [`RUNNABLE`](../../java/lang/Thread.State.html#RUNNABLE)
  正在  Java 虚拟机中执行的线程处于这种状态。 
- [`BLOCKED`](../../java/lang/Thread.State.html#BLOCKED)
  受阻塞并等待某个监视器锁的线程处于这种状态。 
- [`WAITING`](../../java/lang/Thread.State.html#WAITING)
  无限期地等待另一个线程来执行某一特定操作的线程处于这种状态。 
- [`TIMED_WAITING`](../../java/lang/Thread.State.html#TIMED_WAITING)
  等待另一个线程来执行取决于指定等待时间的操作的线程处于这种状态。 
- [`TERMINATED`](../../java/lang/Thread.State.html#TERMINATED)
  已退出的线程处于这种状态。 

在给定时间点上，一个线程只能处于一种状态。这些状态是虚拟机状态，它们并没有反映所有操作系统线程状态。 

### 线程停止

```java
package com.zy.thread_num3;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */

/*
测试stop
1.建议线程正常停止--->利用次数,不建议死循环。
2.建议使用标志位--->设置一个标志位
3.不要使用stop或者destroy 等过时或者JDK不建议使用的方法
*/
public class Demo1 implements Runnable{

    //1.设置一个标识位
    private boolean flag = true;

    @Override
    public void run() {
        int i = 0;
        while (flag){
            System.out.println("线程运行中"+i++);
        }

    }

    //2.设置一个二公开的方法停止线程，转换标志位
    public void stop(){
        this.flag = false;
    }

    public static void main(String[] args) {
        Demo1 demo1 = new Demo1();
        new Thread(demo1).start();

        for (int i = 0; i < 1000; i++) {
            System.out.println("******"+i);
            if (i==900){
                //调用stop方法切换标志位，让线程停止
                demo1.stop();
                System.out.println("线程该停止了");
            }
        }

    }
}
```

### 线程休眠

sleep(时间)指定当前线程阻塞的毫秒数;

sleep存在异常InterruptedException;

sleep时间达到后线程进入就绪状态;

sleep可以模拟网络延时，倒计时等。

每一个对象都有一个锁，sleep不会释放锁;

```java
package com.zy.thread_num3;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */
//模拟十秒倒计时
public class Demo2 {
    public static void main(String[] args) {
        try {
            tenDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void tenDown() throws InterruptedException {
        int num =10;
        while (true){
            Thread.sleep(1000);
            System.out.println(num--);
            if (num<=0){
                break;
            }
        }
    }
}
```

```java
package com.zy.thread_num3;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */

public class Demo2 {
    public static void main(String[] args) {
        //打印当前系统时间
        Date startTime = new Date(System.currentTimeMillis()); //获取系统当前时间
        while (true){
            try {
                Thread.sleep(1000);
                System.out.println(new SimpleDateFormat("HH:mm:ss").format(startTime));
                startTime = new Date(System.currentTimeMillis()); //更新当前时间
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }

    public static void tenDown() throws InterruptedException {
        int num =10;
        while (true){
            Thread.sleep(1000);
            System.out.println(num--);
            if (num<=0){
                break;
            }
        }
    }
}
```

### 线程礼让

```java
package com.zy.thread_num3;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */
//测试礼让线程
    //礼让线程不一定成功
public class Demo3 {
    public static void main(String[] args) {
        TestYield testYield = new TestYield();
        new Thread(testYield,"a").start();
        new Thread(testYield,"b").start();
    }
}

class TestYield implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "线程开始运行");
        Thread.yield();  //礼让
        System.out.println(Thread.currentThread().getName() + "线程停止运行");
    }
}
```

### 线程强制执行_join

Join合并线程，待此线程执行完成后，再执行其他线程，其他线程阻塞

```java
package com.zy.thread_num3;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */
public class TestJoin implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println("插队的来了"+i);
        }
    }

    public static void main(String[] args) {
        //启动线程
        TestJoin testJoin = new TestJoin();
        Thread thread = new Thread(testJoin);
        thread.start();

        //主线程
        for (int i = 0; i < 500; i++) {
            if (i==200){
                try {
                    thread.join();  //插队
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("正常站队--------->"+i);
        }

    }
}
```

### 线程检测

```java
package com.zy.thread_num3;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */
//观测线程状态
public class TestState {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(()->{
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("***********");
        });


        //观察状态
        Thread.State state = thread.getState();
        System.out.println(state);

        //观察启动后
        thread.start();
        state = thread.getState();
        System.out.println(state);

        while (state != Thread.State.TERMINATED){ //只要线程不终止，就一直输出状态
            Thread.sleep(100);
            state = thread.getState(); //更新
            System.out.println(state);
        }
    }
}
```

### 线程的优先级

在应用程序中，要对线程进行调度，最直接的方式就是设置线程的优先级。==优
先级越高的线程获得CPu执行的机会越大，而优先级越低的线程获得cPu执行
的机会越小。==
线程的优先级用1~10之间的整数来表示，==数字越大优先级越高==。
除了可以直接使用数字表示线程的优先级，还可以使用Thread类中提供的三个
静态常量表示线程的优先级。

![image-20210308221536373](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241638559_f3f0d3.webp)

```java
package com.zy.thread_num3;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */
public class TestPriority {
    public static void main(String[] args) {
        //主线称默认优先级
        System.out.println(Thread.currentThread().getName()+"---->"+Thread.currentThread().getPriority()); //5

        MyPriority myPriority = new MyPriority();

        Thread t1 = new Thread(myPriority);
        Thread t2 = new Thread(myPriority);
        Thread t3 = new Thread(myPriority);
        Thread t4 = new Thread(myPriority);
        Thread t5 = new Thread(myPriority);
        Thread t6 = new Thread(myPriority);


        //先设置优先级，再启动
        t1.start(); //5

        t2.setPriority(1);
        t2.start();  //1

        t3.setPriority(5);
        t3.start();  //5

        t4.setPriority(Thread.MAX_PRIORITY);
        t4.start();  //10

        /*t5.setPriority(-1);
        t5.start();  //报错

        t6.setPriority(11);
        t6.start();  //报错
        */



    }
}

class MyPriority implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"---->"+Thread.currentThread().getPriority());
    }
}
```

### 守护(daemon)线程

线程分为用户线程和守护线程

虚拟机必须确保用户线程执行完毕

虚拟机不用等待守护线程执行完毕

```java
package com.zy.thread_num3;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num3
 * @version: 1.0
 */
public class TestDaemon {
    public static void main(String[] args) {
        God god = new God();
        You you = new You();

        Thread thread = new Thread(god);
        thread.setDaemon(true); //默认为false表示是用户线程，正常的线程都是用户线程

        thread.start();  //上帝守护线程启动
        new Thread(you).start(); //用户线程启动
    }
}

class God implements Runnable{

    @Override
    public void run() {
        while (true){
            System.out.println("上帝保佑着你");
        }
    }
}

class You implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("一生都开心的活着");
        }
        System.out.println("good bye this world");
    }
}
```

### 线程同步

​		线程安全问题其实就是由多个线程同时处理共享资源所导致的。要想解决线程安全问题，必须得保证处理共享资源的代码在任意时刻只能有一个线程访问。为此, Java中提供了==线程同步机制==。

**线程不安全案例**

```java
package com.zy.thread_num4;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */

//不安全的买票
public class Demo1 {
    public static void main(String[] args) {
        Buyticket station = new Buyticket();

        new Thread(station,"张三").start();
        new Thread(station,"李四").start();
        new Thread(station,"王五").start();
    }
}
class Buyticket implements Runnable{

    //票
    private int ticketNum =10;
    boolean flag =true;  //外部停止方式

    @Override
    public void run() {
        //买票
        while (flag){
            try {
                buy();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void buy() throws InterruptedException {
        if (ticketNum<=0){
            flag =false;
            return;
        }

        //模拟延迟
        Thread.sleep(100);
        System.out.println(Thread.currentThread().getName()+"买到"+ticketNum--+"张票");
    }
}
```

```java
package com.zy.thread_num4;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */

//不安全的取钱
public class Demo2 {
    public static void main(String[] args) {
        //账户
        Account account = new Account(100,"购房基金");

        Drawing you = new Drawing(account, 50, "你");
        Drawing girlfriend = new Drawing(account, 100, "女朋友");

        you.start();
        girlfriend.start();


    }
}
//账户
class Account{
    int money;
    String name;

    public Account(int money, String name) {
        this.money = money;
        this.name = name;
    }

    public Account(int i) {
    }
}
//银行：模拟取款

class Drawing extends Thread{

    Account account; //账户
    int drawingMoney; //去了多少钱
    int nowMoney; //现在手里有多少钱

    public Drawing(Account account, int drawingMoney, String name) {
        super(name);
        this.account = account;
        this.drawingMoney = drawingMoney;
    }

    @Override
    public void run() {
        //判断有没有钱
        if (account.money-drawingMoney<0){
            System.out.println(Thread.currentThread().getName() + "钱不够，取不了");
            return;
        }


        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //卡内余额 = 余额 - 你取得钱
        account.money = account.money -drawingMoney;

        //你手里钱
        nowMoney = nowMoney + drawingMoney;
        System.out.println(account.name+"余额里的钱为："+account.money);

        //Thread.currentThread().getName() <==>this.getName()
        System.out.println(this.getName()+"手里的钱："+nowMoney);
    }
}
```

### 同步方法

由于我们可以通过private关键字来保证数据对象只能被方法访问，所以我们只
需要针对方法提出一套机制,这套机制就是synchronized关键字，它包括两种用法:synchronized方法和synchronized块．

```java
[修饰符]synchronized 返回值类型 方法名([参数1，.....]){}
```

在方法前面也可以使用synchronized关键字来修饰，被修饰的方法为同步方法,它能实现和同步代码块同样的功能。
被synchronized修饰的方法在某一时刻只允许一个线程访问，访问该方法的其他线程都会发生阻塞，直到当前线程访问完毕后，其他线程才有机会执行。

注意：

同步方法也有锁，它的锁就是当前调用该方法的对象，就是this指向的对象。

Java中静态方法的锁是该方法所在类的class对象，该对象可以直接类名.class的方式获取。

同步代码块和同步方法解决多线程问题有好处也有弊端。同步==解决了多个线程同时访问共享数据时的线程安全问题==，只要加上同一个锁，在同一时间内只能有一条线程执行，但是线程在执行同步代码时每次都会判断锁的状态，非常==消耗资源，效率较低==。

```java
//安全的买票
public class Demo1 {
    public static void main(String[] args) {
        Buyticket station = new Buyticket();

        new Thread(station,"张三").start();
        new Thread(station,"李四").start();
        new Thread(station,"王五").start();
    }
}
class Buyticket implements Runnable{

    //票
    private int ticketNum =10;
    boolean flag =true;  //外部停止方式

    @Override
    public void run() {
        //买票
        while (flag){
            try {
                buy();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    //synchronized 同步方法，锁的是this
    private synchronized void buy() throws InterruptedException {
        if (ticketNum<=0){
            flag =false;
            return;
        }

        //模拟延迟
        Thread.sleep(100);
        System.out.println(Thread.currentThread().getName()+"买到"+ticketNum--+"张票");
    }
}
```

### 同步块



```java
synchronized(lock){  #lock是一个任意类型的锁对象
    //操作共享资源代码块
    ...
}
#synchronized关键字后大括号{}内包含的就是需要同步操作的共享资源代码块
```

注意： lock锁对象可以是任意类型的对象，但多个线程共享的锁对象必须是相同的。锁对象的创建代码不能放到run()方法中，否则每个线程运行到run()方法都会创建一个新对象，这样每个线程都会有一个不同的锁。

```java
//安全的取钱
public class Demo2 {
    public static void main(String[] args) {
        //账户
        Account account = new Account(100,"购房基金");

        Drawing you = new Drawing(account, 50, "你");
        Drawing girlfriend = new Drawing(account, 100, "女朋友");

        you.start();
        girlfriend.start();


    }
}
//账户
class Account{
    int money;
    String name;

    public Account(int money, String name) {
        this.money = money;
        this.name = name;
    }

    public Account(int i) {
    }
}
//银行：模拟取款

class Drawing extends Thread{

    Account account; //账户
    int drawingMoney; //去了多少钱
    int nowMoney; //现在手里有多少钱

    public Drawing(Account account, int drawingMoney, String name) {
        super(name);
        this.account = account;
        this.drawingMoney = drawingMoney;
    }
    //synchronized 默认锁的是this
    @Override
    public void run() {
        //所的对象就是变化的量，需要增删改的对象
        synchronized (account){
            //判断有没有钱
            if (account.money-drawingMoney<0){
                System.out.println(Thread.currentThread().getName() + "钱不够，取不了");
                return;
            }


            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            //卡内余额 = 余额 - 你取得钱
            account.money = account.money -drawingMoney;

            //你手里钱
            nowMoney = nowMoney + drawingMoney;
            System.out.println(account.name+"余额里的钱为："+account.money);

            //Thread.currentThread().getName() <==>this.getName()
            System.out.println(this.getName()+"手里的钱："+nowMoney);
        }

    }
}
```

```java
package com.zy.thread_num4;

import java.util.concurrent.CopyOnWriteArrayList;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */
//测试JUC安全类型的集合
public class Demo3 {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<String>();
        for (int i = 0; i < 10000; i++) {
            new Thread(()->{
                list.add(Thread.currentThread().getName());
            }).start();
        }
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(list.size());
    }
}
```

### 同步锁

从JDK5开始，Java增加了一个功能更强大的Lock锁。其最大的优势在于Lock锁可以让某个线程在持续获取同步锁失败后返回，不再继续等待，另外Lock锁在使用是也更加灵活。

注意：

​	ReentrantLock类是Lock锁接口的实现类，也是常用的同步锁，在该同步锁中除了lock()方法和unlock()方法外，还提供了一些其他同步锁操作的方法，例如tryLock()方法可以判断某个线程锁是否可用。

​	在使用Lock同步锁时，可以根据需要在不同代码位置灵活的上锁和解锁，为了保证所有情况下都能正常解锁以确保其他线程可以执行，通常情况下会在finallyi代码块中调用unlock()方法来解锁。

```java
package com.zy.thread_num4;

import java.util.concurrent.locks.ReentrantLock;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */
//测试Lock锁
public class TestLock {
    public static void main(String[] args) {
        TestLock2 testLock2 = new TestLock2();

        new Thread(testLock2).start();
        new Thread(testLock2).start();
        new Thread(testLock2).start();
    }
}
class TestLock2 implements Runnable{

    int ticketNum = 10;

    //定义lock锁
    private final ReentrantLock lock =new ReentrantLock();

    @Override
    public void run() {
        while (true){

            try {
                lock.lock(); //加锁
                if (ticketNum>0){
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(ticketNum--);
                }else {
                    break;
                }
            }finally {
                lock.unlock(); //解锁
            }

        }
    }
}
```

### 对比

**synchronized 与Lock的对比**

Lock是显式锁（手动开启和关闭锁，别忘记关闭锁) synchronized是隐式锁，出了
作用域自动释放

Lock只有代码块锁，synchronized有代码块锁和方法锁

使用Lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性(提供更多的子类)

优先使用顺序:
	Lock >同步代码块（已经进入了方法体，分配了相应资源)>同步方法（在方法体之外)

### 死锁

多个线程各自占有一些共享资源﹐并且互相等待其他线程占有的资源才能运行﹐而导致两个或者多个线程都在等待对方释放资源﹐都停止执行的情形.某一个同步块同时拥有“两个以上对象的锁”时﹐就可能会发生“死锁”的问题.

```java
//死锁：多个线程互相抱着对方需要的资源，然后形成僵持。
public class DeadLock {
    public static void main(String[] args) {
        Makeup g1 = new Makeup(0, "灰姑娘");
        Makeup g2 = new Makeup(1, "白雪公主");

        g1.start();
        g2.start();

    }
}
//口红
class Lipstick{

}
//镜子
class Mirror{

}

//化妆
class Makeup extends Thread{

    //需要的资源只有一份
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();

    int choice; //选择
    String girlName; //使用化妆品的人

    Makeup(int choice,String girlName){
        this.choice = choice;
        this.girlName = girlName;
    }


    @Override
    public void run() {
        //化妆
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    //化妆，互相持有对方的锁，就是需要拿到对方的资源
    private void makeup() throws InterruptedException {
        if (choice==0){
            synchronized (lipstick){ //获得口红的锁
                System.out.println(this.girlName+"获得口红的锁");
                Thread.sleep(1000);

                synchronized (mirror){ //一秒钟后获得镜子的锁
                    System.out.println(this.girlName+"获得镜子的锁");
                }
            }
        }else {
            synchronized (mirror){ //获得镜子的锁
                System.out.println(this.girlName+"获得镜子的锁");
                Thread.sleep(2000);

                synchronized (lipstick){ //二秒钟后获得口红的锁
                    System.out.println(this.girlName+"获得口红的锁");
                }
            }
        }
    }

}
```

```java
package com.zy.thread_num4;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */


/*
* 产生死锁的四个必要条件:
    1．互斥条件:一个资源每次只能被一个进程使用。
    2．请求与保持条件:一个进程因请求资源而阻塞时，对已获得的资源保持不放。
    3. 不剥夺条件︰进程已获得的资源，在末使用完之前，不能强行剥夺。
    4．循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。
* 解决：
    只要想办法破掉其中的任意一个或多个条件就可以避免死锁发生。   
*避免死锁的发生：
	加锁顺序(线程按照一定的顺序加锁)
	加锁时限(线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁)
	死锁检测
* */

//死锁：多个线程互相抱着对方需要的资源，然后形成僵持。
//解决：将锁放在外面
public class DeadLock {
    public static void main(String[] args) {
        Makeup g1 = new Makeup(0, "灰姑娘");
        Makeup g2 = new Makeup(1, "白雪公主");

        g1.start();
        g2.start();

    }
}
//口红
class Lipstick{

}
//镜子
class Mirror{

}

//化妆
class Makeup extends Thread{

    //需要的资源只有一份
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();

    int choice; //选择
    String girlName; //使用化妆品的人

    Makeup(int choice,String girlName){
        this.choice = choice;
        this.girlName = girlName;
    }


    @Override
    public void run() {
        //化妆
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    //化妆，互相持有对方的锁，就是需要拿到对方的资源
    private void makeup() throws InterruptedException {
        if (choice==0){
            synchronized (lipstick){ //获得口红的锁
                System.out.println(this.girlName+"获得口红的锁");
                Thread.sleep(1000);


            }
            synchronized (mirror){ //一秒钟后获得镜子的锁
                System.out.println(this.girlName+"获得镜子的锁");
            }
        }else {
            synchronized (mirror){ //获得镜子的锁
                System.out.println(this.girlName+"获得镜子的锁");
                Thread.sleep(2000);

            }
            synchronized (lipstick){ //二秒钟后获得口红的锁
                System.out.println(this.girlName+"获得口红的锁");
            }
        }
    }

}
```

### 线程通信

Java在Object类中提供了wait()、notify()、notifyAll()等方法用于解决线程间的通信问题，因为Java中所有类都是Object类的子类或间接子类，因此任何类的实例对象都可以直接使用这些方法。

![image-20210313164903582](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241638613_d79491.webp)

说明︰其中wait()方法用于使当前线程进入等待状态,notify()和notifyAll()方法用于唤醒当前处于等待状态的线程。

注意:wait()、notify()和notifyAll()方法的调用者都应该是同步锁对象，如果这三个方法的调用者不是同步锁对象,Java虚拟机就会抛出IllegalMonitorStateException异常。

```java
package com.zy.thread_num4;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */
//测试：生产者消费者模型--->利用缓冲区解决：管程法
//生产者，消费者，产品，缓冲区
public class TestPc {
    public static void main(String[] args) {
        SynContainer container =new SynContainer();

        new Productor(container).start();
        new Consumer(container).start();
    }
}

//生产者
class Productor extends Thread{
    SynContainer container;

    public Productor(SynContainer synContainer) {
        this.container = synContainer;
    }

    //生产

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("生产了"+i+"只鸡");
            container.push(new Chicken(i));
        }
    }
}

//消费者
class Consumer extends Thread{
    SynContainer container;

    public Consumer(SynContainer synContainer) {
        this.container = synContainer;
    }

    //消费
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了--->"+container.pop().id+"只鸡");
        }
    }
}
//产品
class Chicken{
    int id;  //产品编号

    public Chicken(int id) {
        this.id = id;
    }
}

//缓冲区
class SynContainer{

    //需要一个容器大小
    Chicken[] chickens = new Chicken[10];
    //容器计数器
    int count =0;
    //生产者放入产品
    public synchronized void push(Chicken chicken){
        //如果容器满了，就需要等待消费者消费
        if (count==chickens.length){
            //通知消费者消费，生产者等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果没有满，我们就需要丢入产品
        chickens[count] = chicken;
        count++;

        //可以通知消费者消费了
        this.notifyAll();
    }


    //消费者消费产品
    public synchronized Chicken pop(){
        //判断能否消费
        if (count==0){
            //等待生产者生产，消费者等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果可以消费
        count--;
        Chicken chicken = chickens[count];

        //吃完了，通知生产者生产
        this.notifyAll();
        return chicken;
    }
}
```

```java
package com.zy.thread_num4;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */
//测试生产者和消费者问题2：信号灯法  利用标志位
public class TestPc2 {
    public static void main(String[] args) {
        TV tv = new TV();
        new Player(tv).start();
        new Watcher(tv).start();
    }
}
//生产者--->演员
class Player extends Thread{
    TV tv;
    public Player(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i%2==0){
                this.tv.play("快了大本营播放中。。。");
            }else {
                this.tv.play("抖音：记录美好生活");
            }
        }
    }
}

//消费者--->观众
class Watcher extends Thread{
    TV tv;
    public Watcher(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}

//产品---->节目
class TV{
    //演员表演，观众等待 T
    //观众观看，演员等待 F
    String voice; //表演的节目
    boolean flag = true;

    //表演
    public synchronized void play(String voice){

        if (!flag){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("演员表演了："+voice);
        //通知观众观看
        this.notifyAll(); //通知唤醒
        this.voice = voice;
        this.flag =!this.flag;
    }

    //观看
    public synchronized void watch(){
        if (flag){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观看了："+voice);
        //观看演员表演
        this.notifyAll();
        this.flag = !this.flag;
    }

    //观看
}
```

### 线程池

在大规模应用程序中，创建、分配和释放多线程对象会产生大量内存管理开销。为此，可以考虑使用Java提供的线程池来创建多线程,进一步优化线程管理。

 **Executor接口实现线程池管理**

步骤︰

1.创建一个实现Runnable接口或者Callable接口的实现类，同时重写run()或者call()方法﹔
2.创建Runnable接口或者Callable接口的实现类对象﹔
3.使用Executors线程执行器类创建线程池;
4.使用ExecutorService执行器服务类的submit()方法将Runnable接口或者Callable接口的实现类对象提交到线程池进行管理;
5.线程任务执行完成后，可以使用shutdown()方法关闭线程池。

```java
package com.zy.thread_num4;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */
public class TestPool {
    public static void main(String[] args) {
        //1.创建服务，创建线程池
        //newFixedThreadPool 参数为：线程池大小
        ExecutorService service = Executors.newFixedThreadPool(10);

        //执行
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());

        //2.关闭连接
        service.shutdown();
    }
}
class MyThread implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
ce();
            }
        }

        System.out.println("演员表演了："+voice);
        //通知观众观看
        this.notifyAll(); //通知唤醒
        this.voice = voice;
        this.flag =!this.flag;
    }

    //观看
    public synchronized void watch(){
        if (flag){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观看了："+voice);
        //观看演员表演
        this.notifyAll();
        this.flag = !this.flag;
    }

    //观看
}
```

### 线程池

在大规模应用程序中，创建、分配和释放多线程对象会产生大量内存管理开销。为此，可以考虑使用Java提供的线程池来创建多线程,进一步优化线程管理。

 **Executor接口实现线程池管理**

步骤︰

1.创建一个实现Runnable接口或者Callable接口的实现类，同时重写run()或者call()方法﹔
2.创建Runnable接口或者Callable接口的实现类对象﹔
3.使用Executors线程执行器类创建线程池;
4.使用ExecutorService执行器服务类的submit()方法将Runnable接口或者Callable接口的实现类对象提交到线程池进行管理;
5.线程任务执行完成后，可以使用shutdown()方法关闭线程池。

```java
package com.zy.thread_num4;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @Auther: 赵羽
 * @Description: com.zy.thread_num4
 * @version: 1.0
 */
public class TestPool {
    public static void main(String[] args) {
        //1.创建服务，创建线程池
        //newFixedThreadPool 参数为：线程池大小
        ExecutorService service = Executors.newFixedThreadPool(10);

        //执行
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());

        //2.关闭连接
        service.shutdown();
    }
}
class MyThread implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
```
