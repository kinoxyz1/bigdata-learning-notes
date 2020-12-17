


---
debezium配置文档: https://debezium.io/documentation/reference/connectors/sqlserver.html

confluent配置文档: https://docs.confluent.io/kafka-connect-jdbc/current/source-connector/source_config_options.html

# 一、部署服务
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

# 二、开启 SqlServer CDC
需要为监控的数据库开启cdc功能以便debezium连接器能够获取日志文件。

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

# 三、写配置文件
## 3.1 source
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
## 3.2 sink
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
## 3.3 启动
```bash
$ curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @sqlserver-kafka-pgsql-source.json
$ curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @sqlserver-kafka-pgsql-sink.json
```
## 3.4 查看
```bash
$ curl -H "Accept:application/json" http://localhost:8083/connectors/
```
## 3.5 删除
```bash
$ curl -v -X DELETE http://localhost:8083/connectors/sqlserver-connector-kino
$ curl -v -X DELETE http://localhost:8083/connectors/sqlserver-connector-kino-sink
```

# 四、查看数据
## 4.1 在 SqlServer 中
新增记录
```sqlite-psql
$ insert into test values(1, 'kino1', 1);
$ insert into test values(2, 'kino1', 2);
$ insert into test values(3, 'kino1', 3);
```
## 4.2 在 pgsql 中
查看
```postgres-sql
$ select * from test
```

## 4.3 查看 Kafka
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

## 4.4 查看详细日志
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