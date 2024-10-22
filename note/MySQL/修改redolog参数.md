







# 一、停止mysql
```bash
$ systemctl stop mysqld
```

# 二、备份数据
```bash
$ cp -rp /var/lib/mysql /var/lib/mysql_backup

cp -rp /data/mysql/data /data/mysql/data_backup
```

# 三、修改配置文件
```bash
$ vim /etc/my.cnf
[mysqld]
innodb_log_file_size = 指定大小  # 单位字节
```

# 四、删除旧的日志文件
```bash
$ rm -f /var/lib/mysql/ib_logfile*
```

# 五、重启mysql
```bash
$ systemctl start mysqld
```

# 六、检查日志文件的大小
```bash
$ SHOW VARIABLES LIKE 'innodb_log_file_size';
```





















