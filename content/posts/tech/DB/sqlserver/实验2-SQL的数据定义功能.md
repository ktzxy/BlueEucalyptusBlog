+++
date = '2024-08-26T03:22:00+08:00'
draft = false
title = '实验2 SQL的数据定义功能'
slug = "rk5mm0ex49"
description = "实验2 SQL的数据定义功能"
categories = ["📒数据库"]
tags = ["sqlserver"]
summary = "实验2 SQL的数据定义功能"
series = ["数据库实验系列"]
showSeries= true  
+++

﻿### 1、使用SQL Server Management Studio管理平台完成教材例3-7、例3-8、例3-9、例3-10和例3-11，分别为supermarket数据库创建学生表、商品表、商品种类表、供应商表和销售表。
```sql
create database supermarket
go
create table Student(
	SNO varchar(20) primary key,
	SName varchar(20),
	BirthYear int,
	Ssex varchar(2),
	College varchar(100),
	Major varchar(100),
	WeiXin varchar(100)
)

create table Category(
	CategoryNO varchar(20) primary key,
	CategoryName varchar(100),
	Description varchar(500)
)

create table Supplier(
	SupplierNO varchar(20) primary key,
	SupplierName varchar(100),
	Address varchar(200),
	Telephone varchar(20)
)

create table Goods(
	GoodsNO varchar(20) primary key,
	SupplierNO varchar(20),
	CategoryNO varchar(20),
	GoodsName varchar(100),
	InPrice decimal(18,2),
	SalePrice decimal(18,2),
	Number int,
	ProductTime smalldatetime,
	QGPeriod tinyint,
	foreign key(CategoryNO) references Category (CategoryNO),
	foreign key (SupplierNO) references Supplier (SupplierNO)
)



create table SaleBill(
	GoodsNO varchar(20),
	SNO varchar(20),
	HappenTime datetime,
	Number int,
	primary key (GoodsNO,SNO),
	foreign key (GoodsNO) references Goods (GoodsNO),
	foreign key (SNO) references Student (SNO)
)
```
```sql
use supermarket
go
insert into supermarket.dbo.Student values('S01','李明',2001,'男','CS','IT','wx001');
insert into supermarket.dbo.Student values('S02','徐好',2000,'女','CS','IT','wx002');
insert into supermarket.dbo.Student values('S03','伍民',1998,'男','CS','MIS','wx003');
insert into supermarket.dbo.Student values('S04','闵红',1999,'女','ACC','IT','wx004');
insert into supermarket.dbo.Student values('S05','张小红',1999,'女','ACC','IT','wx005');
insert into supermarket.dbo.Student values('S06','张舒',2001,'男','CS','MIS','wx006');
insert into supermarket.dbo.Student values('S07','王民为',1999,'男','CS','MIS','wx007');
insert into supermarket.dbo.Student values('S08','李士任',2001,'男','ACC','IT','wx008');


insert into supermarket.dbo.Category values('CN001','咖啡','速溶咖啡、咖啡粉、罐装咖啡');
insert into supermarket.dbo.Category values('CN002','洗发水','袋装、瓶装洗发水');
insert into supermarket.dbo.Category values('CN003','方便面','袋装、碗装方便面');

insert into supermarket.dbo.Supplier values('Sup001','卡夫食品（中国）有限公司广州分公司','广州佛山','12348768900');
insert into supermarket.dbo.Supplier values('Sup002','东菀市南城久润食品贸易部','广州东菀','13248768901');
insert into supermarket.dbo.Supplier values('Sup003','重庆飞鹤食品贸易公司','重庆解放碑','12648768901');
insert into supermarket.dbo.Supplier values('Sup004','重庆南山日化品贸易公司','重庆南坪','11648768901');
insert into supermarket.dbo.Supplier values('Sup005','重庆缙云日化品贸易公司','重庆北碚','19648768903');


insert into supermarket.dbo.Goods values('GN0001','Sup001','CN001','麦氏威尔冰咖啡',5.79,7.80,20,'2016-02-08 00:00:00',18);
insert into supermarket.dbo.Goods values('GN0002','Sup002','CN001','捷荣三合一咖啡',12.30,17.30,15,'2017-10-08 00:00:00',18);
insert into supermarket.dbo.Goods values('GN0003','Sup002','CN001','力神咖啡',1.81,2.70,30,'2018-05-06 00:00:00',18);
insert into supermarket.dbo.Goods values('GN0004','Sup001','CN001','麦氏威尔小三合一咖啡',8.12,10.80,20,'2017-05-06 00:00:00',18);
insert into supermarket.dbo.Goods values('GN0005','Sup003','CN001','雀巢香滑咖啡饮料',1.99,2.70,3,'2018-01-01 00:00:00',18);
insert into supermarket.dbo.Goods values('GN0006','Sup003','CN001','雀巢听装咖啡',84.21,113.70,6,'2018-05-06 00:00:00',18);
insert into supermarket.dbo.Goods values('GN0007','Sup004','CN002','夏士莲丝质柔顺洗发水',25.85,35.70,30,'2018-03-08 00:00:00',36);
insert into supermarket.dbo.Goods values('GN0008','Sup005','CN002','飞逸清新爽节洗发水',20.47,30.00,50,'2018-03-09 00:00:00',36);
insert into supermarket.dbo.Goods values('GN0009','Sup005','CN002','力士柔亮洗发水（中/干）',22.65,32.30,20,'2017-12-08 00:00:00',36);
insert into supermarket.dbo.Goods values('GN0010','Sup005','CN002','风影去屑洗发水（清爽）',22.98,34.20,6,'2017-10-07 00:00:00',36);

insert into supermarket.dbo.SaleBill values('GN0001','S01','2018-06-09 00:00:00',3);
insert into supermarket.dbo.SaleBill values('GN0001','S02','2018-05-03 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0001','S03','2018-04-07 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0001','S06','2018-06-12 00:00:00',2);
insert into supermarket.dbo.SaleBill values('GN0002','S02','2018-05-08 00:00:00',2);
insert into supermarket.dbo.SaleBill values('GN0002','S05','2018-06-26 00:00:00',3);
insert into supermarket.dbo.SaleBill values('GN0002','S06','2018-06-16 00:00:00',2);
insert into supermarket.dbo.SaleBill values('GN0003','S01','2018-07-10 00:00:00',2);
insert into supermarket.dbo.SaleBill values('GN0003','S02','2018-07-08 00:00:00',2);
insert into supermarket.dbo.SaleBill values('GN0003','S05','2018-06-01 00:00:00',2);
insert into supermarket.dbo.SaleBill values('GN0003','S06','2018-07-01 00:00:00',2);
insert into supermarket.dbo.SaleBill values('GN0005','S05','2018-06-11 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0006','S03','2018-05-07 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0007','S01','2018-06-09 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0007','S04','2018-06-08 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0007','S05','2018-06-09 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0008','S02','2018-06-04 00:00:00',1);
insert into supermarket.dbo.SaleBill values('GN0008','S06','2018-06-28 00:00:00',1);
```
### 2、使用SQL Server Management Studio管理平台完成教材例3-12、例3-13、例3-14，完成对supermarket数据库中已有数据库表的修改与删除。
```sql
alter table Category add Cat_CategoryNO varchar(20)

alter table Goods drop column Barcode   

alter table Supplier alter column SupplierName nvarchar(200) not null;
```
### 3、使用SQL Server Management Studio管理平台为sjkDB数据库创建数据库表sjktable（表结构自行设计），然后完成例3-15的要求。
![1e53e0bd9b0d0fc0356594725222e7a6](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304170335089.webp)
![e50fb1f1195ccfec72d60c4a49aa198a](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304170416168.webp)

```sql
drop table sjktable
```
### 4、使用SQL语句，为Students数据库创建数据库表Student、Course、Sc，表结构如教材表3-7~表3-9所示。
```sql
use students
go
create table Student(
	Sno char(7) primary key,
	Sname nchar(5) not null,
	Sex nchar(1),
	Sage tinyint,
	Sdept nvarchar(20)
)

create table Course(
	Cno char(6) primary key,
	Cname nvarchar(20) not null,
	Ccredit tinyint,
	Semester tinyint
)
create table Sc(
	Sno char(7),
	Cno char(6),
	Grade tinyint,
	primary key(Sno,Cno)
)
```
### 5、使用SQL语句，完成对Students数据库中数据库表结构的修改：

（1）为表Student添加地址列Address，数据类型为NVARCHAR(50)。

```sql
alter table Student add Address nvarchar(50)
```

（2）将地址列数据类型修改为NVARCHAR(30)。

```sql
alter table Student alter column Address nvarchar(30)
```

（3）删除地址列。

```sql
alter table Student drop column Address
```
### 6、在数据库sjkDB上完成，使用教材P48页的SQL语句创建教材的表3-3和表3-4所示的员工表和薪资表，然后使用SQL Server Management Studio管理平台和SQL语句依次完成例3-25、例3-26和例3-27的操作。
```sql
create unique nonclustered index index_xinz on salary(Saname asc)

create unique index index_yfsf on salary(Sapayabl asc,Psalary desc)

drop index salary.index_xinz,salary.index_yfsf
```
### 7、在数据库supermarket上，使用SQL语句完成下列操作。

（1）为表supplier的字段SupplierName创建一个非聚集、唯一索引。

```sql
use supermarket
go
create unique nonclustered index index_supplier on supplier(SupplierName)
```

（2）使用系统存储过程sp_helpindex查看表Supplier的索引情况，如果已有主码，能否为其再建立一个聚集索引？为什么？

![f3dbfac4b3ac3e93455ac60c08f68251](https://cdn.jsdelivr.net/gh/ktzxy/blog-img@main/2026/20260304170446063.webp)
```sql
聚集索引是通过设置主码来完成的，每个表的主码都是聚集索引，一个表只能有一个聚集索引，
非聚集索引可以有多个。因此无法重复创建聚焦索引
```

（3）删除第（1）题中所建立的索引。

```sql
drop index supplier.index_supplier
```
