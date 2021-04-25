





---
# 一、MySQL RMP 安装包下载
[MySQL 官方下载路径](https://dev.mysql.com/downloads/mysql/)

# 二、卸载系统自带的 MySQL
```bash
$ rpm -qa | grep -i mysql
```

# 三、安装 MySQL
```bash
$ rpm -ivh mysql-community-common-5.7.34-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-common-5.7.34-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-libs-5.7.34-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-devel-5.7.34-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-libs-compat-5.7.34-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-client-5.7.34-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-server-5.7.34-1.el7.x86_64.rpm
```

# 四、启动和开机自启
```bash
# 查看 MySQL 状态
$ systemctl status mysqld
# 启动 MySQL
$ systemctl start mysqld
# 设置开机自启
$ systemctl enable mysqld
```


# 五、设置 ROOT 密码
```bash
$ grep 'temporary password' /var/log/mysqld.log
2021-04-25T15:12:55.574410Z 1 [Note] A temporary password is generated for root@localhost: V2_s+Vz7fEAs
```
# 六、重置 MySQL 密码
```bash
$ mysql -uroot -p

# 密码不能过于简单
mysql> SET PASSWORD FOR 'root'@'localhost'= "Kino123.";
Query OK, 0 rows affected (0.00 sec)
```

# 七、设置 MySQL 远程登录
```bash
$ GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Kino123.' WITH GRANT OPTION;
$ FLUSH PRIVILEGES;
$ exit;
```

# 八、开放 3306 端口
```bash
# firewall-cmd --zone=public --add-port=3306/tcp --permanent
# firewall-cmd --reload
```

# 九、忽略大小写
```bash
$ vim /etc/my.cof
# 在 [mysqld] 加入
lower_case_table_names=1
```

# 十、重启 MySQL
```bash
$ systemctl restart mysqld
```