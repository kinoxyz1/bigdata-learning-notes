



---
# 一、忽略大小写
## 1.1 查看大小写配置
```mysql
mysql> show variables like '%lower_case_table_names%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_table_names | 0     |
+------------------------+-------+
1 row in set (0.00 sec)
```
lower_case_table_names 参数的设置:
- 0(默认值): 大小写敏感;
- 1: 大小写不敏感。创建的表、数据库，都是以小写形式存放在磁盘上，对于sql语句都是转换成为小写对表和数据库进行查找;
- 2: 创建的表、数据库，依据语句上格式存放，凡是查找都是转化为小写进行。

## 1.2 设置忽略大小写
```bash
$ vim /etc/my.cof
# 在 [mysqld] 加入
lower_case_table_names=1
```

## 1.3 设置大小写规则
当想要设置为大小写不敏感时, 要在 `my.cnf` 这个配置文件的 `[mysqld]` 中加入 `lower_case_table_names=1`, 然后重启服务。

但是，要在重启数据库实例前就需要将原来的数据库和表转换为小写，否则将找不到数据库名。

此参数适用于 MySQL5.7。在 MySQL8 下禁止在重新启动 MySQL 时将 `lower_case_table_names` 设置成不同于初始化MySQL 服务时设置的 `lower_case_table_names` 值。 如果非要将 MySQL8 设置为大小写不敏感，具体步骤为:
1. 停止 MySQL 服务;
2. 删除数据目录，即删除 `/var/lib/mysql` 目录;
3. 在 MySQL 配置文件 `/etc/my.cnf` 中添加 `lower_case_table_names=1`
4. 启动 MySQL 服务。


# 二、字符集
在 MySQL8 之前，默认字符集为 `latin1`，utf 字符集指向的是 `utfmb3`。如果管理员忘记修改默认的字符集，就会出现乱码的问题。从 MySQL8 开始，数据库的默认编码改为了 `utfbmb4`, 从而避免乱码问题。

## 2.1 查看字符集
注意: 此处 MySQL5 和 MySQL8 都是通过 docker 安装, 版本如下: `mysql:5.7`、`mysql:8.0`
```mysql
-- mysql5.7
mysql> show variables like '%character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+

-- mysql8
mysql> show variables like '%character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | latin1                         |
| character_set_connection | latin1                         |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | latin1                         |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+
```
在 MySQL5 中, `character_set_server` 和 `character_set_database` 都是 `latin1`，不支持中文，保存中文会报错
```mysql
mysql> create database kinodb1;
Query OK, 1 row affected (0.00 sec)

mysql> use kinodb1
Database changed

mysql> create table test1(id int primary key auto_increment, name varchar(25));
Query OK, 0 rows affected (0.04 sec)

mysql> insert into test1 values(0, '我爱中国');
[HY000][1366] Incorrect string value: '\xE6\x88\x91\xE7\x88\xB1...' for column 'name' at row 1

-- 查看建表语句
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                  |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+
| test1 | CREATE TABLE `test1` (
                                   `id` int(11) NOT NULL AUTO_INCREMENT,
                                   `name` varchar(25) DEFAULT NULL,
                                   PRIMARY KEY (`id`)
          ) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- 查看建库语句
mysql> show create database kinodb1;
+----------+--------------------------------------------------------------------+
| Database | Create Database                                                    |
+----------+--------------------------------------------------------------------+
| kinodb1  | CREATE DATABASE `kinodb1` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+----------+--------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## 2.2 修改字符集
```mysql
$ vim /etc/my.cnf
[mysql]
default-character-set=utf8

[client]
default-character-set=utf8

[mysqld]
character_set_server=utf8

-- 重启 MySQL
# 再次查看字符集(MySQL5)
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```
> 此修改只会针对重启数据库后的后续新建操作生效，历史的误会变更。

## 2.3 修改已有库表的字符集
```mysql
-- 修改已存在的数据库字符集
alter database kinodb1 character set 'utf8';

-- 修改已存在的表字符集
alter table test1 convert to character set 'utf8';
```
> 原来的数据如果使用非 utf8 编码的话，数据本身编码不会发生改变。已有数据需要导出或者删除，然后重新插入。

## 2.4 utf8mb3 和 utf8mb4 
- utf8mb3: 阉割过的 utf8 字符集，只使用 1-3 个字节表示字符。
- utf8mb4: 正宗的 utf8 字符集，使用 1-4 个字节表示字符。

# 三、最大连接数
```bash
$ vim /etc/my.cnf
[mysqld]
...
max_connections=1000

# 查看最大连接数
show variables like '%max_connections%';

# 查看当前连接数
show status like 'Threads%';
```



# 四、修改密码

```mysql
SET PASSWORD FOR 'root'@'localhost'= "Kino123.";
```

# 五、强制修改密码

```bash
```


# 六、max_allowed_packet 
`max_allowed_packet` 是指 mysql server 和 client 在一次传送数据包的过程中最大允许的数据包大小.

```bash
# 临时修改
set global max_allowed_packet = 64 * 1024 * 1024;


# 永久修改
vim my.ini
[mysqld]
max_allowed_packet = 64M 

# 查询
show VARIABLES like '%max_allowed_packet%';
```

# 七、sql_mode
## 7.1 查看当前的 sql_mode
```mysql
-- 三种方式都可以
mysql> select @@session.sql_mode;  
mysql> select @@global.sql_mode; 
mysql> show variables like 'sql_mode';
+-------------------------------------------------------------------------------------------------------------------------------------------+
| @@session.sql_mode                                                                                                                        |
+-------------------------------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```


## 7.2 设置 sql_mode
### 7.2.1 临时设置: 当前窗口中设置 sql_mode
```mysql
set global sql_mode = '...';  -- 全局
set session sql_mode = '...'; -- 当前会话

-- 示例
-- 改为严格模式。在当前会话中生效，关闭当前会话就生效了
set session sql_mode = 'STRICT_TRANS_TABLES';

-- 改为严格模式。在当前服务中生效，重启 MySQL 服务后就失效了
set global sql_mode = 'STRICT_TRANS_TABLES';
```
### 7.2.2 永久设置
修改 `/etc/my.cnf` 设置
```bash
[mysqld]
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

## 7.3 宽松模式
如果设置的是宽松模式，那么在插入数据的时候，即便是给了一个错误的数据，也可以被接收，并且不报错。
```mysql
-- 设置宽松模式
set session sql_mode = '';

-- 创建表
create table test2(id int primary key auto_increment, name varchar(5));

-- insert 记录
insert into test2 values(0, 'abcde123');

-- 查询记录可以看见，name限制为5字符, insert8字符, 实际被截断了，过程中没有异常信息
select * from test2;
+----+-------+
| id | name  |
+----+-------+
|  1 | abcde |
+----+-------+
1 row in set (0.00 sec)
```

## 7.4 严格模式
MySQL5.7 版本就将 sql_mode 默认值改为了严格模式。
```mysql
-- 设置严格模式
set session sql_mode = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';

-- insert 记录会报错
insert into test2 values(0, 'abcde123');
ERROR 1406 (22001): Data too long for column 'name' at row 1
```

## 7.5 sql_mode 取值
1. `ONLY_FULL_GROUP_BY`: 对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么将认为这个SQL是不合法的，因为列不在GROUP BY从句中.
2. `STRICT_TRANS_TABLES`: 在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做任何限制
3. `NO_ZERO_IN_DATE`: 在严格模式，不接受月或日部分为0的日期。如果使用IGNORE选项，我们为类似的日期插入'0000-00-00'。在非严格模式，可以接受该日期，但会生成警告。
4. `NO_ZERO_DATE`: 在严格模式，不要将 '0000-00-00'做为合法日期。你仍然可以用IGNORE选项插入零日期。在非严格模式，可以接受该日期，但会生成警告。
5. `ERROR_FOR_DIVISION_BY_ZERO`: 在严格模式，在INSERT或UPDATE过程中，如果被零除(或MOD(X，0))，则产生错误(否则为警告)。如果未给出该模式，被零除时MySQL返回NULL。如果用到INSERT IGNORE或UPDATE IGNORE中，MySQL生成被零除警告，但操作结果为NULL。
6. `NO_AUTO_CREATE_USER`: 防止GRANT自动创建新用户，除非还指定了密码。
7. `NO_ENGINE_SUBSTITUTION`: 如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常。
8. `ANSI`: 宽松模式，对插入数据进行校验，如果不符合定义类型或长度，对数据类型调整或截断保存，报warning警告。
9. `TRADITIONAL`: 严格模式，当向mysql数据库插入数据时，进行数据的严格校验，保证错误数据不能插入，报error错误。用于事务时，会进行事务的回滚。
10. `STRICT_TRANS_TABLES`: 严格模式，进行数据的严格校验，错误数据不能插入，报error错误。


# 八、修改默认引擎
```mysql
-- 查看数据库默认引擎
mysql> show engines;

-- 设置默认引擎(临时)
mysql> set default_storage_engine=InnoDB;

-- 设置默认引擎(永久)
[mysqld]
default-storage-engine=InnoDB
```

# 九、命令行不能输入中文
```bash
## 在服务器输入
export LANG=en_US.UTF-8

## 再次进入mysql, 即可输入中文
mysql -uroot -p
```













