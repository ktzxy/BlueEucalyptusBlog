+++
date = '2026-03-26T14:27:24+08:00'
draft = false
title = 'Oracle&&SqlServer&&Mysql内置函数对比‌'
slug = "r2tu06odzl"
description = ""
summary = ""
categories = ["📒数据库"]
tags = ["数据库"]
comments = true
+++

## 目录

- [一、字符串函数](#一字符串函数)
- [二、日期函数](#二日期函数)
- [三、数学函数](#三数学函数)
- [四、聚合函数](#四聚合函数)
- [五、类型转换函数](#五类型转换函数)
- [六、条件函数](#六条件函数)
- [七、NULL相关函数](#七null相关函数)
- [八、分组与窗口函数](#八分组与窗口函数)
- [九、JSON处理函数](#九json处理函数)
- [十、空间/地理函数](#十空间地理函数)
- [十一、错误处理与异常](#十一错误处理与异常)
- [十二、序列与自增](#十二序列与自增)
- [十三、批量插入与合并](#十三批量插入与合并)
- [十四、其它常用系统函数](#十四其它常用系统函数)
- [十五、典型SQL语法差异补充](#十五典型sql语法差异补充)
- [十六、特殊函数与高级用法补充](#十六特殊函数与高级用法补充)
- [十七、三大数据库特有函数及特殊场景用法](#十七三大数据库特有函数及特殊场景用法)
- [十八、数据类型对照表](#十八数据类型对照表)
- [十九、迁移注意事项与常见问题](#十九迁移注意事项与常见问题)
- [二十、性能优化建议](#二十性能优化建议)
- [二十一、DDL/DML迁移脚本模板](#二十一ddldml迁移脚本模板)
- [二十二、迁移后测试与验证建议](#二十二迁移后测试与验证建议)

------

> **版本兼容性说明**：
>
> - Oracle 11g+ 支持大部分窗口函数，12c+支持FETCH分页、IDENTITY自增等。
> - SQL Server 2012+ 支持窗口函数、OFFSET分页、TRY_CAST等。
> - MySQL 8.0+ 支持窗口函数、正则、JSON_TABLE等，5.7+支持部分JSON函数。
> - 如无特殊说明，示例均以主流新版本为准。

------

### **一、字符串函数**‌

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | --------------------------- | -------------------------- | -------------------------- | ------------------------------------------------------------ | -------------- |
| 字符串连接   | `CONCAT(str1, str2)`        | `CONCAT(str1, str2)`       | `CONCAT(str1, str2, ...)`  | Oracle最多2个参数，SQL Server/MySQL支持多个参数拼接 | 11g+           |
| 子串截取     | `SUBSTR(str,start,len)`     | `SUBSTRING(str,start,len)` | `SUBSTRING(str, start, len)` | 起始位置从1开始，`len`为截取长度 | 11g+           |
| 字符串长度   | `LENGTH(str)`               | `LEN(str)`                 | `LENGTH(str)`              | 返回字符数（中文按1计算） | 11g+           |
| 去除首尾空格 | `TRIM(str)`                 | `LTRIM(RTRIM(str))`        | `TRIM(str)`                | Oracle支持单边`TRIM(LEADING/TRAILING)`，MySQL同理 | 11g+           |
| 大小写转换   | `LOWER(str)` / `UPPER(str)` | 同左                       | 同左                       | 全数据库通用 | 11g+           |

------

### **SQL示例对照**

#### 1. 字符串函数

- **Oracle**：

  ```sql
  SELECT CONCAT('A', 'B'), SUBSTR('Hello', 2, 3), LENGTH('中国'), TRIM('  abc  '), LOWER('ABC') FROM dual;
  ```

- **SQL Server**：

  ```sql
  SELECT CONCAT('A', 'B'), SUBSTRING('Hello', 2, 3), LEN('中国'), LTRIM(RTRIM('  abc  ')), LOWER('ABC');
  ```

- **MySQL**：

  ```sql
  SELECT CONCAT('A', 'B'), SUBSTRING('Hello', 2, 3), LENGTH('中国'), TRIM('  abc  '), LOWER('ABC');
  ```

------

### **二、日期函数**‌

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------------------------- | ----------------------------- | -------------------------- | ------------------------------------------------------------ | -------------- |
| 当前日期时间 | `SYSDATE`                    | `GETDATE()`                   | `NOW()`                    | 无参数，返回服务器当前时间 | 11g+           |
| 日期格式化   | `TO_CHAR(date,'YYYY-MM-DD')` | `CONVERT(VARCHAR, date, 120)` | `DATE_FORMAT(date, '%Y-%m-%d')` | Oracle格式代码自由，SQL Server用风格代码，MySQL用格式字符串 | 11g+           |
| 日期加减     | `date + N`                   | `DATEADD(unit, N, date)`      | `DATE_ADD(date, INTERVAL N unit)` | Oracle直接加减天数，SQL Server/MySQL需指定单位 | 11g+           |
| 提取日期部分 | `EXTRACT(YEAR FROM date)`    | `DATEPART(YEAR, date)`        | `YEAR(date)`               | 支持year/month/day/hour等时间单位 | 11g+           |

------

### **SQL示例对照**

#### 2. 日期函数

- **Oracle**：

  ```sql
  SELECT SYSDATE, TO_CHAR(SYSDATE, 'YYYY-MM-DD'), SYSDATE + 1, EXTRACT(YEAR FROM SYSDATE) FROM dual;
  ```

- **SQL Server**：

  ```sql
  SELECT GETDATE(), CONVERT(VARCHAR, GETDATE(), 120), DATEADD(day, 1, GETDATE()), DATEPART(YEAR, GETDATE());
  ```

- **MySQL**：

  ```sql
  SELECT NOW(), DATE_FORMAT(NOW(), '%Y-%m-%d'), DATE_ADD(NOW(), INTERVAL 1 DAY), YEAR(NOW());
  ```

------

### **三、数学函数**‌

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ----------------- | -------------- | -------------- | ---------------------------------- | -------------- |
| 绝对值       | `ABS(num)`        | 同左           | 同左           | 全数据库通用 | 11g+           |
| 向上取整     | `CEIL(num)`       | `CEILING(num)` | `CEIL(num)`    | 返回大于等于参数的最小整数 | 11g+           |
| 向下取整     | `FLOOR(num)`      | 同左           | 同左           | 返回小于等于参数的最大整数 | 11g+           |
| 四舍五入     | `ROUND(num,精度)` | 同左           | 同左           | 参数1为数值，参数2为保留小数位数 | 11g+           |

------

### **SQL示例对照**

#### 3. 数学函数

- **Oracle**：

  ```sql
  SELECT ABS(-5), CEIL(2.3), FLOOR(2.7), ROUND(3.1415, 2) FROM dual;
  ```

- **SQL Server**：

  ```sql
  SELECT ABS(-5), CEILING(2.3), FLOOR(2.7), ROUND(3.1415, 2);
  ```

- **MySQL**：

  ```sql
  SELECT ABS(-5), CEIL(2.3), FLOOR(2.7), ROUND(3.1415, 2);
  ```

------

### ‌**四、聚合函数**‌

| ‌**函数功能**‌ | ‌**Oracle**‌                              | ‌**SQL Server**‌                             | ‌**参数说明**‌                                                 | **版本要求** |
| ------------ | --------------------------------------- | ------------------------------------------ | ------------------------------------------------------------ | -------------- |
| 字符串聚合   | `LISTAGG(str,分隔符) WITHIN GROUP(...)` | `STRING_AGG(str,分隔符) WITHIN GROUP(...)` | Oracle支持`ON OVERFLOW TRUNCATE`截断，SQL Server需手动处理超长‌13 | 11g+           |
| 空值替换     | `NVL(expr1, expr2)`                     | `ISNULL(expr1, expr2)`                     | 当`expr1`为NULL时返回`expr2`‌24                               | 11g+           |

------

### **SQL示例对照**

#### 4. 聚合函数

- **Oracle**：

  ```sql
  SELECT LISTAGG(ename, ',') WITHIN GROUP (ORDER BY ename) AS names FROM emp;
  SELECT NVL(comm, 0) FROM emp;
  ```

- **SQL Server**：

  ```sql
  SELECT STRING_AGG(ename, ',') WITHIN GROUP (ORDER BY ename) AS names FROM emp;
  SELECT ISNULL(comm, 0) FROM emp;
  ```

- **MySQL**：

  ```sql
  SELECT GROUP_CONCAT(ename ORDER BY ename SEPARATOR ',') AS names FROM emp;
  SELECT IFNULL(comm, 0) FROM emp;
  ```

------

### ‌**五、类型转换函数**‌

| ‌**函数功能**‌ | ‌**Oracle**‌           | ‌**SQL Server**‌                                   | ‌**参数说明**‌                                              | **版本要求** |
| ------------ | -------------------- | ------------------------------------------------ | --------------------------------------------------------- | -------------- |
| 类型转换     | `TO_NUMBER(str)`     | `CAST(str AS NUMERIC)` / `CONVERT(NUMERIC, str)` | SQL Server推荐使用`TRY_CAST`避免转换失败‌45                | 11g+           |
| 日期转字符串 | `TO_CHAR(date,格式)` | `CONVERT(VARCHAR, date, 格式代码)`               | Oracle格式如`'YYYY-MM-DD'`，SQL Server用数字代码如`112`‌25 | 11g+           |

------

### **SQL示例对照**

#### 5. 类型转换函数

- **Oracle**：

  ```sql
  SELECT TO_NUMBER('123'), TO_CHAR(SYSDATE, 'YYYY-MM-DD') FROM dual;
  ```

- **SQL Server**：

  ```sql
  SELECT CAST('123' AS NUMERIC), CONVERT(VARCHAR, GETDATE(), 112);
  ```

- **MySQL**：

  ```sql
  SELECT CAST('123' AS DECIMAL), DATE_FORMAT(NOW(), '%Y-%m-%d');
  ```

------

### ‌**六、条件函数**‌

| ‌**函数功能**‌ | ‌**Oracle**‌                          | ‌**SQL Server**‌                                     | ‌**参数说明**‌                                                 | **版本要求** |
| ------------ | ----------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | -------------- |
| 条件判断     | `DECODE(expr, val1, res1, default)` | `CASE WHEN expr = val1 THEN res1 ELSE default END` | Oracle特有`DECODE`，SQL Server用标准`CASE`‌24                 | 11g+           |
| 空值处理     | `NVL2(expr, res1, res2)`            | `COALESCE(expr, res1, res2)`                       | Oracle根据`expr`是否为NULL返回不同结果，SQL Server用多参数合并‌46 | 11g+           |

------

### **SQL示例对照**

#### 6. 条件函数

- **Oracle**：

  ```sql
  SELECT DECODE(sex, 'M', '男', 'F', '女', '未知') FROM person;
  SELECT NVL2(comm, '有提成', '无提成') FROM emp;
  ```

- **SQL Server**：

  ```sql
  SELECT CASE WHEN sex = 'M' THEN '男' WHEN sex = 'F' THEN '女' ELSE '未知' END FROM person;
  SELECT COALESCE(comm, '无提成') FROM emp;
  ```

- **MySQL**：

  ```sql
  SELECT IF(sex = 'M', '男', IF(sex = 'F', '女', '未知')) FROM person;
  SELECT IFNULL(comm, '无提成') FROM emp;
  ```

------

### ‌**七、NULL相关函数**‌

| ‌**函数功能**‌ | ‌**Oracle**‌                          | ‌**SQL Server**‌                                     | ‌**参数说明**‌                                                 | **版本要求** |
| ------------ | ----------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | -------------- |
| 条件判断     | `NVL(expr1, expr2)`                     | `ISNULL(expr1, expr2)`                     | 当`expr1`为NULL时返回`expr2`‌24                               | 11g+           |
| 空值处理     | `COALESCE(expr, res1, res2)`            | `COALESCE(expr, res1, res2)`                       | 合并多个参数，返回第一个非NULL值‌46                               | 11g+           |

------

### **SQL示例对照**

#### 7. NULL相关函数

- **Oracle**：

  ```sql
  SELECT NVL(comm, 0), COALESCE(comm, bonus, 0), CASE WHEN comm IS NULL THEN 1 ELSE 0 END FROM emp;
  ```

- **SQL Server**：

  ```sql
  SELECT ISNULL(comm, 0), COALESCE(comm, bonus, 0), CASE WHEN comm IS NULL THEN 1 ELSE 0 END FROM emp;
  ```

- **MySQL**：

  ```sql
  SELECT IFNULL(comm, 0), IFNULL(bonus, 0), IF(comm IS NULL, 1, 0) FROM emp;
  ```

------

### **八、分组与窗口函数**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 行号         | `ROWNUM` / `ROW_NUMBER() OVER(...)` | `ROW_NUMBER() OVER(...)` | `ROW_NUMBER() OVER(...)` | 返回分组内行号 | 11g+           |
| 累计/排名    | `RANK() OVER(...)` / `DENSE_RANK() OVER(...)` | 同左 | 同左 | 分组排名 | 11g+           |
| 分组求和     | `SUM(col) OVER(PARTITION BY ...)` | 同左 | 同左 | 分组累计 | 11g+           |
| 移动平均     | `AVG(col) OVER(ORDER BY ... ROWS BETWEEN N PRECEDING AND CURRENT ROW)` | 同左 | 同左 | 计算滑动窗口平均值 | 11g+           |
| 首/末值      | `FIRST_VALUE(col) OVER(...)` / `LAST_VALUE(col) OVER(...)` | 同左 | 同左 | 获取分组内首/末值 | 11g+           |
| 行间差值     | `LAG(col, N, default) OVER(...)` / `LEAD(col, N, default) OVER(...)` | 同左 | 同左 | 获取前/后N行的值 | 11g+           |

------

### **SQL示例对照**

#### 8. 分组与窗口函数

- **Oracle**：

  ```sql
  SELECT ename, ROW_NUMBER() OVER(ORDER BY sal DESC) AS rn FROM emp;
  SELECT deptno, SUM(sal) OVER(PARTITION BY deptno) AS total_sal FROM emp;
  ```

- **SQL Server**：

  ```sql
  SELECT ename, ROW_NUMBER() OVER(ORDER BY sal DESC) AS rn FROM emp;
  SELECT deptno, SUM(sal) OVER(PARTITION BY deptno) AS total_sal FROM emp;
  ```

- **MySQL**：

  ```sql
  SELECT ename, ROW_NUMBER() OVER(ORDER BY sal DESC) AS rn FROM emp;
  SELECT deptno, SUM(sal) OVER(PARTITION BY deptno) AS total_sal FROM emp;
  ```

------

### **九、JSON处理函数**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 解析JSON     | `JSON_VALUE(json_col, '$.key')` | `JSON_VALUE(json_col, '$.key')` | `JSON_EXTRACT(json_col, '$.key')` | 提取JSON字段值 | 11g+           |
| JSON对象转表 | `JSON_TABLE(json_col, '$.items[*]' COLUMNS(...))` | `OPENJSON(json_col)` | `JSON_TABLE(json_col, '$')` | 拆分JSON数组为多行 | 11g+           |
| 判断JSON有效 | `IS JSON` | `ISJSON(json_col)` | `ISJSON(json_col)` | 判断字符串是否为合法JSON | 11g+           |

------

### **SQL示例对照**

#### 9. JSON处理函数

- **Oracle**：

  ```sql
  SELECT JSON_VALUE('{"a":1}', '$.a') FROM dual;
  ```

- **SQL Server**：

  ```sql
  SELECT JSON_VALUE('{"a":1}', '$.a');
  SELECT * FROM OPENJSON('[{"id":1},{"id":2}]');
  ```

- **MySQL**：

  ```sql
  SELECT JSON_EXTRACT('{"a":1}', '$.a');
  SELECT * FROM JSON_TABLE('[{"id":1},{"id":2}]', '$');
  ```

------

### **十、空间/地理函数**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 创建点       | `SDO_GEOMETRY(...)` | `geometry::STPointFromText(...)` | `ST_GeomFromText(...)` | 创建空间点对象 | 11g+           |
| 距离计算     | `SDO_GEOM.SDO_DISTANCE(...)` | `geometry::STDistance(...)` | `ST_Distance(...)` | 计算空间距离 | 11g+           |
| 空间相交     | `SDO_RELATE(...)` | `geometry::STIntersects(...)` | `ST_Intersects(...)` | 判断空间对象是否相交 | 11g+           |

------

### **SQL示例对照**

#### 10. 空间/地理函数

- **Oracle**：

  ```sql
  SELECT SDO_GEOMETRY(2001, 4326, SDO_POINT_TYPE(116.4, 39.9, NULL), NULL, NULL) FROM dual;
  ```

- **SQL Server**：

  ```sql
  SELECT geometry::STPointFromText('POINT(116.4 39.9)', 4326);
  ```

- **MySQL**：

  ```sql
  SELECT ST_GeomFromText('POINT(116.4 39.9)');
  ```

------

### **十一、错误处理与异常**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 异常捕获     | `BEGIN ... EXCEPTION WHEN ... THEN ... END;` | `BEGIN TRY ... END TRY BEGIN CATCH ... END CATCH` | `BEGIN ... END TRY BEGIN CATCH ... END CATCH` | 均支持块级异常处理 | 11g+           |
| 抛出异常     | `RAISE_APPLICATION_ERROR(-20001, 'msg')` | `THROW 50001, 'msg', 1` | `THROW 50001, 'msg', 1` | 自定义错误抛出 | 11g+           |

------

### **SQL示例对照**

#### 11. 错误处理与异常

- **Oracle**：

  ```sql
  BEGIN
    -- 代码
  EXCEPTION
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('出错');
  END;
  ```

- **SQL Server**：

  ```sql
  BEGIN TRY
    -- 代码
  END TRY
  BEGIN CATCH
    PRINT '出错';
  END CATCH;
  ```

- **MySQL**：

  ```sql
  BEGIN
    -- 代码
  END TRY
  BEGIN CATCH
    SELECT '出错';
  END CATCH;
  ```

------

### **十二、序列与自增**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 创建序列     | `CREATE SEQUENCE seq_name ...` | `CREATE SEQUENCE seq_name ...` | `CREATE SEQUENCE seq_name ...` | 语法类似，参数略有差异 | 12c+           |
| 获取下值     | `seq_name.NEXTVAL` | `NEXT VALUE FOR seq_name` | `LAST_INSERT_ID()` | 获取序列下一个值 | 12c+           |
| 自增主键     | `GENERATED AS IDENTITY`（12c+）/`NUMBER GENERATED BY DEFAULT` | `IDENTITY(1,1)` | `AUTO_INCREMENT` | 字段属性定义自增 | 12c+           |

------

### **SQL示例对照**

#### 12. 序列与自增

- **Oracle**：

  ```sql
  CREATE SEQUENCE seq_test;
  SELECT seq_test.NEXTVAL FROM dual;
  CREATE TABLE t1(id NUMBER GENERATED BY DEFAULT AS IDENTITY, name VARCHAR2(20));
  ```

- **SQL Server**：

  ```sql
  CREATE SEQUENCE seq_test;
  SELECT NEXT VALUE FOR seq_test;
  CREATE TABLE t1(id INT IDENTITY(1,1), name VARCHAR(20));
  ```

- **MySQL**：

  ```sql
  CREATE SEQUENCE seq_test;
  SELECT LAST_INSERT_ID();
  CREATE TABLE t1(id INT AUTO_INCREMENT, name VARCHAR(20));
  ```

------

### **十三、批量插入与合并**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 批量插入     | `INSERT ALL ...` / `INSERT INTO ... SELECT ...` | `INSERT INTO ... SELECT ...` | `INSERT INTO ... SELECT ...` | 批量插入多行数据 | 11g+           |
| 合并（UPSERT）| `MERGE INTO ... USING ... ON ... WHEN MATCHED THEN ...` | 同左 | `MERGE INTO ... ON ... WHEN MATCHED THEN ...` | 两者均支持标准`MERGE`语法 | 11g+           |

------

### **SQL示例对照**

#### 13. 批量插入与合并

- **Oracle**：

  ```sql
  INSERT ALL
    INTO t1 VALUES(1, 'A')
    INTO t1 VALUES(2, 'B')
  SELECT * FROM dual;
  MERGE INTO t1 USING t2 ON (t1.id = t2.id)
    WHEN MATCHED THEN UPDATE SET t1.name = t2.name
    WHEN NOT MATCHED THEN INSERT (id, name) VALUES (t2.id, t2.name);
  ```

- **SQL Server**：

  ```sql
  INSERT INTO t1 (id, name) SELECT 1, 'A' UNION ALL SELECT 2, 'B';
  MERGE t1 USING t2 ON t1.id = t2.id
    WHEN MATCHED THEN UPDATE SET t1.name = t2.name
    WHEN NOT MATCHED THEN INSERT (id, name) VALUES (t2.id, t2.name);
  ```

- **MySQL**：

  ```sql
  INSERT INTO t1 (id, name) VALUES (1, 'A'), (2, 'B');
  MERGE t1 USING t2 ON t1.id = t2.id
    WHEN MATCHED THEN UPDATE SET t1.name = t2.name
    WHEN NOT MATCHED THEN INSERT (id, name) VALUES (t2.id, t2.name);
  ```

------

### **十四、其它常用系统函数**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 获取服务器时间 | `SYSDATE` / `SYSTIMESTAMP` | `GETDATE()` / `SYSDATETIME()` | `NOW()` | 精度略有差异 | 11g+           |
| 获取当前用户   | `USER` / `SYS_CONTEXT('USERENV','SESSION_USER')` | `CURRENT_USER` / `SUSER_SNAME()` | `CURRENT_USER()` | 获取当前登录用户 | 11g+           |
| 获取主机名     | `SYS_CONTEXT('USERENV','HOST')` | `HOST_NAME()` | `@@SERVERNAME` | 获取主机名 | 11g+           |

------

### **SQL示例对照**

#### 14. 其它常用系统函数

- **Oracle**：

  ```sql
  SELECT SYSDATE, USER, SYS_CONTEXT('USERENV','HOST') FROM dual;
  ```

- **SQL Server**：

  ```sql
  SELECT GETDATE(), CURRENT_USER, HOST_NAME();
  ```

- **MySQL**：

  ```sql
  SELECT NOW(), CURRENT_USER, @@SERVERNAME;
  ```

------

### **十五、典型SQL语法差异补充**

1. **LIMIT/OFFSET分页**：  
   - Oracle 12c+：`SELECT ... OFFSET N ROWS FETCH NEXT M ROWS ONLY`  
   - SQL Server 2012+：同上，老版本用`ROW_NUMBER()`子查询分页 | 11g+           |
2. **dual表**：  
   - Oracle：`SELECT 1 FROM dual`  
   - SQL Server：`SELECT 1`（无需dual） | 11g+           |
3. **字符串转义**：  
   - Oracle：单引号用`''`  
   - SQL Server：同上，但部分场景支持`N'...'`表示Unicode | 11g+           |
4. **布尔类型**：  
   - Oracle无布尔类型，常用`NUMBER(1)`或`CHAR(1)`  
   - SQL Server有`BIT`类型 | 11g+           |
5. **注释**：  
   - 单行：`--`  
   - 多行：`/* ... */`  
   - 两者通用 | 11g+           |

------

### **十六、特殊函数与高级用法补充**

| **函数功能** | **Oracle** | **SQL Server** | **MySQL** | **参数说明** | **版本要求** |
| ------------ | ---------- | -------------- | --------- | ------------ | -------------- |
| 分组统计     | `GROUP BY ROLLUP/CUBE` | `GROUP BY ROLLUP/CUBE` | `GROUP BY WITH ROLLUP` | 多维分组统计 | 11g+           |
| 字符查找     | `INSTR(str, substr)` | `CHARINDEX(substr, str)` | `INSTR(str, substr)` | 返回子串位置 | 11g+           |
| 正则匹配     | `REGEXP_LIKE(str, pattern)` | `LIKE`/`PATINDEX` | `REGEXP_LIKE(str, pattern)`/`REGEXP` | MySQL 8+支持标准正则 | 11g+           |
| 批量更新     | `MERGE`/`UPDATE ... WHERE ...` | `MERGE`/`UPDATE ... FROM ...` | `UPDATE ... JOIN ...` | 多表批量更新 | 11g+           |
| 加密解密     | `DBMS_CRYPTO` | `ENCRYPTBYPASSPHRASE`等 | `AES_ENCRYPT/AES_DECRYPT` | 内置加密函数 | 11g+           |
| XML处理      | `XMLTYPE`/`EXTRACTVALUE` | `FOR XML PATH` | `ExtractValue`/`XPath` | 结构化数据处理 | 11g+           |
| JSON处理     | `JSON_VALUE`/`JSON_TABLE` | `JSON_VALUE`/`OPENJSON` | `JSON_EXTRACT`/`JSON_TABLE` | 结构化数据处理 | 11g+           |
| 全文检索     | `CONTAINS` | `CONTAINS` | `MATCH ... AGAINST` | 需建全文索引 | 11g+           |
| 随机数       | `DBMS_RANDOM.VALUE` | `RAND()` | `RAND()` | 生成0-1随机数 | 11g+           |
| UUID         | `SYS_GUID()` | `NEWID()` | `UUID()` | 生成唯一标识符 | 11g+           |

------

#### 特殊函数SQL示例

- **分组统计**：
  - Oracle：

    ```sql
    SELECT deptno, SUM(sal) FROM emp GROUP BY ROLLUP(deptno);
    ```

  - SQL Server：

    ```sql
    SELECT deptno, SUM(sal) FROM emp GROUP BY ROLLUP(deptno);
    ```

  - MySQL：

    ```sql
    SELECT deptno, SUM(sal) FROM emp GROUP BY deptno WITH ROLLUP;
    ```

- **正则匹配**：
  - Oracle：

    ```sql
    SELECT * FROM t WHERE REGEXP_LIKE(name, '^A.*');
    ```

  - SQL Server：

    ```sql
    SELECT * FROM t WHERE name LIKE 'A%';
    ```

  - MySQL：

    ```sql
    SELECT * FROM t WHERE name REGEXP '^A.*';
    ```

- **批量更新**：
  - Oracle：

    ```sql
    MERGE INTO t1 USING t2 ON (t1.id = t2.id) WHEN MATCHED THEN UPDATE SET t1.name = t2.name;
    ```

  - SQL Server：

    ```sql
    UPDATE t1 SET t1.name = t2.name FROM t1 JOIN t2 ON t1.id = t2.id;
    ```

  - MySQL：

    ```sql
    UPDATE t1 JOIN t2 ON t1.id = t2.id SET t1.name = t2.name;
    ```

- **加密解密**：
  - Oracle：

    ```sql
    SELECT RAWTOHEX(DBMS_CRYPTO.ENCRYPT(UTL_I18N.STRING_TO_RAW('abc','AL32UTF8'), 4353, UTL_I18N.STRING_TO_RAW('key','AL32UTF8')) ) FROM dual;
    ```

  - SQL Server：

    ```sql
    SELECT ENCRYPTBYPASSPHRASE('key', 'abc');
    ```

  - MySQL：

    ```sql
    SELECT AES_ENCRYPT('abc', 'key');
    ```

- **UUID**：
  - Oracle：

    ```sql
    SELECT SYS_GUID() FROM dual;
    ```

  - SQL Server：

    ```sql
    SELECT NEWID();
    ```

  - MySQL：

    ```sql
    SELECT UUID();
    ```

------

### **十七、三大数据库特有函数及特殊场景用法**

| **场景/功能** | **Oracle特有** | **SQL Server特有** | **MySQL特有** | **说明** | **版本要求** |
| ------------- | -------------- | ------------------ | ------------- | -------- | -------------- |
| 分析函数扩展  | `RATIO_TO_REPORT(expr) OVER(...)` | `NTILE(n) OVER(...)` | `GROUP_CONCAT(expr)` | Oracle支持分布占比，SQL Server分桶，MySQL字符串聚合 | 11g+           |
| 层次查询      | `CONNECT BY PRIOR` | `CTE递归（WITH ... AS ...）` | `WITH RECURSIVE ...` | 层级树结构遍历 | 11g+           |
| 行转列        | `PIVOT/UNPIVOT` | `PIVOT/UNPIVOT` | `GROUP_CONCAT+CASE WHEN` | 交叉报表、动态列 | 11g+           |
| 伪列          | `ROWNUM`/`ROWID`/`LEVEL` | `%%physloc%%`/`$IDENTITY` | `ROW_NUMBER()` | 伪列/物理定位 | 11g+           |
| 序列号        | `SYS_GUID()` | `NEWSEQUENTIALID()` | `UUID_SHORT()` | 唯一标识生成 | 12c+           |
| XML处理       | `XMLTYPE`/`EXTRACTVALUE` | `FOR XML PATH` | `ExtractValue`/`XPath` | 结构化数据处理 | 11g+           |
| 分区表管理    | `DBMS_PART` | `SWITCH PARTITION` | `PARTITION BY` | 分区表相关管理 | 11g+           |
| 空间分析      | `SDO_GEOM` | `geometry::STBuffer()` | `ST_Buffer()` | GIS空间分析 | 11g+           |
| 计划/执行分析 | `EXPLAIN PLAN FOR ...` | `SET SHOWPLAN_ALL ON` | `EXPLAIN` | SQL执行计划 | 11g+           |
| 触发器扩展    | `AFTER EACH ROW` | `INSTEAD OF` | `BEFORE/AFTER` | 触发器细粒度控制 | 11g+           |
| 其他          | `DBMS_OUTPUT.PUT_LINE` | `PRINT` | `SELECT '...'` | 控制台输出 | 11g+           |

------

#### 特有函数与特殊场景SQL示例

- **Oracle**：
  - 分布占比：

    ```sql
    SELECT deptno, sal, RATIO_TO_REPORT(sal) OVER(PARTITION BY deptno) AS sal_ratio FROM emp;
    ```

  - 层次查询：

    ```sql
    SELECT empno, ename, mgr FROM emp CONNECT BY PRIOR empno = mgr START WITH mgr IS NULL;
    ```

  - 伪列：

    ```sql
    SELECT ROWNUM, emp.* FROM emp;
    SELECT LEVEL FROM DUAL CONNECT BY LEVEL <= 5;
    ```

  - 控制台输出：

    ```sql
    BEGIN DBMS_OUTPUT.PUT_LINE('Hello Oracle!'); END;
    ```

- **SQL Server**：
  - 分桶：

    ```sql
    SELECT Name, Salary, NTILE(4) OVER(ORDER BY Salary) AS Quartile FROM Employees;
    ```

  - 递归CTE：

    ```sql
    WITH OrgChart AS (
      SELECT empid, mgrid FROM emp WHERE mgrid IS NULL
      UNION ALL
      SELECT e.empid, e.mgrid FROM emp e JOIN OrgChart o ON e.mgrid = o.empid
    ) SELECT * FROM OrgChart;
    ```

  - 新顺序GUID：

    ```sql
    SELECT NEWSEQUENTIALID();
    ```

  - 控制台输出：

    ```sql
    PRINT 'Hello SQL Server!';
    ```

- **MySQL**：
  - 字符串聚合：

    ```sql
    SELECT GROUP_CONCAT(name ORDER BY id) FROM users;
    ```

  - 递归查询：

    ```sql
    WITH RECURSIVE cte AS (
      SELECT id, parent_id FROM tree WHERE parent_id IS NULL
      UNION ALL
      SELECT t.id, t.parent_id FROM tree t JOIN cte ON t.parent_id = cte.id
    ) SELECT * FROM cte;
    ```

  - 短UUID：

    ```sql
    SELECT UUID_SHORT();
    ```

  - 控制台输出：

    ```sql
    SELECT 'Hello MySQL!';
    ```

------

### **十八、数据类型对照表**

| 逻辑类型 | Oracle | SQL Server | MySQL | 说明 |
| -------- | ------ | ---------- | ----- | ---- |
| 字符串   | VARCHAR2(n), CHAR(n), CLOB | VARCHAR(n), NVARCHAR(n), TEXT | VARCHAR(n), CHAR(n), TEXT | 长文本类型命名不同 |
| 数值     | NUMBER(p,s), INTEGER | INT, BIGINT, DECIMAL(p,s) | INT, BIGINT, DECIMAL(p,s) | 精度和范围略有差异 |
| 浮点     | BINARY_FLOAT, BINARY_DOUBLE | FLOAT, REAL | FLOAT, DOUBLE | |
| 日期/时间| DATE, TIMESTAMP | DATETIME, DATE, TIME | DATETIME, DATE, TIME, TIMESTAMP | Oracle无TIME类型 |
| 布尔     | 无（NUMBER(1)或CHAR(1)代替） | BIT | TINYINT(1), BOOL | |
| 二进制   | BLOB, RAW | VARBINARY, IMAGE | BLOB, BINARY | |
| JSON     | 无（CLOB/JSON字段） | NVARCHAR+约束/JSON | JSON | MySQL原生支持JSON |

------

### **十九、迁移注意事项与常见问题**

1. **NULL处理差异**：Oracle的空字符串视为NULL，MySQL区分空字符串和NULL。
2. **分页语法**：MySQL 8.0+、SQL Server 2012+、Oracle 12c+支持标准OFFSET/FETCH，老版本需ROWNUM/ROW_NUMBER等。
3. **布尔类型**：Oracle无原生布尔，需用NUMBER(1)或CHAR(1)；MySQL用TINYINT(1)。
4. **字符串截断**：SQL Server超长字符串需手动截断，Oracle可用SUBSTR。
5. **时区/日期精度**：Oracle的TIMESTAMP精度高，MySQL默认无时区。
6. **函数参数顺序**：如SUBSTR/substring参数顺序不同。
7. **正则表达式**：MySQL 8.0+支持标准正则，SQL Server仅LIKE/PATINDEX。
8. **唯一约束/自增**：Oracle 12c+支持IDENTITY，MySQL用AUTO_INCREMENT，SQL Server用IDENTITY。
9. **批量插入/合并**：Oracle支持INSERT ALL，MySQL用多VALUES，SQL Server用MERGE。
10. **分区表/空间数据**：三库分区和空间类型实现差异大，需单独迁移设计。

------

### **二十、性能优化建议**

- 尽量使用索引字段做条件，避免全表扫描。
- 聚合/窗口函数大表上建议分批处理或加索引。
- 批量DML建议分批提交，防止锁表。
- 字符串聚合、JSON处理等大数据量下建议评估执行计划。
- 合理使用分区表、分区索引提升大表查询性能。
- 避免在WHERE子句中对字段做函数运算。

------

### **二十一、DDL/DML迁移脚本模板**

- **表结构迁移**：
  - Oracle：

    ```sql
    CREATE TABLE t1 (
      id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
      name VARCHAR2(50),
      created_at DATE
    );
    ```

  - SQL Server：

    ```sql
    CREATE TABLE t1 (
      id INT IDENTITY(1,1) PRIMARY KEY,
      name VARCHAR(50),
      created_at DATETIME
    );
    ```

  - MySQL：

    ```sql
    CREATE TABLE t1 (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(50),
      created_at DATETIME
    );
    ```

- **索引迁移**：
  - Oracle：`CREATE INDEX idx_name ON t1(name);`
  - SQL Server/MySQL：`CREATE INDEX idx_name ON t1(name);`
- **视图迁移**：
  - Oracle/SQL Server/MySQL：`CREATE VIEW v1 AS SELECT ... FROM ...;`
- **存储过程/函数迁移**：建议逐条分析，语法差异较大。

------

### **二十二、迁移后测试与验证建议**

- **数据量校验**：

  ```sql
  SELECT COUNT(*) FROM t1;
  ```

- **聚合校验**：

  ```sql
  SELECT SUM(amount), AVG(amount) FROM t1;
  ```

- **样本数据对比**：

  ```sql
  SELECT * FROM t1 WHERE id IN (1,2,3);
  ```

- **主键/唯一约束校验**：

  ```sql
  SELECT id, COUNT(*) FROM t1 GROUP BY id HAVING COUNT(*) > 1;
  ```

- **NULL/空值校验**：

  ```sql
  SELECT COUNT(*) FROM t1 WHERE col IS NULL;
  ```

- **性能对比**：
  - 使用EXPLAIN/EXPLAIN PLAN/SHOWPLAN等分析SQL执行计划。

------
