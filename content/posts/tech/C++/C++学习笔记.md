+++
date = '2024-07-14T10:38:44+08:00'
draft = true
title = 'C++学习笔记'
slug = "w9ipl974gh"
description = "学习C++语言笔记"
categories = ["📒C++"]
tags = ["C++"]
summary = "学习C++语言做的笔记"
+++
# C++基础入门

## 1 C++初识

### 1.1  第一个C++程序

编写一个C++程序总共分为4个步骤

* 创建项目
* 创建文件
* 编写代码
* 运行程序

```c++
#include <iostream>
using namespace std;
int main()
{
	cout << "hello C++" << endl;
	system("pause");
	return 0;
}
```

### 1.2 注释

**作用**：在代码中加一些说明和解释，方便自己或其他程序员程序员阅读代码

**两种格式**

1. **单行注释**：`// 描述信息` 
   - 通常放在一行代码的上方，或者一条语句的末尾，==对该行代码说明==
2. **多行注释**： `/* 描述信息 */`
   - 通常放在一段代码的上方，==对该段代码做整体说明==

> 提示：编译器在编译代码时，会忽略注释的内容

### 1.3 变量

**作用**：给一段指定的内存空间起名，方便操作这段内存

**语法**：`数据类型 变量名 = 初始值;`

**示例：**

```c++
#include <iostream>
using namespace std; 
int main(){
	int a = 10;
	cout << "a="<< a << endl;
	system("pause");
	return 0;
}
```

> 注意：C++在创建变量时，必须给变量一个初始值，否则会报错

### 1.4  常量

**作用**：用于记录程序中不可更改的数据

C++定义常量两种方式

1. **\#define** 宏常量： `#define 常量名 常量值`
   * ==通常在文件上方定义==，表示一个常量


2. **const**修饰的变量 `const 数据类型 常量名 = 常量值`
   * ==通常在变量定义前加关键字const==，修饰该变量为常量，不可修改

**示例：**

```c++
#include <iostream>
 using namespace std;
//1、宏变量
#define day 7
 int main(){
 	cout << "一周里总共有" << day << "天"<<endl;
	//2. const 修饰常变量
	const int month = 12; 
	cout << "一年里总共有" << month << "个月"<<endl; 
//	month = 24; 常量不可修改 
 	system("pause");
 	return 0;
 }
```

### 1.5 关键字

**作用：**关键字是C++中预先保留的单词（标识符）

* **在定义变量或者常量时候，不要用关键字**

C++关键字如下：

| asm        | do           | if               | return      | typedef  |
| ---------- | ------------ | ---------------- | ----------- | -------- |
| auto       | double       | inline           | short       | typeid   |
| bool       | dynamic_cast | int              | signed      | typename |
| break      | else         | long             | sizeof      | union    |
| case       | enum         | mutable          | static      | unsigned |
| catch      | explicit     | namespace        | static_cast | using    |
| char       | export       | new              | struct      | virtual  |
| class      | extern       | operator         | switch      | void     |
| const      | false        | private          | template    | volatile |
| const_cast | float        | protected        | this        | wchar_t  |
| continue   | for          | public           | throw       | while    |
| default    | friend       | register         | true        |          |
| delete     | goto         | reinterpret_cast | try         |          |

`提示：在给变量或者常量起名称时候，不要用C++得关键字，否则会产生歧义。`

### 1.6 标识符命名规则

**作用**：C++规定给标识符（变量、常量）命名时，有一套自己的规则

* 标识符不能是关键字
* 标识符只能由字母、数字、下划线组成
* 第一个字符必须为字母或下划线
* 标识符中字母区分大小写

> 建议：给标识符命名时，争取做到见名知意的效果，方便自己和他人的阅读

## 2 数据类型

C++规定在创建一个变量或者常量时，必须要指定出相应的数据类型，否则无法给变量分配内存

### 2.1 整型

**作用**：整型变量表示的是==整数类型==的数据

C++中能够表示整型的类型有以下几种方式，**区别在于所占内存空间不同**：

| **数据类型**        | **占用空间**                                    | 取值范围         |
| ------------------- | ----------------------------------------------- | ---------------- |
| short(短整型)       | 2字节                                           | (-2^15 ~ 2^15-1) |
| int(整型)           | 4字节                                           | (-2^31 ~ 2^31-1) |
| long(长整形)        | Windows为4字节，Linux为4字节(32位)，8字节(64位) | (-2^31 ~ 2^31-1) |
| long long(长长整形) | 8字节                                           | (-2^63 ~ 2^63-1) |

### 2.2 sizeof关键字

**作用：**利用sizeof关键字可以==统计数据类型所占内存大小==

**语法：** `sizeof( 数据类型 / 变量)`

**示例：**

```c++
#include <iostream>
using namespace std;
int main(){
	cout << "int类型所占内存空间为：" << sizeof(int) <<endl; 
	cout << "short类型所占内存空间为：" << sizeof(short) <<endl; 
	cout << "long类型所占内存空间为：" << sizeof(long) <<endl; 
	cout << "long long类型所占内存空间为：" << sizeof(long long) <<endl; 
	system("pause");
	return 0;
}
```

> **整型结论**：==short < int <= long <= long long==

### 2.3 实型（浮点型）

**作用**：用于==表示小数==

浮点型变量分为两种：

1. 单精度float 
2. 双精度double

两者的**区别**在于表示的有效数字范围不同。

| **数据类型** | **占用空间** | **有效数字范围** |
| ------------ | ------------ | ---------------- |
| float        | 4字节        | 7位有效数字      |
| double       | 8字节        | 15～16位有效数字 |

**示例：**

```c++
#include <iostream>
using namespace std;
int main(){
	float a = 3.14f;
	double b = 3.14;
	cout << a << endl;
	cout << b << endl;
	cout << "float的内存空间大小："<< sizeof(float) <<endl; 
	cout << "double的内存空间大小：" << sizeof(double) << endl; 
	
	//科学计数法
	 float c = 3e2; //3*10^2
	 float d = 3e-2; //3*0.1^2
	 cout << c << endl;
	 cout << d << endl;
    system("pause"); 
	return 0;
}
```

### 2.4 字符型

**作用：**字符型变量用于显示单个字符

**语法：**`char ch = 'a';`



> 注意1：在显示字符型变量时，用单引号将字符括起来，不要用双引号

> 注意2：单引号内只能有一个字符，不可以是字符串



- C和C++中字符型变量只占用==1个字节==。
- 字符型变量并不是把字符本身放到内存中存储，而是将对应的ASCII编码放入到存储单元

示例：

```c++
#include <iostream>
using namespace std;
int main(){
	char ch = 'a';
	cout << ch << endl;
	cout << "char的内存空间大小："<<sizeof(char)<<endl;
	cout << (int)ch << endl; //查看字符a对应的ASCII码
	ch = 98; //可以直接用ASCII给字符型变量赋值
	cout << ch << endl;
	system("pause"); 
	return 0;
}
```

ASCII码表格：

| **ASCII**值 | **控制字符** | **ASCII**值 | **字符** | **ASCII**值 | **字符** | **ASCII**值 | **字符** |
| ----------- | ------------ | ----------- | -------- | ----------- | -------- | ----------- | -------- |
| 0           | NUT          | 32          | (space)  | 64          | @        | 96          | 、       |
| 1           | SOH          | 33          | !        | 65          | A        | 97          | a        |
| 2           | STX          | 34          | "        | 66          | B        | 98          | b        |
| 3           | ETX          | 35          | #        | 67          | C        | 99          | c        |
| 4           | EOT          | 36          | $        | 68          | D        | 100         | d        |
| 5           | ENQ          | 37          | %        | 69          | E        | 101         | e        |
| 6           | ACK          | 38          | &        | 70          | F        | 102         | f        |
| 7           | BEL          | 39          | ,        | 71          | G        | 103         | g        |
| 8           | BS           | 40          | (        | 72          | H        | 104         | h        |
| 9           | HT           | 41          | )        | 73          | I        | 105         | i        |
| 10          | LF           | 42          | *        | 74          | J        | 106         | j        |
| 11          | VT           | 43          | +        | 75          | K        | 107         | k        |
| 12          | FF           | 44          | ,        | 76          | L        | 108         | l        |
| 13          | CR           | 45          | -        | 77          | M        | 109         | m        |
| 14          | SO           | 46          | .        | 78          | N        | 110         | n        |
| 15          | SI           | 47          | /        | 79          | O        | 111         | o        |
| 16          | DLE          | 48          | 0        | 80          | P        | 112         | p        |
| 17          | DCI          | 49          | 1        | 81          | Q        | 113         | q        |
| 18          | DC2          | 50          | 2        | 82          | R        | 114         | r        |
| 19          | DC3          | 51          | 3        | 83          | S        | 115         | s        |
| 20          | DC4          | 52          | 4        | 84          | T        | 116         | t        |
| 21          | NAK          | 53          | 5        | 85          | U        | 117         | u        |
| 22          | SYN          | 54          | 6        | 86          | V        | 118         | v        |
| 23          | TB           | 55          | 7        | 87          | W        | 119         | w        |
| 24          | CAN          | 56          | 8        | 88          | X        | 120         | x        |
| 25          | EM           | 57          | 9        | 89          | Y        | 121         | y        |
| 26          | SUB          | 58          | :        | 90          | Z        | 122         | z        |
| 27          | ESC          | 59          | ;        | 91          | [        | 123         | {        |
| 28          | FS           | 60          | <        | 92          | /        | 124         | \|       |
| 29          | GS           | 61          | =        | 93          | ]        | 125         | }        |
| 30          | RS           | 62          | >        | 94          | ^        | 126         | `        |
| 31          | US           | 63          | ?        | 95          | _        | 127         | DEL      |

ASCII 码大致由以下**两部分组**成：

* ASCII 非打印控制字符： ASCII 表上的数字 **0-31** 分配给了控制字符，用于控制像打印机等一些外围设备。
* ASCII 打印字符：数字 **32-126** 分配给了能在键盘上找到的字符，当查看或打印文档时就会出现。

### 2.5 转义字符

**作用：**用于表示一些==不能显示出来的ASCII字符==

现阶段我们常用的转义字符有：` \n  \\  \t`

| **转义字符** | **含义**                                | **ASCII**码值（十进制） |
| ------------ | --------------------------------------- | ----------------------- |
| \a           | 警报                                    | 007                     |
| \b           | 退格(BS) ，将当前位置移到前一列         | 008                     |
| \f           | 换页(FF)，将当前位置移到下页开头        | 012                     |
| **\n**       | **换行(LF) ，将当前位置移到下一行开头** | **010**                 |
| \r           | 回车(CR) ，将当前位置移到本行开头       | 013                     |
| **\t**       | **水平制表(HT)  （跳到下一个TAB位置）** | **009**                 |
| \v           | 垂直制表(VT)                            | 011                     |
| **\\\\**     | **代表一个反斜线字符"\"**               | **092**                 |
| \'           | 代表一个单引号（撇号）字符              | 039                     |
| \"           | 代表一个双引号字符                      | 034                     |
| \?           | 代表一个问号                            | 063                     |
| \0           | 数字0                                   | 000                     |
| \ddd         | 8进制转义字符，d范围0~7                 | 3位8进制                |
| \xhh         | 16进制转义字符，h范围0~9，a~f，A~F      | 3位16进制               |

示例：

```c++
#include <iostream>
using namespace std;
int main(){
	/*
		\n 换行 
		\\  代表一个反斜线字符\
		\t 水平制表 
	*/
	cout << "hello\nworl\\d" <<endl;
	cout << "hello\tworld" <<endl;
	system("pause"); 
	return 0;
}
```

### 2.6 字符串型

**作用**：用于表示一串字符

**两种风格**

1. **C风格字符串**： `char 变量名[] = "字符串值"`

> 注意：C风格的字符串要用双引号括起来

1. **C++风格字符串**：  `string  变量名 = "字符串值"`

示例：

```C++
#include <iostream>
#include <string> 
using namespace std;
int main(){
	//C++风格写法 
	string str = "helloworld";
	//C风格写法 
	char str2[] = "helloworld";
	cout << str << endl;
	cout << str2 << endl;
	system("pause"); 
	return 0;
}
```

> 注意：C++风格字符串，需要加入头文件==#include\<string>==

### 2.7 布尔类型 bool

**作用：**布尔数据类型代表真或假的值 

bool类型只有两个值：

* true  --- 真（本质是1）
* false --- 假（本质是0）

**bool类型占==1个字节==大小**

示例：

```c++
#include<iostream>
using namespace std;
int main(){
	
	bool flag = true;
	cout << flag << endl;
	flag = false;
	cout << flag << endl;
	cout << "布尔的内存空间大小："<< sizeof(bool)<<endl; 
	system("pause");
	return 0;
}
```

### 2.8 数据的输入

**作用：用于从键盘获取数据**

**关键字：**cin

**语法：** `cin >> 变量 `

示例：

```c++
#include<iostream>
using namespace std;
int main(){
	string name;
	int age;
	string sex;
	cout << "请输入你的姓名:"<< endl; 
	cin >> name;
	cout << "请输入你的年龄:"<< endl;
	cin >> age;
	cout << "请输入你的性别:"<< endl;
	cin >> sex;
	cout<< "======================="<<endl;
	cout<< name<<endl;
	cout<< age<<endl;
	cout<< sex<<endl;
	system("pause");
	return 0;
}
```

## 3 运算符

**作用：**用于执行代码的运算

本章我们主要讲解以下几类运算符：

| **运算符类型** | **作用**                               |
| -------------- | -------------------------------------- |
| 算术运算符     | 用于处理四则运算                       |
| 赋值运算符     | 用于将表达式的值赋给变量               |
| 比较运算符     | 用于表达式的比较，并返回一个真值或假值 |
| 逻辑运算符     | 用于根据表达式的值返回真值或假值       |

### 3.1 算术运算符

**作用**：用于处理四则运算 

算术运算符包括以下符号：

| **运算符** | **术语**   | **示例**    | **结果**  |
| ---------- | ---------- | ----------- | --------- |
| +          | 正号       | +3          | 3         |
| -          | 负号       | -3          | -3        |
| +          | 加         | 10 + 5      | 15        |
| -          | 减         | 10 - 5      | 5         |
| *          | 乘         | 10 * 5      | 50        |
| /          | 除         | 10 / 5      | 2         |
| %          | 取模(取余) | 10 % 3      | 1         |
| ++         | 前置递增   | a=2; b=++a; | a=3; b=3; |
| ++         | 后置递增   | a=2; b=a++; | a=3; b=2; |
| --         | 前置递减   | a=2; b=--a; | a=1; b=1; |
| --         | 后置递减   | a=2; b=a--; | a=1; b=2; |

**示例1：**

```c++
#include<iostream>
using namespace std;
int main(){
	int a = 10;
	int b = 4;
	cout << "a+b=" << a+b << endl;
	cout << "a-b=" << a-b << endl;
	cout << "a*b=" << a*b << endl;
	cout << "a/b=" << a/b << endl; //两个整数相除结果依然是整数
	cout << "a%b=" << a%b << endl;

//	b=0;
//	cout << "a%b=" << a%b << endl; //取模运算时，除数也不能为0
	
	float c = 3.14f;
	float d = 4.25;
//	cout << "c%d=" << c%d << endl;	//两个小数不可以取模

	system("pause"); 
	return 0;
}
```

> 总结：在除法运算中，除数不能为0
>
> 只有整型变量可以进行取模运算

**示例2：**

```c++
#include<iostream>
using namespace std;
int main(){
	//后置递增 
	int a = 10;
	int b = a++;
	cout<<a<<endl;	//11
	cout<<b<<endl;	//10
	
	//前置递增
	int a1 = 5;
	int b1 = ++a1;
	cout<<a1<<endl;	//6
	cout<<b1<<endl;	//6
	
	//区别
	//后置递增先计算表达式，后对变量进行++
	int a2 = 10;
	int b2 = a2++ * 10;
	cout << b2 << endl;	//100
	cout << a2 << endl;	//11
	
	//前置递增先对变量进行++，再计算表达式
	int a3 = 10;
	int b3 = ++a3 * 10;
	cout << b3 << endl;	//110
	cout << a3 << endl;	//11
	system("pause"); 
	return 0;
}
```

> 总结：前置递增先对变量进行++，再计算表达式，后置递增相反

### 3.2 赋值运算符

**作用：**用于将表达式的值赋给变量

赋值运算符包括以下几个符号：

| **运算符** | **术语** | **示例**   | **结果**  |
| ---------- | -------- | ---------- | --------- |
| =          | 赋值     | a=2; b=3;  | a=2; b=3; |
| +=         | 加等于   | a=0; a+=2; | a=2;      |
| -=         | 减等于   | a=5; a-=3; | a=2;      |
| *=         | 乘等于   | a=2; a*=2; | a=4;      |
| /=         | 除等于   | a=4; a/=2; | a=2;      |
| %=         | 模等于   | a=3; a%2;  | a=1;      |

**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	int a = 10;
	a += 2;	// a = a + 10 
	cout << "a=" << a<< endl; //12
	
	a=10;
	a -=5;	// a = a - 5
	cout << "a=" << a<<endl;	//5
	
	a=10;
	a /=3;	// a = a / 3
	cout << "a=" << a<<endl;	//3
	
	a=10;
	a %=4;	// a = a % 4
	cout << "a=" << a<<endl;	//2
    system("pause");
	return 0;
}
```

### 3.3 比较运算符

**作用：**用于表达式的比较，并返回一个真值或假值

比较运算符有以下符号：

| **运算符** | **术语** | **示例** | **结果** |
| ---------- | -------- | -------- | -------- |
| ==         | 相等于   | 4 == 3   | 0        |
| !=         | 不等于   | 4 != 3   | 1        |
| <          | 小于     | 4 < 3    | 0        |
| \>         | 大于     | 4 > 3    | 1        |
| <=         | 小于等于 | 4 <= 3   | 0        |
| \>=        | 大于等于 | 4 >= 1   | 1        |

示例：

```c++
#include<iostream>
using namespace std;
int main(){
	int a = 10;
	int b = 20;
	cout << (a == b) << endl;	//0
	cout << (a > b) << endl; 	//0
	cout << (a < b) << endl; 	//1
	cout << (a >= b) << endl; 	//0
	cout << (a <= b) << endl; 	//1
	cout << (a != b) << endl; 	//1
	system("pause");
	return 0;
}
```

> 注意：C和C++ 语言的比较运算中， ==“真”用数字“1”来表示， “假”用数字“0”来表示。== 

### 3.4 逻辑运算符

**作用：**用于根据表达式的值返回真值或假值

逻辑运算符有以下符号：

| **运算符** | **术语** | **示例** | **结果**                                                 |
| ---------- | -------- | -------- | -------------------------------------------------------- |
| !          | 非       | !a       | 如果a为假，则!a为真；  如果a为真，则!a为假。             |
| &&         | 与       | a && b   | 如果a和b都为真，则结果为真，否则为假。                   |
| \|\|       | 或       | a \|\| b | 如果a和b有一个为真，则结果为真，二者都为假时，结果为假。 |

**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	//逻辑运算符  --- 非
	int a = 10;
	cout << !a << endl;		//0
	cout << !!a << endl;	//1
	
	//逻辑运算符  --- 与
	int b = 10;
	cout << (a&&b) << endl;	//1
	a=10;
	b=0;
	cout << (a&&b) << endl;	//0
	a=0;
	b=0;
	cout << (a&&b) << endl;	//0
	
	//逻辑运算符  --- 或
	a = 10;
	b = 10;
	cout << (a||b) << endl; //1
	a = 0;
	cout << (a||b) << endl; //1
	a = 0;
	b = 0;
	cout << (a||b) << endl; //0
	system("pause");
	return 0;
}
```

> 总结：
>
> 逻辑非——真变假，假变真
>
> 逻辑与—— ==同真为真，其余为假==
>
> 逻辑或——同假为假，其余为真

## 4 程序流程结构

C/C++支持最基本的三种程序运行结构：==顺序结构、选择结构、循环结构==

* 顺序结构：程序按顺序执行，不发生跳转
* 选择结构：依据条件是否满足，有选择的执行相应功能
* 循环结构：依据条件是否满足，循环多次执行某段代码



### 4.1 选择结构

#### 4.1.1 if语句

**作用：**执行满足条件的语句

if语句的三种形式

* 单行格式if语句

* 多行格式if语句

* 多条件的if语句

  

1. 单行格式if语句：`if(条件){ 条件满足执行的语句 }`

   ![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151459428.webp)

   示例：

```c++
#include<iostream>
using namespace std;
int main(){
	/*
	选择结构-单行if语句
	输入一个分数，如果分数大于600分，视为考上一本大学，并在屏幕上打印
	*/
	int score = 0;
	cout << "请输入你的分数:" << endl;
	cin >> score;
	//if语句
	//注意事项，在if判断语句后面，不要加分号
	if(score>600){
		cout << "我考上了一本大学！！！" << endl; 
	}
	system("pause");
	return 0;
}
```

> 注意：if条件表达式后不要加分号

2. 多行格式if语句：`if(条件){ 条件满足执行的语句 }else{ 条件不满足执行的语句 };`

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151505187.webp)



示例：

```c++
#include<iostream>
using namespace std;
int main(){
	int score = 0;
	cout << "请输入你的分数:" << endl;
	cin >> score;
	if(score>600){
		cout << "我考上了一本大学！！！" << endl; 
	}else{
		cout << "我未考上一本大学" << endl;
	}
	system("pause");
	return 0;
}
```

3. 多条件的if语句：`if(条件1){ 条件1满足执行的语句 }else if(条件2){条件2满足执行的语句}... else{ 都不满足执行的语句}`

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151511146.webp)







示例：

```c++
#include<iostream>
using namespace std;
int main(){
	int score = 0;
	cout << "请输入你的分数:" << endl;
	cin >> score;
	if(score>600){
		cout << "我考上了一本大学！！！" << endl; 
	}else if(score>500){
		cout << "我考上了二本大学" << endl;
	}else if(score > 400){
		cout << "我考上了三本大学" << endl;
	}else{
		cout << "我未考上本科" << endl;
	}
	system("pause");
	return 0;
}
```

**嵌套if语句**：在if语句中，可以嵌套使用if语句，达到更精确的条件判断

案例需求：

* 提示用户输入一个高考考试分数，根据分数做如下判断
* 分数如果大于600分视为考上一本，大于500分考上二本，大于400考上三本，其余视为未考上本科；
* 在一本分数中，如果大于700分，考入北大，大于650分，考入清华，大于600考入人大。

**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	int score = 0;
	cout << "请用户输入一个高考考试分数:" <<endl;
	cin >> score;
	if(score>600){
		cout << "我考上了一本大学" << endl;
		if(score>700){
			cout << "考上北大！" << endl; 
		}else if(score>650){
			cout << "考上清华！" << endl; 
		}else{
			cout << "考上人大" << endl; 
		}
	}else if(score>500){
		cout << "考上二本！" << endl; 
	}else if(score>400){
		cout << "考上三本！" << endl; 
	}else{
		cout << "未考上本科！" << endl; 
	} 
	system("pause");
	return 0;
}
```

**练习案例：** 三只小猪称体重

有三只小猪ABC，请分别输入三只小猪的体重，并且判断哪只小猪最重？

```c++
#include<iostream>
using namespace std;
int main() {
	//三只小猪称体重，判断哪只最重
	int pig1 = 0;
	int pig2 = 0;
	int pig3 = 0;

	cout << "输入第一只小猪的体重:" << endl;
	cin >> pig1;
	cout << "输入第二只小猪的体重:" << endl;
	cin >> pig2;
	cout << "输入第三只小猪的体重:" << endl;
	cin >> pig3;
	if (pig1 > pig2) //A>B
	{
		if (pig1 > pig3) { //A>C
			cout << "第一只小猪体重最重" << endl;
		}else { //A<C
			cout << "第三只小猪体重最重" << endl;
		}
	}
	else { //A<B
		if (pig2 < pig3) //B<C
		{
			cout << "第三只小猪体重最重" << endl;
		}
		else //B>C
		{
			cout << "第二只小猪体重最重" << endl;
		}
	}
	system("pause");
    return 0;
}
```

#### 4.1.2 三目运算符

**作用：** 通过三目运算符实现简单的判断

**语法：**`表达式1 ? 表达式2 ：表达式3`

**解释：**

如果表达式1的值为真，执行表达式2，并返回表达式2的结果；

如果表达式1的值为假，执行表达式3，并返回表达式3的结果。

**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	int a = 10;
	int b = 20;
	int c;
	c = a>b?a:b;
	cout << "c="<< c << endl; //20
	//C++中三目运算符返回的是变量,可以继续赋值
	(a>b?a:b) = 100;
	cout << "a = " << a << endl; //10
	cout << "b = " << b << endl; //100
	cout << "c = " << c << endl; //20
	system("pause");
	return 0;
}
```

> 总结：和if语句比较，三目运算符优点是短小整洁，缺点是如果用嵌套，结构不清晰

#### 4.1.3 switch语句

**作用：**执行多条件分支语句

**语法：**

```c++
switch(表达式)
{
	case 结果1：执行语句;break;
	case 结果2：执行语句;break;
	...
	default:执行语句;break;
}
```

**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	//请给电影评分 
	//10 ~ 9   经典   
	// 8 ~ 7   非常好
	// 6 ~ 5   一般
	// 5分以下 烂片
	int score = 0;
	cout << "请给电影打分" << endl;
	cin >> score;
	switch (score) {
		case 10:
		case 9:
			cout << "经典" << endl;
			break;
		case 8:
		case 7:
			cout << "非常好" << endl;
			break;
		case 6:
		case 5:
			cout << "一般" << endl;
			break;
		default:
			cout << "烂片" << endl;
			break;
		}
	system("pause");
	return 0;
}
```

> 注意1：switch语句中表达式类型只能是整型或者字符型
>
> 注意2：case里如果没有break，那么程序会一直向下执行
>
> 总结：与if语句比，对于多条件判断时，switch的结构清晰，执行效率高，缺点是switch不可以判断区间

### 4.2 循环结构

#### 4.2.1 while循环语句

**作用：**满足循环条件，执行循环语句

**语法：**` while(循环条件){ 循环语句 }`

**解释：**==只要循环条件的结果为真，就执行循环语句==

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151520914.webp)







**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	int i=1;
	int res=0;
	while(i<=5){	
		res += i++;
	}
	cout << "前5项和为:"<<res<<endl;
    system("pause");
	return 0;
}
```

> 注意：在执行循环语句时候，程序必须提供跳出循环的出口，否则出现死循环

**while循环练习案例：**==猜数字==

**案例描述：**系统随机生成一个1到100之间的数字，玩家进行猜测，如果猜错，提示玩家数字过大或过小，如果猜对恭喜玩家胜利，并且退出游戏。

```c++
#include<iostream>
using namespace std;
int main()
{
	srand(time(0));
	int num = rand() % 100 + 1;
	//cout << num << endl;
	while (1)
	{
		cout << "请输入要猜的数字:" << endl;
		int input = 0;
		cin >> input;
		if (num > input)
		{
			cout << "猜小了" << endl;
		}
		else if (num < input)
		{
			cout << "猜大了" << endl;
		}
		else
		{
			cout << "猜对了" << endl;
			break;
		}
	}
	system("pause");
	return 0;
}
```

#### 4.2.2 do...while循环语句

**作用：** 满足循环条件，执行循环语句

**语法：** `do{ 循环语句 } while(循环条件);`

**注意：**与while的区别在于==do...while会先执行一次循环语句==，再判断循环条件

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151528955.webp)



**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	int i = 1;
	int res = 0;
	do{
		res+=i++;
	}while(i<=5); 
	cout << "前5项和为:"<<res<<endl;
    system("pause");
	return 0;
}
```

> 总结：与while循环区别在于，do...while先执行一次循环语句，再判断循环条件

**练习案例：水仙花数**

**案例描述：**水仙花数是指一个 3 位数，它的每个位上的数字的 3次幂之和等于它本身

例如：1^3 + 5^3+ 3^3 = 153

请利用do...while语句，求出所有3位数中的水仙花数

```c++
#include<iostream>
using namespace std;
int main()
{
	int num = 100;
	do
	{
		int a = num % 10; //个位
		int b = num / 10 % 10;	//十位
		int c = num / 100;	//百位
		if (num == (a *a * a + b *b * b + c *c * c))
		{
			cout << num << endl;
		}
		num++;
	} while (num < 1000);
	system("pause");
	return 0;
}
```

#### 4.2.3 for循环语句

**作用：** 满足循环条件，执行循环语句

**语法：**` for(起始表达式;条件表达式;末尾循环体) { 循环语句; }`

**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	int res = 0;
	for(int i=1;i<=5;i++){
		res += i;
	} 
	cout << "前5项和为:"<<res<<endl;
    system("pause");
	return 0;
}
```

**详解：**

![1541673704101](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151536108.webp)



> 注意：for循环中的表达式，要用分号进行分隔

> 总结：while , do...while, for都是开发中常用的循环语句，for循环结构比较清晰，比较常用

**练习案例：敲桌子**

案例描述：从1开始数到数字100， 如果数字个位含有7，或者数字十位含有7，或者该数字是7的倍数，我们打印敲桌子，其余数字直接打印输出。

```c++
#include<iostream>
using namespace std;
int main()
{
	for (int i = 1; i <= 100; i++)
	{
		if (i % 10 == 7 || i / 10 == 7 || i % 7 == 0)
		{
			cout << "敲桌子 " << i << endl;
		}
		else { cout << i << endl; }
	}
    system("pause");
	return 0;
}
```

#### 4.2.4 嵌套循环

**作用：** 在循环体中再嵌套一层循环，解决一些实际问题

例如我们想在屏幕中打印如下图片，就需要利用嵌套循环

![1541676003486](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151542466.webp)

**示例：**

```c++
#include<iostream>
using namespace std;
int main(){
	for(int i=0;i<10;i++){
		for(int j=0;j<10;j++){
			cout << "*" << " ";
		}
		cout << endl;
	} 
	system("pause"); 
	return 0;
}
```

**练习案例：**乘法口诀表

案例描述：利用嵌套循环，实现九九乘法表

```c++
#include<iostream>
using namespace std;
int main() {
	for (int i = 1; i < 10; i++)
	{
		for (int j = 1; j <= i; j++) {
			cout << i << "*" << j << "=" << (i * j) << "\t";
		}
		cout << endl;
	}
    system("pause");
	return 0;
}
```

### 4.3 跳转语句

#### 4.3.1 break语句

**作用:** 用于跳出==选择结构==或者==循环结构==

break使用的时机：

* 出现在switch条件语句中，作用是终止case并跳出switch
* 出现在循环语句中，作用是跳出当前的循环语句
* 出现在嵌套循环中，跳出最近的内层循环语句

**示例：**

```c++
#include<iostream>
using namespace std;
int main()
{
	//在嵌套循环语句中使用break，退出内层循环
	for (int i = 0; i < 10; i++)
	{
		for (int j = 0; j < 10; j++)
		{
			if (j == 5) { break; }
			cout << "*" << " ";
		}
		cout << endl;
	}
    system("pause");
	return 0;
}
```

> 总结：
>
> 1、在switch 语句中使用break
>
> 2、在循环语句中用break
>
> 3、在嵌套循环语句中使用break，退出内层循环

#### 4.3.2 continue语句

**作用：**在==循环语句==中，跳过本次循环中余下尚未执行的语句，继续执行下一次循环

**示例：**

```c++
#include<iostream>
using namespace std;
int main()
{
	for (int i = 0; i < 10; i++)
	{
		for (int j = 0; j < 10; j++)
		{
			if (j == 5) { continue; }
			cout << "*" << " ";
		}
		cout << endl;
	}
    system("pause");
	return 0;
}
```

> 注意：continue并没有使整个循环终止，而break会跳出循环

#### 4.3.3 goto语句

**作用：**可以无条件跳转语句

**语法：** `goto 标记;`

**解释：**如果标记的名称存在，执行到goto语句时，会跳转到标记的位置

**示例：**

```c++
#include<iostream>
using namespace std;
int main()
{
	cout << "1" << endl;
	goto FLAG;
	cout << "2" << endl;
	cout << "3" << endl;
	cout << "4" << endl;

	FLAG:
	cout << "5" << endl;
	system("pause");
	return 0;
}
```

> 注意：在程序中不建议使用goto语句，以免造成程序流程混乱

## 5 数组

### 5.1 概述

所谓数组，就是一个集合，里面存放了相同类型的数据元素

**特点1：**数组中的每个==数据元素都是相同的数据类型==

**特点2：**数组是由==连续的内存==位置组成的

### 5.2 一维数组

#### 5.2.1 一维数组定义方式

一维数组定义的三种方式：

1. ` 数据类型  数组名[ 数组长度 ]; `
2. `数据类型  数组名[ 数组长度 ] = { 值1，值2 ...};`
3. `数据类型  数组名[ ] = { 值1，值2 ...};`

示例

```c++
#include <iostream>
using namespace std;
int main()
{
	//定义方式1
	//数据类型 数组名[元素个数];
	int num[3];
	//利用下标赋值
	num[0] = 100;
	num[1] = 200;
	num[2] = 300;
	//利用下标输出
	cout << num[0] << endl;
	cout << num[1] << endl;
	cout << num[2] << endl;
	cout << "======================"<< endl;
	//第二种定义方式
	//数据类型 数组名[元素个数] =  {值1，值2 ，值3 ...};
	//如果{}内不足10个数据，剩余数据用0补全
	int num2[10] = {10,20,50,45,60,42};
	//循环输出
	for (int i = 0; i < 10 ; i++)
	{
		cout << num2[i] << "\t";
	}
	cout << endl;
	cout << "======================" << endl;
	//定义方式3
	//数据类型 数组名[] =  {值1，值2 ，值3 ...};
	int num3[] = {10,20,45,63,12,23,49};
	for (int i = 0; i < 7; i++)
	{
		cout << num3[i] << "\t";
	}
    system("pause");
	return 0;
}
```

> 总结1：数组名的命名规范与变量名命名规范一致，不要和变量重名
>
> 总结2：数组中下标是从0开始索引

#### 5.2.2 一维数组数组名

一维数组名称的**用途**：

1. 可以统计整个数组在内存中的长度
2. 可以获取数组在内存中的首地址

**示例：**

```c++
#include <iostream>
using namespace std;
int main()
{
	//数组名用途
	//1、可以获取整个数组占用内存空间大小
	int num[10] = { 10,45,52,16,23,75 };
	cout << "整个数组所占内存空间为：" << sizeof(num) << endl;	//40
	cout << "每个元素所占内存空间为：" << sizeof(num[0]) << endl;		//4
	cout << "数组的元素个数为：" << sizeof(num) / sizeof(num[0]) << endl;

	//2、可以通过数组名获取到数组首地址
	cout << "数组首地址为：" << int(num) << endl;
	cout << "数组中第一个元素地址为： " << (int)&num[0] << endl;
	cout << "数组中第二个元素地址为： " << (int)&num[1] << endl;

	//num = 100;	错误，数组名是常量，因此不可以赋值
	system("pause");
	return 0;
}
```

> 注意：数组名是常量，不可以赋值
>
> 总结1：直接打印数组名，可以查看数组所占内存的首地址
>
> 总结2：对数组名进行sizeof，可以获取整个数组占内存空间的大小

**练习案例1**：五只小猪称体重

**案例描述：**

在一个数组中记录了五只小猪的体重，如：int arr[5] = {300,350,200,400,250};

找出并打印最重的小猪体重。

```c++
#include <iostream>
using namespace std;
int main()
{
	int arr[5] = { 300,350,200,400,250 };
	int max = 0;
	for (int i = 0; i < sizeof(arr) / sizeof(arr[0]); i++)
	{
		if (arr[i] > max)
		{
			max = arr[i];
		}
	}
	cout << "max=" << max << endl;
    system("pause");
	return 0;
}
```

**练习案例2：**数组元素逆置

**案例描述：**请声明一个5个元素的数组，并且将元素逆置.

(如原数组元素为：1,3,2,5,4;逆置后输出结果为:4,5,2,3,1);

```c++
#include <iostream>
using namespace std;
int main()
{
	int arr[5] = {1,3,2,5,4};

	cout << "原数组元素为:" << " ";
	for (int i = 0; i < 5; i++)
	{
		cout << arr[i] << " ";
	}
	cout << endl;
	//
	int start = 0;
	int end = sizeof(arr) / sizeof(arr[0]) - 1;	// 20/4-1=4
	int temp = arr[start];

	while (start<end)
	{
		//元素互换
		temp = arr[start];
		arr[start] = arr[end];
		arr[end] = temp;
		//下标更新
		start++;
		end--;
	}
	//打印逆置后数组元素
	cout << "逆置后数组元素为:" << " ";
	for (int i = 0; i < 5; i++)
	{
		cout << arr[i] << " ";
	}
    system("pause");
	return 0;
}
```

#### 5.2.3 冒泡排序

**作用：** 最常用的排序算法，对数组内元素进行排序

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素做同样的工作，执行完毕后，找到第一个最大值。
3. 重复以上的步骤，每次比较次数-1，直到不需要比较

![1541905327273](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151601739.webp)

![image-20240617232311696](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151555834.webp)

**示例：** 将数组 { 4,2,8,0,5,7,1,3,9 } 进行升序排序

```c++
#include <iostream>
using namespace std;
int main()
{
	int arr[] = {4, 2, 8, 0, 5, 7, 1, 3, 9};
	cout << "排序前:" << " ";
	for (int i = 0; i < 9; i++)
	{
		cout << arr[i] << " ";
	}
	cout << endl;
	for (int i = 0; i < 9 - 1; i++)
	{
		for (int j = 0; j < 9 - i - 1; j++)
		{
			if (arr[j] > arr[j + 1])
			{
				int temp = arr[j];
				arr[j] = arr[j + 1];
				arr[j + 1] = temp;
			}
		}
	}
	cout << "排序后:" << " ";
	for (int i = 0; i < 9; i++)
	{
		cout << arr[i] << " ";
	}
	cout << endl;
	system("pause");
	return 0;
}
```

### 5.3 二维数组

二维数组就是在一维数组上，多加一个维度。

#### 5.3.1 二维数组定义方式

二维数组定义的四种方式：

1. ` 数据类型  数组名[ 行数 ][ 列数 ]; `
2. `数据类型  数组名[ 行数 ][ 列数 ] = { {数据1，数据2 } ，{数据3，数据4 } };`
3. `数据类型  数组名[ 行数 ][ 列数 ] = { 数据1，数据2，数据3，数据4};`
4. ` 数据类型  数组名[  ][ 列数 ] = { 数据1，数据2，数据3，数据4};`

> 建议：以上4种定义方式，利用==第二种更加直观，提高代码的可读性==

示例：

```c++
#include<iostream>
using namespace std;
int main()
{
	//方式1  
	//数组类型 数组名 [行数][列数]
	int arr[2][3];
	arr[0][0] = 1;
	arr[0][1] = 2;
	arr[0][2] = 3;
	arr[1][0] = 4;
	arr[1][1] = 5;
	arr[1][2] = 6;
	//遍历
	for (int i = 0; i < 2; i++)
	{
		for (int j = 0; j < 3; j++)
		{
			cout << arr[i][j] << " ";
		}
		cout << endl;
	}

	//方式2 
	//数据类型 数组名[行数][列数] = { {数据1，数据2 } ，{数据3，数据4 } };
	int arr2[2][3] =
	{
		{1,3,5},
		{2,5,7}
	};

	//方式3
	//数据类型 数组名[行数][列数] = { 数据1，数据2 ,数据3，数据4  };
	int arr3[2][3] = { 1,2,3,4,5,6 };

	//方式4 
	//数据类型 数组名[][列数] = { 数据1，数据2 ,数据3，数据4  };
	int arr4[][3] = { 1,2,3,4,5,6 };

	system("pause");
	return 0;
}
```

> 总结：在定义二维数组时，如果初始化了数据，可以省略行数

#### 5.3.2 二维数组数组名

* 查看二维数组所占内存空间
* 获取二维数组首地址

**示例：**

```c++
#include<iostream>
using namespace std;
int main()
{
	//二维数组数组名
	int arr[2][3] =
	{
		{1,2,3},
		{4,5,6}
	};
	cout << "二维数组大小：" << sizeof(arr) << endl;
	cout << "二维数组一行大小： " << sizeof(arr[0]) << endl;
	cout << "二维数组元素大小： " << sizeof(arr[0][0]) << endl;

	cout << "二维数组行数： " << sizeof(arr) / sizeof(arr[0]) << endl;
	cout << "二维数组列数： " << sizeof(arr[0]) / sizeof(arr[0][0]) << endl;

	//地址
	cout << "二维数组首地址：" << arr << endl;
	cout << "二维数组第一行地址：" << arr[0] << endl;
	cout << "二维数组第二行地址：" << arr[1] << endl;

	cout << "二维数组第一个元素地址：" << &arr[0][0] << endl;
	cout << "二维数组第二个元素地址：" << &arr[0][1] << endl;
	system("pause");
	return 0;
}
```

#### **5.3.3 二维数组应用案例**

**考试成绩统计：**

案例描述：有三名同学（张三，李四，王五），在一次考试中的成绩分别如下表，**请分别输出三名同学的总成绩**

|      | 语文 | 数学 | 英语 |
| ---- | ---- | ---- | ---- |
| 张三 | 100  | 100  | 100  |
| 李四 | 90   | 50   | 100  |
| 王五 | 60   | 70   | 80   |

**参考答案：**

```c++
#include<iostream>
using namespace std;
int main()
{
	int scores[3][3] =
	{
		{100,100,100},
		{90,50,100},
		{60,70,80}
	};
	string name[3] = { "张三","李四","王五" };
	for (int i = 0; i < 3; i++)
	{
		int sum = 0;
		for (int j = 0; j < 3; j++)
		{
			sum += scores[i][j];
		}
		cout << name[i] << "同学总成绩为：" << sum << endl;
	}
	system("pause");
	return 0;
}
```

## 6 函数

### 6.1 概述

**作用：**将一段经常使用的代码封装起来，减少重复代码

一个较大的程序，一般分为若干个程序块，每个模块实现特定的功能。

### 6.2 函数的定义

函数的定义一般主要有5个步骤：

1、返回值类型 

2、函数名

3、参数表列

4、函数体语句 

5、return 表达式

**语法：** 

```c++
返回值类型 函数名 （参数列表）
{
       函数体语句
       return表达式
}
```

* 返回值类型 ：一个函数可以返回一个值。在函数定义中
* 函数名：给函数起个名称
* 参数列表：使用该函数时，传入的数据
* 函数体语句：花括号内的代码，函数内需要执行的语句
* return表达式： 和返回值类型挂钩，函数执行完后，返回相应的数据

**示例：**定义一个加法函数，实现两个数相加

```c++
//函数定义
int add(int num1, int num2)
{
	return num1 + num2;
}
```

### 6.3 函数的调用

**功能：**使用定义好的函数

**语法：**` 函数名（参数）`

**示例：**

```c++
#include<iostream>
using namespace std;
//函数定义
int add(int num1, int num2)//定义中的num1,num2称为形式参数，简称形参
{
	return num1 + num2;
}
int main()
{
	int a, b;
	cout << "请输入一个数字:" << endl;
	cin >> a;
	cout << "请输入一个数字:" << endl;
	cin >> b;
	//调用add函数  调用时的a，b称为实际参数，简称实参
	cout << "两个数相加之和为:" << add(a, b) << endl;
	system("pause");
	return 0;
}
```

> 总结：函数定义里小括号内称为形参，函数调用时传入的参数称为实参

### 6.4 值传递

* 所谓值传递，就是函数调用时实参将数值传入给形参
* 值传递时，==如果形参发生，并不会影响实参==

**示例：**

```c++
#include<iostream>
using namespace std;
//函数定义
void swap(int num1, int num2)
{
	cout << "交换前：" << endl;
	cout << "num1 = " << num1 << endl;
	cout << "num2 = " << num2 << endl;

	int temp = num1;
	num1 = num2;
	num2 = temp;

	cout << "交换后：" << endl;
	cout << "num1 = " << num1 << endl;
	cout << "num2 = " << num2 << endl;

	//return ; 当函数声明时候，不需要返回值，可以不写return
}
int main()
{
	int a = 19;
	int b = 29;
	swap(a, b);
	cout << "mian中的 a = " << a << endl;
	cout << "mian中的 b = " << b << endl;
	system("pause");
	return 0;
}
```

> 总结： 值传递时，形参是修饰不了实参的

### **6.5 函数的常见样式**

常见的函数样式有4种

1. 无参无返
2. 有参无返
3. 无参有返
4. 有参有返

**示例：**

```c++
#include<iostream>
using namespace std;
//函数常见样式
//1、 无参无返
void test01()
{
	cout << "this is test01" << endl;
}
//2、 有参无返
void test02(int a)
{
	cout << "this is test02" << endl;
	cout << "a=" << a << endl;
}
//3、无参有返
int test03()
{
	cout << "this is test03 " << endl;
	return 10;
}
//4、有参有返
int test04(int a, int b)
{
	cout << "this is test04 " << endl;
	int sum = a + b;
	return sum;
}
int main()
{
	//1、 无参无返
	test01();
	//2、 有参无返
	test02(10);
	int num1 = test03();
	cout << num1 << endl;

	int num2 = test04(20,30);
	cout << num2 << endl;
	system("pause");
	return 0;
}
```

### 6.6 函数的声明

**作用：** 告诉编译器函数名称及如何调用函数。函数的实际主体可以单独定义。

*  函数的**声明可以多次**，但是函数的**定义只能有一次**

**示例：**

```c++
#include<iostream>
using namespace std;
//声明可以多次，定义只能一次
//声明
int max(int a, int b);
int max(int a, int b);
//定义
int max(int a, int b)
{
	return a > b ? a : b;
}
int main()
{
	int a = 100;
	int b = 200;
	cout << max(a, b) << endl;
	system("pause");
	return 0;
}
```

### 6.7 函数的分文件编写

**作用：**让代码结构更加清晰

函数分文件编写一般有4个步骤

1. 创建后缀名为.h的头文件  
2. 创建后缀名为.cpp的源文件
3. 在头文件中写函数的声明
4. 在源文件中写函数的定义

**示例：**

```c++
#pragma once
#include<iostream>
using namespace std;
//实现两个数字交换的函数声明
void swap(int a, int b);
```

```c++
#include "swap.h"
int main()
{
	int a = 100;
	int b = 200;
	swap(a, b);
	system("pause");
	return 0;
}
void swap(int a, int b)
{
	int temp = a;
	a = b;
	b = temp;

	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
}
```

## 7 指针

### 7.1 指针的基本概念

**指针的作用：** 可以通过指针间接访问内存

* 内存编号是从0开始记录的，一般用十六进制数字表示
* 可以利用指针变量保存地址


### 7.2 指针变量的定义和使用

指针变量定义语法： `数据类型 * 变量名；`

**示例：**

```c++
int main()
{
	//1.指针的定义
	int a = 10; //定义整型变量a
	//指针定义语法： 数据类型 * 变量名 ;
	int * p;
	//指针变量赋值
	p = &a;	//指针指向变量a的地址
	cout << &a << endl;	//打印数据a的地址
	cout << p << endl;	//打印指针变量p
	//2.指针的使用
	//通过*操作指针变量指向的内存
	cout << "*p = " << *p << endl;
	system("pause");
	return 0;
}
```

指针变量和普通变量的区别

* 普通变量存放的是数据,指针变量存放的是地址
* 指针变量可以通过" * "操作符，操作指针变量指向的内存空间，这个过程称为解引用

> 总结1： 我们可以通过 & 符号 获取变量的地址
>
> 总结2：利用指针可以记录地址
>
> 总结3：对指针变量解引用，可以操作指针指向的内存

### 7.3 指针所占内存空间

提问：指针也是种数据类型，那么这种数据类型占用多少内存空间？

**示例：**

```c++
int main()
{
	int a = 20;
	int* p;
	p = &a;
	cout << *p << endl;	//* 解引用
	cout << sizeof(p) << endl;
	cout << sizeof(char *) << endl;
	cout << sizeof(float *) << endl;
	cout << sizeof(double *) << endl;
	system("pause");
	return 0;
}
```

> 总结：所有指针类型在32位操作系统下是4个字节，64位操作系统下是8个字节

### 7.4 空指针和野指针

**空指针**：指针变量指向内存中编号为0的空间

**用途：**初始化指针变量

**注意：**空指针指向的内存是不可以访问的

**示例1：空指针**

```c++
int main()
{
	//指针变量p指向内存地址编号为0的空间
	int * p = NULL;
	//访问空指针报错 
	//内存编号0 ~255为系统占用内存，不允许用户访问
	cout << *p << endl;
	system("pause");
	return 0;
}
```

**野指针**：指针变量指向非法的内存空间

**示例2：野指针**

```c++
int main()
{
	//指针变量p指向内存地址编号为0x1100的空间
	int* p = (int *)0x1100;
	//访问野指针报错 
	cout << *p << endl;
	system("pause");
	return 0;
}
```

> 总结：空指针和野指针都不是我们申请的空间，因此不要访问。

### 7.5 const修饰指针

const修饰指针有三种情况

1. const修饰指针   --- 常量指针
2. const修饰常量   --- 指针常量
3. const即修饰指针，又修饰常量

**示例：**

```c++
int main()
{
	int a = 10;
	int b = 10;
	//const修饰的是指针，指针指向可以改，指针指向的值不可以更改
	const int* p1 = &a;
	p1 = &b;
	//*p1 = 100;	报错
	//const修饰的是常量，指针指向不可以改，指针指向的值可以更改
	int* const p2 = &a;
	//p2 = &b;	报错
	*p2 = 200;
	//const既修饰指针又修饰常量,指针指向和指针指向的值都不可以更改
	const int* const p3 = &a;
	//p3 = &b;	报错
	//*p3 = 300;	报错
	system("pause");
	return 0;
}
```

> 技巧：看const右侧紧跟着的是指针还是常量, 是指针就是常量指针，是常量就是指针常量

### 7.6 指针和数组

**作用：**利用指针访问数组中元素

**示例：**

```c++
int main()
{
	int arr[] = { 1,2,3,4,5,6,7,8,9 };
	int * p = arr;
	cout << "第一个元素：" << arr[0] << endl;
	cout << "指针访问第一个元素：" << *p << endl;
	
	for (int i = 0; i < 9; i++)
	{
		//利用指针遍历数组
		cout << *p << endl;
		p++;
	}
	system("pause");
	return 0;
}
```

### 7.7 指针和函数

**作用：**利用指针作函数参数，可以修改实参的值

**示例：**

```c++
int main()
{
	int a = 100;
	int b = 200;
	swap2(a, b);	// 值传递不会改变实参
	swap(&a, &b);	//地址传递会改变实参
	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	system("pause");
	return 0;
}
//地址传递
void swap(int* a, int* b)
{
	int temp = *a;
	*a = *b;
	*b = temp;
}
//值传递
void swap2(int a, int b)
{
	int temp = a;
	a = b;
	b = temp;
}
```



> 总结：如果不想修改实参，就用值传递，如果想修改实参，就用地址传递

### 7.8 指针、数组、函数

**案例描述：**封装一个函数，利用冒泡排序，实现对整型数组的升序排序

例如数组：int arr[10] = { 4,3,6,9,1,2,10,8,7,5 };

**示例：**

```c++
int main()
{
	int arr[10] = { 4,3,6,9,1,2,10,8,7,5 };
	int len = sizeof(arr) / sizeof(int);
	cout << "排序前：" << endl;
	printArray(arr, len);
	bubbleSort(arr, len);
	cout << "排序后：" << endl;
	printArray(arr, len);
	system("pause");
	return 0;
}
//冒泡排序函数
void bubbleSort(int* arr, int len)
{
	for (int i = 0; i < len -1; i++)
	{
		for (int j = 0; j < len - i - 1; j++) {
			if (arr[j] > arr[j + 1]) 
			{
				int temp = arr[j];
				arr[j] = arr[j + 1];
				arr[j + 1] = temp;
			}
		}
	}
}
//打印数组函数
void printArray(int arr[], int len)
{
	for (int i = 0; i < len; i++)
	{
		cout << arr[i] << " ";
	}
	cout << endl;
}
```

> 总结：当数组名传入到函数作为参数时，被退化为指向首元素的指针

## 8 结构体

### 8.1 结构体基本概念

结构体属于用户==自定义的数据类型==，允许用户存储不同的数据类型

### 8.2 结构体定义和使用

**语法：**`struct 结构体名 { 结构体成员列表 }；`

通过结构体创建变量的方式有三种：

* struct 结构体名 变量名
* struct 结构体名 变量名 = { 成员1值 ， 成员2值...}
* 定义结构体时顺便创建变量

**示例：**

```c++
struct student
{
	//成员变量
	string name;	//姓名
	int age;		//年龄
	int score;		//分数
}stu3;	//结构体变量创建方式3 
int main()
{
	//结构体变量创建方式1
	struct student stu1;	//struct 关键字可以省略
	stu1.name = "张三";
	stu1.age = 18;
	stu1.score = 98;
	cout << "姓名: " << stu1.name << " 年龄: " << stu1.age << " 分数: " << stu1.score << endl;
	//结构体变量创建方式2
	student stu2 = { "李四",20,100 };
	cout << "姓名: " << stu2.name << " 年龄: " << stu2.age << " 分数: " << stu2.score << endl;
	
	stu3.name = "王五";
	stu3.age = 18;
	stu3.score = 80;
	cout << "姓名：" << stu3.name << " 年龄：" << stu3.age << " 分数：" << stu3.score << endl;
	system("pause");
	return 0;
}
```

> 总结1：定义结构体时的关键字是struct，不可省略
>
> 总结2：创建结构体变量时，关键字struct可以省略
>
> 总结3：结构体变量利用操作符 ''.''  访问成员

### 8.3 结构体数组

**作用：**将自定义的结构体放入到数组中方便维护

**语法：**` struct  结构体名 数组名[元素个数] = {  {} , {} , ... {} }`

**示例：**

```c++
struct student
{
	string name;
	int age;
	int score;
};
int main()
{
	//结构体数组
	student stu[3] =
	{
		{"张三",18,100},
		{"李四",20,98},
		{"王五",19,87}
	};
	for (int i = 0; i < 3; i++)
	{
		cout << "姓名：" << stu[i].name << " 年龄：" << stu[i].age << " 分数：" << stu[i].score << endl;
	}
	system("pause");
	return 0;
}
```

### 8.4 结构体指针

**作用：**通过指针访问结构体中的成员

* 利用操作符 `-> `可以通过结构体指针访问结构体属性

**示例：**

```c++
struct student
{
	string name;
	int age;
	int score;
};
int main()
{
	//结构体数组
	student stu = { "张三",18,100 };
	student* p = &stu;
	p->score = 80;	//指针通过 -> 操作符可以访问成员
	cout << "姓名：" << p->name << " 年龄：" << p->age << " 分数：" << p->score << endl;
	system("pause");
	return 0;
}
```

> 总结：结构体指针可以通过 -> 操作符 来访问结构体中的成员

### 8.5 结构体嵌套结构体

**作用：** 结构体中的成员可以是另一个结构体

**例如：**每个老师辅导一个学员，一个老师的结构体中，记录一个学生的结构体

**示例：**

```c++
struct student
{
	string name;
	int age;
	int score;
};
//教师结构体定义
struct teacher
{
	//成员列表
	int id;	//职工编号
	string name;	//教师姓名
	int age;	//教师年龄
	struct student stu;	//子结构体 学生
};
int main()
{
	teacher t1;
	t1.id = 10001;
	t1.name = "老张";
	t1.age = 40;
	t1.stu.name = "周迅";
	t1.stu.age = 18;
	t1.stu.score = 100;
	cout << "教师 职工编号： " << t1.id << " 姓名： " << t1.name << " 年龄： " << t1.age << endl;
	cout << "辅导学员 姓名： " << t1.stu.name << " 年龄：" << t1.stu.age << " 考试分数： " << t1.stu.score << endl;
	system("pause");
	return 0;
}
```

> **总结：**在结构体中可以定义另一个结构体作为成员，用来解决实际问题

### 8.6 结构体做函数参数 

**作用：**将结构体作为参数向函数中传递

传递方式有两种：

* 值传递
* 地址传递

**示例：**

```c++
struct student
{
	string name;
	int age;
	int score;
};
//值传递
void printStudent1(student stu)
{
	//stu.score = 98;
	cout << "值传递子函数中 姓名：" << stu.name << " 年龄：" << stu.age << " 分数： " << stu.score << endl;
}
//地址传递
void printStudent2(student* stu)
{
	stu->score = 98;
	cout << "地址传递子函数中 姓名：" << stu->name << " 年龄：" << stu->age << " 分数： " << stu->score << endl;
}
int main()
{
	student stu = { "张三",18,100 };
	printStudent1(stu);
	printStudent2(&stu);
	cout << "主函数中 姓名：" << stu.name << " 年龄： " << stu.age << " 分数：" << stu.score << endl;
	system("pause");
	return 0;
}
```

> 总结：如果不想修改主函数中的数据，用值传递，反之用地址传递
>
> 地址传递会导致主函数中的原数据被修改

### 8.7 结构体中 const使用场景

**作用：**用const来防止误操作

**示例：**

```c++
struct student
{
	string name;
	int age;
	int score;
};

void printStudent( const student* stu)	//加const防止函数体中的误操作
{
	//stu->score = 98;	//操作失败，因为加了const修饰
	cout << "姓名：" << stu->name << " 年龄：" << stu->age << " 分数： " << stu->score << endl;
}
int main()
{
	student stu = { "张三",18,100 };
	printStudent(&stu);
	system("pause");
	return 0;
}
```

### 8.8 结构体案例

#### 8.8.1 案例1

**案例描述：**

学校正在做毕设项目，每名老师带领5个学生，总共有3名老师，需求如下

设计学生和老师的结构体，其中在老师的结构体中，有老师姓名和一个存放5名学生的数组作为成员

学生的成员有姓名、考试分数，创建数组存放3名老师，通过函数给每个老师及所带的学生赋值

最终打印出老师数据以及老师所带的学生数据。

**示例：**

```c++
#include <ctime>
struct Student
{
	string name;
	int score;
};
struct Teacher
{
	string name;
	Student stu[5];
};
void allocateSpace(Teacher t[], int len)
{
	string tName = "教师";
	string sName = "学生";
	string nameSeed = "ABCDE";
	for (int i = 0; i < len; i++)
	{
		t[i].name = tName + nameSeed[i];
		for (int j = 0; j < 5; j++)
		{
			t[i].stu[j].name = sName + nameSeed[j];
			t[i].stu[j].score = rand() % 61 + 40;
		}
	}
}
void printTeachers(Teacher t[], int len)
{
	for (int i = 0; i < len; i++)
	{
		cout << t[i].name << endl;
		for (int j = 0; j < 5; j++)
		{
			cout << "\t姓名" << t[i].stu[j].name << " 分数：" << t[i].stu[j].score << endl;
		}
	}
}
int main()
{
	srand((unsigned int)time(NULL));
	Teacher t[3];
	int len = sizeof(t) / sizeof(Teacher);
	allocateSpace(t, len); //创建数据
	printTeachers(t, len); //打印数据
	system("pause");
	return 0;
}
```

#### 8.8.2 案例2

**案例描述：**

设计一个英雄的结构体，包括成员姓名，年龄，性别;创建结构体数组，数组中存放5名英雄。

通过冒泡排序的算法，将数组中的英雄按照年龄进行升序排序，最终打印排序后的结果。

五名英雄信息如下：

```C++
		{"刘备",23,"男"},
		{"关羽",22,"男"},
		{"张飞",20,"男"},
		{"赵云",21,"男"},
		{"貂蝉",19,"女"},
```

**示例：**

```c++
struct Heros
{
	string name;
	int age;
	string gender;
}h;
void bubbleSort(Heros h[], int len)
{
	for (int i = 0; i < len -1; i++)
	{
		for (int j = 0; j < len - i - 1; j++)
		{
			if (h[j].age > h[j + 1].age) 
			{
				Heros temp = h[j];
				h[j] = h[j + 1];
				h[j + 1] = temp;
			}
		}
	}
}
//打印数组
void printHeros(Heros h[], int len)
{
	for (int i = 0; i < len; i++)
	{
		cout << "姓名：" << h[i].name << "年龄：" << h[i].age << "性别：" << h[i].gender << endl;
	}
}
int main()
{
	Heros h[5] =
	{
		{ "刘备", 23, "男" },
		{ "关羽",22,"男" },
		{ "张飞",20,"男" },
		{ "赵云",21,"男" },
		{ "貂蝉",19,"女" },
	};
	int len = sizeof(h) / sizeof(Heros);
	bubbleSort(h, len);
	printHeros(h, len);
	system("pause");
	return 0;
}
```

# C++核心编程

本阶段主要针对C++==面向对象==编程技术做详细讲解，探讨C++中的核心和精髓。

## 1 内存分区模型

C++程序在执行时，将内存大方向划分为**4个区域**

- 代码区：存放函数体的二进制代码，由操作系统进行管理的
- 全局区：存放全局变量和静态变量以及常量
- 栈区：由编译器自动分配释放, 存放函数的参数值,局部变量等
- 堆区：由程序员分配和释放,若程序员不释放,程序结束时由操作系统回收

**内存四区意义：**

不同区域存放的数据，赋予不同的生命周期, 给我们更大的灵活编程

### 1.1 程序运行前

​	在程序编译后，生成了exe可执行程序，**未执行该程序前**分为两个区域

​	**代码区：**

​		存放 CPU 执行的机器指令

​		代码区是**共享**的，共享的目的是对于频繁被执行的程序，只需要在内存中有一份代码即可

​		代码区是**只读**的，使其只读的原因是防止程序意外地修改了它的指令

​	**全局区：**

​		全局变量和静态变量存放在此.

​		全局区还包含了常量区, 字符串常量和其他常量也存放在此.

​		==该区域的数据在程序结束后由操作系统释放==.

**示例：**

```c++
#include "base.h"
//全局变量
int g_a = 10;
int g_b = 10;

//全局常量
const int c_g_a = 10;
const int c_g_b = 10;

int main()
{
	//局部变量
	int a = 10;
	int b = 10;

	//打印地址
	cout << "局部变量a地址为： " << (int)&a << endl;
	cout << "局部变量b地址为： " << (int)&b << endl;

	cout << "全局变量g_a地址为： " << (int)&g_a << endl;
	cout << "全局变量g_b地址为： " << (int)&g_b << endl;

	//静态变量
	static int s_a = 10;
	static int s_b = 10;

	cout << "静态变量s_a地址为： " << (int)&s_a << endl;
	cout << "静态变量s_b地址为： " << (int)&s_b << endl;

	cout << "字符串常量地址为： " << (int)&"hello world" << endl;
	cout << "字符串常量地址为： " << (int)&"hello world1" << endl;

	cout << "全局常量c_g_a地址为： " << (int)&c_g_a << endl;
	cout << "全局常量c_g_b地址为： " << (int)&c_g_b << endl;

	const int c_l_a = 10;
	const int c_l_b = 10;
	cout << "局部常量c_l_a地址为： " << (int)&c_l_a << endl;
	cout << "局部常量c_l_b地址为： " << (int)&c_l_b << endl;
	system("pause");
	return 0;
}
```

总结：

* C++中在程序运行前分为全局区和代码区
* 代码区特点是共享和只读
* 全局区中存放全局变量、静态变量、常量
* 常量区中存放 const修饰的全局常量  和 字符串常量

### 1.2 程序运行后

​	**栈区：**

​		由编译器自动分配释放, 存放函数的参数值,局部变量等

​		注意事项：不要返回局部变量的地址，栈区开辟的数据由编译器自动释放

**示例：**

```c++
#include "base.h"
//栈区数据注意事项 ---不要返回局部变量的地址
//栈区的数据由编译器管理开辟和释
int* fun()
{
	int a = 10;	//局部变量 存放在栈区，栈区的数据在函数执行完后自动释放
	return &a;	//返回局部变量的地址
}
int main()
{
	int* p = fun();
	cout << *p << endl;
	cout << *p << endl;
	system("pause");
	return 0;
}
```

**堆区：**

​		由程序员分配释放,若程序员不释放,程序结束时由操作系统回收

​		在C++中主要利用new在堆区开辟内存

**示例：**

```c++
#include "base.h"
int* fun()
{
	//利用new关键字	可以将数据开辟到堆区
	//指针	本质也是局部变量，放在栈上，指针保存的数据是放在堆区
	int* p = new int(10);
	return p;
}
int main()
{
	//在堆区开辟数据
	int* p = fun();
	cout << *p << endl;
	cout << *p << endl;
	cout << *p << endl;
	cout << *p << endl;
	system("pause");
	return 0;
}
```

**总结：**

堆区数据由程序员管理开辟和释放

堆区数据利用new关键字进行开辟内存

### 1.3 new操作符

​	C++中利用==new==操作符在堆区开辟数据

​	堆区开辟的数据，由程序员手动开辟，手动释放，释放利用操作符 ==delete==

​	语法：` new 数据类型`

​	利用new创建的数据，会返回该数据对应的类型的指针

**示例1： 基本语法**

```c++
//1.new的基本语法
int* fun()
{
	//在堆区创建整型数据
	//new返回是 该数据类型的指针
	int* p = new int(10);
	return p;
}
int main()
{
	int* p = fun();
	cout << *p << endl;
	delete p;	
	cout << *p << endl;	//内存已被释放，再次访问就是非法操作，会报错
	system("pause");
	return 0;
}
```

**示例2：开辟数组**

```c++
//堆区开辟数组
void test1()
{
	//创建10整型数据的数组，在堆区
	int* arr = new int[10];	//10代表数组中有10个元素
	for (int i = 0; i < 10; i++)
	{
		arr[i] = i + 100;
	}
	for (int i = 0; i < 10; i++)
	{
		cout << arr[i] << endl;
	}
    //释放数组 delete 后加 []
	delete[] arr;
}
```

## 2 引用

### 2.1 引用的基本使用

**作用： **给变量起别名

**语法：** `数据类型 &别名 = 原名

**示例：**

```c++
int main()
{
	int a = 10;
	int& b = a;
	
	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	b = 100;
	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	system("pause");
	return 0;
}
```

### 2.2 引用注意事项

* 引用必须初始化
* 引用在初始化后，不可以改变

示例：

```c++
int main()
{
	int a = 10;
	int b = 20;
	//int& c;	//错误，引用必须初始化
	int& c = a;	//一旦初始化后，就不可以更改
	c = b;	//这是赋值操作，不是更改引用

	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	cout << "c = " << c << endl;
	system("pause");
	return 0;
}
```

### 2.3 引用做函数参数

**作用：**函数传参时，可以利用引用的技术让形参修饰实参

**优点：**可以简化指针修改实参

**示例：**

```c++
#include "base.h"
//1.值传递
void swap01(int a, int b)
{
	int temp = a;
	a = b;
	b = temp;
}
//2.地址传递
void swap02(int *a, int *b)
{
	int temp = *a;
	*a = *b;
	*b = temp;
}
//3.引用传递
void swap03(int &a, int &b)
{
	int temp = a;
	a = b;
	b = temp;
}
int main()
{
	int a = 10;
	int b = 20;
	//swap01(a, b);	//值传递，形参不会修饰实参
	//cout << "a:" << a << " b:" << b << endl;
	//swap02(&a, &b);	//地址传递，形参会修饰实参
	//cout << "a:" << a << " b:" << b << endl;
	swap03(a, b);	//引用传递，形参会修饰实参
	cout << "a:" << a << " b:" << b << endl;
	system("pause");
	return 0;
}
```

> 总结：通过引用参数产生的效果同按地址传递是一样的。引用的语法更清楚简单
>
> 地址传递和引用传递，形参会修饰实参

### 2.4 引用做函数返回值

作用：引用是可以作为函数的返回值存在的

注意：**不要返回局部变量引用**

用法：函数调用作为左值

**示例：**

```c++
#include "base.h"
//引用做函数的返回值
//1.不要返回局部变量的引用
int & test()
{
	int a = 10;		//局部变量存放在四区中的 栈区
	return a;
}
//2.函数的调用可以作为左值
int& test2()
{
	static int a = 10;	//静态变量，存放在全局区，全局区上的数据在程序结束后系统释放
	return a;
}
int main()
{
	//int& ref = test();
	//cout << ref << endl;	//第一次结果正确，是因为编译器做了保留
	//cout << ref << endl;	//第二次结果错误，因为a的内存已经释放

	int& ref2 = test2();
	cout << ref2 << endl;
	cout << ref2 << endl;
	test2() = 1000;	//如果函数的返回值是引用，这个函数调用可以作为左值
	cout << ref2 << endl;
	cout << ref2 << endl;
	system("pause");
	return 0;
}
```

### 2.5 引用的本质

本质：**引用的本质在c++内部实现是一个指针常量.**

讲解示例：

```c++
#include "base.h"
//发现是引用，转换为 int* const ref = &a;
void func(int& ref)
{
	ref = 100; //ref是引用，转换为*ref = 100
}
int main()
{
	int a = 10;
	//自动转换为 int * const ref = &a; 指针常量是指针指向不可改，也说明为什么引用不可更改
	int& ref = a;
	ref = 20;	//内部发现ref是引用，自动帮我们转换为： *ref = 20;
	cout << "a = " << a << endl;
	cout << "ref:" << ref << endl;
	func(a);
	system("pause");
	return 0;
}
```

结论：C++推荐用引用技术，因为语法方便，引用本质是指针常量，但是所有的指针操作编译器都帮我们做了

### 2.6 常量引用

**作用：**常量引用主要用来修饰形参，防止误操作

在函数形参列表中，可以加==const修饰形参==，防止形参改变实参

**示例：**

```c++
#include "base.h"
//引用使用的场景，通常用来修饰形参
void showValue(const int& v) {
	//v += 10;
	cout << v << endl;
}
int main()
{
	//常量引用
	//使用场景：用来修饰形参，防止误操作
	//int a = 10;
	//int& ref = 10;	//引用本身需要一个合法的内存空间，因此这行错误
	//加入const就可以了，编译器优化代码，int temp = 10; const int& ref = temp;
	const int& ref = 10;
	//ref = 100;	//加入const后不可以修改变量
	cout << ref << endl;

	//函数中利用常量引用防止误操作修改实参
	int a = 20;
	showValue(a);
	system("pause");
	return 0;
}
```

## 3 函数提高

### 3.1 函数默认参数

在C++中，函数的形参列表中的形参是可以有默认值的。

语法：` 返回值类型  函数名 （参数= 默认值）{}`

**示例：**

```c++
#include "base.h"
//函数默认参数
//如果我们自己传入数据，就用自己的数据，如果没有就用默认值
int func(int a, int b= 20, int c = 30)
{
	return a + b + c;
}
//注意事项
//1.如果某个位置已经有了默认值，那么从这个位置往后，从左到右都必须有默认值
//2.如果函数声明有默认参数，函数实现就不能有默认参数
//声明和实现只能有一个默认参数
int main()
{
	cout << func(10,30) << endl;;
	system("pause");
	return 0;
}
```

### 3.2 函数占位参数

C++中函数的形参列表里可以有占位参数，用来做占位，调用函数时必须填补该位置

**语法：** `返回值类型 函数名 (数据类型){}`

在现阶段函数的占位参数存在意义不大，但是后面的课程中会用到该技术

**示例：**

```c++
#include "base.h"
//函数占位参数 ，占位参数也可以有默认参数
void func(int a, int) {
	cout << "this is func" << endl;
}
int main()
{
	func(10, 10); //占位参数必须填补
	system("pause");
	return 0;
}
```

### 3.3 函数重载

#### 3.3.1 函数重载概述

**作用：**函数名可以相同，提高复用性

**函数重载满足条件：**

* 同一个作用域下
* 函数名称相同
* 函数参数**类型不同**  或者 **个数不同** 或者 **顺序不同**

**注意:**  函数的返回值不可以作为函数重载的条件

**示例：**

```c++
#include "base.h"
//函数重载需要函数都在同一个作用域下
void func()
{
	cout << "func 的调用！" << endl;
}
void func(int a)
{
	cout << "func (int a) 的调用！" << endl;
}
void func(double a)
{
	cout << "func (double a)的调用！" << endl;
}
void func(int a, double b)
{
	cout << "func (int a ,double b) 的调用！" << endl;
}
void func(double a, int b)
{
	cout << "func (double a ,int b)的调用！" << endl;
}

//函数返回值不可以作为函数重载条件
//int func(double a, int b)
//{
//	cout << "func (double a ,int b)的调用！" << endl;
//}
int main() {

	func();
	func(10);
	func(3.14);
	func(10, 3.14);
	func(3.14, 10);

	system("pause");
	return 0;
}
```

#### 3.3.2 函数重载注意事项

* 引用作为重载条件
* 函数重载碰到函数默认参数

**示例：**

```c++
#include "base.h"
//函数重载注意事项
//1、引用作为重载条件
void fun(int &a)	//int &a = 10 不合法
{
	cout << "fun(int &a)调用" << endl;
}
void fun(const int& a)
{
	cout << "fun(const int &a)调用" << endl;
}
//2、函数重载碰到函数默认参数
void fun2(int a, int b = 10)
{
	cout << "fun2(int a，int b = 10)调用" << endl;
}
void fun2(int a)
{
	cout << "fun2(int a)调用" << endl;
}
int main()
{
	/*int a = 10;
	fun(a);*/
	fun(10);
	//fun2(10);	//当函数重载碰到默认参数，出现二义性，报错，尽量避免
	system("pause");
	return 0;
}
```

## **4** 类和对象

C++面向对象的三大特性为：==封装、继承、多态==

C++认为==万事万物都皆为对象==，对象上有其属性和行为

### 4.1 封装

#### 4.1.1  封装的意义

封装是C++面向对象三大特性之一

封装的意义：

* 将属性和行为作为一个整体，表现生活中的事物
* 将属性和行为加以权限控制

**封装意义一：**

​	在设计类的时候，属性和行为写在一起，表现事物

**语法：** `class 类名{   访问权限： 属性  / 行为  };`

**示例1：**设计一个圆类，求圆的周长

**示例代码：**

```c++
#include <iostream>
using namespace std;
const double PI = 3.14;
//1、封装的意义
//将属性和行为作为一个整体，用来表现生活中的事物

//封装一个圆类，求圆的周长
//class代表设计一个类，后面跟着的是类名
class Circle
{
public:	  //访问权限  公共的权限
	//属性
	int m_r;//半径
	//行为
	//获取到圆的周长
	double calculateZC()
	{
		//2 * pi  * r
		//获取圆的周长
		return  2 * PI * m_r;
	}
};
int main()
{
	//通过圆类，创建圆的对象
	// c1就是一个具体的圆
	Circle c1;
	c1.m_r = 10;	//给圆对象的半径 进行赋值操作
	cout << "圆的周长为： " << c1.calculateZC() << endl;

	system("pause");
	return 0;
}
```

**示例2：**设计一个学生类，属性有姓名和学号，可以给姓名和学号赋值，可以显示学生的姓名和学号

**示例2代码：**

```c++
#include <iostream>
using namespace std;
class Student
{
public:
	void setName(string name)
	{
		m_name = name;
	}
	void setId(int id)
	{
		m_id = id;
	}
	void showStudent()
	{
		cout << "name:" << m_name << " ID:" << m_id << endl;
	}
public:
	string m_name;
	int m_id;
};
int main()
{
	Student stu1;
	stu1.setName("张三");
	stu1.setId(1002);
	stu1.showStudent();
	system("pause");
	return 0;
}
```

**封装意义二：**

类在设计时，可以把属性和行为放在不同的权限下，加以控制

访问权限有三种：

1. public        公共权限  
2. protected 保护权限
3. private      私有权限

```c++
#include <iostream>
using namespace std;
//三种权限
//公共权限  public     类内可以访问  类外可以访问
//保护权限  protected  类内可以访问  类外不可以访问
//私有权限  private    类内可以访问  类外不可以访问
class Person
{
public:
	string name;
protected:
	int age;
private:
	char gender;
public:
	void info()
	{
		name = "张三";
		age = 18;
		gender = '男';
		cout << "我的名字是：" << name << " 我今年：" << age << "岁" << " 我的性别是：" << gender << endl;;
	}
};
int main()
{
	Person p;
	p.name = "李四";
	p.info();
	//p.age = 20;	//保护权限类外访问不到
	//p.gender = '女';	//私有权限类外访问不到
	
	system("pause");
	return 0;
}
```

#### 4.1.2 struct和class区别

在C++中 struct和class唯一的**区别**就在于 **默认的访问权限不同**

区别：

* struct 默认权限为公共
* class   默认权限为私有

```c++
#include <iostream>
using namespace std;

class C1
{
	int a;	//默认是私有权限
};
struct C2
{
	int b;	//默认是公共权限
};
int main()
{
	C1 c1;
	c1.a = 10;	//错误，访问权限是私有
	C2 c2;
	c2.b = 20;	//正确，访问权限是公共
	system("pause");
	return 0;
}
```

#### 4.1.3 成员属性设置为私有

**优点1：**将所有成员属性设置为私有，可以自己控制读写权限

**优点2：**对于写权限，我们可以检测数据的有效性

**示例：**

```c++
#include <iostream>
using namespace std;
class Person
{
public:
	void setName(string name)
	{
		m_Name = name;
	}
	string getName() { return m_Name; }
	void setAge(int age)
	{
		m_Age = age;
	}
	
	int getPassword() { return m_Password; }

private:
	string m_Name;	//姓名设置可读可写
	int m_Age;	//只写  年龄
	int m_Password;	//只读 密码
};
int main()
{
	Person p;
	p.setName("张三");
	cout << p.getName() << endl;
	p.setAge(18);
	cout << p.getPassword() << endl;

	system("pause");
	return 0;
}
```

**练习案例1：设计立方体类**

设计立方体类(Cube)

求出立方体的面积和体积

分别用全局函数和成员函数判断两个立方体是否相等。

![1545533548532](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151659061.webp)

```c++
#include <iostream>
using namespace std;
//立方体类设计
//1.创建立方体类
//2.设计属性
//3.设计行为 获取立方体面积和体积
//4.分别用全局函数和成员函数判断两个立方体是否相等
class Cube
{
public:
	void setL(int l){ m_L = l; }
	int getL() { return m_L; }
	void setW(int w) { m_W = w; }
	int getW() { return m_W; }
	void setH(int h) { m_H = h; }
	int getH() { return m_H; }
	int calculateS()	//表面积
	{
		return 2 * (m_L * m_W + m_L * m_H + m_W * m_H);
	}
	int calculateV() { return m_L * m_W * m_H; }	//体积
	//成员函数
	bool isSameByClass(Cube &c)
	{
		if (m_L == c.getL() && m_W == c.getW() && m_H == c.getH())
		{
			return true;
		}
		return false;
	}
private:
	int m_L;	//长
	int m_W;	//宽
	int m_H;	//高
};
//全局函数
bool isSame(Cube& c1, Cube& c2)
{
	if (c1.getL() == c2.getL() && c1.getW() == c2.getW() && c1.getH() == c2.getH())
	{
		return true;
	}
	return false;
}
int main()
{
	Cube c1;
	c1.setL(10);
	c1.setW(10);
	c1.setH(10);
	cout << "长方体的表面积：" << c1.calculateS() << endl;
	cout << "长方体的体积：" << c1.calculateV() << endl;
	Cube c2;
	c2.setL(10);
	c2.setW(10);
	c2.setH(10);
	bool result = isSame(c1, c2);
	if (result)
	{
		cout << "全局函数:c1和c2是相等的" << endl;
	}
	else {
		cout << "全局函数:c1和c2是不相等的" << endl;
	}
	cout << "=======================" << endl;
	result = c1.isSameByClass(c2);
	if (result)
	{
		cout << "成员函数:c1和c2是相等的" << endl;
	}
	else {
		cout << "成员函数:c1和c2是不相等的" << endl;
	}
	system("pause");
	return 0;
}
```

**练习案例2：点和圆的关系**

设计一个圆形类（Circle），和一个点类（Point），计算点和圆的关系。

![1545533829184](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151713609.webp)

```c++
#include <iostream>
using namespace std;
//点和圆的关系案例
// 点类
class Point
{
public:
	void setX(int x) { m_X = x; }
	int getX() { return m_X; }
	void setY(int y) { m_Y = y; }
	int getY() { return m_Y; }
private:
	int m_X;
	int m_Y;
};
//圆类
class Circle
{
public:
	//设置半径
	void setR(int r) { m_R = r; }
	//获取半径
	int getR() { return m_R; }
	//设置圆心
	void setCenter(Point center) { m_Center = center; }
	//获取圆心
	Point getCenter() { return m_Center; }

private:
	int m_R;	//半径
	Point m_Center;	//圆心
};
//判断点和圆关系
void isInCircle(Circle& c, Point& p)
{
	//计算两点之间距离 平方
	int distance = (c.getCenter().getX() - p.getX()) * (c.getCenter().getX() - p.getX()) +
		(c.getCenter().getY() - p.getY()) * (c.getCenter().getY() - p.getY());
	//计算半径的平方
	int rDistance = c.getR() * c.getR();
	//判断关系
	if (distance == rDistance)
	{
		cout << "点在圆上" << endl;
	}
	else if (distance > rDistance)
	{
		cout << "点在圆外" << endl;
	}
	else {
		cout << "点在圆内" << endl;
	}
}
int main()
{
	//创建圆
	Circle c;
	c.setR(10);
	Point center;
	center.setX(10);
	center.setY(0);
	c.setCenter(center);
	//创建点
	Point p;
	p.setX(10);
	p.setY(10);
	isInCircle(c, p);
	system("pause");
	return 0;
}
```

### 4.2 对象的初始化和清理

*  生活中我们买的电子产品都基本会有出厂设置，在某一天我们不用时候也会删除一些自己信息数据保证安全
*  C++中的面向对象来源于生活，每个对象也都会有初始设置以及 对象销毁前的清理数据的设置。

#### 4.2.1 构造函数和析构函数

对象的**初始化和清理**也是两个非常重要的安全问题

​	一个对象或者变量没有初始状态，对其使用后果是未知

​	同样的使用完一个对象或变量，没有及时清理，也会造成一定的安全问题

c++利用了**构造函数**和**析构函数**解决上述问题，这两个函数将会被编译器自动调用，完成对象初始化和清理工作。

对象的初始化和清理工作是编译器强制要我们做的事情，因此如果**我们不提供构造和析构，编译器会提供**

**编译器提供的构造函数和析构函数是空实现。**

* 构造函数：主要作用在于创建对象时为对象的成员属性赋值，构造函数由编译器自动调用，无须手动调用。
* 析构函数：主要作用在于对象**销毁前**系统自动调用，执行一些清理工作。

**构造函数语法：**`类名(){}`

1. 构造函数，没有返回值也不写void
2. 函数名称与类名相同
3. 构造函数可以有参数，因此可以发生重载
4. 程序在调用对象时候会自动调用构造，无须手动调用,而且只会调用一次

**析构函数语法：** `~类名(){}`

1. 析构函数，没有返回值也不写void
2. 函数名称与类名相同,在名称前加上符号  ~
3. 析构函数不可以有参数，因此不可以发生重载
4. 程序在对象销毁前会自动调用析构，无须手动调用,而且只会调用一次

```c++
#include <iostream>
using namespace std;
//对象的初始化和清理

class Person 
{
public:
	//1.构造函数 进行初始化操作
	Person() {
		cout << "构造函数的调用" << endl;
	}
	//2.析构函数 进行清理的操作
	~Person()
	{
		cout << "析构函数的调用" << endl;
	}
};
//构造和析构都是必须有的实现，如果我们自己不提供，编译器会提供一个空实现的构造和析构
void test()
{
	Person p;	//在栈上的数据，test()执行完毕后，释放这个对象
}
int main()
{
	test();
	system("pause");
	return 0;
}
```

#### 4.2.2 构造函数的分类及调用

两种分类方式：

​	按参数分为： 有参构造和无参构造

​	按类型分为： 普通构造和拷贝构造

三种调用方式：

​	括号法

​	显示法

​	隐式转换法

**示例：**

```c++
#include <iostream>
using namespace std;
//1、构造函数分类
// 按照参数分类分为 有参和无参构造   无参又称为默认构造函数
// 按照类型分类分为 普通构造和拷贝构造
class Person
{
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a)
	{
		age = a;
		cout << "有参构造函数!" << endl;
	}
	//拷贝构造函数
	Person(const Person & p)
	{
		cout << "拷贝构造函数!" << endl;
		age = p.age;
	}
	//析构函数
	~Person()
	{
		cout << "析构函数的调用" << endl;
	}
public:
	int age;
};
//2、构造函数的调用

//调用有参的构造函数
void test()
{
	//2.1  括号法，常用
	/*Person p1(10);*/
	//调用无参构造函数
	//注意1：调用无参构造函数不能加括号，如果加了编译器认为这是一个函数声明
	//Person p2();
	/*Person p2;
	Person p3(p1);
	cout << "p1的年龄是：" << p1.age << endl;
	cout << "p3的年龄是：" << p3.age << endl;*/

	//2.2 显式法
	Person p2 = Person(10);
	Person p3 = Person(p2);
	//Person(10)单独写就是匿名对象  当前行结束之后，马上析构

	//2.3 隐式转换法
	Person p4 = 10; // Person p4 = Person(10); 
	Person p5 = p4; // Person p5 = Person(p4); 

	//注意2：不能利用 拷贝构造函数 初始化匿名对象 编译器认为是对象声明
	//Person p5(p4);
}
int main()
{
	test();
	system("pause");
	return 0;
}
```

#### 4.2.3 拷贝构造函数调用时机

C++中拷贝构造函数调用时机通常有三种情况

* 使用一个已经创建完毕的对象来初始化一个新对象
* 值传递的方式给函数参数传值
* 以值方式返回局部对象

**示例：**

```c++
#include <iostream>
using namespace std;

class Person
{
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
		age = 0;
	}
	//有参构造函数
	Person(int a)
	{
		age = a;
		cout << "有参构造函数!" << endl;
	}
	//拷贝构造函数
	Person(const Person& p)
	{
		cout << "拷贝构造函数!" << endl;
		age = p.age;
	}
	//析构函数
	~Person()
	{
		cout << "析构函数的调用" << endl;
	}
	int age;
};
//1. 使用一个已经创建完毕的对象来初始化一个新对象
void test01()
{
	Person p1(20);
	Person p2(p1);
}
//2.值传递的方式给函数参数传值
void doWork(Person p){}
void test02()
{
	Person p;
	doWork(p);
}
//3.值方式返回局部对象
Person doWork2()
{
	Person p1;
	cout << (int*)&p1 << endl;
	return p1;
}
void test03()
{
	Person p = doWork2();
	cout << (int*)&p << endl;
}
int main()
{
	//test01();
	//test02();
	test03();
	system("pause");
	return 0;
}
```

#### 4.2.4 构造函数调用规则

默认情况下，c++编译器至少给一个类添加3个函数

1．默认构造函数(无参，函数体为空)

2．默认析构函数(无参，函数体为空)

3．默认拷贝构造函数，对属性进行值拷贝

构造函数调用规则如下：

* 如果用户定义有参构造函数，c++不在提供默认无参构造，但是会提供默认拷贝构造


* 如果用户定义拷贝构造函数，c++不会再提供其他构造函数

示例：

```c++
#include <iostream>
using namespace std;
//构造函数的调用规则
//1.创建一个类，C++编译器会给每个类都添加至少3个函数
//默认构造 （空实现）
//析构函数 （空实现）
//拷贝构造 （值拷贝）
//2.如果我们写了有参构造函数，编译器就不再提供默认构造，依然提供拷贝构造
//如果我们写了拷贝构造函数，编译器就不再提供其他普通构造函数了
class Person
{
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a)
	{
		age = a;
		cout << "有参构造函数!" << endl;
	}
	//拷贝构造函数
	Person(const Person& p)
	{
		cout << "拷贝构造函数!" << endl;
		age = p.age;
	}
	//析构函数
	~Person()
	{
		cout << "析构函数的调用" << endl;
	}
	int age;
};
void test01()
{
	Person p;
	p.age = 18;
	Person p2(p);
	cout << "p2的年龄:" << p2.age << endl;
}
void test02()
{
	Person p(28);
	Person p2(p);
	cout << "p2的年龄:" << p2.age << endl;
}

int main()
{
	//test01();
	test02();
	system("pause");
	return 0;
}
```

#### 4.2.5 深拷贝与浅拷贝

深浅拷贝是面试经典问题，也是常见的一个坑

浅拷贝：简单的赋值拷贝操作

深拷贝：在堆区重新申请空间，进行拷贝操作

![image-20240712153039770](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151730817.webp)

**示例：**

```c++
#include <iostream>
using namespace std;

class Person
{
public:
	//无参（默认）构造函数
	Person() {
		cout << "无参构造函数!" << endl;
	}
	//有参构造函数
	Person(int a,int b)
	{
		age = a;
		height = new int(b);
		cout << "有参构造函数!" << endl;
	}
	//自己实现拷贝构造函数 解决浅拷贝带来的问题
	Person(const Person& p)
	{
		cout << "Person 拷贝构造函数调用" << endl;
		age = p.age;
		//height = p.height; 编译器默认实现就是这行代码
		//深拷贝操作
		height = new int(*p.height);
	}
	
	//析构函数
	~Person()
	{
		//析构代码，将堆区开辟数据做释放操作
		if (height != NULL)
		{
			delete height;
			height = NULL;
		}
		cout << "析构函数的调用" << endl;
	}
	int age;
	int* height;
};
void test01()
{
	Person p1(18,160);
	cout << "p1的年龄为：" << p1.age << "身高为：" << *p1.height << endl;
	Person p2(p1);
	cout << "p2的年龄为：" << p2.age << "身高为：" << *p2.height << endl;
}

int main()
{
	test01();
	system("pause");
	return 0;
}
```

> 总结：如果属性有在堆区开辟的，一定要自己提供拷贝构造函数，防止浅拷贝带来的问题

#### 4.2.6 初始化列表

**作用：**

C++提供了初始化列表语法，用来初始化属性

**语法：**`构造函数()：属性1(值1),属性2（值2）... {}`

**示例：**

```c++
#include <iostream>
using namespace std;
class Person
{
public:
	//传统初始化操作
	/*Person(int a, int b, int c)
	{
		A = a;
		B = b;
		C = c;
	}*/
	//初始化列表初始化属性
	Person(int a, int b, int c) :A(a), B(b), C(c) {};
	
	int A;
	int B;
	int C;
};
void test() {
	Person p(1, 2, 3);
	cout << "A:" << p.A << endl;
	cout << "B:" << p.B << endl;
	cout << "C:" << p.C << endl;
}
int main()
{
	test();
	system("pause");
	return 0;
}
```

#### 4.2.7 类对象作为类成员

C++类中的成员可以是另一个类的对象，我们称该成员为 对象成员

例如：

```c++
class A {}
class B
{
    A a；
}
```

B类中有对象A作为成员，A为对象成员

那么当创建B对象时，A与B的构造和析构的顺序是谁先谁后？

**示例：**

```c++
#include <iostream>
using namespace std;
//类对象作为类成员
class Phone
{
public:
	Phone(string name)
	{
		cout << "Phone 的构造函数调用" << endl;
		Pname = name;
	};
	~Phone()
	{
		cout << "Phone析构函数调用" << endl;
	}
	string Pname;
};
class Person
{
public:
	Person(string name, string pName) :Name(name), P(pName) 
	{
		cout << "Person 的构造函数调用" << endl;
	};
	~Person()
	{
		cout << "Person析构函数调用" << endl;
	}
	//姓名
	string Name;
	//手机
	Phone P;
};
//当其他类对象作为本类成员，构造时候先构造类对象，再构造自身，析构的顺序与构造相反
void test01()
{
	Person p("张三", "苹果");
	cout << p.Name << "拿着" << p.P.Pname << endl;
}
int main()
{
	test01();
	system("pause");
	return 0;
}
```

#### 4.2.8 静态成员

静态成员就是在成员变量和成员函数前加上关键字static，称为静态成员

静态成员分为：

*  静态成员变量
   *  所有对象共享同一份数据
   *  在编译阶段分配内存
   *  类内声明，类外初始化
*  静态成员函数
   *  所有对象共享同一个函数
   *  静态成员函数只能访问静态成员变量

**示例1 ：**静态成员变量

```c++
#include <iostream>
using namespace std;
//静态成员变量特点：
	//1 在编译阶段分配内存
	//2 类内声明，类外初始化
	//3 所有对象共享同一份数据
class Person
{
public:
	static int A;  //静态成员变量
private:
	static int B; //静态成员变量也是有访问权限的
};
int Person::A = 10;
int Person::B = 20;
void test01()
{
	//静态成员变量两种访问方式
	//1.通过对象访问
	Person p;
	p.A = 100;
	cout << "p.A = " << p.A << endl;
	Person p2;
	p2.A = 200;
	cout << "p.A = " << p.A << endl;
	cout << "p2.A = " << p2.A << endl;
	//2.通过类名访问
	cout << "A = " << Person::A << endl;

	//cout << "B = " << Person::B << endl; //私有权限访问不到

}
int main()
{
	test01();
	system("pause");
	return 0;
}
```

**示例2：**静态成员函数

```c++
#include <iostream>
using namespace std;
//静态成员函数
//所有对象共享同一个函数
//静态成员函数只能访问静态成员变量
class Person
{
public:
	//静态成员函数
	static void func()
	{
		A = 100;	//静态成员函数可以访问 静态成员变量
		//B = 200;	//静态成员函数 不可以访问 非静态成员变量，无法区分到底是哪个对象的B属性
		cout << "static void func调用" << endl;
	}
	static int A; //静态成员变量
	int B;	//非静态成员变量
private:
	static void func2()
	{
		cout << "static void func2调用" << endl;
	}
};
int Person::A = 0;
void test01()
{
	//1.通过对象访问
	Person p;
	p.func();
	//2.通过类名访问
	Person::func();

	//Person::func2(); 类外访问不到私有静态成员函数
	
}
int main()
{
	test01();
	system("pause");
	return 0;
}
```

### 4.3 C++对象模型和this指针

#### 4.3.1 成员变量和成员函数分开存储

在C++中，类内的成员变量和成员函数分开存储

只有非静态成员变量才属于类的对象上

```c++
#include <iostream>
using namespace std;
//成员变量 和 成员函数 分开存储的
class Person
{
	int A; //非静态成员变量
	static int B;	//静态成员变量 不属于类对象上
	void func() {};	//非静态成员函数 不属于类对象上
	static void func2() {}; //静态成员函数 不属于类对象上
};
int Person::B = 0;
void test01()
{
	Person p;
	//空对象占用内存空间为：1
	//C++编译器会给每个空对象也分配一个字节空间，是为了区分空对象占内存的位置
	//每个空对象也应该有一个独一无二的内存地址
	cout << "size of p = " << sizeof(p) << endl;
}
void test02()
{
	Person p;
	cout << "size of p = " << sizeof(p) << endl;
}
int main()
{
	//test01();
	test02();
	system("pause");
	return 0;
}
```

#### 4.3.2 this指针概念

通过4.3.1我们知道在C++中成员变量和成员函数是分开存储的

每一个非静态成员函数只会诞生一份函数实例，也就是说多个同类型的对象会共用一块代码

那么问题是：这一块代码是如何区分那个对象调用自己的呢？

c++通过提供特殊的对象指针，this指针，解决上述问题。**this指针指向被调用的成员函数所属的对象**

this指针是隐含每一个非静态成员函数内的一种指针

this指针不需要定义，直接使用即可

this指针的用途：

*  当形参和成员变量同名时，可用this指针来区分
*  在类的非静态成员函数中返回对象本身，可使用return *this

```c++
#include <iostream>
using namespace std;
class Person
{
public:
	Person(int age)
	{
		//this指针指向 被调用的成员函数所属的对象
		this->age = age;
	}

	Person& PersonAddAge(Person& p)
	{
		this->age += p.age;
		//this指向p2的指针，尔*this指向的就是p2这个对象本体
		return *this;
	}
	int age;
};
//1 解决名称冲突
void test01()
{
	Person p1(18);
	cout << "p1的年龄是：" << p1.age << endl;
}
//2 返回对象本身用*this
void test02()
{
	Person p1(10);
	Person p2(10);
	//链式编程思想
	p2.PersonAddAge(p1).PersonAddAge(p1);
	cout << "p2的年龄是：" << p2.age << endl;
}
int main()
{
	//test01();
	test02();
	system("pause");
	return 0;
}
```

#### 4.3.3 空指针访问成员函数

C++中空指针也是可以调用成员函数的，但是也要注意有没有用到this指针

如果用到this指针，需要加以判断保证代码的健壮性

**示例：**

```c++
#include <iostream>
using namespace std;
//空指针调用成员函数
class Person
{
public:
	void showClassName()
	{
		cout << "this is Person class" << endl;
	}
	void showPersonAge()
	{
		//报错原因是因为传入的指针是为NULL
		if (this == NULL)
		{
			return;
		}
		cout << "age = " << this->Age << endl;
	}
	int Age;
};
void test01()
{
	Person* p = NULL;
	p->showClassName();
	p->showPersonAge();

}
int main()
{
	test01();
	system("pause");
	return 0;
}
```

#### 4.3.4 const修饰成员函数

**常函数：**

* 成员函数后加const后我们称为这个函数为**常函数**
* 常函数内不可以修改成员属性
* 成员属性声明时加关键字mutable后，在常函数中依然可以修改

**常对象：**

* 声明对象前加const称该对象为常对象
* 常对象只能调用常函数

**示例：**

```c++
#include <iostream>
using namespace std;

//常函数
class Person
{
public:
	// this 指针的本质 是指针常量 指针的指向是不可以修改的
	// const Person * const this:
	// 在成员函数后面加const,修饰的是this指向，让指针指向的的值也不可以修改
	void showPerson() const
	{
		this->B = 100;
		//this->A = 100;
		//this = NULL; //this指针不可以修改指针的指向的
	}
	void func()
	{
		A = 200;
	}
	int A;
	mutable int B; //特殊变量，即使在常函数种，也可以修改这个值,加关键字mutable
};
void test01()
{
	Person p;
	p.showPerson();
}
//常对象
void test02()
{
	const Person p; //在对象前加const，变为常对象
	//p.A = 100;
	p.B = 100; //B是特殊值，在常对象下也可以修改

	//常对象只能调用常函数
	p.showPerson();
	//p.func();	//常对象 不可以调用普通成员函数，因为普通成员函数可以修改属性
}
int main()
{
	test01();
	system("pause");
	return 0;
}
```

### 4.4 友元

生活中你的家有客厅(Public)，有你的卧室(Private)

客厅所有来的客人都可以进去，但是你的卧室是私有的，也就是说只有你能进去

但是呢，你也可以允许你的好闺蜜好基友进去。

在程序里，有些私有属性 也想让类外特殊的一些函数或者类进行访问，就需要用到友元的技术

友元的目的就是让一个函数或者类 访问另一个类中私有成员

友元的关键字为  ==friend==

友元的三种实现

* 全局函数做友元
* 类做友元
* 成员函数做友元

#### 4.4.1 全局函数做友元

```c++
```

4.4.2 类做友元

```c++
```

4.4.3 成员函数做友元

```c++
```

### 4.5 运算符重载

运算符重载概念：对已有的运算符重新进行定义，赋予其另一种功能，以适应不同的数据类型

#### 4.5.1 加号运算符重载

作用：实现两个自定义数据类型相加的运算

```c++
```

> 总结1：对于内置的数据类型的表达式的的运算符是不可能改变的
>
> 总结2：不要滥用运算符重载

#### 4.5.2 左移运算符重载

作用：可以输出自定义数据类型

```c++
```

> 总结：重载左移运算符配合友元可以实现输出自定义数据类型

#### 4.5.3 递增运算符重载

作用： 通过重载递增运算符，实现自己的整型数据

```c++
```

> 总结： 前置递增返回引用，后置递增返回值

#### 4.5.4 赋值运算符重载

c++编译器至少给一个类添加4个函数

1. 默认构造函数(无参，函数体为空)
2. 默认析构函数(无参，函数体为空)
3. 默认拷贝构造函数，对属性进行值拷贝
4. 赋值运算符 operator=, 对属性进行值拷贝

如果类中有属性指向堆区，做赋值操作时也会出现深浅拷贝问题

**示例：**

```c++
```

#### 4.5.5 关系运算符重载

**作用：**重载关系运算符，可以让两个自定义类型对象进行对比操作

**示例：**

```c++
```

#### 4.5.6 函数调用运算符重载

* 函数调用运算符 ()  也可以重载
* 由于重载后使用的方式非常像函数的调用，因此称为仿函数
* 仿函数没有固定写法，非常灵活

**示例：**

```c++
```

### 4.6  继承

**继承是面向对象三大特性之一**

有些类与类之间存在特殊的关系，例如下图中：

![1544861202252](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151801053.webp)

我们发现，定义这些类时，下级别的成员除了拥有上一级的共性，还有自己的特性。

这个时候我们就可以考虑利用继承的技术，减少重复代码

#### 4.6.1 继承的基本语法

例如我们看到很多网站中，都有公共的头部，公共的底部，甚至公共的左侧列表，只有中心内容不同

接下来我们分别利用普通写法和继承的写法来实现网页中的内容，看一下继承存在的意义以及好处

**普通实现：**

```c++
```

**继承实现：**

```c++
```

**总结：**

继承的好处：==可以减少重复的代码==

class A : public B; 

A 类称为子类 或 派生类

B 类称为父类 或 基类

**派生类中的成员，包含两大部分**：

一类是从基类继承过来的，一类是自己增加的成员。

从基类继承过过来的表现其共性，而新增的成员体现了其个性。

#### 4.6.2 继承方式

继承的语法：`class 子类 : 继承方式  父类`

**继承方式一共有三种：**

* 公共继承
* 保护继承
* 私有继承

![clip_image002](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151810606.webp)

**示例：**

```c++
```

#### 4.6.3 继承中的对象模型

**问题：**从父类继承过来的成员，哪些属于子类对象中？

**示例：**

```c++
```

利用工具查看：



打开工具窗口后，定位到当前CPP文件的盘符

然后输入： cl /d1 reportSingleClassLayout查看的类名   所属文件名

效果如下图：

> 结论： 父类中私有成员也是被子类继承下去了，只是由编译器给隐藏后访问不到

#### 4.6.4 继承中构造和析构顺序

子类继承父类后，当创建子类对象，也会调用父类的构造函数

问题：父类和子类的构造和析构顺序是谁先谁后？

**示例：**

```c++
```

> 总结：继承中 先调用父类构造函数，再调用子类构造函数，析构顺序与构造相反

#### 4.6.5 继承同名成员处理方式

问题：当子类与父类出现同名的成员，如何通过子类对象，访问到子类或父类中同名的数据呢？

* 访问子类同名成员   直接访问即可
* 访问父类同名成员   需要加作用域

**示例：**

```c++
```

总结：

1. 子类对象可以直接访问到子类中同名成员
2. 子类对象加作用域可以访问到父类同名成员
3. 当子类与父类拥有同名的成员函数，子类会隐藏父类中同名成员函数，加作用域可以访问到父类中同名函数

#### 4.6.6 继承同名静态成员处理方式

问题：继承中同名的静态成员在子类对象上如何进行访问？

静态成员和非静态成员出现同名，处理方式一致

- 访问子类同名成员   直接访问即可
- 访问父类同名成员   需要加作用域

**示例：**

```c++
```

> 总结：同名静态成员处理方式和非静态处理方式一样，只不过有两种访问的方式（通过对象 和 通过类名）

#### 4.6.7 多继承语法

C++允许**一个类继承多个类**

语法：` class 子类 ：继承方式 父类1 ， 继承方式 父类2...`

多继承可能会引发父类中有同名成员出现，需要加作用域区分

**C++实际开发中不建议用多继承**

**示例：**

```c++
```

> 总结： 多继承中如果父类中出现了同名情况，子类使用时候要加作用域

#### 4.6.8 菱形继承



**菱形继承概念：**

​	两个派生类继承同一个基类

​	又有某个类同时继承者两个派生类

​	这种继承被称为菱形继承，或者钻石继承

**典型的菱形继承案例：**

![clip_image002](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304151823210.webp)

**菱形继承问题：**

1.     羊继承了动物的数据，驼同样继承了动物的数据，当草泥马使用数据时，就会产生二义性。

2.     草泥马继承自动物的数据继承了两份，其实我们应该清楚，这份数据我们只需要一份就可以。

**示例：**

```c++
```

总结：

* 菱形继承带来的主要问题是子类继承两份相同的数据，导致资源浪费以及毫无意义
* 利用虚继承可以解决菱形继承问题

### 4.7  多态

#### 4.7.1 多态的基本概念

**多态是C++面向对象三大特性之一**

多态分为两类

* 静态多态: 函数重载 和 运算符重载属于静态多态，复用函数名
* 动态多态: 派生类和虚函数实现运行时多态

静态多态和动态多态区别：

* 静态多态的函数地址早绑定  -  编译阶段确定函数地址
* 动态多态的函数地址晚绑定  -  运行阶段确定函数地址

下面通过案例进行讲解多态

```c++
```

总结：

多态满足条件

* 有继承关系
* 子类重写父类中的虚函数

多态使用条件

* 父类指针或引用指向子类对象

重写：函数返回值类型  函数名 参数列表 完全一致称为重写

#### 4.7.2 多态案例一-计算器类

案例描述：

分别利用普通写法和多态技术，设计实现两个操作数进行运算的计算器类

多态的优点：

* 代码组织结构清晰
* 可读性强
* 利于前期和后期的扩展以及维护

**示例：**

```c++
```

> 总结：C++开发提倡利用多态设计程序架构，因为多态优点很多

#### 4.7.3 纯虚函数和抽象类

在多态中，通常父类中虚函数的实现是毫无意义的，主要都是调用子类重写的内容

因此可以将虚函数改为**纯虚函数**

纯虚函数语法：`virtual 返回值类型 函数名 （参数列表）= 0 ;`

当类中有了纯虚函数，这个类也称为==抽象类==

**抽象类特点**：

 * 无法实例化对象
 * 子类必须重写抽象类中的纯虚函数，否则也属于抽象类

**示例：**

```c++
```

#### 4.7.4 多态案例二-制作饮品

**案例描述：**

制作饮品的大致流程为：煮水 -  冲泡 - 倒入杯中 - 加入辅料

利用多态技术实现本案例，提供抽象制作饮品基类，提供子类制作咖啡和茶叶

**示例：**

```c++
```

#### 4.7.5 虚析构和纯虚析构

多态使用时，如果子类中有属性开辟到堆区，那么父类指针在释放时无法调用到子类的析构代码

解决方式：将父类中的析构函数改为**虚析构**或者**纯虚析构**

虚析构和纯虚析构共性：

* 可以解决父类指针释放子类对象
* 都需要有具体的函数实现

虚析构和纯虚析构区别：

* 如果是纯虚析构，该类属于抽象类，无法实例化对象

虚析构语法：

`virtual ~类名(){}`

纯虚析构语法：

` virtual ~类名() = 0;`

`类名::~类名(){}`

**示例：**

```c++
```

总结：

​	1. 虚析构或纯虚析构就是用来解决通过父类指针释放子类对象

​	2. 如果子类中没有堆区数据，可以不写为虚析构或纯虚析构

​	3. 拥有纯虚析构函数的类也属于抽象类

#### 4.7.6 多态案例三-电脑组装

**案例描述：**

电脑主要组成部件为 CPU（用于计算），显卡（用于显示），内存条（用于存储）

将每个零件封装出抽象基类，并且提供不同的厂商生产不同的零件，例如Intel厂商和Lenovo厂商

创建电脑类提供让电脑工作的函数，并且调用每个零件工作的接口

测试时组装三台不同的电脑进行工作

**示例：**

```c++
```

## 5 文件操作

程序运行时产生的数据都属于临时数据，程序一旦运行结束都会被释放

通过**文件可以将数据持久化**

C++中对文件操作需要包含头文件 ==&lt; fstream &gt;==

文件类型分为两种：

1. **文本文件**     -  文件以文本的**ASCII码**形式存储在计算机中
2. **二进制文件** -  文件以文本的**二进制**形式存储在计算机中，用户一般不能直接读懂它们

操作文件的三大类:

1. ofstream：写操作
2. ifstream： 读操作
3. fstream ： 读写操作

### 5.1文本文件

#### 5.1.1写文件

   写文件步骤如下：

1. 包含头文件   

   \#include <fstream\>

2. 创建流对象  

   ofstream ofs;

3. 打开文件

   ofs.open("文件路径",打开方式);

4. 写数据

   ofs << "写入的数据";

5. 关闭文件

   ofs.close();


文件打开方式：

| 打开方式    | 解释                       |
| ----------- | -------------------------- |
| ios::in     | 为读文件而打开文件         |
| ios::out    | 为写文件而打开文件         |
| ios::ate    | 初始位置：文件尾           |
| ios::app    | 追加方式写文件             |
| ios::trunc  | 如果文件存在先删除，再创建 |
| ios::binary | 二进制方式                 |

**注意：** 文件打开方式可以配合使用，利用|操作符

**例如：**用二进制方式写文件 `ios::binary |  ios:: out`

**示例：**

```c++
```

总结：

* 文件操作必须包含头文件 fstream
* 读文件可以利用 ofstream  ，或者fstream类
* 打开文件时候需要指定操作文件的路径，以及打开方式
* 利用<<可以向文件中写数据
* 操作完毕，要关闭文件

#### 5.1.2读文件

读文件与写文件步骤相似，但是读取方式相对于比较多

读文件步骤如下：

1. 包含头文件   

   \#include <fstream\>

2. 创建流对象  

   ifstream ifs;

3. 打开文件并判断文件是否打开成功

   ifs.open("文件路径",打开方式);

4. 读数据

   四种方式读取

5. 关闭文件

   ifs.close();

**示例：**

```c++
```

总结：

- 读文件可以利用 ifstream  ，或者fstream类
- 利用is_open函数可以判断文件是否打开成功
- close 关闭文件 

### 5.2 二进制文件

以二进制的方式对文件进行读写操作

打开方式要指定为 ==ios::binary==

#### 5.2.1 写文件

二进制方式写文件主要利用流对象调用成员函数write

函数原型 ：`ostream& write(const char * buffer,int len);`

参数解释：字符指针buffer指向内存中一段存储空间。len是读写的字节数

**示例：**

```c++
```

总结：

* 文件输出流对象 可以通过write函数，以二进制方式写数据

#### 5.2.2 读文件

二进制方式读文件主要利用流对象调用成员函数read

函数原型：`istream& read(char *buffer,int len);`

参数解释：字符指针buffer指向内存中一段存储空间。len是读写的字节数

示例：

```c++
```

> 文件输入流对象 可以通过read函数，以二进制方式读数据
