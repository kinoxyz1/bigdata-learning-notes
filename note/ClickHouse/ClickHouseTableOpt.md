


# 一、SQL语法
## 1.1 CREATE
### 1.1.1 CREATE DATABASE
用于创建指定名称的数据库，语法如下：
```sql
CREATE DATABASE [IF NOT EXISTS] db_name
```
如果查询中存在IF NOT EXISTS，则当数据库已经存在时，该查询不会返回任何错误。
```sql
:) create database test;

Ok.
0 rows in set. Elapsed: 0.018 sec.
```

### 1.1.2 CREATE TABLE
对于创建表，语法如下：
```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = engine
```
- DEFAULT expr: 默认值，用法与SQL类似。
- MATERIALIZED expr: 物化表达式，被该表达式指定的列不能被INSERT，因为它总是被计算出来的。 对于INSERT而言，不需要考虑这些列。 另外，在SELECT查询中如果包含星号，此列不会被查询。
- ALIAS expr: 别名。

有三种方式创建表：

① 直接创建
```sql
:) create table t1(id UInt16,name String) engine=TinyLog
```
② 创建一个与其他表具有相同结构的表
```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name AS [db2.]name2 [ENGINE = engine]
```
可以对其指定不同的表引擎声明。如果没有表引擎声明，则创建的表将与db2.name2使用相同的表引擎。
```sql
:) create table t2 as t1 engine=Memory

:) desc t2

DESCRIBE TABLE t2

┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ id   │ UInt16 │              │                    │
│ name   │ String │              │                    │
└──────┴────────┴──────────────┴────────────────────┘
```
③ 使用指定的引擎创建一个与SELECT子句的结果具有相同结构的表，并使用SELECT子句的结果填充它。

语法：
```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name ENGINE = engine AS SELECT ...
```
实例：

先在t2中插入几条数据
```sql
:) insert into t1 values(1,'zhangsan'),(2,'lisi'),(3,'wangwu')

:) create table t3 engine=TinyLog as select * from t1
:) select * from t3
┌─id─┬─name─────┐
│  1 │ zhangsan │
│  2 │ lisi     │
│  3 │ wangwu   │
└────┴──────────┘
```
## 1.2 INSERT INTO
主要用于向表中添加数据，基本格式如下：
```sql
INSERT INTO [db.]table [(c1, c2, c3)] VALUES (v11, v12, v13), (v21, v22, v23), ...
```
实例：
```sql
:) insert into t1 values(1,'zhangsan'),(2,'lisi'),(3,'wangwu')
```
还可以使用select来写入数据：
```sql
INSERT INTO [db.]table [(c1, c2, c3)] SELECT ...
```
实例：
```sql
:) insert into t2 select * from t3
:) select * from t2
┌─id─┬─name─────┐
│  1 │ zhangsan │
│  2 │ lisi     │
│  3 │ wangwu   │
└────┴──────────┘
```

ClickHouse不支持的修改数据的查询：UPDATE, DELETE, REPLACE, MERGE, UPSERT, INSERT UPDATE。

## 1.3 ALTER
ALTER只支持MergeTree系列，Merge和Distributed引擎的表，基本语法：
```sql
ALTER TABLE [db].name [ON CLUSTER cluster] ADD|DROP|MODIFY COLUMN ...
```
参数解析：
- ADD COLUMN: 向表中添加新列
- DROP COLUMN: 在表中删除列
- MODIFY COLUMN: 更改列的类型

案例演示：
① 创建一个MergerTree引擎的表
```sql
create table mt_table (date  Date, id UInt8, name String) ENGINE=MergeTree(date, (id, name), 8192);
```
② 向表中插入一些值
```sql
insert into mt_table values ('2019-05-01', 1, 'zhangsan');
insert into mt_table values ('2019-06-01', 2, 'lisi');
insert into mt_table values ('2019-05-03', 3, 'wangwu');
```
③ 在末尾添加一个新列age
```sql
:)alter table mt_table add column age UInt8
:)desc mt_table
┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ date │ Date   │              │                    │
│ id   │ UInt8  │              │                    │
│ name │ String │              │                    │
│ age  │ UInt8  │              │                    │
└──────┴────────┴──────────────┴────────────────────┘
:) select * from mt_table
┌───────date─┬─id─┬─name─┬─age─┐
│ 2019-06-01 │  2 │ lisi │   0 │
└────────────┴────┴──────┴─────┘
┌───────date─┬─id─┬─name─────┬─age─┐
│ 2019-05-01 │  1 │ zhangsan │   0 │
│ 2019-05-03 │  3 │ wangwu   │   0 │
└────────────┴────┴──────────┴─────┘
```
④ 更改age列的类型
```sql
:)alter table mt_table modify column age UInt16
:)desc mt_table

┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ date │ Date   │              │                    │
│ id   │ UInt8  │              │                    │
│ name │ String │              │                    │
│ age  │ UInt16 │              │                    │
└──────┴────────┴──────────────┴────────────────────┘
```
⑤ 删除刚才创建的age列
```sql
:)alter table mt_table drop column age
:)desc mt_table
┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ date │ Date   │              │                    │
│ id   │ UInt8  │              │                    │
│ name │ String │              │                    │
└──────┴────────┴──────────────┴────────────────────┘
```
## 1.4 DESCRIBE TABLE
查看表结构
```sql
:)desc mt_table
┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ date │ Date   │              │                    │
│ id   │ UInt8  │              │                    │
│ name │ String │              │                    │
└──────┴────────┴──────────────┴────────────────────┘
```
## 1.5 CHECK TABLE

检查表中的数据是否损坏，他会返回两种结果：

- 0: 数据已损坏
- 1: 数据完整

该命令只支持Log，TinyLog和StripeLog引擎。
