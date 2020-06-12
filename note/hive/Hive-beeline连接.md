
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
