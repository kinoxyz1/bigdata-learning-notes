

* [启动hiveserver2服务](#%E5%90%AF%E5%8A%A8hiveserver2%E6%9C%8D%E5%8A%A1)
* [启动beeline](#%E5%90%AF%E5%8A%A8beeline)
* [连接hiveserver2](#%E8%BF%9E%E6%8E%A5hiveserver2)

---

# 启动hiveserver2服务

```bash
[kino@bigdata01.sutpc hive]$ bin/hiveserver2
```

-------------------
# 启动beeline

```bash
[kino@bigdata01.sutpc hive]$ bin/beeline
Beeline version 1.2.1 by Apache Hive
beeline>
```
---
# 连接hiveserver2

```bash
beeline> !connect jdbc:hive2://bigdata01.sutpc:10006（回车）
Connecting to jdbc:hive2://bigdata01.sutpc:10006
Enter username for jdbc:hive2://bigdata01.sutpc:10006: hdfs（回车）
Enter password for jdbc:hive2://bigdata01.sutpc:10006: （直接回车）
Connected to: Apache Hive (version 1.2.1)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://bigdata01.sutpc:10006> show databases;
```
