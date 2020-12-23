


---
debezium配置文档: https://debezium.io/documentation/reference/connectors/sqlserver.html

confluent配置文档: https://docs.confluent.io/kafka-connect-jdbc/current/source-connector/source_config_options.html

# 一、Debezium SqlServer Connector
## 1.1 概述
Debezium SqlServer Connector 可以捕获 SqlServer 数据库的行级别更改事件, 其中包括: `INSERT`、`UPDATE`、`DELETE`

要捕获其事件, 需要开启需要被捕获的表的 SqlServer CDC, 当 Debezium SqlServer Connector 首次连接到 SqlServer 的表时, 将会对数据库表做一致性快照初始化, 初始化完成后, Connector 即可连续捕获其变化的 `INSERT`、`UPDATE`、`DELETE` 事件, 被捕获的事件会将其以流的形式发送到 Kafka 的一个 topic 中(每个表都会有一个对应的 topic)

## 1.2 容错
当 Debezium SqlServer Connector 读取更改并产生的事件时, 它会定期(定期提交的, 发生更改事件时不会提交, 在中断后重启有可能造成重复事件)在数据库日志中记录事件的位置(LSN/日志序列号), 如果 Connector 因为任何原因崩溃而停止, 重新启动后, Connector 会从其上一次读取的最后位置恢复读取 SqlServer CDC 表

容错也适用于一致性快照的初始化



# 二、部署服务
部署 debezium 相关的组件: 
```bash
version: '2'
services:
  zookeeper:
    image: debezium/zookeeper:1.4.0.Alpha2
    ports:
     - 2181:2181
     - 2888:2888
     - 3888:3888
  kafka:
    image: debezium/kafka:1.4.0.Alpha2
    ports:
     - 9093:9092
    links:
     - zookeeper
    environment:
     - ZOOKEEPER_CONNECT=zookeeper:2181
  mysql:
    image: debezium/example-mysql:1.3
    ports:
     - 3309:3306
    environment:
     - MYSQL_ROOT_PASSWORD=debezium
     - MYSQL_USER=mysqluser
     - MYSQL_PASSWORD=mysqlpw
  schema-registry:
    image: confluentinc/cp-schema-registry
    ports:
     - 8181:8181
     - 8081:8081
    environment:
     - SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=zookeeper:2181
     - SCHEMA_REGISTRY_HOST_NAME=schema-registry
     - SCHEMA_REGISTRY_LISTENERS=http://schema-registry:8081
    links:
     - zookeeper
  connect:
    image: debezium/connect:1.4.0.Alpha2
    ports:
     - 8083:8083
    links:
     - kafka
     - mysql
     - schema-registry
    environment:
     - BOOTSTRAP_SERVERS=kafka:9092
     - GROUP_ID=1
     - CONFIG_STORAGE_TOPIC=my_connect_configs
     - OFFSET_STORAGE_TOPIC=my_connect_offsets
     - STATUS_STORAGE_TOPIC=my_connect_statuses
     - INTERNAL_KEY_CONVERTER=org.apache.kafka.connect.json.JsonConverter
     - INTERNAL_VALUE_CONVERTER=org.apache.kafka.connect.json.JsonConverter
```
部署
```bash
$ docker-compose up -d
Pulling zookeeper (debezium/zookeeper:1.4.0.Alpha2)...
1.4.0.Alpha2: Pulling from debezium/zookeeper
9b4ebb48de8d: Pull complete
4aac785d914b: Pull complete
.....
```

部署pgsql
```bash
$ docker run --name postgres-teest -e POSTGRES_PASSWORD=123456 -p 54321:5432 -d --privileged=true postgres:9.6
```

部署SqlServer
```bash
$ docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=123456" -p 14330:1433 --name sqlserver2019 -v /app/data/sqlserver_data:/var/opt/mssql  -d microsoft/mssql-server-linux
$ /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "123456"
```

# 三、开启 SqlServer CDC
需要为监控的数据库开启cdc功能以便debezium连接器能够获取日志文件。

确保开启SQL server agent服务, 如果不开启会报如下错误:
```bash
SQLServerAgent is not currently running so it cannot be notified of this action.
```
开启 SQL server agent服务
```bash
> sp_configure 'show advanced options', 1;   
> GO   
> RECONFIGURE;   
> GO   
> sp_configure 'Agent XPs', 1;   
> GO   
> RECONFIGURE   
> GO 
```


进入要开启 CDC 的数据库
```bash
> use kinodb;
> go
```
为数据库开启 CDC
```bash
# 数据库开启cdc功能，注意不能为master数据库启动该功能
> EXEC sys.sp_cdc_enable_db
> go
```
为表开启 CDC
```bash
# 找出当前filegroup_name的值，一般为PRIMARY
> SELECT FILEGROUP_NAME(1) AS [Filegroup Name];
> GO

# 为监控的表开启cdc
> use kinodb;
> go 

> EXEC sys.sp_cdc_enable_table
> @source_schema = N'dbo',
> @source_name   = N'test',  # 指定要捕获的表的名称
> @role_name     = N'MyRole',  
> @filegroup_name = N'PRIMARY',
> @supports_net_changes = 0
> GO
Job 'cdc.testdb_capture' started successfully.
Job 'cdc.testdb_cleanup' started successfully.
```
查看开启了 CDC 的数据库和表
```bash
# 查看开启了 CDC 的数据库
> select * from sys.databases where is_cdc_enabled = 1
> go
# 查看开启了 CDC 的表
> select name, is_tracked_by_cdc from sys.tables where object_id = OBJECT_ID('dbo.test')
> go
```

CDC 的禁用操作
```bash
# 禁用CDC(表)
> EXEC sys.sp_cdc_disable_table @source_schema = 'dbo', @source_name = 'test', @capture_instance = 'all';
# 禁用CDC(库)
> EXEC sys.sp_cdc_disable_db; 
```

# 四、写配置文件
## 4.1 source
```bash
$ vim sqlserver-kafka-pgsql-source.json
{
    "name": "sqlserver-connector-kino",
    "config": {
        "connector.class" : "io.debezium.connector.sqlserver.SqlServerConnector",
        "tasks.max" : "1",
        "database.server.name" : "kino",
        "database.hostname" : "192.168.1.146",
        "database.port" : "14330",
        "database.user" : "SA",
        "database.password" : "Kino@123.,",
        "database.dbname" : "kinodb",
        "database.history.kafka.bootstrap.servers" : "192.168.1.146:9093",
        "database.history.kafka.topic": "kino.inventory",
        "include.schema.changes": "true",
        "transforms":"unwrap",
        "transforms.unwrap.type":"io.debezium.transforms.ExtractNewRecordState",
        "tombstones.on.delete":"true",
        "transforms.unwrap.drop.tombstones":"false"
    }
}
```
## 4.2 sink
```bash
$ sqlserver-kafka-pgsql-sink.json
{
    "name": "sqlserver-connector-kino-sink",
    "config": {
        "connector.class" : "io.confluent.connect.jdbc.JdbcSinkConnector",
        "connection.url" : "jdbc:postgresql://192.168.1.146:5432/kong",
        "connection.user" : "kong",
        "connection.password" : "Kong@2020",
        "pk.mode": "record_key",
        "pk.fields" : "id",
        "target.tablename" : "test",
        "tasks.max" : "1",
        "topics" : "kino.dbo.test",
        "auto.create" : "true",
        "auto.evolve": "true",
        "auto.offset.reset": "earliest",
        "table.name.format" : "test",
        "insert.mode":"upsert",
        "delete.enabled":"true",
        "batch.size" : "1"
    }
}
```
## 4.3 启动
```bash
$ curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @sqlserver-kafka-pgsql-source.json
$ curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @sqlserver-kafka-pgsql-sink.json
```
## 4.4 查看
```bash
$ curl -H "Accept:application/json" http://localhost:8083/connectors/
```
## 4.5 删除
```bash
$ curl -v -X DELETE http://localhost:8083/connectors/sqlserver-connector-kino
$ curl -v -X DELETE http://localhost:8083/connectors/sqlserver-connector-kino-sink
```

# 五、查看数据
## 5.1 在 SqlServer 中
新增记录
```sqlite-psql
$ insert into test values(1, 'kino1', 1);
$ insert into test values(2, 'kino1', 2);
$ insert into test values(3, 'kino1', 3);
```
## 5.2 在 pgsql 中
查看
```postgres-sql
$ select * from test
```

## 5.3 查看 Kafka
```bash
# 查看topic
$ docker exec -it your CONTAINER ID exec

$ bin/kafka-topics.sh --bootstrap-server kafka:9092 --list
kino
kino.dbo.test
kino.inventory

# 消费 
$ bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --from-beginning --topic kino.dbo.test
```

## 5.4 查看详细日志
需要进入 debezium/connect:1.4.0.Alpha2 镜像中
```bash
$ docker exec -it your CONTAINER ID bash

$ ll -rt 
-rw-r--r-- 1 kafka kafka 1020609 Dec 17 11:51 connect-service.log

$ vi connect-service.log
...
2020-12-17 11:51:33,968 INFO   ||  WorkerSourceTask{id=sqlserver-connector-kino-0} Committing offsets   [org.apache.kafka.connect.runtime.WorkerSourceTask]
2020-12-17 11:51:33,969 INFO   ||  WorkerSourceTask{id=sqlserver-connector-kino-0} flushing 0 outstanding messages for offset commit   [org.apache.kafka.connect.runtime.WorkerSourceTask]
...
```

# 六、表结构变更
如果源表结构发生更改, 例如添加新列, 该更改添加的字段并不会动态出现在 kafka 主题中, 此时必须刷新 变更表的CDC, 才能将更改字段写入到 kafka 主题中

刷新变更表CDC有两种方式:
- 离线模式刷新: 在离线架构中更新, 变更表在停止 Debezium Connector 后会更新
- 在线模式刷新: 在在线模式中更新, 变更表在 Debezium Connector 运行时更新

注意:
- 不论是 在线 还是 离线 更新, 都必须刷新表的CDC, 才能产生变更表后续的更新
- 启用了 CDC 的表不支持某些模式的更改, 例如: 重命名表
- 列本身为 NULL 更改成 NOT NULL(反之也是), Debezium SqlServer Connector 无法正确捕获变更信息, 知道创建一个新的捕获实例之后; @TODO

## 6.1 离线模式更新
脱机模式更新为更新捕获表提供了最安全的方法。但是，离线更新可能不适用于需要高可用性的应用程序。

暂不使用

## 6.2 在线模式更新
比离线模式更新的过程更简单, 在更新过程中无需停机, 但是可能存在潜在的处理差距, 例如在更新过程完成之前这个过程中陆续有数据写入, 则会以旧模式被捕获, 此时的事件中就不会包含新增的字段

案例:

1. 在 test 表中新增一个字段
    ```bash
    > ALTER TABLE test1 ADD phone_number VARCHAR(32);
    > go
    ```
2. 通过运行 `sys.sp_cdc_enable_table` 来创建新的捕获实例
    ```bash
    EXEC sys.sp_cdc_enable_table @source_schema = 'dbo', @source_name = 'test1', @role_name = NULL, @supports_net_changes = 0, @capture_instance = 'dbo_test1_v2';
    GO
    ```
3. 写入一条数据
    ```bash
    INSERT INTO test1 VALUES (111,111,'+1-555-123456');
    GO
    ```
4. 消费 Kafka Connect 查看更新的记录
    ```bash
    ...
    [2020-12-23 11:32:28,256] INFO Multiple capture instances present for the same table: Capture instance "dbo_test1" [sourceTableId=kinodb2.dbo.test1, changeTableId=kinodb2.cdc.dbo_test1_CT, startLsn=00000024:00000a90:0036, changeTableObjectId=1269579561, stopLsn=00000035:00003da8:003e] and Capture instance "dbo_test1_v2" [sourceTableId=kinodb2.dbo.test1, changeTableId=kinodb2.cdc.dbo_test1_v2_CT, startLsn=00000035:00003da8:003e, changeTableObjectId=1330103779, stopLsn=NULL] (io.debezium.connector.sqlserver.SqlServerStreamingChangeEventSource:347)
    [2020-12-23 11:32:28,257] INFO Schema will be changed for Capture instance "dbo_test1_v2" [sourceTableId=kinodb2.dbo.test1, changeTableId=kinodb2.cdc.dbo_test1_v2_CT, startLsn=00000035:00003da8:003e, changeTableObjectId=1330103779, stopLsn=NULL] (io.debezium.connector.sqlserver.SqlServerStreamingChangeEventSource:158)
    ...
    [2020-12-23 11:33:23,216] INFO Migrating schema to Capture instance "dbo_test1_v2" [sourceTableId=kinodb2.dbo.test1, changeTableId=kinodb2.cdc.dbo_test1_v2_CT, startLsn=00000035:00003da8:003e, changeTableObjectId=1330103779, stopLsn=NULL] (io.debezium.connector.sqlserver.SqlServerStreamingChangeEventSource:290)
    [2020-12-23 11:33:23,240] INFO Already applied 21 database changes (io.debezium.relational.history.DatabaseHistoryMetrics:137)
    [2020-12-23 11:33:23,560] INFO Unable to find fields [SinkRecordField{schema=Schema{STRING}, name='phone_number', isPrimaryKey=false}] among column names [id, age] (io.confluent.connect.jdbc.sink.DbStructure:274)
    [2020-12-23 11:33:23,591] INFO Amending TABLE to add missing fields:[SinkRecordField{schema=Schema{STRING}, name='phone_number', isPrimaryKey=false}] maxRetries:10 with SQL: [ALTER TABLE "test1" ADD "phone_number" TEXT NULL] (io.confluent.connect.jdbc.sink.DbStructure:200)
    [2020-12-23 11:33:23,600] INFO Checking PostgreSql dialect for type of TABLE "test1" (io.confluent.connect.jdbc.dialect.PostgreSqlDatabaseDialect:833)
    [2020-12-23 11:33:23,602] INFO Refreshing metadata for table "test1" to Table{name='"test1"', type=TABLE columns=[Column{'phone_number', isPrimaryKey=false, allowsNull=true, sqlType=text}, Column{'id', isPrimaryKey=true, allowsNull=false, sqlType=int4}, Column{'age', isPrimaryKey=false, allowsNull=true, sqlType=int4}]} (io.confluent.connect.jdbc.util.TableDefinitions:86)
    ...
    ```

5. 通过运行 `sys.sp_cdc_disable_table` 存储过程来删除旧的捕获实例。
    ```bash
    EXEC sys.sp_cdc_disable_table @source_schema = 'dbo', @source_name = 'cdc.dbo_test1_CT', @capture_instance = 'cdc.dbo_test1_CT';
    GO
    ```