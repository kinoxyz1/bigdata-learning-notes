









---

# 一、download flink
https://dlcdn.apache.org/flink/flink-1.13.6/flink-1.13.6-bin-scala_2.11.tgz

# 二、install flink(standalone)
[参考 flink 官方部署文档](https://nightlies.apache.org/flink/flink-docs-release-1.14/docs/deployment/resource-providers/standalone/overview/#standalone)

# 三、flink cdc dependent
需要新增的jar

- [flink-sql-connector-mysql-cdc-1.4.0.jar](https://github.com/ververica/flink-cdc-connectors/releases)
- [mysql-connector-java-8.0.23.jar](https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.23/mysql-connector-java-8.0.23.jar)

整个 lib 结构如下
```shell
$ ll ${FLINK_HOME}/lib
总用量 273416
drwxr-xr-x  2 dell nginx      4096 4月  19 17:46 ./
drwxr-xr-x 11 dell nginx      4096 4月  16 02:27 ../
-rw-r--r--  1 root root      92314 2月   4 16:51 flink-csv-1.13.6.jar
-rw-r--r--  1 root root  106542761 2月   4 16:56 flink-dist_2.12-1.13.6.jar
-rw-r--r--  1 root root     148127 2月   4 16:51 flink-json-1.13.6.jar
-rw-r--r--  1 root root   59427729 4月  19 17:46 flink-shaded-hadoop-2-uber-3.0.0-cdh6.3.2-10.0.jar
-rw-r--r--  1 root root    7709740 5月   7  2021 flink-shaded-zookeeper-3.4.14.jar
-r--------  1 root root   27606575 4月  19 17:46 flink-sql-connector-mysql-cdc-1.4.0.jar
-rw-r--r--  1 root root   35053606 2月   4 16:55 flink-table_2.12-1.13.6.jar
-rw-r--r--  1 root root   38622330 2月   4 16:55 flink-table-blink_2.12-1.13.6.jar
-rw-r--r--  1 root root     208006 1月  13 19:06 log4j-1.2-api-2.17.1.jar
-rw-r--r--  1 root root     301872 1月   7 18:07 log4j-api-2.17.1.jar
-rw-r--r--  1 root root    1790452 1月   7 18:07 log4j-core-2.17.1.jar
-rw-r--r--  1 root root      24279 1月   7 18:07 log4j-slf4j-impl-2.17.1.jar
-rw-r--r--  1 root root    2415211 4月  19 17:46 mysql-connector-java-8.0.23.jar
```

# 四、run flink cdc(mysql)
启动 flink 集群
```bash
$ cd ${FLINK_HOME}
$ ./bin/start-cluster.sh
```
在 浏览器中查看 flink webui: http://<you-flink-ip>:8081

进入 sql-client
```bash
$ ./bin/sql-client.sh
```

创建 flink cdc table
```sql
CREATE TABLE source (
  id INT,
  data STRING,
  PRIMARY KEY(id) NOT ENFORCED
) WITH (
'connector' = 'mysql-cdc',
'hostname' = 'jz-desktop-06',
'port' = '3306',
'username' = 'root',
'password' = 'Ioubuy123',
'database-name' = 'kinodb',
'table-name' = 'source'
);
```

查询 
```sql
Flink SQL> select * from source;

Refresh: 1 s                                                                                    Page: Last of 1                                                                            Updated: 18:06:03.607 

                             id                           data
                              1                              1
                              2                              2
                            100                            100
                            101                            101
                            103                            103
                            104                            104
                            102                            102


Q Quit                                   + Inc Refresh                            G Goto Page                              N Next Page                              O Open Row                               
R Refresh                                - Dec Refresh                            L Last Page                              P Prev Page                      
```




























