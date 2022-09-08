



---

# 一、数据目录 /var/lib/mysql/
```mysql
-- 查看数据库文件的存放路径
mysql> show variables like 'datadir';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.00 sec)

-- 查看数据目录
bash-4.2# ls -l /var/lib/mysql/
total 188484
-rw-r----- 1 mysql mysql       56 Aug 23 08:15 auto.cnf
-rw------- 1 mysql mysql     1676 Aug 23 08:15 ca-key.pem
-rw-r--r-- 1 mysql mysql     1112 Aug 23 08:15 ca.pem
-rw-r--r-- 1 mysql mysql     1112 Aug 23 08:15 client-cert.pem
-rw------- 1 mysql mysql     1680 Aug 23 08:15 client-key.pem
-rw-r----- 1 mysql mysql      412 Aug 23 08:46 ib_buffer_pool
-rw-r----- 1 mysql mysql 50331648 Aug 23 09:02 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Aug 23 08:15 ib_logfile1
-rw-r----- 1 mysql mysql 79691776 Aug 23 09:02 ibdata1
-rw-r----- 1 mysql mysql 12582912 Aug 23 08:46 ibtmp1
drwxr-x--- 2 mysql mysql     4096 Aug 23 09:02 kinodb1
drwxr-x--- 2 mysql mysql     4096 Aug 23 08:15 mysql
lrwxrwxrwx 1 mysql mysql       27 Aug 23 08:46 mysql.sock -> /var/run/mysqld/mysqld.sock
drwxr-x--- 2 mysql mysql     4096 Aug 23 08:15 performance_schema
-rw------- 1 mysql mysql     1676 Aug 23 08:15 private_key.pem
-rw-r--r-- 1 mysql mysql      452 Aug 23 08:15 public_key.pem
-rw-r--r-- 1 mysql mysql     1112 Aug 23 08:15 server-cert.pem
-rw------- 1 mysql mysql     1680 Aug 23 08:15 server-key.pem
drwxr-x--- 2 mysql mysql    12288 Aug 23 08:15 sys
```

# 二、配置目录 /etc/mysql、/usr/share/mysql
```bash
-- 查看 /etc/mysql 目录
bash-4.2# ls -l /etc/mysql
total 8
drwxr-xr-x 2 root root 4096 Aug 23 08:46 conf.d
drwxr-xr-x 2 root root 4096 Jul 26 23:29 mysql.conf.d

## 查看 /etc/my.cnf 文件
[mysqld]
...
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

可以发现 /etc/mysql 下的两个目录被mysql配置所包含，自己需要添加配置文件可以在 /etc/mysql/conf.d/ 中添加而不用直接修改 my.cnf

-- 查看 /usr/share/mysql
bash-4.2# ls -l /usr/share/mysql
total 2172
drwxr-xr-x 2 root root    4096 Jul 26 23:29 bulgarian
drwxr-xr-x 2 root root    4096 Jul 26 23:29 charsets
drwxr-xr-x 2 root root    4096 Jul 26 23:29 czech
drwxr-xr-x 2 root root    4096 Jul 26 23:29 danish
-rw-r--r-- 1 root root   25575 Jun  8 08:37 dictionary.txt
drwxr-xr-x 2 root root    4096 Jul 26 23:29 dutch
drwxr-xr-x 2 root root    4096 Jul 26 23:29 english
-rw-r--r-- 1 root root  530589 Jun  8 08:37 errmsg-utf8.txt
drwxr-xr-x 2 root root    4096 Jul 26 23:29 estonian
-rw-r--r-- 1 root root 1073818 Jun  8 08:47 fill_help_tables.sql
drwxr-xr-x 2 root root    4096 Jul 26 23:29 french
drwxr-xr-x 2 root root    4096 Jul 26 23:29 german
drwxr-xr-x 2 root root    4096 Jul 26 23:29 greek
drwxr-xr-x 2 root root    4096 Jul 26 23:29 hungarian
-rw-r--r-- 1 root root    3999 Jun  8 08:37 innodb_memcached_config.sql
-rw-r--r-- 1 root root    2221 Jun  8 09:14 install_rewriter.sql
drwxr-xr-x 2 root root    4096 Jul 26 23:29 italian
drwxr-xr-x 2 root root    4096 Jul 26 23:29 japanese
drwxr-xr-x 2 root root    4096 Jul 26 23:29 korean
-rw-r--r-- 1 root root     773 Jun  8 08:37 magic
-rw-r--r-- 1 root root    2171 Jun  8 08:37 mysql_security_commands.sql
-rw-r--r-- 1 root root  288342 Jun  8 08:37 mysql_sys_schema.sql
-rw-r--r-- 1 root root  155441 Jun  8 08:37 mysql_system_tables.sql
-rw-r--r-- 1 root root    1214 Jun  8 08:37 mysql_system_tables_data.sql
-rw-r--r-- 1 root root   10862 Jun  8 08:37 mysql_test_data_timezone.sql
drwxr-xr-x 2 root root    4096 Jul 26 23:29 norwegian
drwxr-xr-x 2 root root    4096 Jul 26 23:29 norwegian-ny
drwxr-xr-x 2 root root    4096 Jul 26 23:29 polish
drwxr-xr-x 2 root root    4096 Jul 26 23:29 portuguese
drwxr-xr-x 2 root root    4096 Jul 26 23:29 romanian
drwxr-xr-x 2 root root    4096 Jul 26 23:29 russian
drwxr-xr-x 2 root root    4096 Jul 26 23:29 serbian
drwxr-xr-x 2 root root    4096 Jul 26 23:29 slovak
drwxr-xr-x 2 root root    4096 Jul 26 23:29 spanish
drwxr-xr-x 2 root root    4096 Jul 26 23:29 swedish
drwxr-xr-x 2 root root    4096 Jul 26 23:29 ukrainian
-rw-r--r-- 1 root root    1243 Jun  8 09:14 uninstall_rewriter.sql
```

# 三、查看默认数据库
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
四个默认的数据库功能如下:
1. `mysql`: MySQL 系统自带的核心数据库, 它存储了 MySQL 的用户账户和权限信息，一些存储过程、事件的定义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。
2. `information_schema`: MySQL 系统自带的数据库，这个数据库保存着 MySQL 服务器**维护的所有其他数据库的信息**，比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述信息，有时候也被称之为 **元数据**。在这个数据库中提供了一些以 `innodb_sys` 开头的表，用于表示内部系统的表。
3. `performance_schema`: MySQL 系统自带的数据库，这个数据库主要保存 MySQL 服务器运行过程中的一些状态信息，可以用来 监控 **MySQL 服务的各类性能指标**。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多场的时间，内存的使用情况等。
4. `sys`: MySQL 系统自带的数据库，这个数据库主要通过 **视图** 的形式把 `information_schema` 和 `performance_schema` 结合起来，帮助系统管理员和开发人员监控 MySQL 的技术性能。

# 四、数据库在文件系统中的表示
```bash
bash-4.2# cd /var/lib/mysql
bash-4.2# ll
总用量 189980
-rw-r-----. 1 mysql mysql 56 7月 28 00:27 auto.cnf
-rw-r-----. 1 mysql mysql 179 7月 28 00:27 binlog.000001
-rw-r-----. 1 mysql mysql 820 7月 28 01:00 binlog.000002
-rw-r-----. 1 mysql mysql 179 7月 29 14:08 binlog.000003
-rw-r-----. 1 mysql mysql 582 7月 29 16:47 binlog.000004
-rw-r-----. 1 mysql mysql 179 7月 29 16:51 binlog.000005
-rw-r-----. 1 mysql mysql 179 7月 29 16:56 binlog.000006
-rw-r-----. 1 mysql mysql 179 7月 29 17:37 binlog.000007
-rw-r-----. 1 mysql mysql 24555 7月 30 00:28 binlog.000008
-rw-r-----. 1 mysql mysql 179 8月 1 11:57 binlog.000009
-rw-r-----. 1 mysql mysql 156 8月 1 23:21 binlog.000010
-rw-r-----. 1 mysql mysql 156 8月 2 09:25 binlog.000011
-rw-r-----. 1 mysql mysql 1469 8月 4 01:40 binlog.000012
-rw-r-----. 1 mysql mysql 156 8月 6 00:24 binlog.000013
-rw-r-----. 1 mysql mysql 179 8月 6 08:43 binlog.000014
-rw-r-----. 1 mysql mysql 156 8月 6 10:56 binlog.000015
-rw-r-----. 1 mysql mysql 240 8月 6 10:56 binlog.index
-rw-------. 1 mysql mysql 1676 7月 28 00:27 ca-key.pem
-rw-r--r--. 1 mysql mysql 1112 7月 28 00:27 ca.pem
-rw-r--r--. 1 mysql mysql 1112 7月 28 00:27 client-cert.pem
-rw-------. 1 mysql mysql 1676 7月 28 00:27 client-key.pem
drwxr-x---. 2 mysql mysql 4096 7月 29 16:34 dbtest
-rw-r-----. 1 mysql mysql 196608 8月 6 10:58 #ib_16384_0.dblwr
-rw-r-----. 1 mysql mysql 8585216 7月 28 00:27 #ib_16384_1.dblwr
-rw-r-----. 1 mysql mysql 3486 8月 6 08:43 ib_buffer_pool
-rw-r-----. 1 mysql mysql 12582912 8月 6 10:56 ibdata1
-rw-r-----. 1 mysql mysql 50331648 8月 6 10:58 ib_logfile0
-rw-r-----. 1 mysql mysql 50331648 7月 28 00:27 ib_logfile1
-rw-r-----. 1 mysql mysql 12582912 8月 6 10:56 ibtmp1
drwxr-x---. 2 mysql mysql 4096 8月 6 10:56 #innodb_temp
drwxr-x---. 2 mysql mysql 4096 7月 28 00:27 mysql
-rw-r-----. 1 mysql mysql 26214400 8月 6 10:56 mysql.ibd
srwxrwxrwx. 1 mysql mysql 0 8月 6 10:56 mysql.sock
-rw-------. 1 mysql mysql 5 8月 6 10:56 mysql.sock.lock
drwxr-x---. 2 mysql mysql 4096 7月 28 00:27 performance_schema
-rw-------. 1 mysql mysql 1680 7月 28 00:27 private_key.pem
-rw-r--r--. 1 mysql mysql 452 7月 28 00:27 public_key.pem
-rw-r--r--. 1 mysql mysql 1112 7月 28 00:27 server-cert.pem
-rw-------. 1 mysql mysql 1680 7月 28 00:27 server-key.pem
drwxr-x---. 2 mysql mysql 4096 7月 28 00:27 sys
drwxr-x---. 2 mysql mysql 4096 7月 29 23:10 temp
-rw-r-----. 1 mysql mysql 16777216 8月 6 10:58 undo_001
-rw-r-----. 1 mysql mysql 16777216 8月 6 10:58 undo_002
```
这个数据目录下的文件和子目录比较多，除了 information_schema 这个系统数据库外，其他的数据库
在 数据目录 下都有对应的子目录。

以我的 temp 数据库为例，在MySQL5.7 中打开：
```bash
bash-4.2# cd ./temp
bash-4.2# ll
总用量 1144
-rw-r-----. 1 mysql mysql 8658 8月 18 11:32 countries.frm
-rw-r-----. 1 mysql mysql 114688 8月 18 11:32 countries.ibd
-rw-r-----. 1 mysql mysql 61 8月 18 11:32 db.opt
-rw-r-----. 1 mysql mysql 8716 8月 18 11:32 departments.frm
-rw-r-----. 1 mysql mysql 147456 8月 18 11:32 departments.ibd
-rw-r-----. 1 mysql mysql 3017 8月 18 11:32 emp_details_view.frm
-rw-r-----. 1 mysql mysql 8982 8月 18 11:32 employees.frm
-rw-r-----. 1 mysql mysql 180224 8月 18 11:32 employees.ibd
-rw-r-----. 1 mysql mysql 8660 8月 18 11:32 job_grades.frm
-rw-r-----. 1 mysql mysql 98304 8月 18 11:32 job_grades.ibd
-rw-r-----. 1 mysql mysql 8736 8月 18 11:32 job_history.frm
-rw-r-----. 1 mysql mysql 147456 8月 18 11:32 job_history.ibd
-rw-r-----. 1 mysql mysql 8688 8月 18 11:32 jobs.frm
-rw-r-----. 1 mysql mysql 114688 8月 18 11:32 jobs.ibd
-rw-r-----. 1 mysql mysql 8790 8月 18 11:32 locations.frm
-rw-r-----. 1 mysql mysql 131072 8月 18 11:32 locations.ibd
-rw-r-----. 1 mysql mysql 8614 8月 18 11:32 regions.frm
-rw-r-----. 1 mysql mysql 114688 8月 18 11:32 regions.ibd
```
在MySQL8.0中打开：
```bash
bash-4.2# cd ./temp
bash-4.2# ll
总用量 1080
-rw-r-----. 1 mysql mysql 131072 7月 29 23:10 countries.ibd
-rw-r-----. 1 mysql mysql 163840 7月 29 23:10 departments.ibd
-rw-r-----. 1 mysql mysql 196608 7月 29 23:10 employees.ibd
-rw-r-----. 1 mysql mysql 114688 7月 29 23:10 job_grades.ibd
-rw-r-----. 1 mysql mysql 163840 7月 29 23:10 job_history.ibd
-rw-r-----. 1 mysql mysql 131072 7月 29 23:10 jobs.ibd
-rw-r-----. 1 mysql mysql 147456 7月 29 23:10 locations.ibd
-rw-r-----. 1 mysql mysql 131072 7月 29 23:10 regions.ibd
```

# 五、表在文件系统中的表示
## 5.1 InnoDB存储引擎模式
### 5.1.1 表结构
为了保存表结构， InnoDB 在 数据目录 下对应的数据库子目录下创建了一个专门用于 描述表结构的文件 ，文件名是这样：
> 表名.frm

如果在数据库中创建一个名为 test 的表
```mysql
mysql> use kinodb1;
mysql> create table test(id int);
```
那么在数据库 kinodb1 对应的子目录下就会创建一个名为 `test.frm` 的用于描述表结构的文件。`.frm` 文件的格式在不同的平台上都是相同的。这个后缀名为 `.frm` 是以**二进制**存储的，直接打开是乱码的。

### 5.1.2 表中数据和索引
#### 系统表空间(system tablespace)
默认情况下，InnoDB 会在数据目录下创建一个名为 `ibdata1`，大小为 `12M` 的文件，这个文件就是对应的**系统表空间**在文件系统上的表示。这个文件不够用的时候会自己增加文件的大小。

`ibdata1` 是可以存在多个也可以修改名称的，需要在 MySQL 启动时配置对应的文件路径以及它们的大小:
```bash
$ vim /etc/my.cnf
[server]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

#### 独立表空间(file-per-table tablespace)
在 MySQL5.6.6之后(含)，InnoDB 并不会默认的把各个表的数据存储到系统表空间中，而是为**每一个表创建一个独立的表空间**，也就是说创建了多少表，就有多少个独立表空间。使用**独立表空间**来存储数据的话，会在该表所属数据库对应的子目录下创建一个表示该独立表空间的文件，文件名和表名相同，只不过添加了一个 `.ibd` 的扩展名，完整的扩展名如下:
> 表名.ibd

例如
```bash
bash-4.2# ls -l
total 220
-rw-r----- 1 mysql mysql    65 Aug 23 08:39 db.opt
-rw-r----- 1 mysql mysql  8586 Aug 23 08:40 test1.frm
-rw-r----- 1 mysql mysql 98304 Aug 23 08:40 test1.ibd
-rw-r----- 1 mysql mysql  8586 Aug 23 09:02 test2.frm
-rw-r----- 1 mysql mysql 98304 Aug 23 09:02 test2.ibd
```
其中 `test1.ibd` 文件就用来存储 `test1` 表中的数据和索引。

### 系统表空间与独立表空间的设置
我们可以自己指定使用 **系统表空间** 还是 **独立表空间** 来存储数据，这个功能由启动参数 `innodb_file_per_table` 控制，比如我们想刻意将表数据都存储到 **系统表空间** 时，可以在启动 MySQL 服务器的时候这样配置:
```bash
[server]
innodb_file_per_table=0 # 0：代表使用系统表空间； 1：代表使用独立表空间
```
默认情况:
```mysql
mysql> show variables like 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name | Value |
+-----------------------+-------+
| innodb_file_per_table | ON |
+-----------------------+-------+
1 row in set (0.01 sec)
```

### 其他类型的表空间
随着 MySQL 的发展，出了上述两种老牌表空间之外，现在还新提出了一些不同类型的表空间，比如通用表空间(general tablespace)、临时表空间(temporary tablespace)等。

## 5.2 MyISAM 存储引擎模式
### 5.2.1 表结构
在存储表结构方面，MyISAM 和 InnoDB 一样，也是在数据目录下对应的数据库子目录创建一个专门用于描述表结构的文件:
> 表名.frm

### 5.2.2 表中数据和索引
在 MyISAM 中的索引全部都是二级索引，该存储引擎的数据和索引是分开存放的。所以在文件系统中也是使用不同的文件来存储数据文件和索引文件，同时表数据都存放在对应的数据库子目录下。

假如 test 表使用 MyISAM 存储引擎的话，那么在它所在数据库对应的目录下会为 test 表创建这三个文件
> test.frm:       存储表结构
> 
> test.MYD:       存储数据(MYDATA)
> 
> test.MYI:       存储索引(MYIndex)

例如创建一个 MyISAM 表，使用 ENGINE 选项显示指定引擎(因为默认引擎是 InnoDB)
```mysql
CREATE TABLE student_myisam(
    id bigint not null auto_increment,
    name varchar(64) default null,
    age int default null,
    sex varchar(2) default null,
    primary key(id) 
) engine=MYISAM auto_increment=0 default charset=utf8mb3;
```








































