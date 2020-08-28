


* [一、表引擎](#%E4%B8%80%E8%A1%A8%E5%BC%95%E6%93%8E)
  * [1\.1 TinyLog](#11-tinylog)
  * [1\.2 Memory](#12-memory)
  * [1\.3 Merge](#13-merge)
  * [1\.4 MergeTree](#14-mergetree)
  * [1\.5 ReplacingMergeTree](#15-replacingmergetree)
  * [1\.6 SummingMergeTree](#16-summingmergetree)
  * [1\.7 Distributed](#17-distributed)
  
  
---

# 一、表引擎
表引擎（即表的类型）决定了：
- 数据的存储方式和位置，写到哪里以及从哪里读取数据
- 支持哪些查询以及如何支持。
- 并发数据访问。
- 索引的使用（如果存在）。
- 是否可以执行多线程请求。
- 数据复制参数。

ClickHouse的表引擎有很多，下面介绍其中几种，对其他引擎有兴趣的可以去查阅官方文档：https://clickhouse.yandex/docs/zh/operations/table_engines/
## 1.1 TinyLog
最简单的表引擎，用于将数据存储在磁盘上。每列都存储在单独的压缩文件中，写入时，数据将附加到文件末尾。

该引擎没有并发控制 
- 如果同时从表中读取和写入数据，则读取操作将抛出异常；
- 如果同时写入多个查询中的表，则数据将被破坏。

这种表引擎的典型用法是 write-once: 首先只写入一次数据，然后根据需要多次读取。此引擎适用于相对较小的表（建议最多1,000,000行）。如果有许多小表，则使用此表引擎是适合的，因为它比需要打开的文件更少。当拥有大量小表时，可能会导致性能低下。      不支持索引。

案例：创建一个TinyLog引擎的表并插入一条数据
```sql
:)create table t (a UInt16, b String) ENGINE=TinyLog;
:)insert into t (a, b) values (1, 'abc');
```
此时我们到保存数据的目录 `/var/lib/clickhouse/data/default/t` 中可以看到如下目录结构:
```bash
[root@kino100 t]# ls
a.bin  b.bin  sizes.json
```
a.bin 和 b.bin 是压缩过的对应的列的数据，sizes.json 中记录了每个 *.bin 文件的大小:
```bash
[root@kino100 t]# cat sizes.json 
{"yandex":{"a%2Ebin":{"size":"28"},"b%2Ebin":{"size":"30"}}}
```
## 1.2 Memory
内存引擎，数据以未压缩的原始形式直接保存在内存当中，服务器重启数据就会消失。读写操作不会相互阻塞，不支持索引。简单查询下有非常非常高的性能表现（超过10G/s）。

一般用到它的地方不多，除了用来测试，就是在需要非常高的性能，同时数据量又不太大（上限大概 1 亿行）的场景。
## 1.3 Merge
Merge 引擎 (不要跟 MergeTree 引擎混淆) 本身不存储数据，但可用于同时从任意多个其他的表中读取数据。 读是自动并行的，不支持写入。读取时，那些被真正读取到数据的表的索引（如果有的话）会被使用。 

Merge 引擎的参数：一个数据库名和一个用于匹配表名的正则表达式。

案例：先建t1，t2，t3三个表，然后用 Merge 引擎的 t 表再把它们链接起来。
```sql
:)create table t1 (id UInt16, name String) ENGINE=TinyLog;
:)create table t2 (id UInt16, name String) ENGINE=TinyLog;
:)create table t3 (id UInt16, name String) ENGINE=TinyLog;

:)insert into t1(id, name) values (1, 'first');
:)insert into t2(id, name) values (2, 'second');
:)insert into t3(id, name) values (3, 'i am in t3');

:)create table t (id UInt16, name String) ENGINE=Merge(currentDatabase(), '^t');

:) select * from t;
┌─id─┬─name─┐
│  2 │ second │
└────┴──────┘
┌─id─┬─name──┐
│  1 │ first │
└────┴───────┘
┌─id─┬─name───────┐
│ 3	 │ i am in t3 │
└────┴────────────┘
```
## 1.4 MergeTree
Clickhouse 中最强大的表引擎当属 MergeTree （合并树）引擎及该系列（*MergeTree）中的其他引擎。

MergeTree 引擎系列的基本理念如下。当你有巨量数据要插入到表中，你要高效地一批批写入数据片段，并希望这些数据片段在后台按照一定规则合并。相比在插入时不断修改（重写）数据进存储，这种策略会高效很多。

格式：
```java
ENGINE [=] MergeTree(date-column [, sampling_expression], (primary, key), index_granularity)
```
参数解读：

- date-column: 类型为 Date 的列名。ClickHouse 会自动依据这个列按月创建分区。分区名格式为 "YYYYMM" 。
- sampling_expression: 采样表达式。
- (primary, key): 主键。类型为Tuple()
- index_granularity: 索引粒度。即索引中相邻”标记”间的数据行数。设为 8192 可以适用大部分场景。

案例：
```sql
create table mt_table (date  Date, id UInt8, name String) ENGINE=MergeTree(date, (id, name), 8192);

insert into mt_table values ('2019-05-01', 1, 'zhangsan');
insert into mt_table values ('2019-06-01', 2, 'lisi');
insert into mt_table values ('2019-05-03', 3, 'wangwu');
```
在/var/lib/clickhouse/data/default/mt_tree下可以看到：
```bash
[root@kino100 mt_table]# ls
20190501_20190501_2_2_0  20190503_20190503_6_6_0  20190601_20190601_4_4_0  detached
```

随便进入一个目录:
```bash
[root@hadoop102 20190601_20190601_4_4_0]# ls
checksums.txt  columns.txt  date.bin  date.mrk  id.bin  id.mrk  name.bin  name.mrk  primary.idx
```
- *.bin: 按列保存数据的文件
- *.mrk: 保存块偏移量
- primary.idx: 保存主键索引

## 1.5 ReplacingMergeTree
这个引擎是在 MergeTree 的基础上，添加了“处理重复数据”的功能，该引擎和MergeTree的不同之处在于它会删除具有相同主键的重复项。数据的去重只会在合并的过程中出现。合并会在未知的时间在后台进行，所以你无法预先作出计划。有一些数据可能仍未被处理。因此，ReplacingMergeTree 适用于在后台清除重复的数据以节省空间，但是它不保证没有重复的数据出现。

格式：
```java
ENGINE [=] ReplacingMergeTree(date-column [, sampling_expression], (primary, key), index_granularity, [ver])
```
可以看出他比MergeTree只多了一个ver，这个ver指代版本列，他和时间一起配置，区分哪条数据是最新的。

案例:
```sql
create table rmt_table (date  Date, id UInt8, name String,point UInt8) ENGINE= ReplacingMergeTree(date, (id, name), 8192,point);
```

插入一些数据:
```sql
insert into rmt_table values ('2019-07-10', 1, 'a', 20);
insert into rmt_table values ('2019-07-10', 1, 'a', 30);
insert into rmt_table values ('2019-07-11', 1, 'a', 20);
insert into rmt_table values ('2019-07-11', 1, 'a', 30);
insert into rmt_table values ('2019-07-11', 1, 'a', 10);
```

等待一段时间或optimize table rmt_table手动触发merge，后查询
```sql
:) select * from rmt_table;
┌───────date─┬─id─┬─name─┬─point─┐
│ 2019-07-11 │  1 │ a    │    30 │
└────────────┴────┴──────┴───────┘
```
## 1.6 SummingMergeTree
该引擎继承自 MergeTree。区别在于，当合并 SummingMergeTree 表的数据片段时，ClickHouse 会把所有具有相同主键的行合并为一行，该行包含了被合并的行中具有数值数据类型的列的汇总值。如果主键的组合方式使得单个键值对应于大量的行，则可以显著的减少存储空间并加快数据查询的速度，对于不可加的列，会取一个最先出现的值。

语法：
```bash
ENGINE [=] SummingMergeTree(date-column [, sampling_expression], (primary, key), index_granularity, [columns])
```
- columns: 包含将要被汇总的列的列名的元组

案例：
```sql
create table smt_table (date Date, name String, a UInt16, b UInt16) ENGINE=SummingMergeTree(date, (date, name), 8192, (a))
```
插入数据：
```sql
insert into smt_table (date, name, a, b) values ('2019-07-10', 'a', 1, 2);
insert into smt_table (date, name, a, b) values ('2019-07-10', 'b', 2, 1);
insert into smt_table (date, name, a, b) values ('2019-07-11', 'b', 3, 8);
insert into smt_table (date, name, a, b) values ('2019-07-11', 'b', 3, 8);
insert into smt_table (date, name, a, b) values ('2019-07-11', 'a', 3, 1);
insert into smt_table (date, name, a, b) values ('2019-07-12', 'c', 1, 3);
```
等待一段时间或optimize table smt_table手动触发merge，后查询
```sql
:) select * from smt_table 

┌───────date─┬─name─┬─a─┬─b─┐
│ 2019-07-10 │ a    │ 1 │ 2 │
│ 2019-07-10 │ b    │ 2 │ 1 │
│ 2019-07-11 │ a    │ 3 │ 1 │
│ 2019-07-11 │ b    │ 6 │ 8 │
│ 2019-07-12 │ c    │ 1 │ 3 │
└────────────┴──────┴───┴───┘
```

发现2019-07-11，b的a列合并相加了，b列取了8（因为b列为8的数据最先插入）。

## 1.7 Distributed
分布式引擎，本身不存储数据, 但可以在多个服务器上进行分布式查询。 读是自动并行的。读取时，远程服务器表的索引（如果有的话）会被使用。 
```java
Distributed(cluster_name, database, table [, sharding_key])
```
参数解析：

- cluster_name: 服务器配置文件中的集群名,在/etc/metrika.xml中配置的
- database: 数据库名
- table: 表名
- sharding_key: 数据分片键

案例演示：
① 在hadoop102，hadoop103，hadoop104上分别创建一个表t
```sql
:)create table t(id UInt16, name String) ENGINE=TinyLog;
```
② 在三台机器的t表中插入一些数据
```sql
:)insert into t(id, name) values (2, 'lisi');
:)insert into t(id, name) values (1, 'zhangsan');
```
③ 在hadoop102上创建分布式表
```sql
:)create table dis_table(id UInt16, name String) ENGINE=Distributed(perftest_3shards_1replicas, default, t, id);
```
④ 往dis_table中插入数据
```sql
:) insert into dis_table select * from t
```
⑤ 查看数据量
```sql
:) select count() from dis_table 
FROM dis_table 

┌─count()─┐
│       8 │
└─────────┘
:) select count() from t

SELECT count()
FROM t 

┌─count()─┐
│       3 │
└─────────┘
```
可以看到每个节点大约有1/3的数据
