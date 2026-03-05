+++
date = '2024-08-15T18:28:38+08:00'
draft = false
title = '实验7 T SQL编程'
slug = "d1lx0hx0un"
description = "实验7 T SQL编程"
categories = ["📒数据库"]
tags = ["sqlserver","sql编程"]
summary = "实验7 T SQL编程"
+++

﻿### 在超市管理数据库SuperMarket的基础上进行实验。
### 1、自定义数据类型GoodID_Type，用于描述商品的编号。
```sql
--查看该表的数据类型
sp_columns Goods
--创建（方法一）
create type GoodID_Type from varchar(20) not null
--创建（方法二）
exec sp_addtype GoodID_Type,'varchar(20)','not null'
--查看
exec sp_help GoodID_Type
```
### 2、在SuperMarket数据库中创建表good，表结构与Goods类似，而GoodsNO的数据类型为自定义数据类型GoodID_type。
```sql
drop table if exists good;
create table good(
	GoodsNO GoodID_type primary key,
	SupplierNO varchar(20),
	CategoryNO varchar(20),
	GoodsName varchar(100),
	Barcode varchar(100),
	InPrice decimal(18,2),
	SalePrice decimal(18,2),
	Number int,
	ProductTime smalldatetime,
	QGPeriod tinyint,
	foreign key(CategoryNO) references Category (CategoryNO),
	foreign key (SupplierNO) references Supplier (SupplierNO)
)
go
```
### 3、创建一个局部变量goods_type，并在SELECT语句中使用该变量查找商品表中所有毛巾类商品的名称和售价。
```sql
declare @goods_type varchar(20)
set @goods_type = '毛巾'
select GoodsName,SalePrice from Goods
where CategoryNO in (
					select CategoryNO
					from Category
					where CategoryName like @goods_type)
go
```
### 4、判断商品表Goods是否存在商品类型为“白酒”的商品，如果存在则显示该类别的所有商品信息，否则显示无此类商品。
```sql
declare @gname varchar(20),@num int
set @gname='白酒'
select @num=COUNT(*) from Goods
where CategoryNO in (
					select CategoryNO
					from Category
					where CategoryName like @gname)
if @num > 0
	select * from Goods where CategoryNO in (select CategoryNO from Category where CategoryName like @gname)
else
	print '无此类商品'
go


declare @s int,@goods_type varchar(100)
set @goods_type='白酒'
select @s=(select count(*)
from goods, category
where goods.CategoryNO=Category.CategoryNO and CategoryName=@goods_type)

if @s>0
	select *
	from goods, category
	where goods.CategoryNO=Category.CategoryNO and CategoryName=@goods_type
else
	print  '无此类商品'
go
```
### 5、如果商品表Goods中存在商品数量小于10的情况，则将所有商品数量增加10，反复执行直到所有商品的数量都不小于10为止。
```sql
while (select MAX(Number) from Goods) >=10
begin
	if(select MIN(Number) from Goods) <10
		begin
			update Goods set Number = Number + 10
			break
		end
end
go

while exists(select * from goods where number<10)
begin 
	update goods set number=number+10
end
go
```
### 6、声明一个游标，用于对“饼干”类商品的售价降价5%。
```sql
declare @gname varchar(20)
declare @now_goods_id varchar(20)
declare @total int
set @gname = '饼干'
--声明游标
declare sale_cur cursor
for select GoodsNO from Goods where CategoryNO in (
	select CategoryNO from Category where CategoryName like @gname)

--给局部变量赋值
select @total=COUNT(*) from Goods where CategoryNO in (
	select CategoryNO from Category where CategoryName like @gname)
--打开游标
open sale_cur
--使用游标
while @total > 0 
		begin
			--游标下移
			fetch sale_cur into @now_goods_id
			--更新数据
			begin
				update Goods set SalePrice = SalePrice - SalePrice * 0.05 where GoodsNO like @now_goods_id
				--break 不可以设置break,否则只生效一条数据
			end
			set @total = @total -1
		end
--关闭游标
close sale_cur

--释放游标
deallocate sale_cur
go



declare @goods_no varchar(20)
declare cur_pie cursor
	for 
		select GoodsNO
		from goods, category
		where goods.CategoryNO=Category.CategoryNO and CategoryName='咖啡'
	for update of SalePrice
open cur_pie
fetch next from cur_pie into @goods_no
while @@FETCH_STATUS=0
	begin
		update goods set SalePrice=SalePrice*0.95 where GoodsNO=@goods_no
		fetch next from cur_pie into @goods_no
	end
close cur_pie
deallocate cur_pie
go
```
### 7、创建自定义函数，用于统计销售表SaleBill 中某段时间内的销售情况。并调用该函数输出执行结果。
```sql
create function fun_saleinfo(@pre smalldatetime,@last smalldatetime)
returns int
begin
	return(select SUM(Number) from SaleBill where HappenTime >= @pre and HappenTime<=@last)
end
go
declare @formerly smalldatetime,@latter smalldatetime
set @formerly = '2018-04-01'
set @latter = '2018-05-01'
select dbo.fun_saleinfo(@formerly,@latter)
go


create function salereport(@begindate smalldatetime, @enddate smalldatetime)
returns decimal(18,2)
as
	begin
		declare @sum_amount decimal(18,2)
		select @sum_amount=(select sum(saleprice*S.number)
			from salebill S,goods G
			where S.GoodsNO=G.GoodsNO)
		return @sum_amount
	end
go

declare @begindate smalldatetime, @enddate smalldatetime
set @begindate='2018-7-1'
set @enddate='2018-7-31'
select dbo.salereport(@begindate,@enddate) as '2018年7月份销售金额'
go
```
### 8、创建自定义函数，用于显示商品表Goods中售价大于指定价格的商品信息。并调用该函数输出执行结果。
```sql
create function showGoods(@saleprice decimal(18,2))
returns table
as
	return select * from goods where saleprice>@saleprice
go

select * from dbo.showGoods(50)
go
```
