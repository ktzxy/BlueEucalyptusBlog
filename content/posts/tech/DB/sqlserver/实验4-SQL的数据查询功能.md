+++
date = '2025-08-10T11:25:17+08:00'
draft = false
title = '实验4 SQL的数据查询功能'
slug = "9v82ah4b0i"
description = "实验4 SQL的数据查询功能"
categories = ["📒数据库"]
tags = ["sqlserver"]
summary = "实验4 SQL的数据查询功能"
series = ["数据库实验系列"]
showSeries= true  
+++

﻿### 1、使用SQL Server Management Studio管理平台，为supermarket数据库添加如教材P53页图3-4~图3-8所示的示例数据。
> 前面实验2已经加过，不再执行

### 2、完成教材上的例3-28~例3-69的操作。
##### 查询全体学生姓名、学号、专业
```sql
select SName,SNO,Major from Student
```
##### 查询全体学生的详细信息
```sql
select * from Student;
```
##### 查询全体学生的学号、姓名、年龄
```sql
select SNO,SName,YEAR(GETDATE())-BirthYear Age from Student
```
##### 查询购买了商品的学生学号
```sql
select distinct SNO from SaleBill
```
##### 查询管理信息系统专业学生名单
```sql
select * from Student where Major = 'MIS'
```
##### 查询年龄不大于20的学生名单
```sql
select * from Student where YEAR(GETDATE())-BirthYear !> 20;
```
##### 查询现存货量为3~10的商品信息
```sql
select * from Goods where Number between 3 and 10
```
##### 查询2017年生产的商品信息
```sql
select * from Goods where ProductTime between '2017-1-1' and '2017-12-31'
```
##### 查询姓名在“李明”和“闵红”之间的学生信息
```sql
select * from Student where SName between '李明' and '闵红'
```
##### 查询商品编号分别为GN0001、GN0002的销售信息
```sql
select * from SaleBill where GoodsNO in ('GN0001','GN0002')
```
##### 查询不是MIS专业的学生信息
```sql
select * from Student where Major != 'MIS'
```
##### 查询商品名称中包含“咖啡”的商品信息
```sql
select * from Goods where GoodsName like '%咖啡%'
```
##### 查询学生姓名第二个字为“民”的学生信息
```sql
select * from Student where SName like '_民%'
```
##### 查询商品编号最后一位不是1、4、7的商品信息
```sql
select * from Goods where GoodsNO not like '%[147]'

select * from Goods where GoodsNO like '%[^147]'
```
##### 查询AC专业的学生和MIS专业男生的信息
```sql
select * from Student where Major = 'AC' or Major = 'MIS' and Ssex = '男'
```
##### 查询学生信息，按出生年升序排列
```sql
select * from Student order by BirthYear
```
##### 查询商品名包含“咖啡”的商品的商品编号、商品名、现货存量和生产时间。按现货存量升序、生产日期降序排列。
```sql
select GoodsNO,GoodsName,Number,ProductTime from Goods 
where GoodsName like '%咖啡%' order by Number,ProductTime desc
```
##### 查询商品表的商品编号、商品名称、现货存量、生产日期、保质期剩余天数，按保质期剩余天数升序排列
```sql
select GoodsNO,GoodsName,Number,ProductTime,
QGPeriod * 30 - DATEDIFF(DAY,ProductTime,GETDATE()) days_remaining 
from Goods
order by days_remaining
```
##### 查询商品个数
```sql
select COUNT(*) 商品个数 from Goods
```
##### 查询售出商品种类
```sql
select COUNT(distinct GoodsNO) 商品种类 from SaleBill
```
##### 统计销售表中最多、最少和平均销售量
```sql
select MAX(Number) 最大销售量,MIN(Number) 最小销售量,AVG(Number) 平均销售量 from SaleBill
```
##### 统计每个学生购买的商品种类
```sql
select SNO,COUNT(*) 商品种类 from SaleBill group by SNO
```
##### 统计每个学生购买的商品种类，列出购买3种或3种以上商品学生的学号，购买商品种类
```sql
select SNO,COUNT(*) 商品种类 from SaleBill group by SNO having COUNT(*)>=3
```
##### 统计学生表中每年出生的男、女生人数，按出生年降序、人数升序排列。
```sql
select Student.BirthYear,Student.Ssex,COUNT(*) from Student
group by BirthYear,Ssex
order by BirthYear desc,count(*)
```
##### 查询学生购物情况
```sql
select * from Student S join SaleBill SA on S.SNO = SA.SNO
```
##### 查询MIS专业学生的购物情况
```sql
select * from Student S 
join SaleBill SA on S.SNO = SA.SNO
where Major = 'MIS'
```
##### 查询“CS”学校各学生的消费金额
```sql
select college,SName,sum(SA.Number * SalePrice) 消费金额
from Student S join SaleBill SA on S.SNO = SA.SNO join Goods G on SA.GoodsNO = G.GoodsNO
where college = 'CS'
group by college,SName
```
##### 查询与商品“麦氏威尔冰咖啡”同一类别的商品的商品编号、商品名
```sql
select G2.GoodsNO,G2.GoodsName 
from Goods G join Goods G2 on G.CategoryNO = G2.CategoryNO
where G.GoodsName = '麦氏威尔冰咖啡'
and G2.GoodsName != '麦氏威尔冰咖啡'
```
##### 查询没人购买的商品，列出商品名与现货存量
```sql
select GoodsName,G.Number 现货存量
from Goods G left join SaleBill GA on GA.GoodsNO = G.GoodsNO
where GA.SNO is null
```
##### 查询与商品“麦氏威尔冰咖啡”同一类别的商品的商品编号、商品名
```sql
select GoodsName from Goods 
where CategoryNO = (select CategoryNO from Goods
where GoodsName = '麦氏威尔冰咖啡') and GoodsName != '麦氏威尔冰咖啡';
```
##### 查询进价大于平均进价的商品名称和进价
```sql
select GoodsName,InPrice from Goods
where InPrice > (select avg(InPrice) from Goods)
```
##### 查询购买了“东菀市南城久润食品贸易部”经销的商品的学生学号和姓名。
```sql
select SNO,SName from Student where SNO
in(select distinct SNO from SaleBill where GoodsNO 
in(select GoodsNO from Goods where SupplierNO
= (select SupplierNO from Supplier 
where SupplierName = '东菀市南城久润食品贸易部')))
```
##### 查询超过同类商品平均进价的商品信息
```sql
select * from Goods
where InPrice > (select avg(InPrice) from Goods G where CategoryNO = Goods.CategoryNO)
```
##### 查询购买了商品的学生信息
```sql
select * from Student where exists
(select * from SaleBill where SNO = Student.SNO)
```
##### 查询至少购买了学生S02购买的全部商品的学生学号
```sql
select distinct SNO from SaleBill S1 where
S1.SNO != 'S02' and not exists
(select * from SaleBill S2 where S2.SNO = 'S02' and not exists
(select * from SaleBill S3 where S3.SNO = S1.SNO and S3.GoodsNO = S2.GoodsNO))
```
##### 查询MIS专业或出生年晚于1999年的学生信息
```sql
select * from Student where Major = 'MIS'
union 
select * from Student where BirthYear > 1999
```
##### 查询MIS专业，出生年晚于1991年的学生信息
```sql
select * from Student where Major = 'MIS'
intersect
select * from Student where BirthYear > 1991
```
##### 查询至少购买了学生S02购买的全部商品的学生学号
```sql
select distinct SNO from SaleBill
where SNO!='S02' and not exists
(select GoodsNO from SaleBill where SNO = 'S02'
except
select GoodsNO from SaleBill S where S.SNO = SaleBill.SNO)
```
##### 查询各类别商品的商品种类名和平均售价
```sql
select C.CategoryName,AVG_CA.AVGSALEPRICE
from Category C join
(select CategoryNO,avg(SalePrice) from Goods group by CategoryNO) 
as AVG_CA(CategoryNO,AVGSALEPRICE) on C.CategoryNO = AVG_CA.CategoryNO
```
##### 查询购买了GN0002商品的学生信息
```sql
select * from Student S join
(select SNO,GoodsNO from SaleBill where GoodsNO = 'GN0002') SA_SNO on S.SNO = SA_SNO.SNO
```
##### 查询销售额前三的商品与销售额
```sql
select top 3 G.GoodsNO,sum(SA.Number * G.SalePrice) GOODSUM
from Goods G join SaleBill SA
on SA.GoodsNO = SA.GoodsNO
group by G.GoodsNO
order by GOODSUM desc
```
##### 查询年龄最大的3名学生的信息
```sql
select top 3 with ties * from Student
order by BirthYear
```
### 3、在数据库supermarket上完成下列操作。

（1）查询商品种类信息。

```sql
select CategoryNO,CategoryName,Description from Category
```

（2）查询IT专业所有学生信息。

```sql
select * from Student where Major = 'IT'
```

（3）查询MIS专业年龄小于20岁的学生信息。并为MIS列取别名为“信息管理系统”。

```sql
select SNO,SName,BirthYear,Ssex,College,Major 信息管理系统,WeiXin 
from Student where YEAR(GETDATE()) - BirthYear <22 and Major = 'MIS'
```

（4）查询利润率大于30%的商品编号与商品名。

```sql
select GoodsNO,GoodsName,ROUND((SalePrice-InPrice)/InPrice,2) 利润率
from Goods where (SalePrice-InPrice)/InPrice > 0.3
```

（5）查询广州佛山供应的商品信息。

```sql
select * from Goods G join Supplier S on G.SupplierNO = S.SupplierNO
where S.Address = '广州佛山'
```

（6）查询购买了商品种类为咖啡的MIS专业的学生信息。

```sql
select * from Student where SNO in(
	select SNO from SaleBill where GoodsNO in(
		select GoodsNO 
		from Goods G join Category C 
		on G.CategoryNO = C.CategoryNO
		and CategoryName = '咖啡'
		)
) and Major = 'MIS'
```

（7）查询购买了商品种类为咖啡的各专业的学生人数。

```sql
select Major,count(Major) 人数 from Student where SNO in(
	select SNO from SaleBill where GoodsNO in(
		select GoodsNO 
		from Goods G join Category C 
		on G.CategoryNO = C.CategoryNO
		and CategoryName = '咖啡'
		)
) group by Major
go

--方法二
select Major, count(*) 人数
from (
	select distinct Student.*
	from Salebill,Student,Goods,Category
	where SaleBill.SNO=Student.SNO
	   and SaleBill.GoodsNO=Goods.GoodsNO
	   and Goods.CategoryNO=Category.CategoryNO
	   and CategoryName='咖啡'
) S
group by Major
go

--方法三
select Major,count(distinct(Student.SNO)) 人数
from Salebill,Student,Goods,Category
where SaleBill.SNO=Student.SNO
   and SaleBill.GoodsNO=Goods.GoodsNO
   and Goods.CategoryNO=Category.CategoryNO
   and CategoryName='咖啡'
group by Major
go
```

（8）查询购买各商品种类的各专业的学生人数。

```sql
select CategoryName,Major, count(*) 人数
from (
	select distinct Student.*,CategoryName
	from Salebill,Student,Goods,Category
	where SaleBill.SNO=Student.SNO
	   and SaleBill.GoodsNO=Goods.GoodsNO
	   and Goods.CategoryNO=Category.CategoryNO
) S
group by CategoryName,Major
order by CategoryName
go
```

（9）查询从未购买过商品的学生信息。

```sql
select * from Student where SNO not in(
	select distinct SNO from SaleBill where GoodsNO in(
		select GoodsNO 
		from Goods G join Category C 
		on G.CategoryNO = C.CategoryNO
		)
)
go


select * from student
except
select distinct student.*
from SaleBill,Student
where SaleBill.SNO=Student.SNO
go
```

（10）查询与商品编号GN0005相同产地的商品编号、商品名。

```sql
select GoodsNO,GoodsName from Goods where SupplierNO in (
	select SupplierNO from Supplier where Address =(
		select Address 
		from Supplier S join Goods G 
		on S.SupplierNO = G.SupplierNO 
		where GoodsNO = 'GN0005'
		)
)
go

select GoodsNo, GoodsName
from Goods,Supplier
where Goods.SupplierNO=Supplier.SupplierNO and Goods.GoodsNO!='GN0005' and address=(
	select address 
	from goods,supplier 
	where goods.GoodsNO='GN0005' and Goods.SupplierNO=Supplier.SupplierNO 
	)
go
```

（11） 使用派生表查询各供应商的存货量。

```sql
select SupplierName 供应商名称,S2.SUM_Number 存货量
from Supplier S
join (
	select SupplierNO,SUM(Number) SUM_Number 
	from Goods 
	group by SupplierNO) S2
on S.SupplierNO = S2.SupplierNO
go

select supplierName 供应商,存货量
from supplier, (
select Supplier.supplierNO,sum(goods.number) 存货量
from goods,supplier
where goods.SupplierNO=supplier.SupplierNO
group by supplier.SupplierNO) N
where supplier.SupplierNO=N.SupplierNO
go
```

（12） 查询售价大于该种类商品售价均值的商品号、商品名。

```sql
select GoodsNO,GoodsName
from Goods G join (
	select CategoryNO,ROUND(avg(SalePrice),2) avg_salePrice
	from Goods
	group by CategoryNO) G2
on G.CategoryNO = G2.CategoryNO
and SalePrice > avg_salePrice

go
select GoodsNO, GoodsName
from Goods G
where SalePrice>(select avg(SalePrice) from goods where CategoryNO=G.CategoryNO)
go
```

（13）分别用子查询与连接查询查询购买了商品编号为“GN0003”和“GN0007”的学生学号与姓名。

```sql
select SNO,SName from Student
where SNO in (
	select SNO 
	from SaleBill 
	where GoodsNO in('GN0003','GN0007')
	group by SNO
	having count(GoodsNO) = 2)

go

select S.SNO,SName from Student S
right join (
	select SNO
	from SaleBill
	where GoodsNO in('GN0003','GN0007')
	group by SNO
	having count(GoodsNO) = 2
	) g
on S.SNO = g.SNO
order by S.SNO

select S.SNO,S.SName
from SaleBill SB, Student S
where SB.GoodsNO='GN0003' and Exists (select * from SaleBill where SaleBill.GoodsNO='GN0007' and SB.SNO=SaleBill.SNO) and SB.SNO=S.SNO
go
```

（14）查询各校销售额。

```sql
select College,sum(sum_Number_salePrice) 销售额
from Student S join(
	select SA.SNO,sum(SA.Number * G.SalePrice) sum_Number_salePrice
	from SaleBill SA join Goods G
	on SA.GoodsNO = G.GoodsNO
	group by SNO) g
on S.SNO = g.SNO
group by College
go


select College, Sum(SaleBill.Number*Goods.SalePrice)
from SaleBill,Goods,Student
where SaleBill.GoodsNO=Goods.GoodsNO and SaleBill.SNO=Student.SNO
group by College
go
```

（15）查询购买额前三的校名、专业名。

```sql
select top 3 S.College,S.Major,sum(sum_Number_salePrice) amount
from Student S
join (
	select SA.SNO,sum(SA.Number * G.SalePrice) sum_Number_salePrice
	from SaleBill SA join Goods G
	on SA.GoodsNO = G.GoodsNO
	group by SNO) g
on S.SNO = g.SNO
group by College,Major
order by amount desc

go
select top 3 college,major,sum(saleprice*salebill.number) amount
from salebill,student,goods
where salebill.sno=student.sno and salebill.goodsno=goods.goodsno
group by college,major
order by amount desc
go
```

（16）使用集合查询方式查询生产日期早于2018-1-1 或库存量小于30的商品信息。

```sql
select * from Goods where ProductTime <'2018-1-1'
union
select * from Goods where Number <30

go
select * from goods where datediff(day,'2018-1-1',producttime)<0
union
select * from goods where number<30
go
```
