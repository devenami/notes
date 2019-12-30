## SQL简介

### SQL是什么

* SQL 指结构化查询语言
* SQL 使我们有能力访问数据库
* SQL 是一种 ANSI (美国国家标准化组织)的标准计算机语言

### SQL能做什么

* SQL面向数据库执行查询
* SQL可从数据库取回数据
* SQL可在数据库中插入新的记录
* SQL可更新数据库中的数据
* SQL可从数据库删除记录
* SQL可创建新数据库
* SQL可在数据库中创建新表
* SQL可在数据库中创建存储过程
* SQL可在数据库中创建视图
* SQL可以设置表、存储过程和视图权限

## SQL语法

### DML数据操作语言

* **SELECT** - 从数据库表中获取数据
* **UPDATE** - 更新数据库表中的数据
* **DELETE** - 从数据库表中删除数据
* **INSERT INTO** - 向数据库表中插入数据

### DDL 数据定义语言

* **CREATE DATABASE** - 创建新数据库
* **ALTER DATABASE** - 修改数据库
* **CREATE TABLE** - 创建新表
* **ALTER TABLE** - 变更（改变）数据库表
* **DROP TABLE** - 删除表
* **CREATE INDEX** - 创建索引（搜索键）
* **DROP INDEX** - 删除索引

### SQL select 语句

```sql
SELECT 列名称 FROM 表名称
```

以及

```sql
SELECT * FROM 表名称
```
星号(*****)是选取所有列的快捷方式

### distinct

**DISTINCT**用于返回唯一不同的值。（所有查询列的数据都不相同才会被当做唯一）

```sql
SELECT DISTINCT 列名称 FROM 表名称
```

### where 

**WHERE**子句用于条件过滤

```sql
SELECT 列名称 FROM 表名称 WHERE 列 运算符 值
```

**运算符**

| 操作符   | 描述         |
| :------- | :----------- |
| =        | 等于         |
| <> 或 != | 不等于       |
| >        | 大于         |
| <        | 小于         |
| >=       | 大于等于     |
| <=       | 小于等于     |
| BETWEEN  | 在某个范围内 |
| LIKE     | 搜索某种模式 |


SQL 使用单引号（**‘ ’**）来环绕**文本值**。如果是**数组**，请不要使用引号。

### and & or

```sql
SELECT 列名 FROM 表名 WHERE 列 运算符 值 （AND | OR） 列 运算符 值 ...
```

* 只有**AND**连接的所有条件都成立时，才会显示记录
* 只要**OR**连接的条件中有一个成立的，就会显示记录
* **AND**优先级大于**OR**

### order by

```sql
SELECT 列名 FROM 表名 ORDER BY 列名 （DESC）
```

* ORDER BY 语句用于根据指定的列队结果集进行排序
* ORDER BY 语句默认按照升序对记录进行排序
* **DESC** 可以使记录降序排序

### insert

INSERT INTO 语句用于向表格中插入新的行。

```sql
INSERT INTO 表名 VALUES (值1, 值2,...)
```

也可以指定所要插入数据的列

```sql
INSERT INTO 表名 (列1,列2,...) VALUES (值1, 值2,...)
```

### update

UPDATE 语句用于修改表中的数据。

```sql
UPDATE 表名 SET 列名 = 新值 WHERE 列名 = 某值
```

### delete

DELETE用于删除表中的行

```sql
DELETE FROM 表名 WHERE 列名 = 值
```

### limit

limit在MySQL中用于分页查询

```sql
SELECT 列名 FROM 表名 LIMIT [开始的下标,]每页显示的数量
```

### like

LIKE操作用于在WHERE子句中搜索列中的指定模式

```sql
SELECT 列名 FROM 表名 WHERE 列名 LIKE 匹配模式
```

**通配符**

| 通配符                     | 描述                       |
| :------------------------- | :------------------------- |
| %                          | 替代一个或多个字符         |
| _                          | 仅替代一个字符             |
| [charlist]                 | 字符列中的任何单一字符     |
| [^charlist]或者[!charlist] | 不在字符列中的任何单一字符 |

### in

IN 操作符允许我们在WHERE子句中规定多个值。

```sql
SELECT 列名 FROM 表名 WHERE 列名 IN (值1, 值2,...)
```

### between 

操作符 BETWEEN ... AND 会选取介于两个值之间的数据范围。这些值可以是数值、文本或者日期。

```sql
SELECT 列名 FROM 表名 WHERE 列名 (NOT) BETWEEN 值1 AND 值2
```

* BETWEEN a AND b，包括a，但是不包括b
* **NOT** 修饰符用于查询不在 between and 范围内的数据

### alias（别名）

SQL中使用 **AS** 关键字为表或字段设置别名，**AS** 可以省略

```sql
SELECT 列名 AS 列别名 FROM 表名 AS 表别名
```

### join

**引用两张表**

```sql
SELECT 表1.字段1, 表2.字段2 FROM 表1, 表2 WHERE 表1.KEY = 表2.KEY
```

**join**

```sql
SELECT 表1.字段1, 表2.字段2 FROM 表1 INNER JOIN 表2 ON 表1.KEY = 表2.KEY
```

**join**不同类型

* **JOIN** ：如果表中至少有一个匹配，则返回行
* **LEFT JOIN** ：即使右表中没有匹配，也从左表返回所有的行
* **RIGHT JOIN** ：即使左表中没有匹配，也从右表中返回所有的行
* **INNER JOIN** ：左右表中同时匹配才返回行
* **FULL JOIN** ：只要其中一个表中存在匹配，就返回行

### union

**UNION** 操作符用于合并两个或多个 **SELECT** 语句的结果集

* UNION 内部的 SELECT 语句必须拥有**相同数量的列**。
* 列也必须拥有相似的**数据类型**
* 每条 SELECT 语句中的列的**顺序**必须相同
* UNION 结果集中的列总是等于 UNION 中**第一个SELECT**语句中的列名

**union 语法**

```sql
SELECT 列名 FROM 表1
UNION
SELECT 列名 FROM 表2
```

**UNION操作符会选取不同的值，相同的值则只留一条**

**union all 语法**

```sql
SELECT 列名 FROM 表1
UNION ALL
SELECT 列名 FROM 表2
```

**UNION ALL操作符允许相同的数据存在**

### select into

SELECT INTO 语句可用于创建表的备份复件， SELECT INTO 语句从一个表中获取数据，然后把数据插入另一个表中。

```sql
SELECT 列名 
INTO 新表名 [IN 其他的数据库]
FROM 旧表名
```

* 可用于制作数据副本 **SELECT * INTO 备份表 FROM 原表**
* 用带**WHERE**条件
* 可进行**表关联**

### create db

```sql
CREATE DATABASE 数据库名称
```

### create table 

```sql
CREATE TABLE 表名 
(
	列名1 数据类型,
    列名2 数据类型,
    列名3 数据类型,
    ...
)
```

**数据类型**

| 数据类型                                          | 描述                                                         |
| :------------------------------------------------ | :----------------------------------------------------------- |
| integer(size)int(size)smallint(size)tinyint(size) | 仅容纳整数。在括号内规定数字的最大位数。                     |
| decimal(size,d)numeric(size,d)                    | 容纳带有小数的数字。"size" 规定数字的最大位数。"d" 规定小数点右侧的最大位数。 |
| char(size)                                        | 容纳固定长度的字符串（可容纳字母、数字以及特殊字符）。在括号中规定字符串的长度。 |
| varchar(size)                                     | 容纳可变长度的字符串（可容纳字母、数字以及特殊的字符）。在括号中规定字符串的最大长度。 |
| date(yyyymmdd)                                    | 容纳日期。                                                   |
| datetime(yyyymmdd HH:MM:ss)                       | 容纳具体时间。                                               |

### not null

NOT NULL 约束强制列不接受 NULL 值

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

### unique

UNIQUE 约束唯一标识数据库表中的每条记录

UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。

PRIMARY KEY 拥有自动定义的 UNIQUE 约束。

请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

**创建表时添加 unique 约束**

*MySQl* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (Id_P)
)
```

*SQL Server / Oracle / MS Access* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL UNIQUE,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果要为多个列定义 UNIQUE 约束，并为该约束设置名称，可以使用 **CONSTRAINT** 关键字

*MySQL / SQL Server / Oracle / MS Access* ：

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
)
```

**增加unique约束**

当表已被创建时，如需在 "Id_P" 列创建 UNIQUE 约束，请使用下列 SQL：

*MySQL / SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ADD UNIQUE (Id_P)
```

如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

*MySQL / SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
```

**删除unique约束**

如需撤销 UNIQUE 约束，请使用下面的 SQL：

*MySQL* :

```sql
ALTER TABLE Persons
DROP INDEX uc_PersonID
```

*SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
DROP CONSTRAINT uc_PersonID
```

### primary key

PRIMARY KEY 约束唯一标识数据库表中的每条记录。

主键必须包含唯一的值。

主键列不能包含 NULL 值。

每个表都应该有一个主键，并且每个表只能有一个主键。

**创建表时设置主键**

*MySQL* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (Id_P)
)
```

*SQL Server / Oracle / MS Access* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL PRIMARY KEY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

*MySQL / SQL Server / Oracle / MS Access* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
)
```

**增加主键约束**

*MySQL / SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ADD PRIMARY KEY (Id_P)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

*MySQL / SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
```

**注释：**如果您使用 ALTER TABLE 语句添加主键，必须把主键列声明为不包含 NULL 值（在表首次创建时）

**删除主键约束**

*MySQL* :

```sql
ALTER TABLE Persons
DROP PRIMARY KEY
```

*SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```

### check

CHECK 约束用于限制列中的值的范围。

如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。

**创建表时增加check约束**

*My SQL* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (Id_P>0)
)
```

*SQL Server / Oracle / MS Access* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL CHECK (Id_P>0),
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法：

*MySQL / SQL Server / Oracle / MS Access* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
)
```

**增加check约束**

*MySQL / SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ADD CHECK (Id_P>0)
```

如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法：

*MySQL / SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ADD CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
```

**删除check约束**

*SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
DROP CONSTRAINT chk_Person
```

*MySQL* :

```sql
ALTER TABLE Persons
DROP CHECK chk_Person
```

### default

DEFAULT 约束用于向列中插入默认值。

如果没有规定其他的值，那么会将默认值添加到所有的新记录。

**创建表时设置默认值**

*My SQL / SQL Server / Oracle / MS Access* :

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255) DEFAULT 'Sandnes'
)
```

通过使用类似 GETDATE() 这样的函数，DEFAULT 约束也可以用于插入系统值：

```sql
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
OrderDate date DEFAULT GETDATE()
)
```

**增加默认值**

*MySQL* :

```sql
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
```

*SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ALTER COLUMN City SET DEFAULT 'SANDNES'
```

**删除默认值**

*MySQL* :

```sql
ALTER TABLE Persons
ALTER City DROP DEFAULT
```

*SQL Server / Oracle / MS Access* :

```sql
ALTER TABLE Persons
ALTER COLUMN City DROP DEFAULT
```

### create index

CREATE INDEX 语句用于在表中创建索引。

在不读取整个表的情况下，索引使数据库应用程序可以更快的查找数据

更新一个包含索引的表需要比更新一个没有索引的表要花费更多的时间，这是由于索引本身也需要更新。

因此，理想的做法是仅在常被搜索的列(以及表)上面创建索引。

**简单索引**

```sql
CREATE INDEX index_name
ON table_name (column_name)
```

**唯一性索引**

唯一性索引意味着两个行不能拥有相同的索引值

```sql
CREATE UNIQUE INDEX index_name
ON table_name (column_name)
```

### drop

使用 drop 语句，可以删除索引、表和数据库

**删除索引**

*用于 Microsoft SQLJet (以及 Microsoft Access) 的语法* :

```sql
DROP INDEX index_name ON table_name
```

*用于 MS SQL Server 的语法* :

```sql
DROP INDEX table_name.index_name
```

*用于 IBM DB2 和 Oracle 语法* :

```sql
DROP INDEX index_name
```

*用于 MySQL 的语法* :

```sql
ALTER TABLE table_name DROP INDEX index_name
```

**删除表**

```sql
DROP TABLE 表名
```

**删除数据库**

```sql
DROP DATABASE 数据库名称
```

**清空表中数据，但是不删除表**

```sql
TRUNCATE TABLE 表名称
```

### alter

如需在表中添加列，请使用下列语法:

```sql
ALTER TABLE table_name
ADD column_name datatype
```

要删除表中的列，请使用下列语法：

```sql
ALTER TABLE table_name 
DROP COLUMN column_name
```

**注释：**某些数据库系统不允许这种在数据库表中删除列的方式 (DROP COLUMN column_name)。

要改变表中列的数据类型，请使用下列语法：

```sql
ALTER TABLE table_name
ALTER COLUMN column_name datatype
```

### increment

MySQL 使用 **AUTO_INCREMENT** 关键字来执行 auto-increment 任务。

默认地，AUTO_INCREMENT 的开始值是 1，每条新记录递增 1。

要让 AUTO_INCREMENT 序列以其他的值起始，请使用下列 SQL 语法：

```sql
ALTER TABLE Persons AUTO_INCREMENT=100
```

MS SQL/SQL Server 使用 IDENTITY 关键字来执行 auto-increment 任务。

默认地，IDENTITY 的开始值是 1，每条新记录递增 1。

要规定 "P_Id" 列以 20 起始且递增 10，请把 identity 改为 IDENTITY(20,10)

### view

在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表。

视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。我们可以向视图添加 SQL 函数、WHERE 以及 JOIN 语句，我们也可以提交数据，就像这些来自于某个单一的表。

**注释：**数据库的设计和结构不会受到视图中的函数、where 或 join 语句的影响。

**创建视图**

```
CREATE VIEW view_name AS
SELECT column_name(s)
FROM table_name
WHERE condition
```

**注释：**视图总是显示最近的数据。每当用户查询视图时，数据库引擎通过使用 SQL 语句来重建数据。

**更新视图**

```sql
CREATE OR REPLACE VIEW view_name AS
SELECT column_name(s)
FROM table_name
WHERE condition
```

**删除视图**

```sql
DROP VIEW view_name
```

### date

MySQL 使用下列数据类型在数据库中存储日期或日期/时间值：

- DATE - 格式 YYYY-MM-DD
- DATETIME - 格式: YYYY-MM-DD HH:MM:SS
- TIMESTAMP - 格式: YYYY-MM-DD HH:MM:SS
- YEAR - 格式 YYYY 或 YY

下面的表格列出了 MySQL 中最重要的内建日期函数：

| 函数          | 描述                                |
| :------------ | :---------------------------------- |
| NOW()         | 返回当前的日期和时间                |
| CURDATE()     | 返回当前的日期                      |
| CURTIME()     | 返回当前的时间                      |
| DATE()        | 提取日期或日期/时间表达式的日期部分 |
| EXTRACT()     | 返回日期/时间按的单独部分           |
| DATE_ADD()    | 给日期添加指定的时间间隔            |
| DATE_SUB()    | 从日期减去指定的时间间隔            |
| DATEDIFF()    | 返回两个日期之间的天数              |
| DATE_FORMAT() | 用不同的格式显示日期/时间           |

**实例表**

| OrderId | ProductName | OrderDate               |
| :------ | :---------- | :---------------------- |
| 1       | 'Computer'  | 2008-12-29 16:25:46.635 |

**DATE(date)**

```sql
SELECT ProductName, DATE(OrderDate) AS OrderDate
FROM Orders
WHERE OrderId=1
```

结果：

| ProductName | OrderDate  |
| :---------- | :--------- |
| 'Computer'  | 2008-12-29 |

**EXTRACT(unit FROM date)**

*date* 参数是合法的日期表达式。*unit* 参数可以是下列的值：

| Unit 值            |
| :----------------- |
| MICROSECOND        |
| SECOND             |
| MINUTE             |
| HOUR               |
| DAY                |
| WEEK               |
| MONTH              |
| QUARTER            |
| YEAR               |
| SECOND_MICROSECOND |
| MINUTE_MICROSECOND |
| MINUTE_SECOND      |
| HOUR_MICROSECOND   |
| HOUR_SECOND        |
| HOUR_MINUTE        |
| DAY_MICROSECOND    |
| DAY_SECOND         |
| DAY_MINUTE         |
| DAY_HOUR           |
| YEAR_MONTH         |

```sql
SELECT EXTRACT(YEAR FROM OrderDate) AS OrderYear,
EXTRACT(MONTH FROM OrderDate) AS OrderMonth,
EXTRACT(DAY FROM OrderDate) AS OrderDay
FROM Orders
WHERE OrderId=1
```

结果：

| OrderYear | OrderMonth | OrderDay |
| :-------- | :--------- | :------- |
| 2008      | 12         | 29       |

**DATE_ADD(date,INTERVAL expr type)**

*date* 参数是合法的日期表达式。*expr* 参数是您希望添加的时间间隔。type 参照 EXTRACT() 函数的 unit

```
SELECT OrderId,DATE_ADD(OrderDate,INTERVAL 2 DAY) AS OrderPayDate
FROM Orders
```

结果：

| OrderId | OrderPayDate            |
| :------ | :---------------------- |
| 1       | 2008-12-31 16:25:46.635 |

**DATEDIFF(date1,date2)**
*date1* 和 *date2* 参数是合法的日期或日期/时间表达式。

**注释：**只有值的日期部分参与计算。

**DATE_FORMAT(date,format)**

*date* 参数是合法的日期。*format* 规定日期/时间的输出格式。

可以使用的格式有：

| 格式 | 描述                                           |
| :--- | :--------------------------------------------- |
| %a   | 缩写星期名                                     |
| %b   | 缩写月名                                       |
| %c   | 月，数值                                       |
| %D   | 带有英文前缀的月中的天                         |
| %d   | 月的天，数值(00-31)                            |
| %e   | 月的天，数值(0-31)                             |
| %f   | 微秒                                           |
| %H   | 小时 (00-23)                                   |
| %h   | 小时 (01-12)                                   |
| %I   | 小时 (01-12)                                   |
| %i   | 分钟，数值(00-59)                              |
| %j   | 年的天 (001-366)                               |
| %k   | 小时 (0-23)                                    |
| %l   | 小时 (1-12)                                    |
| %M   | 月名                                           |
| %m   | 月，数值(00-12)                                |
| %p   | AM 或 PM                                       |
| %r   | 时间，12-小时（hh:mm:ss AM 或 PM）             |
| %S   | 秒(00-59)                                      |
| %s   | 秒(00-59)                                      |
| %T   | 时间, 24-小时 (hh:mm:ss)                       |
| %U   | 周 (00-53) 星期日是一周的第一天                |
| %u   | 周 (00-53) 星期一是一周的第一天                |
| %V   | 周 (01-53) 星期日是一周的第一天，与 %X 使用    |
| %v   | 周 (01-53) 星期一是一周的第一天，与 %x 使用    |
| %W   | 星期名                                         |
| %w   | 周的天 （0=星期日, 6=星期六）                  |
| %X   | 年，其中的星期日是周的第一天，4 位，与 %V 使用 |
| %x   | 年，其中的星期一是周的第一天，4 位，与 %v 使用 |
| %Y   | 年，4 位                                       |
| %y   | 年，2 位                                       |

```sql
DATE_FORMAT(NOW(),'%b %d %Y %h:%i %p')
DATE_FORMAT(NOW(),'%m-%d-%Y')
DATE_FORMAT(NOW(),'%d %b %y')
DATE_FORMAT(NOW(),'%d %b %Y %T:%f')
```

结果类似：

```sql
Dec 29 2008 11:45 PM
12-29-2008
29 Dec 08
29 Dec 2008 16:25:46.635
```

### null

如果表中的某个列是可选的，那么我们可以在不向该列添加值的情况下插入新记录或更新已有的记录。这意味着该字段将以 NULL 值保存。

NULL 值不能使用比较运算符，如 = ， < 或者 <>， 只能使用 IS NULL 和 IS NOT NULL 操作符

**ISNULL(列名, 默认值)** 为 SQLServer 提供的处理 NULL 值的函数

**IFNULL(列名, 默认值)** 为 MySQL 提供的处理 NULL 的函数

### for update

**场景**

存在**高并发**，并且对于数据的**准确性**很有要求的场景

**原则**

一锁二判三更新

**使用方式**

```sql
SELECT * FROM TABLE WHERE xxx FOR UPDATE
```

**锁级别**

* InnoDB默认是行级别的锁
* 有明确指定的主键时为行级别的锁，否则为表级别

**注：**

* for update 仅适用于InnoDB，并且必须开启事务，在begin与commit之间才生效。
* 要测试for update的锁表情况，可以利用MySQL的Command Mode，开启二个视窗来做测试。

## SQL函数

### function

**函数的语法**

内建 SQL 函数的语法是：

```
SELECT function(列) FROM 表
```

**函数的类型**

在 SQL 中，基本的函数类型和种类有若干种。函数的基本类型是：

- Aggregate 函数
- Scalar 函数

**合计函数（Aggregate functions）**

Aggregate 函数的操作面向一系列的值，并返回一个单一的值。

**注释：**如果在 SELECT 语句的项目列表中的众多其它表达式中使用 SELECT 语句，则这个 SELECT 必须使用 GROUP BY 语句！

**Scalar 函数**

Scalar 函数的操作面向某个单一的值，并返回基于输入值的一个单一的值。

### avg()

AVG 函数返回数值列的平均值。NULL 值不包括在计算中。

```sql
SELECT AVG(column_name) FROM table_name
```

### count()

**SQL COUNT(column_name) 语法**

COUNT(column_name) 函数返回指定列的值的数目（NULL 不计入）：

```sql
SELECT COUNT(column_name) FROM table_name
```

**SQL COUNT(*) 语法**

COUNT(*) 函数返回表中的记录数：

```sql
SELECT COUNT(*) FROM table_name
```

**SQL COUNT(DISTINCT column_name) 语法**

COUNT(DISTINCT column_name) 函数返回指定列的不同值的数目：

```sql
SELECT COUNT(DISTINCT column_name) FROM table_name
```

**注释：**COUNT(DISTINCT) 适用于 ORACLE 和 Microsoft SQL Server，但是无法用于 Microsoft Access。

### first()

FIRST() 函数返回指定的字段中第一个记录的值。

**提示：**可使用 ORDER BY 语句对记录进行排序。

**SQL FIRST() 语法**

```sql
SELECT FIRST(column_name) FROM table_name
```

### last()

LAST() 函数返回指定的字段中最后一个记录的值。

**提示：**可使用 ORDER BY 语句对记录进行排序。

**SQL LAST() 语法**

```sql
SELECT LAST(column_name) FROM table_name
```

### max()、min()

MAX 函数返回一列中的最大值。NULL 值不包括在计算中。

MIN 函数返回一列中的最小值。NULL 值不包括在计算中。

**SQL MAX() 语法**

```sql
SELECT MAX(column_name) FROM table_name
```

**注释：**MIN 和 MAX 也可用于文本列，以获得按字母顺序排列的最高或最低值。

### sum()

SUM 函数返回数值列的总数（总额）。

**SQL SUM() 语法**

```sql
SELECT SUM(column_name) FROM table_name
```

### group by

GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。

**SQL GROUP BY 语法**

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
```

### having

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。

**SQL HAVING 语法**

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```

### ucase()、lcase()

UCASE 函数把字段的值转换为大写。LCASE 函数把字段的值转换为小写。

**SQL UCASE() 语法**

```sql
SELECT UCASE(column_name) FROM table_name
```

### mid()

MID 函数用于从文本字段中提取字符。

**SQL MID() 语法**

```sql
SELECT MID(column_name,start[,length]) FROM table_name
```

| 参数        | 描述                                                        |
| :---------- | :---------------------------------------------------------- |
| column_name | 必需。要提取字符的字段。                                    |
| start       | 必需。规定开始位置（起始值是 1）。                          |
| length      | 可选。要返回的字符数。如果省略，则 MID() 函数返回剩余文本。 |

### len()

LEN 函数返回文本字段中值的长度。

**SQL LEN() 语法**

```sql
SELECT LEN(column_name) FROM table_name
```

### round()

ROUND 函数用于把数值字段四舍五入为指定的小数位数。

**SQL ROUND() 语法**

```sql
SELECT ROUND(column_name,decimals) FROM table_name
```

| 参数        | 描述                         |
| :---------- | :--------------------------- |
| column_name | 必需。要舍入的字段。         |
| decimals    | 必需。规定要返回的小数位数。 |

