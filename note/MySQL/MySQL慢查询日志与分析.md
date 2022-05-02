


---


MySQL 查询超过设置阈值(long_query_time)的语句，会记录到慢查询日志中.

# 一、开启 MySQL 慢查询日志
MySQL 默认情况下是没有开启慢查询日志的, 如果不是调优需要，不建议打开，慢查询日志开启会带来一定的性能影响，慢查询日志支持将日志记录写入文件。

## 1.1 查看是否开启
```sql
-- 查看慢查询日志是否开启
mysql> show variables like '%slow_query_log%';
+---------------------+---------------------------------------+
| Variable_name       | Value                                 |
+---------------------+---------------------------------------+
| slow_query_log      | OFF                                   |
| slow_query_log_file | /var/lib/mysql/jz-desktop-06-slow.log |
+---------------------+---------------------------------------+
2 rows in set (0.00 sec)
```

## 1.2 开启
```sql
-- 开启慢查询日志，只对当前数据库生效，并且重启数据库后失效
set global slow_query_log = 1;
```

## 1.3 慢查询日志的阈值
```sql
-- 查看慢查询日志的阈值, 默认 10s
mysql> show variables like '%long_query_time%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)

-- 设置阈值
set long_query_time = 3;
```

## 1.4 永久开始慢查询日志
修改 my.cnf
```sh
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/jz-desktop-06-slow.log
long_query_time=3
log_output=FILE
```













