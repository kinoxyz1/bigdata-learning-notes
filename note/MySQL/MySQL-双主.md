





用docker方式模拟搭建
```bash
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
lower_case_table_names=1
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
skip-character-set-client-handshake
skip-name-resolve
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# 主从同步
server-id = 1
log-bin = mysql-bin
sync_binlog = 1
binlog_checksum = none
binlog_format = mixed
auto-increment-increment = 2
auto-increment-offset = 1
slave-skip-errors = all
event_scheduler = 1
max_allowed_packet = 64M
# Custom config should go here
!includedir /etc/mysql/conf.d/
```

docker
```bash
$ docker run -it -d --name mysql1 \
  -p 3306:3306 \
  -v ./data:/var/lib/mysql \
  -v ./my.cnf:/etc/mysql/my.cnf \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e TZ=Asia/Shanghai \
  mysql:5.7
```

查看当前 master 状态
```bash
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      150 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

创建用于主从复制的用户
```bash
CREATE USER 'replica' IDENTIFIED WITH mysql_native_password BY 'replica123';
GRANT REPLICATION SLAVE ON *.* TO 'replica';
FLUSH PRIVILEGES;
```

MySQL2 部署
```bash
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# 主从同步
server-id = 2
log-bin = mysql-bin
sync_binlog = 1
binlog_checksum = none
binlog_format = mixed
auto-increment-increment = 2
auto-increment-offset = 2
slave-skip-errors = all
event_scheduler = 1
max_allowed_packet = 64M
# Custom config should go here
!includedir /etc/mysql/conf.d/
```


docker
```bash
$ docker run -it -d --name mysql2 \
-p 3305:3306 \
-v ./data:/var/lib/mysql \
-v ./my.cnf:/etc/mysql/my.cnf \
-e MYSQL_ROOT_PASSWORD=root123 \
-e TZ=Asia/Shanghai \
mysql:5.7
```

查看当前 master 状态
```bash
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      150 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

创建用于主从复制的用户
```bash
CREATE USER 'replica' IDENTIFIED WITH mysql_native_password BY 'replica123';
GRANT REPLICATION SLAVE ON *.* TO 'replica';
FLUSH PRIVILEGES;
```

配置主主复制

在 MySQL1 中执行:
```bash
mysql> change master to master_host='192.168.1.249',master_port=3305,master_user='replica',master_password='replica123',master_log_file='mysql-bin.000003',master_log_pos=889;
```
master_host 和 master_port 是 MySQL2 的信息

启动同步进程
```bash
mysql> start slave;
```

查看同步状态
```bash
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.249
                  Master_User: replica
                  Master_Port: 3305
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 889
               Relay_Log_File: 3dcf2a742bdf-relay-bin.000002
                Relay_Log_Pos: 312
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes   <----
            Slave_SQL_Running: Yes   <----
```

如果配置有问题, 可以执行如下命令重新配置
```bash
stop slave;
reset slave all;
```

在 mysql2 中执行
```bash
change master to master_host='192.168.1.249',master_port=3306,master_user='replica',master_password='replica123',master_log_file='mysql-bin.000003',master_log_pos=740;
```

启动同步进程
```bash
mysql> start slave;
```

查看同步状态
```bash
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.249
                  Master_User: replica
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 740
               Relay_Log_File: 754b2b2118d2-relay-bin.000002
                Relay_Log_Pos: 312
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes   <----
            Slave_SQL_Running: Yes   <----
```

测试主主复制

在 mysql1 中执行如下命令
```bash
mysql> create database kinodb;
Query OK, 1 row affected (0.00 sec)
```
在 mysql2 中查看
```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kinodb             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

在 mysql2 中建表
```bash
mysql> create table t1(id int);
Query OK, 0 rows affected (0.00 sec)
```
在 mysql1 中查看
```bash
mysql> use kinodb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_kinodb |
+------------------+
| t1               |
+------------------+
1 row in set (0.00 sec)
```

在 mysql1 中插入数据
```bash
insert into t1 values(1);
```
在 mysql2 中查询
```bash
mysql> select * from t1;
+------+
| id   |
+------+
|    1 |
+------+
1 row in set (0.00 sec)
```

配置 keepalived


在每台服务器上安装 Keepalived：

```bash
yum install keepalived -y
```

MySQL1 Keepalived 配置文件 (/etc/keepalived/keepalived.conf):
```bash
vrrp_script chk_mysql {
    script "/etc/keepalived/chk_mysql.sh"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.22.0.100
    }
    track_script {
        chk_mysql
    }
}
```

MySQL2 Keepalived 配置文件 (/etc/keepalived/keepalived.conf):
```bash
vrrp_script chk_mysql {
    script "/etc/keepalived/chk_mysql.sh"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.22.0.100
    }
    track_script {
        chk_mysql
    }
}
```
健康检查脚本 (/etc/keepalived/chk_mysql.sh):
```bash
#!/bin/bash
if ! mysqladmin ping -h 127.0.0.1 --silent; then
    exit 1
fi
exit 0
```
设置脚本可执行权限：
```bash
chmod +x /etc/keepalived/chk_mysql.sh
```
启动 Keepalived：
```bash
systemctl start keepalived
systemctl enable keepalived
```