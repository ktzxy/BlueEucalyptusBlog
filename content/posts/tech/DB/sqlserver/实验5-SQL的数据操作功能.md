+++
date = '2024-08-20T17:52:34+08:00'
draft = false
title = '实验5 SQL的数据操作功能'
slug = "v6ygzp6ydl"
description = "实验5 SQL的数据操作功能"
categories = ["📒数据库"]
tags = ["sqlserver"]
summary = "实验5 SQL的数据操作功能"
+++

﻿### 1、完成教材例3-70~例3-76的操作。
```sql
insert into Student values('S09','程浩',1999,'男','CS','IT','wx009');


use supermarket;
create table SubGoods(
	GoodName varchar(100),
	Number int
)
insert into SubGoods
	select GoodsName,G.Number 
	from Goods G left join SaleBill SA 
	on G.GoodsNO = SA.GoodsNO
	where SA.SNO is null



update Goods set Number = Number + 2


update Goods set Number = 0
where DATEDIFF(DAY,ProductTime,GETDATE()) - QGPeriod * 30 >0


update Goods set SalePrice = SalePrice * 1.1
from Supplier S join Goods G on S.SupplierNO = G.SupplierNO
where SupplierName = '重庆缙云日化品贸易公司'


delete SubGoods


delete from Goods
from Supplier S join Goods G on S.SupplierNO = G.SupplierNO
where SupplierName = '重庆缙云日化品贸易公司'
```
### 2、在数据库supermarket上完成下列操作。

（1） 添加新品“GN0011  Sup002  CN001  乐至三合一咖啡  12. 30  17. 30  100  2018-11-12  18”。

```sql
insert into Goods values('GN0011','Sup002','CN001','乐至三合一咖啡',12.30,17.30,100,'2018-11-12 00:00:00',18);
```

（2）先建立一张新表，使用子查询将各月的销售额插入该表，存储月份及销售额。

```sql
if exists(select name from sysobjects where name='sale_report')
	drop table sale_report
create table sale_report(
	 月份 char(7),
	 销售额 decimal(18,2)
 )
 insert into sale_report 
	 select convert(char(7),HappenTime,120), sum(s.number*g.SalePrice)  /*子查询,获取月份及销售额,结果集作为values被插入到目标表*/
	 from SaleBill s, Goods g
	 where s.GoodsNO=g.GoodsNO
	 group by convert(char(7),HappenTime,120)
go
```

（3） 使用子查询将各学生的购买额插入新表，由系统自建新表，存储学生学号、姓名、销售额。

```sql
if exists(select * from sysobjects where name='sale_report_stu')
	drop table sale_report_stu
 select s.sno 学号, s.SName 姓名, sa.amount 销售额 into stu_sale_report
	from student s join  
		(select student.sno sno, sum(salebill.number*goods.saleprice) amount  /* 子查询,获取学号及其销售额,结果作为派生表*/
			from SaleBill,Student,Goods 
			where SaleBill.SNO=Student.SNO and SaleBill.GoodsNO=Goods.GoodsNO 
			group by Student.SNO
		) sa  /*为了方便他处引用派生表的字段,为派生表指定别名*/
	on s.sno=sa.sno
go
```

（4）将所有商品存量增加2。

```sql
update supermarket.dbo.Goods set Number = Number + 2
```

（5）将保质期还有30天的商品价格打8折。

```sql
 update goods set saleprice=saleprice*0.8
 where QGPeriod*30-datediff(day,producttime,getdate())<=30
```

（6）分别使用子查询方式与连接方式将广州地区供货商的商品加价10%。

```sql
update Goods set SalePrice = SalePrice * 1.1
from Supplier S join Goods G on S.SupplierNO = G.SupplierNO
where Address like '广州%'

update Goods set SalePrice = SalePrice * 1.1
where SupplierNO in(select SupplierNO from Supplier where Address like '广州%')
```

（7）将销售额后两位的商品下架。

```sql
alter table SaleBill nocheck constraint all;
delete from Goods where GoodsNO in(
	select GoodsNO from (
		select top 2 G.GoodsNO,SUM(SA.Number * G.SalePrice) GOODSUM
		from Goods G join SaleBill SA
		on G.GoodsNO = SA.GoodsNO
		group by G.GoodsNO
		order by GOODSUM) as s)
alter table SaleBill check constraint all;
```

（8）删除销售额最小的供应商信息。

```sql
alter table Goods nocheck constraint all;
delete from Supplier
where SupplierNO in(
	select * from (
		select top 1 S.SupplierNO from Supplier S
		join Goods G on S.SupplierNO = G.SupplierNO
		join SaleBill SA on G.GoodsNO = SA.GoodsNO
		group by S.SupplierNO
		order by SUM(SalePrice * SA.Number)
	) as a)
alter table Goods check constraint all;
```
