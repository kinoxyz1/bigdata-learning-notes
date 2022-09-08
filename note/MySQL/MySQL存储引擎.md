










---
没有说明版本的命令，即为在MySQL8.0中操作的。

# 一、查看MySQL支持的存储引擎
```mysql
-- 查看 MySQL 提供的存储引擎
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

# 二、设置系统默认的存储引擎
```mysql
-- 查看默认的存储引擎
mysql> show variables like '%storage_engine%';
+---------------------------------+-----------+
| Variable_name                   | Value     |
+---------------------------------+-----------+
| default_storage_engine          | InnoDB    |
| default_tmp_storage_engine      | InnoDB    |
| disabled_storage_engines        |           |
| internal_tmp_mem_storage_engine | TempTable |
+---------------------------------+-----------+
4 rows in set (0.00 sec)

-- 或者
mysql> select @@default_storage_engine;
+--------------------------+
| @@default_storage_engine |
+--------------------------+
| InnoDB                   |
+--------------------------+
1 row in set (0.00 sec)
```

创建表时没有显示的指定表的存储引擎，那就会用默认的存储引擎。可以通过以下命令修改默认存储引擎。
```mysql
mysql> set default_storage_engine=MyISAM;
```
或者修改 my.cnf 文件(需要重启MySQL)
```mysql
[mysqld]
default-storage-engine=MyISAM
```

# 三、设置表的存储引擎
如果创建表时不想用系统默认，也可以在创建的时候就指定存储引擎。

## 3.1 创建表时设置存储引擎
语法
```mysql
mysql> create table 表名(
         字段
       ) engine = 存储引擎名称;
```
示例
```mysql
mysql> create table test1(
        id int primary key auto_increment,
        name varchar(20)
    ) engine = MEMORY;
```

## 3.2 修改表的存储引擎
语法
```mysql
mysql> alter table 表名 engine = 存储引擎名称;
```
示例
```mysql
mysql> alter table test1 engine = InnoDB;
```

# 四、存储引擎介绍
## 4.1 InnoDB 引擎
InnoDB 引擎具备外键支持功能的事务存储引擎。

- MySQL从3.23.34a开始就包含InnoDB存储引擎。`大于等于5.5之后，默认采用InnoDB引擎`。 


































































