+++
date = '2024-02-08T10:49:58+08:00'
draft = false
title = 'Excel学习'
slug = "1o1lhzmfgj"
description = "Excel学习"
categories = ["📒成长"]
tags = ["Excel"]
summary = "Excel学习"
+++
说明：根据王佩丰Excel24讲学习所作（部分）

# 一、认识

### 1.快速到达数据底部或者顶部

​		鼠标点击任意单元格，箭头放在上边框或者下边框，点击两次

### 2.冻结窗口

​		滚动工作表其余部分时，保持首行/首列不变

### 3.填充柄

​		选择任意一个或多个填充过的单元格，鼠标点击右下角，（鼠标点击不放）进行下拉；鼠标右击下拉可进行以年/月/工作日填充（日期为例）

​		自定义填充

![image-20230624102748660](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155315759.webp)

# 二、设置单元格格式

### 1.使用单元格格式工具美化表格

鼠标右击——设置单元格格式

或者点击

![image-20230624104325369](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155324073.webp)

弹出

![image-20230624104354060](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155330178.webp)

### 2.单元格数字格式

| 类型     | 原格式    | 转变后的格式       |
| -------- | --------- | ------------------ |
| 数值     | -25636    | -25,636            |
| 货币     | 10000     | ¥10,000.00         |
| 会计专用 | 1555      | $     1,555.00     |
| 日期     | 39814     | 2009年1月1日       |
| 时间     | 0.6980556 | 下午4时45分12秒    |
| 百分比   | 0.11      | 11.00%             |
| 分数     | 0.1       | 0                  |
| 科学计数 | 1.2E+14   | 1.2E+14            |
| 文本     | 2422      | 2422               |
| 特殊     | 25368     | 二万五千三百六十八 |

数字格式-自定义格式

![image-20230624112731204](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155340215.webp)

![image-20230624112803262](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155343235.webp)

![image-20230624112818936](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155346438.webp)

数据分列

【数据】-【分列】

![image-20230624113959551](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155348911.webp)

# 三、查找替换定位

### 替换

替换颜色

![image-20230628182649495](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155354685.webp)

匹配字体替换

输入要查找的要素，结合通配符“*和?”，可以实现高级模糊查找。在Excel的查找和替换中使用`星号“*”`可查找任意字符串，例如 查找`“IT* ”`可找到“IT主站”和“IT论坛”等。使用`问号`可查找任意单个字符。例如查找“?23” 可找到“023”和“423”等

![image-20230628183959966](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155410763.webp)

注意：结合单元格匹配使用

![image-20230628184249531](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155413340.webp)

当替换项含有*号时，可在前加入~使其不生效

### 定位

通过名称框定位单元格及区域位置

![image-20230628184745229](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155416376.webp)

定义名称

![image-20230628184956954](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155421522.webp)

修改批注图形形状

1. 空白区域插入形状，并在菜单形状右击【添加到快速访问工具栏】
2. 选中批注图形，在左上角【快速访问工具栏】中更改形状

通过定位自动填充单元格

1. 选中所有区域，用于定位条件筛选
2. 按键`= 和 ⬆`
3. 按住Alt，然后回车

# 四、排序与选择

### 多条件排序

![image-20230628220826149](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155430365.webp)

按颜色排序

自定义排序

![image-20230628221429779](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155428190.webp)

利用排序插入行

![image-20230628222457091](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155435634.webp)

![image-20230628222442909](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155433705.webp)

### 筛选



**一般筛选**

选中首行，点击筛选

==注意：==若要复制筛选后的数据，还需选择定位条件当中的可见单元格

**条件筛选**

从众多科目中筛选不重复的科目

![image-20230701131205057](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155439585.webp)

筛选部门是一车间且科目是邮寄费的发生额？

![image-20230701131754420](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155441869.webp)

筛选部门是一车间或科目是邮寄费的发生额？

![image-20230701132004710](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155456749.webp)

筛选出一车间或大于3000的二车间或发生额大于10000的数据？

![image-20230701133007053](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155444263.webp)

# 五、分类汇总与数据有效性

### 分类汇总工具

使用分类汇总前先排序

![image-20230709231547981](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155451039.webp)

**分地区与产品分类统计数量、金额、成本的总计**

1. 先排序（所属区域基础上再给产品类别排序），然后选中所属区域分类汇总

   ![image-20230710090823478](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155447415.webp)

   ![image-20230710090857745](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155509159.webp)

2. 再选中产品类别分类汇总

![image-20230710090929209](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155512395.webp)

**使用分类汇总批量合并内容相同的单元格**

1. 将所属区域进行排序

2. 将所属区域进行分类汇总，汇总方式为计数

   ![image-20230710092615557](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155518504.webp)

3. 选中该列单元格，定位到空值，并合并后居中

   ![image-20230710092657112](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155527091.webp)

4. 取消分类汇总

5. 选择格式刷

   ![image-20230710092856909](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155523530.webp)

### 数据有效性

【数据】【数据验证】

选中该列，点击数据验证，设置相应条件

![image-20230710093803025](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155535754.webp)

![image-20230710093916341](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155541635.webp)

![image-20230710093932431](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160317562.webp)

# 六、数据透视表

【插入】【数据透视表】

数据透视表选项——>显示——>勾选经典数据透视表布局

![image-20230711164838158](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155548427.webp)

**数据透视表中的结合**

![image-20230711165735038](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155551064.webp)

![image-20230711174532118](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155555078.webp)

**批量创建工作表**

1. 选择任意字段，插入数据透视表
2. 将该字段放到报表筛选字段处
3. 再将该字段拖至值字段处
4. 点击数据透视表选项，显示报表筛选页
5. 然后利用Shift选中所有新创的工作表将原有内容替换掉空白

# 七、认识函数与公式

| 1、运算符           |                              |
| ------------------- | ---------------------------- |
|                     | 算术运算符+ - * / % & ^      |
|                     | 比较运算符= > < >=  <= <>    |
|                     |                              |
| 2、公式中的比较判断 |                              |
|                     | 比较运算符的结果：TRUE FALSE |
| 3、运算符优先级     |                              |
| －                  | 负号                         |
| %                   | 百分比                       |
| ^                   | 求幂                         |
| * /                 | 乘和除                       |
| + -                 | 加和减                       |
| &                   | 文本连接                     |
| =,<,>,<=,>=,<>      | 比较                         |
| 4、单元格引用       |                              |
|                     | 相对引用：A1                 |
|                     | 绝对引用：$A$1               |
|                     | 混合引用：$A1  A$1           |

![image-20230712104925191](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155600545.webp)

函数求和，可结合定位工具实现跳跃自动选区求和

使用公式时，不方便拖拽时，可使用定位，再Ctr + 回车 批量填充

# 八、IF函数逻辑判断

**IF函数的基本用法**

函数语法：IF(logical_test,[value_if_true],[value_if_false])

### IF函数

![image-20230712230644880](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155603298.webp)

![image-20230712230704694](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160331583.webp)

![image-20230712231836178](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155606203.webp)

### VLOOKUP函数

![image-20230713000302611](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155608319.webp)

### ISERROR函数

![image-20230713000559002](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155610400.webp)

### AND 和 OR

![image-20230713001759173](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155612925.webp)

![image-20230713001818318](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155616471.webp)

# 九、COUNTIF函数

![image-20260221223545252](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160343802.webp)

![image-20230713100846992](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155619476.webp)

![image-20230713101335361](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155625265.webp)

![image-20230713101945282](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155623082.webp)

Countif函数超过15位字符时的错误，则在后面加上*

![image-20230713103637505](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160355918.webp)

背景填充：【条件格式】【新建规则】

![image-20230713103851708](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155628811.webp)

![image-20230713104848503](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155631328.webp)

![image-20230713105053074](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155634067.webp)

![image-20230713105603242](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155636417.webp)

![image-20230713105648294](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155639274.webp)

# 十、SUMIF函数

![image-20260221223206076](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155644865.webp)

![image-20230715152541965](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155643159.webp)

![image-20230715152736571](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160413103.webp)

Sumif函数超过15位字符时的错误

![image-20230715153212631](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155648200.webp)

关于第三参数简写时的注意事项

![image-20230715154232254](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160426082.webp)

**sumifs**

![image-20260221223421668](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155654093.webp)

![image-20230715155936493](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155656432.webp)

**替代vlookup**

![image-20230715160354761](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160435157.webp)

数据有效性

![image-20230715161733520](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155658832.webp)

# 十一、Vlookup函数

### Vlookup函数语法

VLOOKUP(lookup_value,table_array,col_index_num,[range_lookup])

![image-20260221222557968](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155701465.webp)

### 基本应用

![image-20230726220559973](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155703889.webp)

通配符查找

![image-20230726221308938](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155706425.webp)

近似匹配

![image-20230726223106976](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155710196.webp)

### 数字格式问题

![image-20230726223916267](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155712930.webp)

![image-20230726224032044](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155715624.webp)

![image-20230726224630107](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160452222.webp)

# 十二、Match和Index函数

MATCH(lookup_value,lookup_array,[match_type])

INDEX(array,row_num,[column_num])

![image-20230727225819835](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155719065.webp)

### 单元格引用原理

![image-20230727230415744](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160459737.webp)

### 返回多列结果

![image-20230727231343795](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155722365.webp)

![image-20230727233054945](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155727610.webp)

# 十三、邮件合并

在工作当中，需要生成多个文档或者电子邮件之类的，大体内容保持不变，关键信息变动，使用邮件合并更加方便达到要求。

### 批量生成多个文档

【邮件】【开始邮件合并】【邮件合并分步向导】

在完成合并步骤前可选择完成完成合并并编辑单个文档

### 利用word发送邮件

同理

### 每页显示多条记录

第一步，选择目录

![image-20230802232037340](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155735563.webp)

效果如图

![image-20230802232055378](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155732992.webp)

### 邮件合并后的数字格式处理

数字格式	`\# "#,##0"`

日期格式	`\@ "M/d/yyyy"`	（m需要大写）

![image-20230802232605125](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160513777.webp)

按下Alt + F9键

![image-20230802232626226](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155741956.webp)

修改后

![image-20230802233126402](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155743503.webp)

# 十四、日期函数

![image-20231010165141563](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160521220.webp)

![image-20231010205004001](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155746908.webp)

![image-20231010205656982](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160528188.webp)

![image-20231010205953232](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155750790.webp)

![image-20231010210326308](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155753048.webp)

![image-20231010210642269](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160539562.webp)

![image-20231010211646869](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155755067.webp)

![image-20231010212410760](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155801208.webp)

![image-20231010212615445](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155757731.webp)

# 十五、简单文本函数

## 截取字符串

![image-20231021114017610](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155803937.webp)

![image-20231021114027774](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155805920.webp)

![image-20231021114045754](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155808491.webp)

![image-20231021115554721](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306155811090.webp)

## 获取文本中的信息

![image-20231021120151267](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160116076.webp)

![image-20231021120209655](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160118366.webp)

![image-20231021131602857](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160120643.webp)

## 身份证

![image-20231021132301879](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160123830.webp)

![image-20231021132602903](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160600732.webp)

# 十六、数学函数

round、roundup、rounddown、int、mod（求余数）、row、column

![image-20231021224203249](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160126841.webp)

![image-20231021224620620](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160129320.webp)

![image-20231021225431674](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160132113.webp)

![image-20231021225921141](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160135256.webp)

![image-20231021230228647](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160137672.webp)

# 十七、vlookup函数与数组

回顾sumif、sumifs函数

![image-20231022145630574](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160140235.webp)

# 十八、indirect函数

![image-20231022150926284](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160143158.webp)

![image-20231022151303522](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160145375.webp)

跨表引用

![image-20231022151801438](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160148872.webp)

顺序不同如何处理

![image-20231022152407962](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160151222.webp)

混合引用

![image-20231022153234769](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160153656.webp)

![image-20231022154114497](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160156132.webp)

根据省份确定城市

![image-20231022154526715](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160158450.webp)

![image-20231022154626546](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160201377.webp)

![image-20231022155144599](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160204085.webp)

# 十九、动态图表

先在空白处写出公式

=OFFSET($B$1,COUNTA($B:$B)-10,0,10,1)

然后添加到名称

![image-20231022210458382](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160209015.webp)

最后插入图表

![image-20231022210616664](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160211185.webp)

案例2

1. 开发工具 插入滚动条

2. 设置最小值和单元格链接

3. 写出公式=OFFSET($B$1,$F$2,0,$F$4,1)

4. 公式下定义名称

   ![image-20231022212411021](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260306160215809.webp)

5. 插入图表，选择数据，添加

   系类名称：成交量

   系列值：=Sheet1!成交量

6. 滑动滚动条

# 二十、实用技巧

1.数字自动占位补全，【设置单元格格式】自定义 类型填写000

2.选定区域内容后面批量加下划线，【设置单元格格式】自定义 类型填写@*_

3.该列前面输入多个内容，alt+下键  可显示前面输入过的内容，可直接选择

4.批量创建文件夹，="md "&文件夹名字  创建文本文档，将函数结果复制到文本中，另存为bat文件，ANSI编码

5.防看错行列的聚光灯效果 创建公式 =OR(cell("row")=ROW(),cell("col")=COLUMN())

6.复制自定义格式单元格 =text(A1,"000")

7.批量插入图片  选中所有图片，插入excel，格式 调整大小 光标定位到任意单元格 拖动最后一张照片 到合适位置 ctrl + g 定位条件 【对象】 对齐对象 纵向分布 左对齐

8.单元格带单位求和 先去掉元求和 再【设置单元格格式】自定义 类型 原来基础上加元字

9.批量对文件重命名 先在excel中对原名处理 ="前缀"&A1  然后="ren 原名列 新名列"，将函数结果复制到文本中，另存为bat文件，ANSI编码

10.通过地址插入图片 ="<table><img src"""地址"""width=""101""height=""122>" 右键选择性粘贴 unicode文本

11.一键批量提取工作表名称 新建名称 =getworkbook(1)  然后 = index(名字,row(A1))

12.批量套打小数位很长  Alt + F9 放在结果后面大括号前面 ==\\#"0.00"== Alt + F9再切换回来

13.筛选数据自动重新编号 =subtotal(3,$C$6:C6)*1

14.  文件夹中提取文件名 bat脚本 dir * . * /b>提取文件名.txt
15.  设置数字以万元显示  设置单元格格式 自定义 0!.0,万
16.  多表透视表 Alt + D + P 三个分开按
17.  excel证件照换背景 【格式】删除背景，更改填充色
18.  











