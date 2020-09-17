


---
# 一、查看数据库编码
```mysql-sql
MariaDB [(none)]> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |   <----------
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |   <----------
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

# 二、修改数据库编码
修改mysql配置文件/etc/my.cnf
```bash
[mysqld]
character-set-server=utf8 
[client]
default-character-set=utf8 
[mysql]
default-character-set=utf8
```

# 三、重启MySQL
```bash
# 重启 mariadb
systemctl restart mariadb
# 重启 mysql
systemctl restart mysqld
```

# 四、查看数据库编码
```mysql-sql
MariaDB [(none)]> show variables like '%char%';
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

# 五、测试
```mysql-sql
MariaDB [test]> create table student(id int, name varchar(12));
Query OK, 0 rows affected (0.01 sec)

MariaDB [test]> insert into student values(1, '张三');
Query OK, 1 row affected (0.01 sec)

MariaDB [test]> select * from student;
+------+--------+
| id   | name   |
+------+--------+
|    1 | 张三   |
+------+--------+
1 row in set (0.00 sec)
```