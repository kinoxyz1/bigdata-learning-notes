





---



# 一、flink cdc environment
* [FLink CDC](flinkcdc.md)

# 二、Iceberg install
Iceberg 提供了可用安装包, 也可以自行编译

[官方下载](https://repo.maven.apache.org/maven2/org/apache/iceberg/iceberg-flink-runtime-1.13/0.13.1/iceberg-flink-runtime-1.13-0.13.1.jar)

```bash
$ cp iceberg-flink-runtime-1.13-0.13.1.jar ${FLINK_HOME}/lib
```

# 三、hadoop install
* [Hadoop3.0 完全分部署安装部署](../hadoop/Hadoop3.0完全分部署安装部署.md)

# 四、start flink cluster
flink 集群可以使  standalone、YARN、K8S

以 standalone 为例:
```bash
$ cd ${FLINK_HOME}
$ ./bin/start-cluster.sh
```

# 五、run demo
```bash
# 启动 flink sql-client
$ ./bin/sql-client.sh

# 设置 checkpoint
$ SET 'execution.checkpointing.interval' = '3s';   

# 创建 cdc 表
$ CREATE TABLE source (
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

# 创建 iceberg 表
# hadoop type
$ CREATE TABLE sample (
      id int,
      data    STRING,
      PRIMARY KEY (id) not ENFORCED
    ) WITH (
      'connector'='iceberg',
      'catalog-name'='iceberg_catalog',
      'catalog-type'='hadoop',  
      'warehouse'='file:///user/hadoop/warehouse',   ## 存本地
      'format-version'='2'
    );


# hive type
$ CREATE TABLE sample1 (
    id   BIGINT,
    data STRING
) WITH (
    'connector'='iceberg',
    'catalog-name'='hive_prod',
    'catalog-database'='hive_db',
    'catalog-table'='hive_iceberg_table',
    'uri'='thrift://jz-desktop-10:9083',
    'warehouse'='hdfs:///user/hadoop/warehouse'
);
  
# 插入数据
$ insert into sample1 select * from source;

# 查询(实时)
$ select * from sample1 /*+ OPTIONS('streaming'='true', 'monitor-interval'='1s')*/ ;

# 在mysql中插入数据
$ insert into source values(100, '100');

# 在 mysql 中修改数据
```




















