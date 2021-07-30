







---
# 一、Flink CDC
[基于 Flink SQL CDC的实时数据同步方案 直播回顾](https://www.bilibili.com/video/BV1zt4y1D7kt/)


# 一、使用 Flink CDC 之前遇到的问题
## 1.1 写多存储系统
在实时任务同步开始的时候可能规划的是从 MySQL 同步到 Hive, 但是在后面往往可能需要写入到其他的存储系统中, 例如希望做全文检索就需要存一份 ES, 需要做缓存, 还需要写 Redis 等等。

在 Flink 中, 会极大的减少增加需求带来的工作量, 仅需要写一个目标库的建表语句、FlinkSQL 的建表语句, 即可简单的实现这个功能。



# 二、案例
## 2.1 部署 Flink(单机)

### 2.1.1 下载
[Flink1.13下载地址](https://mirrors.bfsu.edu.cn/apache/flink/flink-1.13.1/flink-1.13.1-bin-scala_2.11.tgz)


### 2.1.2 安装
```bash
$ tar -zxvf flink-1.13.1-bin-scala_2.11.tgz
```

### 2.1.3 修改配置
这个案例中会用到 FlinkSQL, 需要有相应的配置文件, 但是下载下来的安装包中没有, 可以再下载一个低版本的Flink, 把 conf 下面的 `sql-client-defaults.yaml` 文件拷贝出来

修改 `sql-client-defaults.yaml` 文件
```yaml
configuration:  # 打开注释
  execution.checkpointing.interval: 3000   # 增加(最好是加上)

execution:
  current-catalog: myhive   # 填 catalog 的名字
  current-database: flink   # 选择一个数据库(类似于进Docker容器的工作目录)

catalogs:  # 打开注释
  - name: myhive  # 和上面的 catalog 名字对应上
    type: hive
    hive-conf-dir: /etc/hive/conf/    # 选择 hive-site.xml 文件所在的目录
    default-database: flink   # 默认的数据库, 和上面类似
```

### 2.1.4 增加驱动
FlinkSQL 创建 Kafka、Hive 相关的数据库连接, 需要不同的驱动, 将如下的驱动加入到 `$FLINK_HOME/lib/` 下
```bash
[root@cdh-host-01 flink-1.13.1]# ll lib/
total 283644
-rw-r--r-- 1 root root    130802 Jul 30 11:53 aircompressor-0.8.jar
-rw-r--r-- 1 root root    167761 Jul 30 11:42 antlr-runtime-3.5.2.jar
-rw-r--r-- 1 root root     92311 Jul 30 11:40 flink-csv-1.13.1.jar
-rw-r--r-- 1 root root 115530972 Jul 30 11:40 flink-dist_2.11-1.13.1.jar
-rw-r--r-- 1 root root    148131 Jul 30 11:40 flink-json-1.13.1.jar
-rw-r--r-- 1 root root   7709740 Jul 30 11:40 flink-shaded-zookeeper-3.4.14.jar
-rw-r--r-- 1 root root  42200352 Jul 30 13:02 flink-sql-connector-hive-2.2.0_2.11-1.13.1.jar
-rw-r--r-- 1 root root   3666045 Jul 30 13:10 flink-sql-connector-kafka_2.11-1.13.1.jar
-rw-r--r-- 1 root root  36417228 Jul 30 11:40 flink-table_2.11-1.13.1.jar
-rw-r--r-- 1 root root  40965908 Jul 30 11:40 flink-table-blink_2.11-1.13.1.jar
-rw-r--r-- 1 root root  40623959 Jul 30 13:02 hive-exec-3.1.2.jar
-rw-r--r-- 1 root root     67114 Jul 30 11:40 log4j-1.2-api-2.12.1.jar
-rw-r--r-- 1 root root    276771 Jul 30 11:40 log4j-api-2.12.1.jar
-rw-r--r-- 1 root root   1674433 Jul 30 11:40 log4j-core-2.12.1.jar
-rw-r--r-- 1 root root     23518 Jul 30 11:40 log4j-slf4j-impl-2.12.1.jar
-rw-r--r-- 1 root root    733071 Jul 30 11:53 orc-core-1.4.3.jar
```

## 2.2 部署 Oracle & Debezium
这里有两篇文章, 按照说明正常部署即可:

[debezium集成Oracle攻略(上)](https://blog.csdn.net/weixin_43926300/article/details/108508672#comments_17627343)

[debezium集成Oracle攻略(下)](https://blog.csdn.net/weixin_43926300/article/details/108512985)

使用 Debezium 同步 Oracle 数据的配置文件:
```json
{
  "connector.class":"io.debezium.connector.oracle.OracleConnector",
  "database.user":"c##dbzuser",
  "database.dbname":"ORCLCDB",
  "tasks.max":"1",
  "database.pdb.name":"ORCLPDB1",
  "database.connection.adapter":"xstream",
  "database.history.kafka.bootstrap.servers":"192.168.1.112:9092,192.168.1.149:9092,192.168.1.184:9092",
  "database.history.kafka.topic":"schema-changes.inventory",
  "transforms":"unwrap",
  "database.server.name":"server1",
  "database.port":"1521",
  "decimal.handling.mode":"string",
  "database.hostname":"192.168.1.182",
  "database.password":"dbz",
  "transforms.unwrap.drop.tombstones":"false",
  "name":"oracle-inventory-connector",
  "database.out.server.name":"dbzxout",
  "transforms.unwrap.type":"io.debezium.transforms.ExtractNewRecordState"
}
```
如果加了: `transforms.unwrap.type`, FlinkSQL 则使用 json 连接器, 如果没有加使用 debezium-json 连接器

## 2.3 同步
### 2.3.1 Source
在 sql-client 中执行
```sql
DROP TABLE IF EXISTS PRODUCTS;
show tables;
CREATE TABLE PRODUCTS (
                          ID int,
                          NAME string,
                          DESCRIPTION string,
                          WEIGHT string
) WITH (
      'connector' = 'kafka',
      'topic' = 'server1.DEBEZIUM.PRODUCTS',
      'properties.bootstrap.servers' = '192.168.1.112:9092',
      'properties.group.id' = 'aa',
      'scan.startup.mode' = 'earliest-offset',
      'value.format' = 'json'
      );
```

在 Idea 中写 Java 代码:

先引入pom:
```xml
<properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <flink.version>1.13.0</flink.version>
        <scala.version>2.12</scala.version>
        <hadoop.version>3.1.3</hadoop.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-jdbc_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-filesystem_2.11</artifactId>
            <version>1.11.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-csv</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-json</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-shaded-hadoop-2-uber</artifactId>
            <version>2.8.3-10.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-sql-avro</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-sql-connector-hive-2.2.0_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-hive_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-sql-connector-kafka_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-sql-orc_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-sql-parquet_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>org.apache.flink</groupId>-->
<!--            <artifactId>flink-table_2.11</artifactId>-->
<!--            <version>${flink.version}</version>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.apache.flink</groupId>-->
<!--            <artifactId>flink-table-blink_2.11</artifactId>-->
<!--            <version>${flink.version}</version>-->
<!--        </dependency>-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner-blink_${scala.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
        </dependency>
    </dependencies>
```
代码:
```java

EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().build();
TableEnvironment tableEnv = TableEnvironment.create(settings);
tableEnv.getConfig().getConfiguration().setString("table.exec.hive.fallback-mapred-reader", "true");

System.out.println("Oracle2Hive.main start...");

// //定义hive的Catalog
String catalogName = "myhive";
String defaultDatabase = "flink";
String hiveConfDir = "D:/work/jz-dmp/dc_streaming/src/main/resources/hiveconf/";
HiveCatalog catalog = new HiveCatalog(catalogName, defaultDatabase, hiveConfDir);
//注册
tableEnv.registerCatalog(catalogName, catalog);
tableEnv.useCatalog(catalogName);
tableEnv.useDatabase(defaultDatabase);

/**********************************************Source**************************************************/
tableEnv.executeSql("DROP TABLE IF EXISTS PRODUCTS");  // 删除表
StringBuffer sourceBuffer = new StringBuffer();
sourceBuffer.append(" CREATE TABLE PRODUCTS ( ");
sourceBuffer.append("  ID int, ");
sourceBuffer.append("  NAME string, ");
sourceBuffer.append("  DESCRIPTION string, ");
sourceBuffer.append("  WEIGHT string ");
sourceBuffer.append(") WITH (");
sourceBuffer.append("  'connector' = 'kafka',");
sourceBuffer.append("  'topic' = 'server1.DEBEZIUM.PRODUCTS',");
sourceBuffer.append("  'properties.bootstrap.servers' = '192.168.1.112:9092',");
sourceBuffer.append("  'properties.group.id' = 'aa',");
sourceBuffer.append("  'scan.startup.mode' = 'earliest-offset',");
sourceBuffer.append("  'value.format' = 'json'");   // 这个看情况
sourceBuffer.append(")");
tableEnv.executeSql(sourceBuffer.toString());
```


### 2.3.2 Sink
在 sql-client 中执行
```sql
SET table.sql-dialect=hive;
DROP TABLE IF EXISTS PRODUCTS_SINK;
show tables;
create table PRODUCTS_SINK(
	ID int,
  NAME string,
  DESCRIPTION string,
  WEIGHT string
) STORED AS parquet
TBLPROPERTIES (
	'sink.partition-commit.delay'='1 min',
	'sink.partition-commit.policy.kind'='metastore,success-file',
	'sink.partition-commit.watermark-time-zone'='Asia/Shanghai'
);

insert into PRODUCTS_SINK select * from PRODUCTS;
```

在 Idea 中写 Java 代码
```java
tableEnv.getConfig().setSqlDialect(SqlDialect.HIVE);
StringBuffer sinkBuffer = new StringBuffer();
sinkBuffer.append("create table PRODUCTS_SINK(\n");
sinkBuffer.append("\tID int,\n");
sinkBuffer.append("  NAME string,\n");
sinkBuffer.append("  DESCRIPTION string,\n");
sinkBuffer.append("  WEIGHT string\n");
sinkBuffer.append(") STORED AS parquet\n");
sinkBuffer.append("TBLPROPERTIES (\n");
sinkBuffer.append("\t'sink.partition-commit.delay'='1 min',\n");
sinkBuffer.append("\t'sink.partition-commit.policy.kind'='metastore,success-file',\n");
sinkBuffer.append("\t'sink.partition-commit.watermark-time-zone'='Asia/Shanghai'\n");
sinkBuffer.append(")");
tableEnv.executeSql(sinkBuffer.toString());

tableEnv.executeSql("insert into PRODUCTS_SINK select * from PRODUCTS");
```

## 2.5 查询 Hive 表
```sql
0: jdbc:hive2://192.168.1.184:10000> select * from products_sink;
INFO  : Compiling command(queryId=hive_20210730164528_99345640-4589-4695-a472-efdc31374400): select * from products_sink
                                                                                                               INFO  : Semantic Analysis Completed
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:products_sink.id, type:int, comment:null), FieldSchema(name:products_sink.name, type:string, comment:null), FieldSchema(name:products_sink.description, type:string, comment:null), FieldSchema(name:products_sink.weight, type:string, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hive_20210730164528_99345640-4589-4695-a472-efdc31374400); Time taken: 0.175 seconds
INFO  : Executing command(queryId=hive_20210730164528_99345640-4589-4695-a472-efdc31374400): select * from products_sink
                                                                                                                                                                                                                                                                                                                                           INFO  : Completed executing command(queryId=hive_20210730164528_99345640-4589-4695-a472-efdc31374400); Time taken: 0.001 seconds
INFO  : OK
+-------------------+---------------------+----------------------------------------------------+-----------------------+
| products_sink.id  | products_sink.name  |             products_sink.description              | products_sink.weight  |
+-------------------+---------------------+----------------------------------------------------+-----------------------+
| 101               | scooter             | Small 2-wheel scooter                              | 3.14                  |
| 102               | car battery         | 12V car battery                                    | 8.1                   |
| 103               | 12-pack drill bits  | 12-pack of drill bits with sizes ranging from #40 to #3 | 0.8                   |
| 104               | hammer              | 12oz carpenter's hammer                            | 0.75                  |
| 105               | hammer              | 14oz carpenter's hammer                            | 0.875                 |
| 106               | hammer              | 16oz carpenter's hammer                            | 1                     |
| 107               | rocks               | box of assorted rocks                              | 5.3                   |
| 108               | jacket              | water resistent black wind breaker                 | 0.1                   |
| 109               | spare tire          | 24 inch spare tire                                 | 22.2                  |
| 110               | aaa                 | ffff                                               | 1.1111                |
| 111               | ff                  | fff                                                | 1111                  |
| 999               | uio                 | yuk                                                | 789                   |
+-------------------+---------------------+----------------------------------------------------+-----------------------+
```


























