

* [一、下载](#%E4%B8%80%E4%B8%8B%E8%BD%BD)
* [二、解压](#%E4%BA%8C%E8%A7%A3%E5%8E%8B)
* [三、修改配置文件名](#%E4%B8%89%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%90%8D)
* [四、配置 hive\-env\.sh文件](#%E5%9B%9B%E9%85%8D%E7%BD%AE-hive-envsh%E6%96%87%E4%BB%B6)
* [五、配置 Hadoop 集群](#%E4%BA%94%E9%85%8D%E7%BD%AE-hadoop-%E9%9B%86%E7%BE%A4)
  * [5\.1 必须启动 hdfs 和 yarn](#51-%E5%BF%85%E9%A1%BB%E5%90%AF%E5%8A%A8-hdfs-%E5%92%8C-yarn)
  * [5\.2 在 HDFS 上创建文件夹并赋予权限](#52-%E5%9C%A8-hdfs-%E4%B8%8A%E5%88%9B%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%B9%B6%E8%B5%8B%E4%BA%88%E6%9D%83%E9%99%90)
* [六、安装 MariaDB, 配置权限](#%E5%85%AD%E5%AE%89%E8%A3%85-mariadb-%E9%85%8D%E7%BD%AE%E6%9D%83%E9%99%90)
  * [6\.1 安装](#61-%E5%AE%89%E8%A3%85)
  * [6\.2 配置](#62-%E9%85%8D%E7%BD%AE)
* [七、Hive 元数据配置到 MariaDB](#%E4%B8%83hive-%E5%85%83%E6%95%B0%E6%8D%AE%E9%85%8D%E7%BD%AE%E5%88%B0-mariadb)
  * [7\.1 下载 MariaDB 驱动包](#71-%E4%B8%8B%E8%BD%BD-mariadb-%E9%A9%B1%E5%8A%A8%E5%8C%85)
  * [7\.2 解压后 将 驱动包 拷贝到 $HIVE\_HOME/lib 目录下](#72-%E8%A7%A3%E5%8E%8B%E5%90%8E-%E5%B0%86-%E9%A9%B1%E5%8A%A8%E5%8C%85-%E6%8B%B7%E8%B4%9D%E5%88%B0-hive_homelib-%E7%9B%AE%E5%BD%95%E4%B8%8B)
  * [7\.3 配置 Metastore 到 MariaDB](#73-%E9%85%8D%E7%BD%AE-metastore-%E5%88%B0-mariadb)
* [八、连接到hive](#%E5%85%AB%E8%BF%9E%E6%8E%A5%E5%88%B0hive)
* [九、HiveHDBC访问](#%E4%B9%9Dhivehdbc%E8%AE%BF%E9%97%AE)
  * [9\.1 启动 hiveserver2](#91-%E5%90%AF%E5%8A%A8-hiveserver2)



----
# 一、下载
http://archive.apache.org/dist/hive/

# 二、解压
```bash
[root@hadoop1 hive-3.1.2]# tar -zxvf apache-hive-3.1.2-bin.tar.gz -C /usr/bigdata/
[root@hadoop1 hive-3.1.2]# cd /usr/bigdata/
[root@hadoop1 hive-3.1.2]# mv apache-hive-3.1.2-bin/ hive-3.1.2
[root@hadoop1 hive-3.1.2]# cd hive-3.1.2/
```

# 三、修改配置文件名
修改 /usr/bigdata/hive-3.1.2/conf 目录下的 hive-env.sh.template 名称为 hive-env.sh 
```bash
[root@hadoop1 conf]# mv hive-env.sh.template hive-env.sh
``` 

# 四、配置 hive-env.sh文件
① 配置HADOOP_HOME路径
```bash
[root@hadoop1 conf]# vim hive-env.sh
export HADOOP_HOME=/usr/bigdata/hadoop-3.1.1
```
② 配置HIVE_CONF_DIR路径
```bash
[root@hadoop1 conf]# vim hive-env.sh
export HIVE_CONF_DIR=/usr/bigdata/hive-3.1.2
```

# 五、配置 Hadoop 集群
## 5.1 必须启动 hdfs 和 yarn
```bash
[root@hadoop1 conf]# start-all.sh
```
## 5.2 在 HDFS 上创建文件夹并赋予权限
```bash
[root@hadoop1 conf]# hadoop fs -mkdir /tmp
[root@hadoop1 conf]# hadoop fs -mkdir -p /user/hive/warehouse
[root@hadoop1 conf]# hadoop fs -chmod g+w /tmp
[root@hadoop1 conf]# hadoop fs -chmod g+w /user/hive/warehouse
```

# 六、安装 MariaDB, 配置权限
## 6.1 安装
[CentOS7安装MariaDB](../../note/MariaDB/CentOS7安装MariaDB.md)

## 6.2 配置
① 连接到mysql
```bash
[root@hadoop1 conf]# mysql -uroot -p
输入密码
```
② 查询user表
```bash
MariaDB [(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> select User, Host, Password from user;
+------+-----------+-------------------------------------------+
| User | Host      | Password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
| root | hadoop1   | *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
| root | 127.0.0.1 | *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
| root | ::1       | *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
+------+-----------+-------------------------------------------+
4 rows in set (0.00 sec)
```
③ 修改user表，把Host表内容修改为%
```bash
MariaDB [mysql]> update user set host='%' where host='localhost';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
④ 删除root用户的其他host
```bash
MariaDB [mysql]> delete from user where Host='hadoop1';
Query OK, 1 row affected (0.00 sec)

MariaDB [mysql]> delete from user where Host='127.0.0.1';
Query OK, 1 row affected (0.00 sec)

MariaDB [mysql]> delete from user where Host='::1';
Query OK, 1 row affected (0.00 sec)
```
⑤ 刷新
```bash
MariaDB [mysql]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [mysql]> quit
Bye
```

# 七、Hive 元数据配置到 MariaDB
## 7.1 下载 MariaDB 驱动包
下载地址: https://downloads.mariadb.com/Connectors/java/connector-java-2.1.2/mariadb-java-client-2.1.2.jar

## 7.2 解压后 将 驱动包 拷贝到 $HIVE_HOME/lib 目录下
```bash
[root@hadoop1 opt]# mv mariadb-java-client-2.1.2.jar /usr/bigdata/hive-3.1.2/lib/
```

## 7.3 配置 Metastore 到 MariaDB
① 在 $HIVE_HOME/conf 目录下创建 hive-site.xml
```bash
[root@hadoop1 conf]# vim hive-site.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
        <property>
          <name>javax.jdo.option.ConnectionURL</name>
          <value>jdbc:mysql://hadoop1:3306/metastore?createDatabaseIfNotExist=true</value>
          <description>JDBC connect string for a JDBC metastore</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionDriverName</name>
          <value>org.mariadb.jdbc.Driver</value>
          <description>Driver class name for a JDBC metastore</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionUserName</name>
          <value>root</value>
          <description>username to use against metastore database</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionPassword</name>
          <value>123</value>
          <description>password to use against metastore database</description>
        </property>
</configuration>
```
② 初始化hive元数据库
```bash
[root@hadoop1 bin]# schematool -dbType mysql -initSchema
```
③ 查看 MariaDB, metastore 数据已经生成了
```bash
[root@hadoop1 opt]# mysql -uroot -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 7
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| metastore          |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)
```
# 八、连接到hive
```bash
[root@hadoop1 bin]# hive
hive> show databases;
OK
default
Time taken: 0.591 seconds, Fetched: 1 row(s)

hive> set hive.exec.mode.local.auto=true;
hive> set mapreduce.map.memory.mb=1025;
hive> set mapreduce.reduce.memory.mb=1025; 
hive> set mapreduce.job.reduces=30;
hive> create table test(id int, name string);
hive> show tables;
OK
test
Time taken: 0.027 seconds, Fetched: 1 row(s)

hive> insert into test values(1, 'kino');
hive> select * from test;
OK
1	kino
Time taken: 0.099 seconds, Fetched: 1 row(s)
```

# 九、HiveHDBC访问
## 9.1 启动 hiveserver2
```bash
[root@hadoop1 lib]# hiveserver2

```