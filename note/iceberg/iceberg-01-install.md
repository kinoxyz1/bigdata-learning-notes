







---


# 环境说明

 服务 | 版本 
 -- | -- 
 Hadoop |3.0.0-cdh6.3.2  
 Flink |1.13.6 
 Iceberg |0.13.1 

# 一、standalone mode env building

## 1.1 download flink

```bash
FLINK_VERSION=1.13.6
SCALA_VERSION=2.12
APACHE_FLINK_URL=archive.apache.org/dist/flink/
wget ${APACHE_FLINK_URL}/flink-${FLINK_VERSION}/flink-${FLINK_VERSION}-bin-scala_${SCALA_VERSION}.tgz
tar xzvf flink-${FLINK_VERSION}-bin-scala_${SCALA_VERSION}.tgz
```

## 1.2 start flink cluster

```bash
# HADOOP_HOME is your hadoop root directory after unpack the binary package.
export HADOOP_CLASSPATH=`$HADOOP_HOME/bin/hadoop classpath`

# Start the flink standalone cluster
./bin/start-cluster.sh
```

## 1.3 start flink sql client

```bash
# HADOOP_HOME is your hadoop root directory after unpack the binary package.
export HADOOP_CLASSPATH=`$HADOOP_HOME/bin/hadoop classpath`

# iceberg jar cp $FLINK_HOME/lib 下启动
./bin/sql-client.sh embedded shell

# 指定 iceberg jar 启动
./bin/sql-client.sh embedded -j <flink-runtime-directory>/iceberg-flink-runtime-xxx.jar shell
```

## 1.4 note: 引入cdh hadoop

```bash
# 拷贝 cdh /opt/cloudera 包到 flink 集群
scp -r /opt/cloudera user@host:/opt/cloudera

# 拷贝 cdh hive conf 到 flink 集群
mkdir -p $FLINK_HOME/hadoop
scp -r /etc/hive/conf/*.xml user@host:$FLINK_HOME/hadoop

# 加入环境变量
vim /etc/profile
export HADOOP_HOME=/opt/cloudera/parcels/CDH/lib/hadoop
export HADOOP_CONF_DIR=/app/apache/flink/flink-1.13.6/hadoop
# export HADOOP_COMMON_HOME=/app/apache/flink/flink-1.13.6/hadoop
# export HADOOP_HDFS_HOME=/app/apache/flink/flink-1.13.6/hadoop
# export HADOOP_YARN_HOME=/app/apache/flink/flink-1.13.6/hadoop
# export HADOOP_MAPRED_HOME=/app/apache/flink/flink-1.13.6/hadoop
# export HADOOP_CLASSPATH=`hadoop classpath`
# export PATH=$PATH:$HADOOP_CLASSPATH

source /etc/profile
```

# 二、Creating catalogs

支持三种 catalogs：

- hive catalog
- hadoop catalog
- custom catalog

## 2.1 hive catalog

```sql
CREATE CATALOG hive_catalog WITH (
  'type'='iceberg',
  'catalog-type'='hive',
  'uri'='thrift://jz-desktop-10:9083',
  'clients'='5',
  'property-version'='1',
  'warehouse'='hdfs://nameservice1/user/hive/warehouse/iceberg/',
  'format-version'='2'
);
```

## 2.2 hadoop catalog

```sql
CREATE CATALOG hadoop_catalog WITH (
  'type'='iceberg',
  'catalog-type'='hadoop',
  'warehouse'='hdfs://nameservice1/user/hive/warehouse/iceberg/',
  'property-version'='1'
);
```

## 2.3 custom catalog
```sql
CREATE CATALOG my_catalog WITH (
  'type'='iceberg',
  'catalog-impl'='com.my.custom.CatalogImpl',
  'my-additional-catalog-config'='my-value'
);
```

## 2.4 用 YAML 配置创建

```yaml
catalogs: 
  - name: my_catalog
    type: iceberg
    catalog-type: hadoop
    warehouse: hdfs://nameservice1/user/hive/warehouse/iceberg/
```


# 三、sql 操作

```sql
-- start flinksql client
bin/sql-client.sh embedded shell
```

## 3.1 create catalog

```sql
-- create catalog
CREATE CATALOG hadoop_catalog WITH (
  'type'='iceberg',
  'catalog-type'='hadoop',
  'warehouse'='hdfs://nameservice1/kino/warehouse/iceberg/hadoop/',
  'property-version'='1'
);
```

## 3.2 create database

```sql
-- create database
create database iceberg_db;
use iceberg_db;
```

## 3.3 create table

flink iceberg 不支持 隐藏分区

```sql
-- 常规建表
create table testA( 
  id bigint,
  name string, 
  age int,
  dt string
)
PARTITIONED by(dt);

-- like 建表(两张表结构一模一样)
create table testB like testA;
```

## 3.4 insert

```bash
insert into testA values(1001,' 张三',18,'2021-07-01'),(1001,' 李四',19,'2021-07-02');
```

> 1. 可以访问 ip:8081 在 flink webui 查看任务执行情况；
>
> 2. 同时也可以在 hdfs 上查看数据：hdfs dfs -ls /kino/warehouse/iceberg/hadoop/iceberg_db/testA

## 3.5 insert overwrite

```sql
insert overwrite testA values(1,' 王 五 ',18,'2021-07-01'),(2,' 马 六',19,'2021-07-02');
[INFO] Submitting SQL update statement to the cluster...
[ERROR] Could not execute SQL statement. Reason:
java.lang.IllegalStateException: Unbounded data stream doesn't support overwrite operation.
```

flink 默认使用流的方式插入数据，这个时候流的插入是不支持 overwrite 操作的

需要将插入模式进行修改，改成批的插入方式,再次使用 overwrite 插入数据。如需要改回流式操作参数设置为 `SET execution.type = streaming;`

```sql
SET execution.type = batch;
insert overwrite testA values(1,' 王 五 ',18,'2021-07-01'),(2,' 马 六',19,'2021-07-02');
```

## 3.6 select

```sql
select * from testA;
```

## 3.7 alter

### 3.7.1 alter table name

```sql
ALTER TABLE testA RENAME TO testB;
[ERROR] Could not execute SQL statement. Reason:
java.lang.UnsupportedOperationException: Cannot rename Hadoop tables
```

### 3.7.2 alter column

```sql
ALTER TABLE testA ADD COLUMN (c1 string comment 'new_column docs');
[ERROR] Could not execute SQL statement. Reason:
org.apache.flink.sql.parser.impl.ParseException: Encountered "COLUMNS" at line 2, column 5.
Was expecting one of:
    "CONSTRAINT" ...
    "PRIMARY" ...
    "UNIQUE" ...
```





