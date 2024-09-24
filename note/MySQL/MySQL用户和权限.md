








---
# 一、用户管理
## 1.1 登录 MySQL 服务器
```bash
mysql -h hostname|hostIp -P port -u username -p password DatabaseName -e "SQL 语句"
```
参数说明:
- `-h`: 要连接 MySQL 的主机名或者IP地址。
- `-P`: 要连接 MySQL 的端口，不填默认为 3306。
- `-u`: MySQL 登录用户名。
- `-p`: MySQL 密码。
- `DatabaseName`: 指明登录到哪一个数据库中，不指定默认登录到 MySQL 数据库中。
- `-e`: 要执行的 sql 语句，执行完成后退出 MySQL。

示例:
```mysql
mysql -h localhost -uroot -p123456 -P3306 mysql -e "select host,user from user"
```

## 1.2 创建用户
语法:
```mysql
create user 用户名 [IDENTIFIED BY '密码'][, 用户名 [IDENTIFIED BY '密码']];
```
- `用户名`: 表示新建用户的账户，由 `用户(User)` 和 `主机名(hosts)` 构成;
- `[]`: 表示可选， 可以指定用户登录时需要密码验证，也可以不指定密码验证，这样用户可以直接登录。
- `create user` 语句可以同时创建多个用户。

示例:
```mysql
create user zhangsan IDENTIFIED BY '123123'; # 默认hosts是 %

create user 'zhangsan'@'localhost' IDENTIFIED  BY '123123';  # zhangsan 只能由 localhost 登录
```

## 1.3 修改用户
```mysql
update mysql.user set user='zhangsan' where user='zhangsan1';
flush privileges;
```

## 1.4 删除用户
### 方式一: 使用 drop 方式删除(推荐)
需要用户具有 drop user 权限
```mysql
drop user user[, user]...;
```
示例:
```mysql
drop user zhangsan1; ## 默认删除 host 为 % 的用户，否则会报错: ERROR 1396 (HY000): Operation DROP USER failed for 'zhangsan1'@'%'

drop user 'zhangsan'@'localhost';
```

### 方式二: 使用 delete 方式删除
```mysql
delete from  mysql.user where user = 'zhangsan' and host = 'localhost';
flush privileges;
```
执行完 delete 需要 flush 来使用户生效:
```mysql
flush privileges;
```
> 此方式会导致系统有残留信息保留。drop user 会删除用户以及对应的权限，执行命令后会发现 mysql.user 表和 mysql.db 表的相应记录都消失了。

## 1.5 设置当前用户密码
### 方法一
```mysql
-- MySQL5.7
SET PASSWORD = PASSWORD('new_password');
```
### 方法二
```mysql
ALTER USER USER() IDENTIFIED BY 'new_password';
```
### 方法三
```mysql
SET PASSWORD = 'new_password'
```
该方法会自动将密码加密后再赋给当前用户

## 1.6 修改其他用户密码
### 方法一
```mysql
ALTER USER user [IDENTIFIED BY 'new_password'][, user[IDENTIFIED BY 'new_password']];
```
### 方式二
```mysql
SET PASSWORD FOR 'username'@'hostname' = 'new_password';
```
### 方式三(不推荐)
```mysql
UPDATE mysql.user SET authentication_string=PASSWORD('new_password')
WHERE user = 'username' and host = 'hostname';
```

## 1.7 MySQL8 密码管理
### 1.7.1 密码过期策略
在 MySQL 中，数据库管理员可以手动设置账号密码过期，也可以建立一个自动密码过期的策略；

过期策略可以使全局的，也可以为每个账号单独设置过期策略。

例如设置用户立马过期
```mysql
ALTER USER 'zhangsan'@'localhost' PASSWORD EXPIRE;
```
该语句将用户zhangsan的密码设置为过期，zhangsan 用户仍然可以登录进去数据库，但是无法查询。密码过期后，只有重新设置了新的密码，才能正常班使用。

### 1.7.2 手动指定时间过期
**全局方式**

如果密码使用的时间大于允许的时间，服务器会自动设置为过期，不需要手动设置。

MySQL 使用 `default_password_lifetime` 系统变量建立全局密码过期策略。
- 默认值是0，表示禁用自动密码过期；
- 它允许的值是正整数N，表示允许的密码生存期。密码必须每隔N天修改一次。

两种实现方式:
1. 使用 sql 语句更改变量的值并持久化
    ```mysql
    SET PERSIST default_password_lifetime = 180;   -- 建立全局策略，设置密码每隔 180 天过期
    ```
2. 配置文件进行维护
    ```bash
    vim /etc/my.cnf
    [mysqld]
    default_password_lifetime=180   # 建立全局策略，设置密码每隔 180 天过期
    ```
**单独设置**
每个账号既可以延用全局密码过期策略，也可以单独设置策略。在 `create user` 和 `alter user` 语句上加入 `PASSWORD EXPIRE` 选项可以实现单独设置过期策略，下面是一些示例:
```mysql
# 设置 zhangsan 账号密码每 90 天过期
CREATE USER 'zhangsan'@'%' PASSWORD EXPIRE INTERVAL 90 DAY;
ALTER USER 'zhangsan'@'%' PASSWORD EXPIRE INTERVAL 90 DAY;

# 设置密码永不过期
CREATE USER 'zhangsan'@'%' PASSWORD EXPIRE NEVER;
ALTER USER 'zhangsan'@'%' PASSWORD EXPIRE NEVER;

# 延用全局密码过期策略
CREATE USER 'zhangsan'@'%' PASSWORD EXPIRE DEFAULT;
ALTER USER 'zhangsan'@%' PASSWORD EXPIRE DEFAULT;
```

### 1.7.2 密码重用策略 
MySQL 限制使用已用过的密码。重用限制策略基于 **密码更改的数量** 和 **使用的时间**。重用策略可以使全局的，也可以为每个账号设置单独的策略。

- 账号的历史密码包含过去该账号所使用的密码。MySQL 基于以下规则来限制密码的重用：
  - 如果账号的密码限制基于**密码更改的数量**，那么新密码不能从最近限制的密码数量中选择。例如，如果密码更改的最小值为3，那么新密码不能与最近3个密码中任何一个相同。
  - 如果账号密码限制**基于时间**，那么新密码不能从规定时间内选择。例如，如果密码重用周期为 60 天，那么新密码不能从最近 60 天内使用的密码中选择。
- MySQL 使用 `password_history` 和 `password_reuse_interval` 系统变量设置密码重用策略
  - `password_hisory`: 规定密码重用的数量；
  - `password_reuse_interval`: 规定密码重用的周期。
- 这两个值可以在 `服务器的配置文件` 中进行维护，也可以在运行期间 `使用SQL语句更改` 该变量的值并持久化。

**手动设置密码重用方式1(全局)：**
1. 使用sql
    ```mysql
    SET PERSIST password_history = 6; -- 设置密码不能选择最近使用过的 6 个密码
    
    SET PERSIST password_reuse_interval = 365; -- 设置不能选择最近一年内的密码
    ```
2. my.cnf 配置文件
```mysql
[mysqld]
password_history=6
password_reuse_interval=365
```

**手动设置密码重用方式2(单独设置)：**
```mysql
-- 不能使用最近 5 个密码
CREATE USER 'zhangsan'@'%' PASSWORD HISTORY 5;
ALTER USER 'zhangsan'@'%' PASSWORD HISTORY 5;

-- 不能使用最近 365 天的密码
CREATE USER 'zhangsan'@'%' PASSWORD REUSE INTERVAL 365 DAY;
ALTER USER 'zhangsan'@'%' PASSWORD REUSE INTERVAL 365 DAY;

-- 既不能使用最近 5 个密码，也不能使用 365 天内的密码
CREATE USER 'zhangsan'@'%' PASSWORD HISTORY 5 PASSWORD REUSE INTERVAL 365 DAY;
ALTER USER 'zhangsan'@'%' PASSWORD HISTORY 5 PASSWORD REUSE INTERVAL 365 DAY;

-- 延用全局策略
CREATE USER 'zhangsan'@'%' PASSWORD HISTORY DEFAULT PASSWORD REUSE INTERVAL DEFAULT;
ALTER USER 'zhangsan'@'%' PASSWORD HISTORY DEFAULT PASSWORD REUSE INTERVAL DEFAULT;
```


# 二、权限管理
## 2.1 MySQL 权限列表
查看所有权限
```mysql
show privileges;
+-------------------------+---------------------------------------+-------------------------------------------------------+
| Privilege               | Context                               | Comment                                               |
+-------------------------+---------------------------------------+-------------------------------------------------------+
| Alter                   | Tables                                | To alter the table                                    |
| Alter routine           | Functions,Procedures                  | To alter or drop stored functions/procedures          |
| Create                  | Databases,Tables,Indexes              | To create new databases and tables                    |
| Create routine          | Databases                             | To use CREATE FUNCTION/PROCEDURE                      |
| Create temporary tables | Databases                             | To use CREATE TEMPORARY TABLE                         |
| Create view             | Tables                                | To create new views                                   |
| Create user             | Server Admin                          | To create new users                                   |
| Delete                  | Tables                                | To delete existing rows                               |
| Drop                    | Databases,Tables                      | To drop databases, tables, and views                  |
| Event                   | Server Admin                          | To create, alter, drop and execute events             |
| Execute                 | Functions,Procedures                  | To execute stored routines                            |
| File                    | File access on server                 | To read and write files on the server                 |
| Grant option            | Databases,Tables,Functions,Procedures | To give to other users those privileges you possess   |
| Index                   | Tables                                | To create or drop indexes                             |
| Insert                  | Tables                                | To insert data into tables                            |
| Lock tables             | Databases                             | To use LOCK TABLES (together with SELECT privilege)   |
| Process                 | Server Admin                          | To view the plain text of currently executing queries |
| Proxy                   | Server Admin                          | To make proxy user possible                           |
| References              | Databases,Tables                      | To have references on tables                          |
| Reload                  | Server Admin                          | To reload or refresh tables, logs and privileges      |
| Replication client      | Server Admin                          | To ask where the slave or master servers are          |
| Replication slave       | Server Admin                          | To read binary log events from the master             |
| Select                  | Tables                                | To retrieve rows from table                           |
| Show databases          | Server Admin                          | To see all databases with SHOW DATABASES              |
| Show view               | Tables                                | To see views with SHOW CREATE VIEW                    |
| Shutdown                | Server Admin                          | To shut down the server                               |
| Super                   | Server Admin                          | To use KILL thread, SET GLOBAL, CHANGE MASTER, etc.   |
| Trigger                 | Tables                                | To use triggers                                       |
| Create tablespace       | Server Admin                          | To create/alter/drop tablespaces                      |
| Update                  | Tables                                | To update existing rows                               |
| Usage                   | Server Admin                          | No privileges - allow connect only                    |
+-------------------------+---------------------------------------+-------------------------------------------------------+
31 rows in set (0.00 sec)
```

1. CREATE 和 DROP 权限: 可以 创建/删除 数据库和表。
2. INSERT、UPDATE、DELETE: 允许在一个数据库现有的表上实施操作。
3. SELECT: 只有在真正从一个表检索行时才会被用到。
4. INDEX: 允许创建或删除索引，INDEX 适用于已有的表。如果具有某个表的 CREATE 权限，就可以在 CREATE TABLE 语句中创建包括索引的定义。
5. ALTER: 可以使用 ALTER TABLE 来更改表的结构和重新命令表。
6. CREATE ROUTINE: 用来创建保存的程序(函数), CREATE ROUTINE 权限用来更改和删除保存的程序
7. EXECUTE: 用来执行保存的程序。
8. GRANT: 允许授权给其他用户，可用于数据库、表和保存程序。
9. FILE: 使用户可以使用 LOAD DATA INFILE 和 SELECT ... INTO OUTFILE 语句读或写服务器上的文件，任何被授予 FILE 权限的用户都能读或者写 MySQL 服务器上的任何文件(说明：用户可以读任何数据库目录下的文件，因为服务器可以访问这些文件)

## 2.2 授权
授权命令
```mysql
GRANT 权限1,权限2,...权限n ON 数据库名称.表名称 TO 用户名@用户地址 [IDENTIFIED BY '密码'];
```
如果发现被授权用户不存在，会直接创建一个用户。

示例
```mysql
-- 授予 zhangsan 用户在 kinodb 数据库下所有表的增删改查的权限
GRANT SELECT, INSERT, DELETE, UPDATE ON kiondb.* TO 'zhangsan'@'%';

-- 授予 zhangsan 用户所有库所有表的所有权限
GRANT ALL PRIVILEGES ON *.* TO 'zhangsan'@'%' IDENTIFIED BY '123456';
```

## 2.3 查看权限
```mysql
-- 查看当前用户选线
SHOW GRANTS;

SHOW GRANTS FOR CURRENT_USER;

SHOW GRANTS FOR CURRENT_USER();

-- 查看某用户的全局权限
SHOW GRANTS FOR 'user'@'主机地址';
```

## 2.4 收回权限
收回权限命令:
```mysql
REVOKE 权限1, 权限2,...权限n ON 数据库.表名 FROM 用户名@用户地址;
```
示例
```mysql
-- 收回 zhangsan 用户所有权限
REVOKE ALL PRIVILEGES ON *.* FROM 'zhangsan'@'%';

-- 收回 mysql 库下所有表的增删改查权限
REVOKE SELECT,INSERT,UPDATE,DELETE ON mysql.* FROM 'zhangsan'@'%';
```
> 需要用户重新登陆才生效

# 三、权限表
## 3.1 user 表
```mysql
mysql> desc mysql.user;
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Field                  | Type                              | Null | Key | Default               | Extra |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Host                   | char(60)                          | NO   | PRI |                       |       |
| User                   | char(32)                          | NO   | PRI |                       |       |
| Select_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Insert_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Update_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Delete_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Create_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Drop_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Reload_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Shutdown_priv          | enum('N','Y')                     | NO   |     | N                     |       |
| Process_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| File_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Grant_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| References_priv        | enum('N','Y')                     | NO   |     | N                     |       |
| Index_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Show_db_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Super_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tmp_table_priv  | enum('N','Y')                     | NO   |     | N                     |       |
| Lock_tables_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Execute_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_slave_priv        | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_client_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Create_view_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Show_view_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Create_routine_priv    | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_routine_priv     | enum('N','Y')                     | NO   |     | N                     |       |
| Create_user_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Event_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Trigger_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tablespace_priv | enum('N','Y')                     | NO   |     | N                     |       |
| ssl_type               | enum('','ANY','X509','SPECIFIED') | NO   |     |                       |       |
| ssl_cipher             | blob                              | NO   |     | NULL                  |       |
| x509_issuer            | blob                              | NO   |     | NULL                  |       |
| x509_subject           | blob                              | NO   |     | NULL                  |       |
| max_questions          | int(11) unsigned                  | NO   |     | 0                     |       |
| max_updates            | int(11) unsigned                  | NO   |     | 0                     |       |
| max_connections        | int(11) unsigned                  | NO   |     | 0                     |       |
| max_user_connections   | int(11) unsigned                  | NO   |     | 0                     |       |
| plugin                 | char(64)                          | NO   |     | mysql_native_password |       |
| authentication_string  | text                              | YES  |     | NULL                  |       |
| password_expired       | enum('N','Y')                     | NO   |     | N                     |       |
| password_last_changed  | timestamp                         | YES  |     | NULL                  |       |
| password_lifetime      | smallint(5) unsigned              | YES  |     | NULL                  |       |
| account_locked         | enum('N','Y')                     | NO   |     | N                     |       |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
45 rows in set (0.00 sec)
```
这些字段可以分成4类
- 范围列(或用户列)
- 权限列
- 安全列
- 资源控制列

1. 范围列(或用户列)
   - host: 表示连接类型
     - `%`: 表示所有远程通过 TCP 方式的连接。
     - `IP 地址`: 通过定制 IP 地址进行的 TCP 方式的连接。
     - `机器名`: 通过定制网络中的机器名进行的 TCP 方式连接。
     - `::1`: IPv6 的本地 IP 地址，等同于 IPv4 的 127.0.0.1；
     - `localhost`: 本地方式通过命令行的方式连接，比如 `mysql -uroot -pxxx` 方式的连接。
   - user: 表示用户名， 用一用户通过不同方式链接的权限是不一样的。
   - password: 密码
2. 权限列
   - Grant_priv: 表示是否拥有 GRANT 权限。
   - Shutdown_priv: 表示是否拥有停止 MySQL 服务的权限。
   - Super_priv: 表示是否拥有超级权限。
   - Execute_priv: 表示是否拥有 EXECUTE 权限。拥有这个权限，可以执行存储过程和函数。
   - Select_priv、Insert_priv 等: 为该用户所拥有的权限。
3. 安全列: 安全列只有 6 个字段，其中两个是 ssl 相关的(ssl_type、ssl_cipher), 用于**加密**；两个是 x509 相关的(x509_issuer、x509_subject), 用户**标识用户**；另外两个 Plugin 字段用户 **验证用户身份** 的插件，该字段不能为空。如果该字段为空，服务器内部就使用内建授权验证机制验证用户身份。
4. 资源控制列: 资源控制列的字段用来 **限制用户使用的资源**， 包含 4 个字段:
   - max_questions: 用户每小时允许执行的查询操作次数；
   - max_updates: 用户每小时允许执行的更新操作次数;
   - max_connections: 用户每小时允许执行的连接操作次数;
   - max_user_connections: 用户允许同时建立的连接次数。

## 3.2 db类
使用 describe 查看 db 表的基本结构
```mysql
mysql> describe mysql.db;
+-----------------------+---------------+------+-----+---------+-------+
| Field                 | Type          | Null | Key | Default | Extra |
+-----------------------+---------------+------+-----+---------+-------+
| Host                  | char(60)      | NO   | PRI |         |       |
| Db                    | char(64)      | NO   | PRI |         |       |
| User                  | char(32)      | NO   | PRI |         |       |
| Select_priv           | enum('N','Y') | NO   |     | N       |       |
| Insert_priv           | enum('N','Y') | NO   |     | N       |       |
| Update_priv           | enum('N','Y') | NO   |     | N       |       |
| Delete_priv           | enum('N','Y') | NO   |     | N       |       |
| Create_priv           | enum('N','Y') | NO   |     | N       |       |
| Drop_priv             | enum('N','Y') | NO   |     | N       |       |
| Grant_priv            | enum('N','Y') | NO   |     | N       |       |
| References_priv       | enum('N','Y') | NO   |     | N       |       |
| Index_priv            | enum('N','Y') | NO   |     | N       |       |
| Alter_priv            | enum('N','Y') | NO   |     | N       |       |
| Create_tmp_table_priv | enum('N','Y') | NO   |     | N       |       |
| Lock_tables_priv      | enum('N','Y') | NO   |     | N       |       |
| Create_view_priv      | enum('N','Y') | NO   |     | N       |       |
| Show_view_priv        | enum('N','Y') | NO   |     | N       |       |
| Create_routine_priv   | enum('N','Y') | NO   |     | N       |       |
| Alter_routine_priv    | enum('N','Y') | NO   |     | N       |       |
| Execute_priv          | enum('N','Y') | NO   |     | N       |       |
| Event_priv            | enum('N','Y') | NO   |     | N       |       |
| Trigger_priv          | enum('N','Y') | NO   |     | N       |       |
+-----------------------+---------------+------+-----+---------+-------+
22 rows in set (0.00 sec)
```
1. 用户列: db 表用户列有 3 个字段，分别是 host、user、db。这三个字段分别表示主机名、用户名、数据库名。表示从某个主机某个用户对某个数据库的操作权限，这 3 个字段的组合构成了 db 表的主键。
2. 权限列: `Create_routine_priv` 和 `Alter_routine_priv` 这两个字段决定用户是否具有创建和修改存储过程的权限。

## 3.3 table_priv 表和 columns_priv 表
tables_priv 表用来对表**设置操作权限**，columns_priv 表用来对表的**某一列设置权限**。

```mysql
mysql> desc mysql.tables_priv;
+-------------+-----------------------------------------------------------------------------------------------------------------------------------+------+-----+-------------------+-----------------------------+
| Field       | Type                                                                                                                              | Null | Key | Default           | Extra                       |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------+------+-----+-------------------+-----------------------------+
| Host        | char(60)                                                                                                                          | NO   | PRI |                   |                             |
| Db          | char(64)                                                                                                                          | NO   | PRI |                   |                             |
| User        | char(32)                                                                                                                          | NO   | PRI |                   |                             |
| Table_name  | char(64)                                                                                                                          | NO   | PRI |                   |                             |
| Grantor     | char(93)                                                                                                                          | NO   | MUL |                   |                             |
| Timestamp   | timestamp                                                                                                                         | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
                                                                                                                                                                                            | Table_priv  | set('Select','Insert','Update','Delete','Create','Drop','Grant','References','Index','Alter','Create View','Show view','Trigger') | NO   |     |                   |                             |
| Column_priv | set('Select','Insert','Update','References')                                                                                      | NO   |     |                   |                             |
+-------------+-----------------------------------------------------------------------------------------------------------------------------------+------+-----+-------------------+-----------------------------+
8 rows in set (0.00 sec)
```
- Host、Db、User 和 Table_name 四个字段分别表示主机名、数据库名、用户名和表名。
- Grantor 表示修改该记录的用户。
- Timestamp 表示修改该记录的时间。
- Table_priv 表示对象的操作权限。包括 Select、Insert、Update、Delete、Create、Drop、Grant、Reference、Index 和 Alter。
- Column_priv 表示对表中的列的操作权限，包括 Select、Insert、Update 和 References。

```mysql
mysql> desc mysql.columns_priv;
+-------------+----------------------------------------------+------+-----+-------------------+-----------------------------+
| Field       | Type                                         | Null | Key | Default           | Extra                       |
+-------------+----------------------------------------------+------+-----+-------------------+-----------------------------+
| Host        | char(60)                                     | NO   | PRI |                   |                             |
| Db          | char(64)                                     | NO   | PRI |                   |                             |
| User        | char(32)                                     | NO   | PRI |                   |                             |
| Table_name  | char(64)                                     | NO   | PRI |                   |                             |
| Column_name | char(64)                                     | NO   | PRI |                   |                             |
| Timestamp   | timestamp                                    | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| Column_priv | set('Select','Insert','Update','References') | NO   |     |                   |                             |
+-------------+----------------------------------------------+------+-----+-------------------+-----------------------------+
7 rows in set (0.00 sec)
```

## 3.4 procs_priv 表
procs_priv 表可以对存储过程和存储函数设置操作权限，表结构如下:
```mysql
mysql> desc mysql.procs_priv;
+--------------+----------------------------------------+------+-----+-------------------+-----------------------------+
| Field        | Type                                   | Null | Key | Default           | Extra                       |
+--------------+----------------------------------------+------+-----+-------------------+-----------------------------+
| Host         | char(60)                               | NO   | PRI |                   |                             |
| Db           | char(64)                               | NO   | PRI |                   |                             |
| User         | char(32)                               | NO   | PRI |                   |                             |
| Routine_name | char(64)                               | NO   | PRI |                   |                             |
| Routine_type | enum('FUNCTION','PROCEDURE')           | NO   | PRI | NULL              |                             |
| Grantor      | char(93)                               | NO   | MUL |                   |                             |
| Proc_priv    | set('Execute','Alter Routine','Grant') | NO   |     |                   |                             |
| Timestamp    | timestamp                              | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+--------------+----------------------------------------+------+-----+-------------------+-----------------------------+
8 rows in set (0.00 sec)
```

# 四、访问控制

## 4.1 连接核实阶段
当用户试图连接 MySQL 服务器时，服务器基于用户的身份以及用户是否能提供正确的密码验证身份来确定接受或者拒接连接。即客户端用户会在连接请求中提供用户名、主机地址、用户密码，MySQL 服务器接收到用户请求后，会**使用 user 表中的 host、user 和 authentication_string 这 3 个字段匹配客户端提供的信息。**

服务器只有在 user 表记录在 Host 和 User 字段匹配客户端主机名和用户名，并且提供正确的密码时才接受连接。如果连接核实没有通过，服务器就完全拒绝访问；否则，服务器接收连接，然后进入阶段2等待用户请求。

## 4.2 请求核实阶段
一旦建立了连接，服务器就进入了访问控制的阶段2，也就是请求核实阶段。对此连接上进来的每个请求，服务器检查该请求要执行什么操作、是否有足够的权限来执行它，这正是需要授权表中的权限列发挥作用的地方。这些权限可以来自 user、db、table_priv 和 column_priv 表。

确认权限时，MySQL 首先检查 user 表, 如果指定的权限没有在 user 表中被授予，那么 MySQL 就会继续 **检查db表**，db表是下一安全层级，其中的权限限定于数据库层级，在该层级的 SELECT 权限允许用户查看指定数据库的所有表中的数据；如果再该层级没有找到限定的权限，MySQL 继续 **检查tables_priv表**以及 **columns_priv 表**，如果若有权限表都检查完毕，但是还是没有找到允许的权限操作，MySQL 将返回错误信息，用户请求的操作不能执行，操作失败。

> MySQL 通过向下层级的顺序(从 User 表到 columns_priv 表)检查权限表，但并不是所有的权限都要执行该过程。例如，一个用户登录到 MySQL 服务器之后对 MySQL 的管理操作，此时只设计到管理权限，因此 MySQL 只检查 user 表。另外，如果请求的权限操作不被允许，MySQL 也不会继续检查下一层级的表。



# 五、角色管理
角色是 MySQL8 中引入的新功能。
## 5.1 创建角色
语法
```mysql
CREATE ROLE 'role_name'[@'host_name'][,'role'[@'host_name']]...
```
角色名称的命令规则和用户名类似。如果 host_name 省略，默认为 %, role_name 不可以省略，不可为空。

示例
```mysql
-- 创建 manager 角色
CREATE ROLE 'manager'@'%';
```

## 5.2 给角色赋予权限
```mysql
GRANT privileges ON table_name TO 'role_name'[@'host_name'];
```
privileges 代表权限的名称，多个权限以逗号隔开。可使用 SHOW 语句查询权限名称，如下:
```mysql
mysql> SHOW PRIVILEGES\G;
*************************** 1. row ***************************
Privilege: Alter
Context: Tables
  Comment: To alter the table
*************************** 2. row ***************************
Privilege: Alter routine
  Context: Functions,Procedures
  Comment: To alter or drop stored functions/procedures
*************************** 3. row ***************************
Privilege: Create
Context: Databases,Tables,Indexes
  Comment: To create new databases and tables
*************************** 4. row ***************************
Privilege: Create routine
  Context: Databases
  Comment: To use CREATE FUNCTION/PROCEDURE
*************************** 5. row ***************************
Privilege: Create role
    Context: Server Admin
  Comment: To create new roles
*************************** 6. row ***************************
Privilege: Create temporary tables
  Context: Databases
  Comment: To use CREATE TEMPORARY TABLE
*************************** 7. row ***************************
...
```

示例: 给 manager 角色赋予 kinodb 的 user 表只读权限
```mysql
GRANT SELECT ON kinodb.user TO 'manager'@'%';
```

## 5.3 查看角色的权限
```mysql
mysql> show grants for 'manager';
+--------------------------------------------------+
| Grants for manager@%                             |
+--------------------------------------------------+
| GRANT USAGE ON *.* TO `manager`@`%`              |
| GRANT SELECT ON `kinodb`.`user` TO `manager`@`%` |
+--------------------------------------------------+
2 rows in set (0.00 sec)
```

## 5.4 收回角色的权限
语法
```mysql
REVOKE privileges ON tablename FROM 'role_name';
```
示例: 撤销 manager 角色的SELECT权限
```mysql
REVOKE SELECT ON kinodb.user FROM 'manager';
```

## 5.5 删除角色
语法
```mysql
DROP ROLE role_name [, role2]...
```
示例: 删除角色 manager
```mysql
DROP ROLE 'manager';
```

## 5.6 给用户赋予角色
语法
```mysql
GRANT role [, role2, ...] TO user [, user2...];
```
role 代表角色，user代表用户，可以将多个角色同时赋予多个用户，用逗号隔开。

示例: 给 kino 用户赋予 manager 角色
```mysql
-- 创建用户
CREATE USER kino@'%' IDENTIFIED BY '123456';

-- 创建角色 
CREATE ROLE manager@'%';

-- 创建数据库
CREATE DATABASE kinodb;

-- 创建 user 表
use kinodb;
CREATE TABLE user(id int primary key auto_increment, name varchar(200));

-- 插入数据
INSERT INTO user values(0, 'kino1');

-- 给 manager 角色赋予 kinodb 所有读权限
GRANT SELECT ON kinodb.* FROM 'manager'@'%';

-- 登录 kino 用户查看是否能查询到 user 表
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| performance_schema |
+--------------------+
2 rows in set (0.00 sec)

-- 给 kino 用户赋予 manager 角色
GRANT 'manager'@'%' TO 'kino'@'%';

-- 添加完成后使用 SHOW 语句查看是否添加成功
SHOW GRANTS FOR 'manager'@'%';

-- 使用 kino 用户登录，查询当前角色，如果角色未激活，结果将显示 NONE
SELECT CURRENT_ROLE();
+----------------+
| CURRENT_ROLE() |
+----------------+
| NONE           |
+----------------+
1 row in set (0.00 sec)

-- 查看是否能查询到 user 表
此时还是看不到权限，是因为还没激活角色
```

## 5.7 激活角色
方式一
```mysql
-- 为 kinodb 用户激活所有已拥有的角色
SET DEFAULT ROLE ALL TO 'kino'@'%';

-- 此时再登录 kino 用户查看 kinodb.user
SELECT * FROM kinodb.user;
+----+-------+
| id | name  |
+----+-------+
|  1 | kino1 |
+----+-------+
1 row in set (0.00 sec)
```

方式二: 将 `activate_all_roles_on_login` 设置为 `ON`
```mysql
-- 查看 activate_all_roles_on_login 的设置
SHOW VARIABLES LIKE 'activate_all_roles_on_login'
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| activate_all_roles_on_login | OFF   |
+-----------------------------+-------+
1 row in set (0.01 sec)

-- 设置
SET GLOBAL activate_all_roles_on_login=ON;  -- 对所有角色永久激活，执行完之后用户才真正拥有了赋予角色的所有权限
```

## 5.8 撤销用户的角色
语法
```mysql
REVOKE role FROM user;
```
示例: 撤销 kino 用户的 manager 角色
```mysql
-- 撤销
REVOKE 'manager'@'%' FROM 'kino'@'%';

-- 查看用户的角色信息
SHOW GRANTS FOR 'kino'@'%';
+----------------------------------+
| Grants for kino@%                |
+----------------------------------+
| GRANT USAGE ON *.* TO `kino`@`%` |
+----------------------------------+
1 row in set (0.00 sec)
```
执行完之后，再次登录 kino 用户，发现已经没有 kinodb.user 的查询权限。

## 5.9 设置强制角色(mandatory role)
方式一: 服务启动前设置
```bash
[mysqld]
mandatory_role='role1,role2@localhost,role3@192.168.1.0';
```
方式二: 运行时设置
```mysql
SET PERSIST mandatory_role='role1,role2@localhost,role3@192.168.1.0'; -- 系统重启后仍然生效

SET GLOBAL mandatory_role='role1,role2@localhost,role3@192.168.1.0'; -- 系统重启后失效
```

























