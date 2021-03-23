



--- 
# Binlog 的作用
- MySQL 主从复制: MySQL Replication在Master端开启binlog，Master把它的二进制日志传递给slaves来达到master-slave数据一致的目的
- 数据恢复: 通过使用 mysqlbinlog 工具来使恢复数据
- 数据实时同步: canal/flink CDC/debezium等 将 MySQL 数据实时同步到对应目标库中


# 一、Centos 安装mysql
https://blog.csdn.net/Java_Road_Far/article/details/98041498


# 二、开启 Binlog 
修改 `/etc/mysql/my.cnf` 配置文件的 `log-bin` 选项
```bash
$ vim /etc/my.cnf
[mysqld]
# MySQL5.7 开启 binlog 需要添加该参数
server_id=2
# 表示开启 binlog, 并指定 binlog 文件名
log-bin=mysql-bin
# 默认参数
binlog_format = ROW
# binlog 保留天数
expire_logs_days = 7
```
重启 MySQL即可

可以使用 `SET SQL_LOG_BIN=0` 命令停止使用日志文件，然后可以通过 `SET SQL_LOG_BIN=1` 命令来启用。

创建数据库创建表:
```mysql
mysql> create database kino;
mysql> use kino;
mysql> create table test1(id int, name varchar(20));
mysql> insert into test1 values(1,'kino1');
mysql> insert into test1 values(2,'kino2');
mysql> delete from test1 where id = 2;
```

# 三、查看 Binlog
查看当前服务器使用的二进制文件及大小:
```mysql
mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |      1291 |
+------------------+-----------+
1 row in set (0.00 sec)
```
显示主服务器使用的二进制文件及大小:
```mysql
mysql> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |      1291 |
+------------------+-----------+
1 row in set (0.00 sec)
```
显示当前使用的二进制文件及所处位置:
```mysql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     1291 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
> binlog_do_db：此参数表示只记录制定数据库的二进制日志
> 
> binlog_ignore_db：此参数标示不记录指定的数据库的二进制日志

查看 一个binlog 文件具体内容:
```mysql
mysql> show binlog events in 'mysql-bin.000001';
+------------------+------+----------------+-----------+-------------+----------------------------------------------------------+
| Log_name         | Pos  | Event_type     | Server_id | End_log_pos | Info                                                     |
+------------------+------+----------------+-----------+-------------+----------------------------------------------------------+
| mysql-bin.000001 |    4 | Format_desc    |         2 |         123 | Server ver: 5.7.26-log, Binlog ver: 4                    |
| mysql-bin.000001 |  123 | Previous_gtids |         2 |         154 |                                                          |
| mysql-bin.000001 |  154 | Anonymous_Gtid |         2 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                     |
| mysql-bin.000001 |  219 | Query          |         2 |         313 | create database kino                                     |
| mysql-bin.000001 |  313 | Anonymous_Gtid |         2 |         378 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                     |
| mysql-bin.000001 |  378 | Query          |         2 |         496 | use `kino`; create table test1(id int, name varchar(20)) |
| mysql-bin.000001 |  496 | Anonymous_Gtid |         2 |         561 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                     |
| mysql-bin.000001 |  561 | Query          |         2 |         633 | BEGIN                                                    |
| mysql-bin.000001 |  633 | Table_map      |         2 |         684 | table_id: 108 (kino.test1)                               |
| mysql-bin.000001 |  684 | Write_rows     |         2 |         730 | table_id: 108 flags: STMT_END_F                          |
| mysql-bin.000001 |  730 | Xid            |         2 |         761 | COMMIT /* xid=21 */                                      |
| mysql-bin.000001 |  761 | Anonymous_Gtid |         2 |         826 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                     |
| mysql-bin.000001 |  826 | Query          |         2 |         898 | BEGIN                                                    |
| mysql-bin.000001 |  898 | Table_map      |         2 |         949 | table_id: 108 (kino.test1)                               |
| mysql-bin.000001 |  949 | Write_rows     |         2 |         995 | table_id: 108 flags: STMT_END_F                          |
| mysql-bin.000001 |  995 | Xid            |         2 |        1026 | COMMIT /* xid=22 */                                      |
| mysql-bin.000001 | 1026 | Anonymous_Gtid |         2 |        1091 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                     |
| mysql-bin.000001 | 1091 | Query          |         2 |        1163 | BEGIN                                                    |
| mysql-bin.000001 | 1163 | Table_map      |         2 |        1214 | table_id: 108 (kino.test1)                               |
| mysql-bin.000001 | 1214 | Delete_rows    |         2 |        1260 | table_id: 108 flags: STMT_END_F                          |
| mysql-bin.000001 | 1260 | Xid            |         2 |        1291 | COMMIT /* xid=23 */                                      |
+------------------+------+----------------+-----------+-------------+----------------------------------------------------------+
21 rows in set (0.00 sec)
```

或者使用 `mysqlbinlog` 命令, 在shell终端输入:
```bash
$ pwd
/var/lib/mysql

$ mysqlbinlog mysql-bin.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#210323 14:29:47 server id 2  end_log_pos 123 CRC32 0xaad3e97b 	Start: binlog v 4, server v 5.7.26-log created 210323 14:29:47 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
24pZYA8CAAAAdwAAAHsAAAABAAQANS43LjI2LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAADbillgEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
AXvp06o=
'/*!*/;
# at 123
#210323 14:29:47 server id 2  end_log_pos 154 CRC32 0xfae4e8a5 	Previous-GTIDs
# [empty]
# at 154
#210323 14:37:37 server id 2  end_log_pos 219 CRC32 0x2bea37b1 	Anonymous_GTID	last_committed=0	sequence_number=1	rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 219
#210323 14:37:37 server id 2  end_log_pos 313 CRC32 0x49f3b909 	Query	thread_id=3	exec_time=0	error_code=0
SET TIMESTAMP=1616481457/*!*/;
SET @@session.pseudo_thread_id=3/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
create database kino
/*!*/;
# at 313
#210323 14:38:15 server id 2  end_log_pos 378 CRC32 0x9a66d815 	Anonymous_GTID	last_committed=1	sequence_number=2	rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 378
#210323 14:38:15 server id 2  end_log_pos 496 CRC32 0xf02528be 	Query	thread_id=3	exec_time=0	error_code=0
use `kino`/*!*/;
SET TIMESTAMP=1616481495/*!*/;
create table test1(id int, name varchar(20))
/*!*/;
# at 496
#210323 14:38:29 server id 2  end_log_pos 561 CRC32 0x8ac26097 	Anonymous_GTID	last_committed=2	sequence_number=3	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 561
#210323 14:38:29 server id 2  end_log_pos 633 CRC32 0xf4a985a4 	Query	thread_id=3	exec_time=0	error_code=0
SET TIMESTAMP=1616481509/*!*/;
BEGIN
/*!*/;
# at 633
#210323 14:38:29 server id 2  end_log_pos 684 CRC32 0x7c5ca4d0 	Table_map: `kino`.`test1` mapped to number 108
# at 684
#210323 14:38:29 server id 2  end_log_pos 730 CRC32 0xafb9c961 	Write_rows: table id 108 flags: STMT_END_F

BINLOG '
5YxZYBMCAAAAMwAAAKwCAAAAAGwAAAAAAAEABGtpbm8ABXRlc3QxAAIDDwIUAAPQpFx8
5YxZYB4CAAAALgAAANoCAAAAAGwAAAAAAAEAAgAC//wBAAAABWtpbm8xYcm5rw==
'/*!*/;
# at 730
#210323 14:38:29 server id 2  end_log_pos 761 CRC32 0x6eb0d9b8 	Xid = 21
COMMIT/*!*/;
# at 761
#210323 14:38:34 server id 2  end_log_pos 826 CRC32 0xc5a12639 	Anonymous_GTID	last_committed=3	sequence_number=4	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 826
#210323 14:38:34 server id 2  end_log_pos 898 CRC32 0x1c50150d 	Query	thread_id=3	exec_time=0	error_code=0
SET TIMESTAMP=1616481514/*!*/;
BEGIN
/*!*/;
# at 898
#210323 14:38:34 server id 2  end_log_pos 949 CRC32 0x8ccd9297 	Table_map: `kino`.`test1` mapped to number 108
# at 949
#210323 14:38:34 server id 2  end_log_pos 995 CRC32 0x9914f5b2 	Write_rows: table id 108 flags: STMT_END_F

BINLOG '
6oxZYBMCAAAAMwAAALUDAAAAAGwAAAAAAAEABGtpbm8ABXRlc3QxAAIDDwIUAAOXks2M
6oxZYB4CAAAALgAAAOMDAAAAAGwAAAAAAAEAAgAC//wCAAAABWtpbm8ysvUUmQ==
'/*!*/;
# at 995
#210323 14:38:34 server id 2  end_log_pos 1026 CRC32 0x76a9f6c0 	Xid = 22
COMMIT/*!*/;
# at 1026
#210323 14:44:05 server id 2  end_log_pos 1091 CRC32 0xcc781003 	Anonymous_GTID	last_committed=4	sequence_number=5	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1091
#210323 14:44:05 server id 2  end_log_pos 1163 CRC32 0xcfe6bf4d 	Query	thread_id=3	exec_time=0	error_code=0
SET TIMESTAMP=1616481845/*!*/;
BEGIN
/*!*/;
# at 1163
#210323 14:44:05 server id 2  end_log_pos 1214 CRC32 0x2ebd09bb 	Table_map: `kino`.`test1` mapped to number 108
# at 1214
#210323 14:44:05 server id 2  end_log_pos 1260 CRC32 0xfd67254a 	Delete_rows: table id 108 flags: STMT_END_F

BINLOG '
NY5ZYBMCAAAAMwAAAL4EAAAAAGwAAAAAAAEABGtpbm8ABXRlc3QxAAIDDwIUAAO7Cb0u
NY5ZYCACAAAALgAAAOwEAAAAAGwAAAAAAAEAAgAC//wCAAAABWtpbm8ySiVn/Q==
'/*!*/;
# at 1260
#210323 14:44:05 server id 2  end_log_pos 1291 CRC32 0xae3f5bd3 	Xid = 23
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

# 四、Binlog 删除
```mysql
mysql> show variables like 'expire_log_days';
mysql>  set global expire_log_days=3;  //过期删除
mysql> reset master; //删除master的binlog
mysql> reset slave; //删除slave的中继日志
mysql> purge master logs before '2016-10-20 16:25:00';//删除指定日期前的日志索引中binlog日志文件
mysql> purge master logs to 'binlog.000002';//删除指定日志文件
```

# 五、Binlog 写入时机
Binlog 的写入是发生在 事务提交 之后的, Binlog 什么时候刷新到磁盘, 跟 sync_binlog 相关

- 如果 sync_binlog 设置为 0, 则表示 MySQL 不控制 Binlog 的刷新, 由文件系统去控制它缓存的刷新;
- 如果设置为不为 0 的值, 则表示每 sync_binlog 次事务, MySQL调用文件系统的刷新操作刷新Binlog 到磁盘中;
- 如果设置为 1, 在系统故障时最多丢失一个事务的更新, 这个是最安全的, 但是会对性能有所影响;

如果 sync_binlog=0 或 sync_binlog>1, 当发生电源故障或操作系统崩溃时, 可能有一部分已提交但是Binlog 未被同步到磁盘的事务被丢失, 此时恢复程序就无法恢复这部分事务.

在 MySQL5.7.7 之前, 默认值 sync_binlog 是 0, MySQL5.7.7 之后使用的默认值是 1, 这样最安全, 一般情况下会设置 100 或 0, 牺牲一定的一致性获取更好的性能.

# 六、Binlog 文件扩展
当遇到如下3种情况之一时, 会形成一个新的 Binlog 文件, 且文件序号递增
1. MySQL 服务器停止或者重启, MySQL 会在重启时生成一个新的Binlog 文件;
2. 使用 flush logs 命令时, 会生成一个新的 Binlog 文件;
3. 当 Binlog 文件大小超过 max_binlog_size 系统变量配置的上线时, 会生成一个新的 Binlog 文件;

binlog文件的最大值和默认值是1GB，该设置并不能严格控制binlog的大小，尤其是binlog比较靠近最大值而又遇到一个比较大事务时，为了保证事务的完整性，不可能做切换日志的动作，只能将该事务的所有SQL都记录到当前日志，直到事务结束。

# 七、Binlog 日志格式
支持三种格式:
1. STATEMENT: 基于 SQL 语句的复制(statement-based replication, SBR)
2. ROW: 基于行的复制(row-based replication, RBR)
3. MIXED: 混合模式复制(mixed-based replication, MBR)

在 MySQL5.7.7 之前, 默认格式是 STATEMENT, 在 MySQL5.7.7 之后, 默认值是 ROW, 日志通过 binlog-format 指定, 如 `binlog-format=binlog-format`、`binlog-format=ROW`、`binlog-format=MIXED`

## 7.1 STATEMENT
每一条会修改数据库的 SQL 都会记录在 Binlog 中

优点: 不需要记录每一行的变化, 减少了 binlog 的日志量, 节约了 IO, 提高了性能;

缺点: 由于记录的只是执行语句, 为了这些语句能在 slave 上正确运行, 因此还必须记录每条语句在执行的时候的一些相关信息, 以保证所有语句能在 slave 和 master 端执行的时候是相同的结果, 另外 MySQL 的复制, 像一些特定函数的功能, slave 可与 master 上保持一致会有很多相关问题(如 sleep函数, last_insert_id函数, user_defined function(udf)会出问题 ).

另外使用如下函数也无法被复制:
- LOAD_FILE()
- UUID()
- USER()
- FOUND_ROWS()
- SYSDATE() // 除非启动时启用了 --sysdate-is-now 选项


## 7.2 ROW
MySQL5.1.5 才开始支持 ROW LEVEL 的复制, 它不记录 sql 语句上下文相关的信息, 仅保存哪条记录被修改.

优点: Binlog 中可以不记录执行的 SQL 语句的上下文信息, 仅需要记录一条记录被修改成什么了, 所以 ROW 的日志内容会非常清楚的记录下每一行数据修改的细节, 而且不会出现某些特定情况下的存储过程, 或函数, 以及trigger 的调用和触发无法被正确复制的问题.

缺点: 所有的执行的语句当记录到日志中的时候, 都将以每行记录的修改来记录, 这样可能会产生大量的日志内容。

新版本的 MySQL 中对 ROW LEVEL 模式做了优化, 并不是所有的修改都会以 ROW LEVEL 来记录, 像遇到表结构更改的时候就会以 STATEMENT 模式来记录, 如果 SQL 语句确实就是 update 或 delete 等修改数据的语句, 那么还是会记录所有的行的变更.

## 7.3 MIXED 
从 MySQL5.1.8 开始, MySQL 提供了 MIXED 格式, 实际上就是 STATEMENT 和 ROW 的结合

在 MIXED 模式下, 一般的语句修改适用的是  STATEMENT 格式保存 Binlog, 如一些函数,STATEMENT 无法完成主从复制的操作, 则采用 ROW 格式保存 Binlog, MySQL 会根据执行的每一条具体的 SQL 语句来区分对待记录的日志形式, 也就是在 STATEMENT 和 ROW 中选择一种


# 八、 Binlog 恢复
由于 Binlog 文件是二级制的, 并不能直接查看, 所以需要用 mysqlbinlog 命令转换为 普通文件 才能查看

## 8.1 直接显示(显示所有)
```bash
$ mysqlbinlog -v --base64-output=decode-rows /var/lib/mysql/master.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
mysqlbinlog: File '/var/lib/mysql/master.000001' not found (Errcode: 2 - No such file or directory)
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@k8s-node2 mysql]# mysqlbinlog -v --base64-output=decode-rows master.000001
...
```
## 8.2 直接显示(显示指定时间段/范围段)
```bash
# 显示 开始时间 到 结束时间 && 开始未知 到 结束为止 的 Binlog
$ mysqlbinlog -v --base64-output=decode-rows mysql-bin.000001 \
>     --start-datetime="2019-03-01 00:00:00"  \
>     --stop-datetime="2022-03-10 00:00:00"   \
>     --start-position="0"    \
>     --stop-position="20"
mysqlbinlog: [Warning] option 'start-position': unsigned value 0 adjusted to 4
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#210323 14:29:47 server id 2  end_log_pos 123 CRC32 0xaad3e97b 	Start: binlog v 4, server v 5.7.26-log created 210323 14:29:47 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

## 8.3 将 Binlog 输出到文件中
```bash
$ mysqlbinlog mysql-bin.000001 > binlog.txt
$ cat binlog.txt

/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#210323 14:29:47 server id 2  end_log_pos 123 CRC32 0xaad3e97b  Start: binlog v 4, server v 5.7.26-log created 210323 14:29:47 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
24pZYA8CAAAAdwAAAHsAAAABAAQANS43LjI2LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAADbillgEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
AXvp06o=
'/*!*/;
# at 123
#210323 14:29:47 server id 2  end_log_pos 154 CRC32 0xfae4e8a5  Previous-GTIDs
# [empty]
# at 154
#210323 14:37:37 server id 2  end_log_pos 219 CRC32 0x2bea37b1  Anonymous_GTID  last_committed=0        sequence_number=1       rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 219
#210323 14:37:37 server id 2  end_log_pos 313 CRC32 0x49f3b909  Query   thread_id=3     exec_time=0     error_code=0
SET TIMESTAMP=1616481457/*!*/;
SET @@session.pseudo_thread_id=3/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
create database kino
/*!*/;
# at 313
#210323 14:38:15 server id 2  end_log_pos 378 CRC32 0x9a66d815  Anonymous_GTID  last_committed=1        sequence_number=2       rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 378
#210323 14:38:15 server id 2  end_log_pos 496 CRC32 0xf02528be  Query   thread_id=3     exec_time=0     error_code=0
use `kino`/*!*/;
SET TIMESTAMP=1616481495/*!*/;
create table test1(id int, name varchar(20))
/*!*/;
# at 496
#210323 14:38:29 server id 2  end_log_pos 561 CRC32 0x8ac26097  Anonymous_GTID  last_committed=2        sequence_number=3       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 561
#210323 14:38:29 server id 2  end_log_pos 633 CRC32 0xf4a985a4  Query   thread_id=3     exec_time=0     error_code=0
SET TIMESTAMP=1616481509/*!*/;
BEGIN
/*!*/;
# at 633
#210323 14:38:29 server id 2  end_log_pos 684 CRC32 0x7c5ca4d0  Table_map: `kino`.`test1` mapped to number 108
# at 684
#210323 14:38:29 server id 2  end_log_pos 730 CRC32 0xafb9c961  Write_rows: table id 108 flags: STMT_END_F

BINLOG '
5YxZYBMCAAAAMwAAAKwCAAAAAGwAAAAAAAEABGtpbm8ABXRlc3QxAAIDDwIUAAPQpFx8
5YxZYB4CAAAALgAAANoCAAAAAGwAAAAAAAEAAgAC//wBAAAABWtpbm8xYcm5rw==
'/*!*/;
# at 730
#210323 14:38:29 server id 2  end_log_pos 761 CRC32 0x6eb0d9b8  Xid = 21
COMMIT/*!*/;
# at 761
#210323 14:38:34 server id 2  end_log_pos 826 CRC32 0xc5a12639  Anonymous_GTID  last_committed=3        sequence_number=4       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 826
#210323 14:38:34 server id 2  end_log_pos 898 CRC32 0x1c50150d  Query   thread_id=3     exec_time=0     error_code=0
SET TIMESTAMP=1616481514/*!*/;
BEGIN
/*!*/;
# at 898
#210323 14:38:34 server id 2  end_log_pos 949 CRC32 0x8ccd9297  Table_map: `kino`.`test1` mapped to number 108
# at 949
#210323 14:38:34 server id 2  end_log_pos 995 CRC32 0x9914f5b2  Write_rows: table id 108 flags: STMT_END_F

BINLOG '
6oxZYBMCAAAAMwAAALUDAAAAAGwAAAAAAAEABGtpbm8ABXRlc3QxAAIDDwIUAAOXks2M
6oxZYB4CAAAALgAAAOMDAAAAAGwAAAAAAAEAAgAC//wCAAAABWtpbm8ysvUUmQ==
'/*!*/;
# at 995
#210323 14:38:34 server id 2  end_log_pos 1026 CRC32 0x76a9f6c0         Xid = 22
COMMIT/*!*/;
# at 1026
#210323 14:44:05 server id 2  end_log_pos 1091 CRC32 0xcc781003         Anonymous_GTID  last_committed=4        sequence_number=5       rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1091
#210323 14:44:05 server id 2  end_log_pos 1163 CRC32 0xcfe6bf4d         Query   thread_id=3     exec_time=0     error_code=0
SET TIMESTAMP=1616481845/*!*/;
BEGIN
/*!*/;
# at 1163
#210323 14:44:05 server id 2  end_log_pos 1214 CRC32 0x2ebd09bb         Table_map: `kino`.`test1` mapped to number 108
# at 1214
#210323 14:44:05 server id 2  end_log_pos 1260 CRC32 0xfd67254a         Delete_rows: table id 108 flags: STMT_END_F

BINLOG '
NY5ZYBMCAAAAMwAAAL4EAAAAAGwAAAAAAAEABGtpbm8ABXRlc3QxAAIDDwIUAAO7Cb0u
NY5ZYCACAAAALgAAAOwEAAAAAGwAAAAAAAEAAgAC//wCAAAABWtpbm8ySiVn/Q==
'/*!*/;
# at 1260
#210323 14:44:05 server id 2  end_log_pos 1291 CRC32 0xae3f5bd3         Xid = 23
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/
```
查看普通文件能看到每条被记录下来的操作语句的起始位置(# at number), 例如在 起始位置是: 1163 这里, 记录下来的操作语句是: `DELETE` , 一直到 结束为止: 1260 才结束 

## 8.1 可选参数
- `--stop-datetime`: 从二进制日志中的第 N 个时间或晚于 stop-datetime变量 的事件停止读;
- `--start-datetime`: 从二进制日志中的第 N 个时间或晚于 stop-datetime变量 的事件起读;
- `--start-position`: 从二进制日志中第1个位置等于N参量时的事件开始读;
- `--stop-position`: 从二进制日志中第1个位置等于和大于N参量时的事件起停止读;
- `-d`: 指定恢复binlog日志中的某个库的日志;

## 8.2 基于位置恢复

### 8.2.1 基于结束位置恢复
先删除kino数据库
```mysql
$ drop database kino;
```
基于位置恢复, 恢复结束为止是删除 delete id=2 之前, 从转换来的普通文本可以看见, delete 操作的开始位置是 1163, 所以恢复语句是: 
```bash
$ mysqlbinlog --stop-position='1163' mysql-bin.000001 | mysql -uroot -p
Enter password: 
```
恢复完成后去MySQL中查看回复好的数据:
```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kino               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use kino;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_kino |
+----------------+
| test1          |
+----------------+
1 row in set (0.00 sec)

mysql> select * from test1;
+------+-------+
| id   | name  |
+------+-------+
|    1 | kino1 |
|    2 | kino2 |
+------+-------+
2 rows in set (0.00 sec)
```
数据就恢复出来了, 而且是执行误操作 delete id=2 之前的数据.

### 8.2.2 基于开始位置恢复
从最开始的位置恢复
```bash
$ mysqlbinlog --start-position='4' mysql-bin.000001 | mysql -uroot -p
Enter password: 
```
查看数据库记录:
```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```
发现kino数据库并没有被恢复, 原因为后面写了drop database 被记录到 Binlog 中了, 所以需要设置起止位置和结束为止

### 8.2.3 基于起始位置 和 结束为止 恢复

此时再次查看 Binlog 对应的 普通文件:
```bash
$ mysqlbinlog mysql-bin.000001 > binlog1.txt
```
由于此时的 Binlog 文件比较长这里不做展示, 在 vim 中搜索 drop 发现, drop操作起始位置是: 1356
```bash
...
# at 1260
#210323 14:44:05 server id 2  end_log_pos 1291 CRC32 0xae3f5bd3         Xid = 23
COMMIT/*!*/;
# at 1291
#210323 16:52:07 server id 2  end_log_pos 1356 CRC32 0x213fce18         Anonymous_GTID  last_committed=5        sequence_number=6       rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1356
#210323 16:52:07 server id 2  end_log_pos 1448 CRC32 0xd821b18d         Query   thread_id=3     exec_time=0     error_code=0
SET TIMESTAMP=1616489527/*!*/;
drop database kino
/*!*/;
# at 1448
...
```
所以恢复语句是:
```bash
$ mysqlbinlog --start-position='4' --stop-position='1356' mysql-bin.000001 | mysql -uroot -p
```
再次查看数据库:
```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kino               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use kino;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_kino |
+----------------+
| test1          |
+----------------+
1 row in set (0.00 sec)

mysql> select * from test1;
+------+-------+
| id   | name  |
+------+-------+
|    1 | kino1 |
+------+-------+
1 row in set (0.00 sec)
```
数据已经被恢复到第一次删库之后的时候了

## 8.3 基于时间点恢复
根据第二次查看的基本文件搜索 drop可以看到 第一次执行 drop 命令是 , 而 Binlog 第一次有数据的起止时间是 210323 14:29:47, 所以指定 开始时间 和 结束时间 为如下命令
```bash
$ mysqlbinlog --start-datetime="2021-03-23 14:29:47" --stop-datetime="2021-03-23 16:52:07" mysql-bin.000001 | mysql -uroot -p
```
需要注意的是, start-datetime 和 stop-datetime 需要传入的时间类型为: `yyyy-MM-dd HH:mm:ss`, 否则会报错


查看恢复的数据:
```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kino               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use kino;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_kino |
+----------------+
| test1          |
+----------------+
1 row in set (0.00 sec)

mysql> select * from test1;
+------+-------+
| id   | name  |
+------+-------+
|    1 | kino1 |
+------+-------+
1 row in set (0.00 sec)
```
可以看见数据已经被恢复了

## 8.4 恢复指定的数据库
只用加一条 -d 就可以了, 例如: 
```bash
$ mysqlbinlog -d kino --start-datetime="2021-03-23 14:29:47" --stop-datetime="2021-03-23 16:52:07" mysql-bin.000001 | mysql -uroot -p
```
查看数据库:
```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kino               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use kino;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_kino |
+----------------+
| test1          |
+----------------+
1 row in set (0.00 sec)

mysql> select * from test1;
+------+-------+
| id   | name  |
+------+-------+
|    1 | kino1 |
+------+-------+
1 row in set (0.00 sec
```
数据已经被恢复