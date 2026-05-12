+++
date = '2024-11-10T17:25:29+08:00'
draft = false
title = 'Day 03 java流程控制'
slug = "f8ec3i652s"
description = "Day 03 java流程控制"
categories = ["📒开发"]
tags = ["Java"]
summary = "Day 03 java流程控制"
comments = true  
+++
﻿
# Day-03-java流程控制

### Scanner对象

java.util.Scanner是Java5的新特性，我们可以通过Scanner类来获取用户的输入。

基本语法

```java
Scanner s = new Scanner(System.in);
```

通过Scanner类的next()与nextLine()方法获取输入的字符串，在读取前我们一般需要使用hasNext() 与hasNextLine()判断是否还有输入的数据。

```java
public static void main(String[] args) {
    //创建一个扫描对象，用于接受键盘数据
    Scanner scanner = new Scanner(System.in);
    System.out.println("使用next方式接受：");
    //判断用户有没有输入字符串
    if (scanner.hasNext()){
        //使用next方式接受
        String str = scanner.next();
        System.out.println("输出的内容为："+str);
        //凡是属于IO流的类如果不关闭会一直占用资源
        scanner.close();
    }
}
```

```java
public static void main(String[] args) {
    //从键盘接收数据
    Scanner scanner = new Scanner(System.in);
    System.out.println("使用nextLine方式接受");
    //判断是否还有输入
    if (scanner.hasNextLine()){
        String str = scanner.nextLine();
        System.out.println("输出的内容是："+str);
        scanner.close();
    }
}
```

**next():**
1、一定要读取到有效字符后才可以结束输入。
2、对输入有效字符之前遇到的空白，next()方法会自动将其去掉。

3、只有输入有效字符后才将其后面输入的空白作为分隔符或者结束符。

4、next()不能得到带有空格的字符串。
**nextLine():**
1、以Enter为结束符,也就是说nextLine()方法返回的是输入回车之前的所有字符。

2、可以获得空白。

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);

    int i = 0;
    float f = 0.0f;
    System.out.println("请输入整数：");

    if (scanner.hasNextInt()){
        i = scanner.nextInt();
        System.out.println("整数数据："+i);
    }else {
        System.out.println("输入的不是整数数据！");
    }
    System.out.println("请输入小数：");

    if (scanner.hasNextFloat()){
        f = scanner.nextFloat();
        System.out.println("小数数据："+f);
    }else {
        System.out.println("输入的不是小数数据！");
    }
```

```java
public static void main(String[] args) {
    //我们可以输入多个数字，并求其总和与平均数，每输入一个数字用回车确认，通过输入非数字来结束输入并输出执行结果:
    Scanner scanner=  new Scanner(System.in);
    //和
    double sum = 0;
    //计算输入了多少个数字
    int m =0;
    System.out.println("请输入数据：");
    //通过循环判断是否还有输入，并在里面对每一次进行求和和统计
    while (scanner.hasNextDouble()){
        double x = scanner.nextDouble();
        //m = m + 1;
        m++;
        sum = sum + x;
        System.out.println("你输入了第"+m+"个数据，然后当前结果sum："+sum);
    }
    System.out.println(m+"个数的和为："+sum);
    System.out.println(m + "个数的平均数为："+(sum/m));
    scanner.close();
}
```

### 顺序结构

JAVA的基本结构就是顺序结构，除非特别指明，否则就按照顺序一句一句执行。顺序结构是最简单的算法结构。

![在这里插入图片描述](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624955_c16cf9.webp)


语句与语句之间，框与框之间是按从上到下的顺序进行的，它是由若干个依次执行的处理步骤组成的，它是任何一个算法都离不开的一种基本算法结构。

### 选择结构

**if单选择结构**

**if双选择结构**

**if多选择结构**

**嵌套的if结构**
**switch多选择结构**

#### if单选择结构

我们很多时候需要去判断(个东西是否可行，然后我们才去执行，这样一个过程在程序中用if语句来表示

![在这里插入图片描述](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241624044_e9d3aa.webp)


**语法：**

```java
if(布尔表达式){
	//如果布尔表达式为true将执行的语句
}
```

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    System.out.println("请输入内容：");
    String s = scanner.nextLine();
    if (s.equals("Hello")){
        System.out.println(s);
    }
    System.out.println("End");
    scanner.close();
}
```

#### if 双选择结构

![在这里插入图片描述](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241625371_3a3a4f.webp)


**语法：**

```java
if(布尔表达式){
	//如果布尔表达式的值为true
}else{
	//如果布尔表达式的值为false
}
```

```java
public static void main(String[] args) {
    //如果分数大于60就是及格，小于60就是不及格
    Scanner scanner = new Scanner(System.in);
    System.out.println("请输入你的成绩：");
    int score = scanner.nextInt();
    if (score<60){
        System.out.println("不及格");
    }else {
        System.out.println("及格");
    }

    scanner.close();
}
```

#### if 多选择结构

![在这里插入图片描述](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202310241625771_a5f232.webp)


**语法：**

```java
if(布尔表达式1){
	//如果布尔表达式1的值为true执行代码
}else if(布尔表达式2){
	//如果布尔表达式2的值为true执行代码
}else if(布尔表达式3){
	//如果布尔表达式3的值为true执行代码
}else {
    //如果以上布尔表达式都不为true执行代码
}
```

```java
public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入你的成绩：");
        int score = scanner.nextInt();
        if (score==100){
            System.out.println("恭喜满分！");
        }else if (score<60 && score>=0){
            System.out.println("不及格");
        }else if (score<70 && score>=60){
            System.out.println("及格");
        }else if (score<80 && score>=70){
            System.out.println("差");
        }else if (score<90 && score>=80){
            System.out.println("良");
        }else if (score<100 && score>=90){
            System.out.println("优秀");
        }else {
            System.out.println("输入不合法！");
        }
        scanner.close();
    }
```

#### 嵌套的if结构

使用嵌套的if...else语句是合法的。也就是说你可以在另一个if或者else if语句中使用if或者else if 语句。你可以像if 语句一样嵌套else if...else。
语法:

```java
if(布尔表达式1){
	//如果布尔表达式1的值为true执行代码
    if(布尔表达式2){
		//如果布尔表达式2的值为true执行代码
    }
}
```

#### 嵌套练习

```java
public class demo2 {
    public static void main(String[] args) {
        //1 给出积分
        //键盘接受数据
        Scanner sc = new Scanner(System.in);
        
        System.out.println("请输入会员积分：");
        
        //先判断键盘输入的数据是不是整型
        if (sc.hasNextInt()==true){
            int score = sc.nextInt();
            //判断这个整数是否为正数
            if (score>=0){
                String discount = "";
                //2 根据积分判断折扣
                if (score>=8000){
                    discount = "0.6";
                }else if (score>=4000){
                    discount = "0.7";
                }else if (score>=2000){
                    discount = "0.8";
                }else{
                    discount = "0.9";
                }
                
                System.out.println("该会员享受的折扣为："+discount);
                
            }else {
                System.out.println("你输入的数据是负数，不符合要求");
            }
        }else {
            System.out.println("你输入的数据不是整型");
        }
    }
}
```

#### switch多选择结构

多选择结构还有一个实现方式就是switch case语句。

switch case语句判断一个变量与一系列值中某个值是否相等，每个值称为一个分支。

为了防止代码的“穿透”效果：在每个分支后面加上一个关键字break，遇到break这个分支就结束了

类似else的“兜底”“备胎”的分支：default分支

default分支可以写在任意的位置上，但是如果没有在最后一行，后面必须加上break关键字

相邻分支逻辑是一样的，那么就可以只保留最后一个分支，上面的都可以省去不写了

switch分支和if分支区别：

表达式是等值判断的话——if，switch都可以

如果表达式时区间判断的情况——if最好

switch应用场合：就是等值判断，等值的情况比较少的情况下

```java
switch(expression){
	case value :
		//语句
		break;//可选
    case value :
		//语句
		break; //可选
	//你可以有任意数量的case语句
    default ://可选
		//语句
}
```

**switch语句中的变量类型可以是:**

byte、short、int或者char，String，枚举。

同时case标签必须为字符串常量或字面量。

```java
public static void main(String[] args) {
    //case 穿透  //switch 匹配一个具体的值
    char grade = 'F';
    switch (grade){
        case 'A':
            System.out.println("优秀");
            break;
        case 'B':
            System.out.println("良好");
            break;
        case 'C':
            System.out.println("及格");
            break;
        case 'D':
            System.out.println("再接再厉");
            break;
        default:
            System.out.println("未知等级");
    }
}
```

从Java SE7开始

switch支持字符串String类型了

```java
public static void main(String[] args) {
    String name = "赵云";
    switch (name){
        case "英雄":
            System.out.println("英雄");
            break;
        case "五虎上将":
            System.out.println("五虎上将");
            break;
        default:
            System.out.println("不知道");
    }
}
```

### 循环结构

**while循环**

**do...while**

**循环for循环**

在Java5中引入了一种主要用于数组的增强型for循环。

#### while循环

while是最基本的循环，它的结构为:

```java
while(布尔表达式 ){
	/循环内容
}
```

只要布尔表达式为true，循环就会一直执行下去。
我们大多数情况是会让循环停止下来的，我们需要一个让表达式失效的方式来结束循环。

少部分情况需要循环一直执行，比如服务器的请求响应监听等。
循环条件一直为true就会造成无限循环【死循环】，我们正常的业务编程中应该尽量避免死循环。会影响程序性能或者造成程序卡死奔溃!

```java
public static void main(String[] args) {
    //计算1+2+...+100?
    int i =1;    //条件初始化
    int sum = 0;  
    while (i<=100){  //条件判断
		sum+=i;      //循环体
        i++;         //迭代
    }
    System.out.println(sum);
    //计算2+4+...+998+1000
    int i =2;    
    int sum = 0;  
    while (i<=1000){  
		sum += i;     
        i = i +2;      
    }
    System.out.println(sum);
    //计算1*3*5*7*9*11*13
    int i = 1;
    int result = 1;
    while (i<=13){
        result *= i;
        i += 2;
    }
    System.out.println(result);
}
```

#### do...while循环

对于while语句而言，如果不满足条件，则不能进入循环。但有时候我们需要即使不满足条件,也至少执行一次。
do...while循环和while循环相似，不同的是，do...while循环至少会执行一次。

```java
do {
	//代码语句
}while(布尔表达式);
```

```java
public static void main(String[] args) {
    int i = 0;
    int sum = 0;
    do {
        sum+=i;
        i++;
    }while (i<=100);
    System.out.println(sum);
}
```

While和do-While的区别:
	while：先判断，后执行。

​	dowhile：先执行，后判断! 至少被执行一次，第二次才开始判断！

​	Do..while总是保证循环体会被至少执行一次!这是他们的主要差别。

```java
public static void main(String[] args) {
    int a =0;
    while (a<0){
        System.out.println(a);  //不输出
        a++;
    }
    System.out.println("===============");
    do {
        System.out.println(a);  //输出0
        a++;
    }while (a<0);
}
```

#### For循环

虽然所有循环结构都可以用while或者do...while表示，但Java提供了另一种语句——for循环，使一些循环结构变得更加简单。

for循环语句是支持迭代的一种通用结构，**是最有效、最灵活的循环结构**。

for循环执行的次数是在执行前就确定的。

语法格式如下:

```java
for(初始化;条件判断;迭代){
	//代码语句
}
```

**关于for循环有以下几点说明:**

​	最先执行初始化步骤。可以声明一种类型，但可初始化一个或多个循环控制变量，也可以是空语句。

​	然后，检测布尔表达式的值。如果为 true，循环体被执行。如果为false，循环终止，开始执行循环体后面的语句。

​	执行一次循环后，更新循环控制变量(迭代因子控制循环变量的增减)。

​	再次检测布尔表达式。循环执行上面的过程。

​	i的作用域：作用范围：离变量最近{}

​	循环分为两大类：

​	第一类：当型  while(){}   for(){}

​	第二类：直到型    do{}while();

```java
//死循环
for (;;){
    
}
while(true){
    
}
do{
    
}while(true);
```

```java
public static void main(String[] args) {
    //练习一：计算0到100之间的奇数和偶数的和
    int oddSum = 0;
    int evenSum = 0;
    for (int i = 0; i <= 100; i++) {
        if (i%2!=0){
            oddSum+=i;
        }else {
            evenSum+=i;
        }
    }
    System.out.println("奇数的和："+oddSum);  //2500
    System.out.println("偶数的和："+evenSum);  //2550
}
```

```java
public static void main(String[] args) {
    //练习二：用while或for循环输出1——1000之间能被5整除的数，并且每行输出3个
    for (int i = 0; i <= 1000; i++) {
        if (i%5==0){
            System.out.print(i+"\t");
        }
        if (i%(3*5)==0){  //每行输出3个
            System.out.println();
            //System.out.println("\n");
        }
    }
    //println  输出完会换行
    //print  输出完不会换行
}
```

```java
public static void main(String[] args) {
    /*打印九九乘法表
    1.先打印出第一列
    2.再用一个循环包起来
    3.去掉重复项 i <= j
    4.调整样式
     */
    for (int i = 1; i <=9 ; i++) {
            for (int j = 1; j <=i ; j++) {
                System.out.print(i + "*" + j + "=" + (i*j) + "\t");
            }
            System.out.println();
        }
        System.out.println("===============================");
        for (int i = 9; i >=1 ; i--) {
            for (int j = 1; j <=i ; j++) {
                System.out.print(i + "*" + j + "=" + (i*j) + "\t");
            }
            System.out.println();
        }
}
```

#### 增强for循环

数组重点使用

Java5引入了一种主要用于数组或集合的增强型for循环。

Java 增强for循环语法格式如下:

```java
for(声明语句∶表达式)
{
	//代码句子
}
```

声明语句:声明新的局部变量，该变量的类型必须和数组元素的类型匹配。其作用域限定在循环语句块，其值与此时数组元素的值相等。

表达式:表达式是要访问的数组名，或者是返回值为数组的方法。

```java
public static void main(String[] args) {
        int [] numbers = {10,20,30,40,50};  //定义了一个数组

        for (int i =0;i<5;i++){
            System.out.println(numbers[i]);
        }
        System.out.println("=====================");
        //遍历数组的元素
        for (int x:numbers){
            System.out.println(x);
        }
    }
```

### break continue

break在任何循环语句的主体部分，均可用break控制循环的流程。**break用于强行退出循环**，不执行循环中剩余的语句。(break语句也在switch语句中使用)

continue语句用在循环语句体中，**用于终止某次循环过程**，即跳过循环体中尚未执行的语句，接着进行下一次是否执行循环的判定。

return的作用，**结束当前所在方法的执行**。

```java
break的作用：停止最近的循环
public static void main(String[] args) {
        for (int i = 1; i <=100 ; i++) {
            System.out.println(i);
            while (i==36){
                break;  //break停止的是里面的while循环，而不是外面的for循环
            }
        }
    }
```

```java
/*
	功能：输出1——100中被6整除的数：
        方式一：
        for (int i = 1; i <=100 ; i++) {
            if (i%6==0){
                System.out.println(i);
            }
        }

         */
        /*
        方式二
         
        for (int i = 1; i <=100 ; i++) {
            if (i%6!=0){
                continue;  //停止本次循环，继续下一次循环
            }
            System.out.println(i);
        }
        
         */
```



```java
public static void main(String[] args) {
    int i =0;
    while (i<100){
        i++;
        System.out.println(i);
        if (i==30){
            break;
        }
    }
    //continue
    int d = 0;
    while (d<100){
        d++;
        if (d%10==0){
            System.out.println();
            continue;
        }
        System.out.print(d);
    }
    /*
    1.break在任何循环语句的主体部分，均可用break控制循环的流程。
    2.break用于强行退出得环，不执行循环中剩余的语句。(break 语句也在switch语句中使用)
    3.continue 语句用在循环语句体中,用于终s止t某次循环过程，即跳过循环体中尚未执行的语句，接着进行下一 次是否执行循环的判定。
     */
}
```

**关于goto关键字**

​	goto关键字很早就在程序设计语言中出现。尽管goto仍是Java的一个保留字，但并未在语言中得到正式使用;Java没有goto。然而，在break和continue这两个关键字的身上，我们仍然能看出一些goto的影子---带标签的break和continue。
​	“标签”是指后面跟一个冒号的标识符，例如:label
​	对Java来说唯一用到标签的地方是在循环语句之前。而在循环之前设置标签的唯一理由是:我们希望在其中嵌套另一个循环，由于break和continue关键字通常只中断当前循环，但若随同标签使用，它们就会中断到存在标签的地方。

```java
public static void main(String[] args) {
    //打印101-150之间所有的质数
    //质数是指大于1的自然数中，除了1和它本身以外不再有其他因数的自然数
    //不建议使用
    int count = 0;
    outer:for (int i =101;i<150;i++){
        for (int j = 2; j<i/2;j++){
            if (i % j ==0){
                continue outer;
            }
        }
        System.out.print(i+" ");
    }
}
```

### 练习一：

```java
//输出输出1——100被5整除的数，每行输出6个
int sount = 0;  //引入一个计数器，初始值为0
for (int i = 1; i <=100 ; i++) {
    if (i%5==0){  //被5整除的数
        System.out.print(i+"\t");
        sount ++;  //每在控制台输出一个数，count就加1操作
        if (sount%6==0){
            System.out.println(); //换行
        }
    }
}
```

### 练习二

```java
/*
        功能：
            1.请输入10个整数，当输入的数是666的时候，退出程序
            2.判断其中录入正数的个数并输出
            3.判断系统的退出状态：是正常退出还是被迫退出
         */
        int count = 0;  //引入一个计数器
        boolean flag = true; //引入一个布尔类型的变量     --》理解为一个“开关”，默认情况下开关是开着的
        Scanner sc = new Scanner(System.in);
        for (int i = 1; i <=10 ; i++) { //i：循环次数
            System.out.println("请录入第"+i+"个数：");
            int num = sc.nextInt();
            if (num>0){ //录入的正数
                count++;
            }
            if (num==666){
                flag=false; //当遇到666的时候，“开关”被关上
                //退出循环
                break;
            }
        }
        System.out.println("你录入的正数个数为"+count);

        if (flag){  //flag=true
            System.out.println("正常退出");
        }else {   //flag = false
            System.out.println("被迫退出");
        }
```

### 练习三

```java
public static void main(String[] args) {
    //打印三角形  5行
    for (int i = 1; i <= 5; i++){
        for (int j = 5;j >=i ; j--){
            System.out.print(" ");
        }
        for (int j =1;j <=i ; j++){
            System.out.print("*");
        }
        for (int j =1;j < i ; j++){
            System.out.print("*");
        }
        System.out.println();
    }
}
```

### 练习四

```java
//前面带有空隙的长方形
for (int j = 1; j <=4 ; j++) { //控制行数
    for (int i = 1; i <=5 ; i++) { //控制空格的个数
        System.out.print(" ");
    }
    for (int i = 1; i <=9 ; i++) { //控制*的个数
        System.out.print("*");
    }
    System.out.println();
}
//平行四边形
		for (int j = 1; j <=4 ; j++) { //控制行数
            for (int i = 1; i <=(9-j) ; i++) { //控制空格的个数
                System.out.print(" ");
            }
            for (int i = 1; i <=9 ; i++) { //控制*的个数
                System.out.print("*");
            }
            System.out.println();
        }
//实心菱形一
		//上面三角形
        for (int j = 1; j <=4 ; j++) { //控制行数
            for (int i = 1; i <=(9-j) ; i++) { //控制空格的个数
                System.out.print(" ");
            }
            for (int i = 1; i <=(2*j-1) ; i++) { //控制*的个数
                System.out.print("*");
            }
            System.out.println();
        }
        //下面三角形
        for (int j = 1; j <=3 ; j++) { //控制行数
            for (int i = 1; i <=(j+5) ; i++) { //控制空格的个数
                System.out.print(" ");
            }
            for (int i = 1; i <=(7-2*j) ; i++) { //控制*的个数
                System.out.print("*");
            }
            System.out.println();
        }
//实心菱形二
		int size = 15;
        int startNum = size/2+1; //起始列号
        int endNum = size/2+1;  //结束列号
        boolean flag = true;
        for (int j = 1; j <=size ; j++) {
            for (int i = 1; i <=size ; i++) {
                if (i>=startNum && i<=endNum){
                    System.out.print("*");
                }else {
                    System.out.print(" ");
                }
            }
            //换行
            System.out.println();
            if (endNum==size){
                flag = false;
            }
            if (flag){
                startNum--;
                endNum++;
            }else {
                startNum++;
                endNum--;
            }
        }

//空心菱形一
		//上面三角形
        for (int j = 1; j <=4 ; j++) { //控制行数
            for (int i = 1; i <=(9-j) ; i++) { //控制空格的个数
                System.out.print(" ");
            }
            for (int i = 1; i <=(2*j-1) ; i++) { //控制*的个数
                if (i==1 || i == (2*j-1)){
                    System.out.print("*");
                }else {
                    System.out.print(" ");
                }
            }
            System.out.println();
        }
        //下面三角形
        for (int j = 1; j <=3 ; j++) { //控制行数
            for (int i = 1; i <=(j+5) ; i++) { //控制空格的个数
                System.out.print(" ");
            }
            for (int i = 1; i <=(7-2*j) ; i++) { //控制*的个数
                if (i==1 || i == (7-2*j)){
                    System.out.print("*");
                }else {
                    System.out.print(" ");
                }
            }
            System.out.println();
        }
//空心菱形二
		int size = 15;
        int startNum = size/2+1; //起始列号
        int endNum = size/2+1;  //结束列号
        boolean flag = true;
        for (int j = 1; j <=size ; j++) {
            for (int i = 1; i <=size ; i++) {
                if (i==startNum || i==endNum){
                    System.out.print("*");
                }else {
                    System.out.print(" ");
                }
            }
            //换行
            System.out.println();
            if (endNum==size){
                flag = false;
            }
            if (flag){
                startNum--;
                endNum++;
            }else {
                startNum++;
                endNum--;
            }
        }
```

