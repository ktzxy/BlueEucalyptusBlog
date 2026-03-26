+++
date = '2024-04-02T10:31:35+08:00'
draft = false
title = 'Kettle精讲'
slug = "nk1fwk43b9"
description = "Kettle精讲"
categories = ["📒成长"]
tags = ["kettle"]
summary = "Kettle精讲"
+++
# **Kettle精讲**

## 一、kettle简介

### 1. kettle的发展史

Kettle最早是一个开源的ETL工具，全称为KDE Extraction, Transportation, Transformation and Loading Environment。KDE源于最开始的计划是在K Desktop Environment(www.kde.org)上开发这个软件，但这个计划被取消。在2006年，Pentaho公司收购了Kettle项目，原Kettle项目发起人Matt Casters加入了Pentaho团队，成为Pentaho套件数据集成架构师，从此，Kettle成为企业级数据集成及商业智能套件Pentaho的主要组成部分，**Kettle亦重命名为Pentaho Data Integration（PDI）**。Pentaho公司于2015年被Hitachi（日立） Data Systems收购，Hitachi Data Systems于2017年改名为Hitachi Vantara。

Pentaho Data Integration以Java开发，支持跨平台运行，可以在Window、Linux、Unix上运行，绿色无需安装，数据抽取高效稳定。Pentaho Data Integration分为商业版与开源版，开源版的截止2021年1月的累计下载量达836万，其中19%来自中国。在中国，一般人仍习惯把Pentaho Data Integration（PDI）的开源版称为Kettle。

### 2. kettle 与ETL

ETL（Extract-Transform-Load的缩写，即数据抽取、转换、装载的过程）

在各个企业中，对数据的处理几乎成为其数字化发展的必要流程，而数据的处理，无外乎抽取、统计分析、转换、装载，因此，各个企业目前都需要ETL工程师来完成数据的处理工作。

kettle是一个ETL工具，允许管理来自不同异构数据源的数据，并在基于**图形化**的工具中，完成对ETL的操作。

### 3. kettle的架构

![PixPin_2025-12-16_09-00-27](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512160901530.webp)

**Transformation**：转换

- 在大部分场景下，可以直接称之为“数据流”
- 它可以完成对数据的【输入】-->【处理】-->【输出】
- 一旦启动一个转换任务，则其中的所有组件会同时启动，并根据配置，逐条处理数据

**Job**：作业

- 作业可以称之为步骤流或者控制流
- 在作业中，可以挂载（调用）转换任务，也可以挂载（调用）job任务
- 作业中的各个组件，按照顺序执行，可以对执行的结果进行判断并处理分支
- 作业可以检测数据表、文件是否存在，执行Shell脚本，执行SQL脚本，获取数据，发送邮件等

**核心组件：**

| 组件    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| spoon   | 【勺子】是kettle的图形化工具，可以通过简单的拖拉拽方式完成kettle任务的设计、运行与调试，为kettle最常用的组件。 |
| Pan     | 【煎锅】Transformation执行器(命令行方式)，Pan用于在终端执行Transformation，没有图形界面 |
| Kitchen | 【厨房】Job执行器(命令行方式)，Kitchen用于在终端执行Job，没有图形界面。 |
| Carte   | 嵌入式Web服务，用于远程执行Job或Transformation，Kettle通过Carte建立集群 |

### 4. kettle的特点

1. **免费**开源：基于java的免费开源的软件，对商业用户也没有限制，可以在任何的公司中使用。
2. 容易配置：可以在Window、Linux、Unix上运行，绿色无需安装，数据抽取高效稳定。
3. **兼容**各种数据源：ETL工具集，它允许你管理来自不同数据库的数据。
4. **简单**开发：通过图形界面设计与开发任务，无需写代码实现。

## 二、kettle的安装

安装要求：

- 安装所在的服务器或者Windows中，需要jdk1.8

在获取到压缩包之后，将压缩包解压至无中文路径下即可，注意，是整体路径中，任何一级目录中都不包含中文

解压后的目录结构如下：

![image-20251216092113530](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512160921600.webp)

![image-20251216092353419](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512160923506.webp)

![image-20251216092538808](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512160925879.webp)

## 三、kettle的初体验

需求：将一个csv文件中的数据内容输出到Excel文件中

### 1. 新增转换任务

新增一个转换任务的方式：

![image-20251216093857565](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512160938634.webp)

### 2. 配置csv输入组件（step）

![image-20251216094744305](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512160947378.webp)

![image-20251216095933710](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512160959794.webp)

==注意：内容包含中文，可在文件编码选择utf-8；根据字段内容选择合适的类型==

### 3. 配置Excel输出组件（step）

1）通过拖拽的方式将Excel输出放入编辑页面中

![image-20251216101525842](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161015981.webp)

2）将输入组件与输出组件连接到一起：

![image-20251216101726579](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161017639.webp)

3）双击Excel输出组件，对内容进行配置

![image-20251216153930619](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161539707.webp)

### 4. 创建作业（job）

1）双击【主对象树】中的作业或点击【文件】-【新建】-【作业】

2）每个任务由一个start组件开始

![image-20251216154947185](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161549250.webp)

### 5. 在作业中挂载转换任务

配置转换任务与结束节点

![image-20251216155200727](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161552789.webp)

![image-20251216155318668](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161553743.webp)

### 测试运行

保存任务并执行

## 四、kettle名词解释

### 1. 转换

转换（transformation）是ETL解决方案中最主要的部分，它处理抽取、转换、加载各阶段各种对数据的操作。转换包含一个或多个“Step-步骤”，例如读取文件，过滤数据，数据加载等操作都是步骤。转换里的步骤通过“Hop-跳”来连接，跳定义了一个单向通道，允许数据从一个步骤向另一个步骤流动。此外，转换中的每个步骤还可以注释，目的主要是使转换文档化。

每个“Transformation-转换”对应的保存文件名称为“xx.ktr”

### 2. Step-步骤

Step是转换里的基本组成部分。

“CSV文件输入”和“Excel输出”显示了两个Step步骤。

每个Step都有唯一的一个名字，一个Step可以有多个输出跳，一个“步骤”的数据有多个输出跳时可以设置数据“分发”或者“复制”，“分发”是目标“步骤”轮流接收数据，“复制”是所有的记录被同时发送到所有的目标“步骤”。

==注意：分发会导致一份文件中的内容被发送到不通的文件中，输出到两个文件中的内容不同。==

### 3. Hop-跳

“Hop-跳”就是步骤之间带箭头的连线，跳定义了步骤之间的数据通路。在转换中“跳”不能循环，因为每个步骤都依赖前一个步骤获取字段值。所以转换任务是一个DAG（有向无环图）

“Hop-跳”实际上是两个“Step-步骤”之间的记录行的缓存，缓存数据量可以在转换配置中设置（双击配置页面空白处），当缓存记录数满了，写数据的步骤停止写入，此时输出步骤不会停止，持续的读取缓存数据，直到缓存中有空间，写数据步骤继续运行。当缓存清空后，读取数据的步骤停止读取数据，直到缓存中又有数据。

当单击“跳”时，连线变灰色，代表不使用，再次单击变蓝代表启用。

### 4. 并行（转换）

当“Transformation-转换”启动后，所有“Step-步骤”都同时启动，这些“Step-步骤”都是并发方式运行，**各自从对应的输入跳中读取数据**，并把处理过的数据写到输出跳，直到输入跳里不再有数据，就终止步骤的运行，当所有的步骤都终止了，整个转换就停止了。

“Transformation-转换”里的步骤几乎是同时启动的，所以不可能判断出哪个步骤是第一个启动的步骤。如果想要一个任务沿着指定的顺序执行，那么就要使用“Job-作业”。

### 5. 数据类型

数据以数据行的形式沿着步骤移动。一个数据行是零到多个字段的集合，字段包括下面几种数据类型：

- String： 字符类型
- Number： 双精度浮点数（3.14）
- Integer： 带符号长整型
- BigNumber： 任意精度数值（3.141592653）
- Date： 带毫秒精度的日期时间值
- Boolean： 取值为true和false的布尔值
- Binary： 二进制字段可以包括图形、声音、视频及其他类型的二进制数据

### 6. Job-作业

 大多数ETL项目都需要完成各种各样的操作，而且这些操作要按照一定顺序完成。因为转换以并行方式执行，就需要一个可以串行执行的作业来处理这些操作。

作业是步骤流，转换是数据流，这是作业和转换的最大区别。

 作业的每个步骤必须等到前面的步骤都跑完了，后面的步骤才会执行

 而转换会一次性把所有的控件全部先启动（一个控件对应启动一个线程）然后数据流会从第一个控件开始，一条记录，一条记录的流向最后的控件。

 一个作业包括一个或多个作业项，这些作业项以某种顺序来执行。作业执行顺序由作业项之间的跳和每个作业项的执行结果来决定。 可以单机作业跳来改变作业跳的状态，有三种状态（锁-必须执行、对号-执行成功时、错号-执行失败时，这三种状态可以通过单击连线进行切换）。

![image-20251216162803048](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161628172.webp)

上图中的“Start”是“job-作业”的起点，一个作业只能定义一个“Start”。上图中每个“转换”就是作业的作业项，作业项是作业的基本构成部分。默认情况下作业中作业项都是以串行的方式制定，只是在特殊的情况下以并行方式执行。

当作业中有多条路径时，会采用回溯算法来执行作业项，如下图：

![image-20251216162839298](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161628387.webp)

回溯算法就是：假设扫行到了图里的一条路径的某个节点时，要依次扫行这个节点的所有子路径，直到没有再可以执行的子路径，就返回该节点的上一节点，再反复这个过程。

上图中的三个作业的执行顺序如下：

首先“开始”作业项搜索所有下一个节点作业项，找到了“A”和“C”

1. 执行“A”
2. 搜索“A”后面的作业项，发现了“B”
3. 执行“B”
4. 搜索“B”后面的作业项，没有找到任何作业项
5. 回到“A”，也没有发现其他作业项（需要被执行的作业项）
6. 回到Start，发现另一个要执行的作业项“C”
7. 执行“C”
8. 搜索“C”后面的作业项，没有找到任何作业项
9. 回到Start，没有找到任何作业项
10. 作业结束。

以上执行过程就是Start->A->B->C,也有可能是Start->C->A->B。

作业除了以上串行执行外，还可以并行执行：

![PixPin_2025-12-16_16-29-49](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161630942.webp)

每个“Job-作业”对应的保存文件名称为“xx.kjb”。

## 五、转换核心对象

### 1. 输入

#### 1) CSV文件输入

![image-20251216164716883](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161647994.webp)

#### 2) Excel输入

![image-20251216165559970](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161656075.webp)

注意：kettle不支持读取使用Excel2007创建的Excel2003文件；03是xls，07以后是xlsx

#### 3) 文本文件输入

![image-20251216172626607](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202512161726708.webp)

#### 4）生成记录

![image-20260218214304746](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182143821.webp)

**补充：如何将任务放入到linux中执行**

- 首先将kettle的压缩包放入linux中，并通过unzip进行解压，如果没有指令可以通过yum install unzip 进行安装
- 解压指令：unzip 包名 -d /opt/installs
- 将配置好的任务传入linux中，建议在kettle的目录中新建一个job目录，将其放入
- vim 配置文件.ktr 将其中的输出与输入目录改为linux中的具体目录
- 在kettle的主目录中，执行：./pan.sh -file=./job/generate_input.ktr

#### 5) 表输入

Kettle支持抽取数据库表中的数据，以MySQL为例， 需要将mysql的驱动包放入Kettle解压目录“...\data-integration\lib”下，这里放入之后，需要重新启动Kettle。

![image-20260218214657739](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182146826.webp)

在新建的一个【转换】配置了mysql数据库的连接，但在其他的转换任务中，无法直接使用，需要将此数据库连接共享才可以完成所有转换任务的共同使用。

![image-20260218215005819](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182150877.webp)

### 2. 输出

#### 1) Excel输出/MicrosoftExcel输出

Excel输出”和“MicrosoftExcel输出”都是Excel输出，不同的是“Excel输出”写出支持“xls”格式，“MicrosoftExcel输出”支持“xls”和“xlsx”格式。

![image-20260218220203879](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182202964.webp)

![image-20260218220549938](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182205021.webp)

#### 2) SQL文件输出

在某些场景下，需要对数据库中的某个表进行备份或者迁移的动作，可以使用【SQL文件输出】步骤进行数据的处理。

![image-20260218221048220](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182210296.webp)

#### 3) 文本文件输出

文本文件输出可以将各种数据源中的数据生成为文本文件，大多数的使用场景是将数据放入linux中，供hive等数据库数据分析使用。

![image-20260218221500246](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182215327.webp)

![image-20260218221756673](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602182217747.webp)

#### 4) 表输出

kettle9.3连接mysql8.0x需要选项中加这三个参数，否则点击测试没有反应，mysql驱动用8.0.28，不能用localhost，要用127.0.0.1

| 参数 (Parameter)        | 值 (Value)    |
| :---------------------- | :------------ |
| useSSL                  | false         |
| allowPublicKeyRetrieval | true          |
| serverTimezone          | Asia/Shanghai |

![image-20260219174255113](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602191742184.webp)

![image-20260219175104047](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602191751139.webp)

```
注意：如果数据中存在中文，则需要在数据库配置处添加命名参数
```

命名参数：characterEncoding

值：utf8

![image-20260219164212494](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602191642596.webp)

### 3. 转换

#### 1) Concat fields



在输出的时候，记得设定合并后的字段的长度不能太短

#### 2) 值映射

当遇到需要将某个字段中的值，根据值的内容来进行数据映射的时候，使用【值映射】步骤

类似于 0 代表男 1 代表女，输出的时候不输出 0 或者1 ，而是输出 男或者女

要求：在值映射的组件中进行字段值的配置时，“源值”与“目标值”的字段类型需要一致。



#### 3) 增加序列

相当于 row_number() over()



#### 4) 字段选择

当需要对抽取的数据只获取其中的某几列字段的时候，可以使用字段选择step

【字段选择】step还可以对字段的名称进行更改，以及变更其数据类型





# Kettle调优

1、调整JVM大小进行性能优化，修改Kettle根目录下的Spoon脚本。

![img](https://fastly.jsdelivr.net/gh/ktzxy/blog-img@main/2026/202602191758224.webp) 

参数参考：

-Xmx2048m：设置JVM最大可用内存为2048M。

-Xms1024m：设置JVM促使内存为1024m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

-Xmn2g：设置年轻代大小为2G。整个JVM内存大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。

-Xss128k：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。

2、 调整提交（Commit）记录数大小进行优化，Kettle默认Commit数量为：1000，可以根据数据量大小来设置Commitsize：1000~50000

3、尽量使用数据库连接池；

4、尽量提高批处理的commit size；

5、尽量使用缓存，缓存尽量大一些（主要是文本文件和数据流）；

6、Kettle是Java做的，尽量用大一点的内存参数启动Kettle；

7、可以使用sql来做的一些操作尽量用sql；

Group , merge , stream lookup,split field这些操作都是比较慢的，想办法避免他们.，能用sql就用sql；

8、插入大量数据的时候尽量把索引删掉；

9、尽量避免使用update , delete操作，尤其是update,如果可以把update变成先delete,  后insert；

10、能使用truncate table的时候，就不要使用deleteall row这种类似sql合理的分区，如果删除操作是基于某一个分区的，就不要使用delete row这种方式（不管是deletesql还是delete步骤）,直接把分区drop掉，再重新创建；

11、尽量缩小输入的数据集的大小（增量更新也是为了这个目的）；

12、尽量使用数据库原生的方式装载文本文件(Oracle的sqlloader, mysql的bulk loader步骤)。
