# SQLite

主要记一些基础知识和概念，具体用法会简单记录一些

## RDBMS

关系型数据库管理系统（Relational Database Management System，RDBMS），RDBMS将数据组织为相关的行和列，它通过数据、关系、和对数据的约束组成的数据模型来存放和管理数据

**特点**

- 数据以表格的形式出现
- 每行为各种记录名称
- 每列为记录名称所对应的数据域
- 许多的行和列组成一张表单
- 若干表单组成database

**为什么要用SQLite**

- 不需要单独的服务器或操作系统
- 无需配置，无需安装管理
- 一个完整的SQLite是存储在一个单一的跨平台的磁盘文件
- 轻量级
- 自给自足，无需外部依赖
- 事务兼容ACID*
- 支持SQL92（SQL2)标准的大多数查询语言的功能
- 用ANSI-C编写
- 多平台支持

**ACID**：指数据库事务正确执行的四个基本要素，原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability），必须要有这四种特性，才能保证在事务过程（Transaction Processing）中保证数据的正确性。

## 安装SQLite3

到https://www.sqlite.org/download.html下载带autoconf的tar.gz源码包

tar -zxvf ???

make

sudo make install

就ok了

---

## SQLite命令

CREATE SELECT INSERT UPDATE DELETE DROP

**DDL 数据定义语言**

- CREATE 创建一个新的表，一表的视图，或者数据库中的其他对象
- ALTER 修改数据库中的某个已有的数据库对象，比如一个表
- DROP 删除整个表，或者表的视图，或者数据库中的其他对象

**DML 数据操作语言**

- INSERT 创建一条记录
- UPDATE 修改记录
- DELETE 删除记录

**DQL 数据查询语言**

- SELECT 从一个或多个表中检索某些记录

*在命令行键入sqlite3，可以使用命令*

`.help`：可以输出可用的命令清单

| 命令                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| `.backup ?DB? FILE`     | 备份 DB 数据库（默认是 "main"）到 FILE 文件。                |
| `.bail ON\|OFF`         | 发生错误后停止。默认为 OFF。                                 |
| `.databases`            | 列出数据库的名称及其所依附的文件。                           |
| `.dump ?TABLE?`         | 以 SQL 文本格式转储数据库。如果指定了 TABLE 表，则只转储匹配 LIKE 模式的 TABLE 表。 |
| `.echo ON\|OFF`         | 开启或关闭 echo 命令。                                       |
| `.exit`                 | 退出 SQLite 提示符。                                         |
| `.explain ON\|OFF`      | 开启或关闭适合于 EXPLAIN 的输出模式。如果没有带参数，则为 EXPLAIN on，及开启 EXPLAIN。 |
| `.header(s) ON\|OFF`    | 开启或关闭头部显示。                                         |
| `.help`                 | 显示消息。                                                   |
| `.import FILE TABLE`    | 导入来自 FILE 文件的数据到 TABLE 表中。                      |
| `.indices ?TABLE?`      | 显示所有索引的名称。如果指定了 TABLE 表，则只显示匹配 LIKE 模式的 TABLE 表的索引。 |
| `.load FILE ?ENTRY?`    | 加载一个扩展库。                                             |
| `.log FILE\|off`        | 开启或关闭日志。FILE 文件可以是 stderr（标准错误）/stdout（标准输出）。 |
| `.mode MODE`            | 设置输出模式，MODE 可以是下列之一：<br>**csv** 逗号分隔的值<br>**column** 左对齐的列<br>**html** HTML 的 <table> 代码<br>**insert** TABLE 表的 SQL 插入（insert）语句<br>**line** 每行一个值<br>**list** 由 .separator 字符串分隔的值<br>**tabs** 由 Tab 分隔的值<br>**tcl** TCL 列表元素 |
| `.nullvalue STRING`     | 在 NULL 值的地方输出 STRING 字符串。                         |
| `.output FILENAME`      | 发送输出到 FILENAME 文件。                                   |
| `.output stdout`        | 发送输出到屏幕。                                             |
| `.print STRING...`      | 逐字地输出 STRING 字符串。                                   |
| `.prompt MAIN CONTINUE` | 替换标准提示符。                                             |
| `.quit`                 | 退出 SQLite 提示符。                                         |
| `.read FILENAME`        | 执行 FILENAME 文件中的 SQL。                                 |
| `.schema ?TABLE?`       | 显示 CREATE 语句。如果指定了 TABLE 表，则只显示匹配 LIKE 模式的 TABLE 表。 |
| `.separator STRING`     | 改变输出模式和 .import 所使用的分隔符。                      |
| `.show`                 | 显示各种设置的当前值。                                       |
| `.stats ON\|OFF`        | 开启或关闭统计。                                             |
| `.tables ?PATTERN?`     | 列出匹配 LIKE 模式的表的名称。                               |
| `.timeout MS`           | 尝试打开锁定的表 MS 毫秒。                                   |
| `.width NUM NUM`        | 为 "column" 模式设置列宽度。                                 |
| `.timer ON\|OFF`        | 开启或关闭 CPU 定时器。                                      |

命令都是以.开头的，我觉得是为了区分SQL和sqlite3的命令吧

**sqlite_master 表格**

```sql lite
.schema sqlite_master
```

将输出

```sql lite
CREATE TABLE sqlite_master (
  type text,
  name text,
  tbl_name text,
  rootpage integer,
  sql text
);
```

## SQLite语法

### 大小写

部分区分大小写

### 注释

不能嵌套

单行注释，两个`-`

可以用`/**/`

## SQLite语句

以任何关键字开始，如SELECT、INSERT、UPDATE、DELETE、ALTER、DROP等，以分号（;）结束

### SQLite ANALYZE 语句：

```sql lite
ANALYZE;
```

```sql lite
ANALYZE database_name;
```

```sql lite
ANALYZE database_name.table_name;
```

### SQLite AND/OR 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  CONDITION-1 {AND|OR} CONDITION-2;
```

### SQLite ALTER TABLE 语句：

```sql lite
ALTER TABLE table_name ADD COLUMN column_def...;
```

### SQLite ALTER TABLE 语句（Rename）：

```sql lite
ALTER TABLE table_name RENAME TO new_table_name;
```

### SQLite ATTACH DATABASE 语句：

```sql lite
ATTACH DATABASE 'DatabaseName' As 'Alias-Name';
```

### SQLite BEGIN TRANSACTION 语句：

```sql lite
BEGIN;
```

```sql lite
BEGIN EXCLUSIVE TRANSACTION;
```

### SQLite BETWEEN 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name BETWEEN val-1 AND val-2;
```

### SQLite COMMIT 语句：

```sql lite
COMMIT;
```

### SQLite CREATE INDEX 语句：

```sql lite
CREATE INDEX index_name
ON table_name ( column_name COLLATE NOCASE );
```

### SQLite CREATE UNIQUE INDEX 语句：

```sql lite
CREATE UNIQUE INDEX index_name
ON table_name ( column1, column2,...columnN);
```

### SQLite CREATE TABLE 语句：

```sql lite
CREATE TABLE table_name(
   column1 datatype,
   column2 datatype,
   column3 datatype,
   .....
   columnN datatype,
   PRIMARY KEY( one or more columns )
);
```

### SQLite CREATE TRIGGER 语句：

```sql lite
CREATE TRIGGER database_name.trigger_name 
BEFORE INSERT ON table_name FOR EACH ROW
BEGIN 
   stmt1; 
   stmt2;
   ....
END;
```

### SQLite CREATE VIEW 语句：

```sql lite
CREATE VIEW database_name.view_name  AS
SELECT statement....;
```

### SQLite CREATE VIRTUAL TABLE 语句：

```sql lite
CREATE VIRTUAL TABLE database_name.table_name USING weblog( access.log );
or
CREATE VIRTUAL TABLE database_name.table_name USING fts3( );
```

### SQLite COMMIT TRANSACTION 语句：

```sql lite
COMMIT;
```

### SQLite COUNT 子句：

```sql lite
SELECT COUNT(column_name)
FROM   table_name
WHERE  CONDITION;
```

### SQLite DELETE 语句：

```sql lite
DELETE FROM table_name
WHERE  {CONDITION};
```

### SQLite DETACH DATABASE 语句：

```sql lite
DETACH DATABASE 'Alias-Name';
```

### SQLite DISTINCT 子句：

```sql lite
SELECT DISTINCT column1, column2....columnN
FROM   table_name;
```

### SQLite DROP INDEX 语句：

```sql lite
DROP INDEX database_name.index_name;
```

### SQLite DROP TABLE 语句：

```sql lite
DROP TABLE database_name.table_name;
```

### SQLite DROP VIEW 语句：

```sql lite
DROP VIEW view_name;
```

### SQLite DROP TRIGGER 语句：

```sql lite
DROP TRIGGER trigger_name
```

### SQLite EXISTS 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name EXISTS (SELECT * FROM   table_name );
```

### SQLite EXPLAIN 语句：

```sql lite
EXPLAIN INSERT statement...;
or 
EXPLAIN QUERY PLAN SELECT statement...;
```

### SQLite GLOB 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name GLOB { PATTERN };
```

### SQLite GROUP BY 子句：

```sql lite
SELECT SUM(column_name)
FROM   table_name
WHERE  CONDITION
GROUP BY column_name;
```

### SQLite HAVING 子句：

```sql lite
SELECT SUM(column_name)
FROM   table_name
WHERE  CONDITION
GROUP BY column_name
HAVING (arithematic function condition);
```

### SQLite INSERT INTO 语句：

```sql lite
INSERT INTO table_name( column1, column2....columnN)
VALUES ( value1, value2....valueN);
```

### SQLite IN 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name IN (val-1, val-2,...val-N);
```

### SQLite Like 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name LIKE { PATTERN };
```

### SQLite NOT IN 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name NOT IN (val-1, val-2,...val-N);
```

### SQLite ORDER BY 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  CONDITION
ORDER BY column_name {ASC|DESC};
```

### SQLite PRAGMA 语句：

```sql lite
PRAGMA pragma_name;

For example:

PRAGMA page_size;
PRAGMA cache_size = 1024;
PRAGMA table_info(table_name);
```

### SQLite RELEASE SAVEPOINT 语句：

```sql lite
RELEASE savepoint_name;
```

### SQLite REINDEX 语句：

```sql lite
REINDEX collation_name;
REINDEX database_name.index_name;
REINDEX database_name.table_name;
```

### SQLite ROLLBACK 语句：

```sql lite
ROLLBACK;
or
ROLLBACK TO SAVEPOINT savepoint_name;
```

### SQLite SAVEPOINT 语句：

```sql lite
SAVEPOINT savepoint_name;
```

### SQLite SELECT 语句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name;
```

### SQLite UPDATE 语句：

```sql lite
UPDATE table_name
SET column1 = value1, column2 = value2....columnN=valueN
[ WHERE  CONDITION ];
```

### SQLite VACUUM 语句：

```sql lite
VACUUM;
```

### SQLite WHERE 子句：

```sql lite
SELECT column1, column2....columnN
FROM   table_name
WHERE  CONDITION;
```
## SQLite数据类型

SQLite数据类型是一个用来指定任何对象的数据类型的属性，每一列，每个变量和表达式都有相关的数据类型。

可以在创建表的同时使用数据类型，SQLite使用一个更普遍的动态类型系统，在SQLite中，值的数据类型与值本身相关，而不是与创建它的容器相关

### SQLite存储类

| 存储类  | 描述                                       |
| ------- | ------------------------------------------ |
| NULL    | 值是一个NULL值                             |
| INTEGER | 有符号整数，根据值的大小保存在123468字节中 |
| REAL    | 浮点数，8字节IEEE浮点数字                  |
| TEXT    | 字符串，UTF-8、UTF-16BE、UTF-16LE          |
| BLOB    | 根据输入存储                               |

### SQLite亲和（Affinity）类型

| 亲和类型 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| TEXT     | 数值类型在插入之前，先被转换为文本格式，之后再插入目标字段   |
| NUMERIC  | 文本数据被插入到亲和类型为NUMERIC字段中时，如果转换操作不会导致数据信息丢失并完全可逆，那么SQLite就会将该文本数据转换为INTEGER或REAL类型，若转换失败，会以TEXT存储。对于NULL和BLOB类型数据将不进行任何转换，直接以BULL或BLOB类型存储。浮点格式的常量文本，如果在转换为INTEGER的时候不会丢失信息，那么就会转换为INTEGER的方式进行存储。 |
| INTEGER  | 亲缘类型为INTEGER类型的字段，其规则等同于NUMERIC。差别是执行CAST表达式时。 |
| REAL     | 规则基本等同于NUMERIC，差别是不会将3000.0这样的文本数据转换为INTEGER的方式存储。 |
| NONE     | 不做任何转换                                                 |

### SQL亲和类型（Affinity）及类型名称

| 数据类型                                                     | 亲和类型 |
| ------------------------------------------------------------ | -------- |
| - INT<br>INTEGER<br>TINYINT<br>SMAILLINT<br>MEDIUMINT<br>BIGINT<br>UNSIGNED BIG INT<br>INT2<br>INT8 | INTEGER  |
| CHARACTER(20)<br>VARCHAR(25)<br>VARYING CHARACTER(255)<br>NCHAR(55)<br>NATIVE CHARACTER(70)<br>NVARCHAR(100)<br>TEXT<br>CLOB | TEXT     |
| BLOB<br>no datatype specified                                | NONE     |
| REAL<br>DOUBLE<br>DOUBLE PRECISION<br>FLOAT                  | REAL     |
| NUMERIC<br>DECIMAL(10,5)<br>BOOLEAN<br>DATE<br>DATETIME      | NUMERIC  |

### Boolean数据类型

并没有这个数据类型，被储存为0(false)和1(true)

### Date与Time数据类型

也没有这个数据类型，可以把日期和时间存储为TEXT、REAL、INTEGER

| 存储类  | 日期格式                                           |
| ------- | -------------------------------------------------- |
| TEXT    | 格式为"YYYY-MM-DD HH:MM:SS.SSS"                    |
| REAL    | 从公元前4714年11月24日格林尼治时间的正午算起的天数 |
| INTEGER | 从1970-01-01 00:00:00 UTC算起的秒数                |

可以使用内置的时间函数来转换格式

## SQLite创建数据库

### 语法

```sql lite
sqlite3 [DatabaseName.db]
```

通常数据库名应该是唯一的

### 实例

*测试的例子都写在了`~/work_dir/C_test_prjs/sqlite3_test`下面*

```sql lite
sqlite3 test.db
sqlite> .databases
sqlite> test.db .dump > test.sql
sqlite> .quit
```

用`.databases`查看，但是却没有发现刚刚创建的数据库

用`.dump`导出到文本文件

用`.quit`退出

## 附加数据库

同时有多个数据库可用的时候，使用`ATTACH DATABASE`用来选择一个特定的数据库

### 语法

```sql lite
sqlite> ATTACH DATABASE 'DatabaseName' AS 'Alias-Name';
```

实例

```sql lite
sqlite> ATTACH DATABASE 'test.db' AS 'test';
```

注意单引号，此时执行

```bash
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main                                                                       
2    test             /home/jiangdongchao/work_dir/C_test_prjs/sqlite3_test/dbs/
```

main和temp是保留的

## 分离数据库

把连接的别名和数据库进行分离。连接是之前的`ATTACH DATABASE`进行的，如果有一个数据库文件有多个别名，那么只会断开相应的那个，main和temp无法被分离

### 语法

```sql lite
sqlite> DETACH DATABASE 'Alias-Name';
```

## 创建表

`CREATE TABLE`创建表，涉及到命名表、定义列及每列的数据类型

### 语法

```sql lite
sqlite> CREATE TABLE database_name.table_name(
    column1 datatype PRIMARY KEY(one or more columns),
    column2 datatype,
    ...
    column3 datatype,
);
```

### 实例

```sql lite
sqlite> CREATE TABLE test.phnbk(
   ...> ID INT PRIMARY KEY NOT NULL,
   ...> NAME TEXT NOT NULL,
   ...> AGE INT NOT NULL,
   ...> ADDRESS CHAR(50),
   ...> SALARY ERAL
   ...> );
```

这里：

```sql lite
sqlite3 anothertest.db
sqlite> ATTACH DATABASE `anothertest` AS `another`
sqlite> CREATE TABLE anothertest.dept(
   ...> ID INT PRIMARY KEY NOT NULL,
   ...> DEP CHAR(50) NOT NULL,
   ...> EMP_ID INT NOT NULL
   ...> );
```

此时

```sql lite
sqlite> .tables
another.dept  test.phnbk  
```

也就是`.tables`会列出当前附加的数据库中的所有表

**emm**，测试`.schema`怎么都输出不了有用的东西

## 删除表

`DROP TABLE`用于删除表的定义，数据，索引，触发器，约束和表的权限规范

### 语法

```sql lite
sqlite> DROP TABLE database_name.table_name;
```

### 实例

若表不存在，会提示

**另外**，当两个都`ATTACH`了的数据库中有相同名字的某个表时，执行`DROP`

```sql lite
sqlite> .tables
another.testT  test.phnbk     test.testT   
sqlite> DROP TABLE testT;
sqlite> .tables         
test.phnbk  test.testT
sqlite> DROP TABLE testT;
sqlite> .tables         
test.phnbk
```

所以我觉得应该尽量加上数据库名，防止出现误删的情况。

## INSERT

`INSERT INTO`

## 语法

```sql lite
sqlite> INSERT INTO TABLE_NAME [(column1, column2, column3, ...columnN)]
   ...> VALUES (value1, value2, value3, ...valueN);
```

如果给出了列名，后面的值要对应，并且非空列必须给出

如果没给出列名，那么后面值中必须所有的值都写出来

### 实例

```sql lite
sqlite> INSERT INTO test.phnbk (AGE, NAME, ID)
   ...> VALUES (22, 'lilili', 1);
sqlite> INSERT INTO test.phnbk VALUES (2, 'lalala', 90, '2st sr', 3330);
```

上述两种用法可以正常使用

*可以使用SELECT语句从一个表中将数据填充到另一个表中*

## SELECT

## 语法

```sql lite
sqlite> SELECT column1, column2, columnN FROM table_name;
```

```sql lite
sqlite> SELECT * FROM table_name;
```

## 实例

```sql lite
sqlite> SELECT * FROM phnbk;
1|lilili|22||
2|lalala|90|2st sr|3330
sqlite> .header on
sqlite> SELECT * FROM phnbk;
ID|NAME|AGE|ADDRESS|SALARY
1|lilili|22||
2|lalala|90|2st sr|3330
sqlite> .mode column
sqlite> SELECT * FROM phnbk;
ID          NAME        AGE         ADDRESS     SALARY    
----------  ----------  ----------  ----------  ----------
1           lilili      22                                
2           lalala      90          2st sr      3330      
sqlite> SELECT ID, NAME FROM phnbk;
ID          NAME      
----------  ----------
1           lilili    
2           lalala    
sqlite> .width 5, 10, 5; 
sqlite> SELECT * FROM phnbk;      
ID     NAME        AGE    ADDRESS     SALARY    
-----  ----------  -----  ----------  ----------
1      lilili      22                           
2      lalala      90     2st sr      3330   
```

### Schema信息

sqlite_master表是每个数据库的一个系统表，不能被更新，他自动进行更新，可以

```sql lite
sqlite> .schema sqlite_master
CREATE TABLE sqlite_master (
  type text,
  name text,
  tbl_name text,
  rootpage integer,
  sql text
);
```

*但是*`SELECT *`*的时候输出空*

## SQLite运算符

- 算数运算符
- 比较运算符
- 逻辑运算符
- 位运算符

### SQLite算术运算符

假设变量 a=10，变量 b=20，则：

| 运算符 | 描述                                    | 实例             |
| ------ | --------------------------------------- | ---------------- |
| +      | 加法 - 把运算符两边的值相加             | a + b 将得到 30  |
| -      | 减法 - 左操作数减去右操作数             | a - b 将得到 -10 |
| *      | 乘法 - 把运算符两边的值相乘             | a * b 将得到 200 |
| /      | 除法 - 左操作数除以右操作数             | b / a 将得到 2   |
| %      | 取模 - 左操作数除以右操作数后得到的余数 | b % a 将得到 0   |

```sql lite
sqlite> .mode line
sqlite> select 10 + 10;
10 + 10 = 20
```

