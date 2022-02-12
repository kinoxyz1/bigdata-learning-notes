



---
# 一、忽略大小写
```bash
$ vim /etc/my.cof
# 在 [mysqld] 加入
lower_case_table_names=1
```

# 二、字符集
```bash
# 查看 默认字符集
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

# 修改字符集
$ vim /etc/my.cnf
[mysql]
default-character-set=utf8

[client]
default-character-set=utf8

[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'

# 再次查看字符集
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

