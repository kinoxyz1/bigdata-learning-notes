






---
# 一、MySQL 环境准备
* [Docker 部署 MySQL](../../note/docker/Docker常用安装.md)

开启binlog

```bash
vi /kino/mysql/conf/my.conf
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
server-id=1
expire_logs_days = 30
```

# 二、create mysql table 
```sql
drop table if exists user_action;
CREATE TABLE if not exists `user_action` (
  `uuid` int(11) NOT NULL,
  `user_code` varchar(200) COLLATE utf8_unicode_ci DEFAULT NULL,
  `page` varchar(200) COLLATE utf8_unicode_ci DEFAULT NULL,
  `action_time` datetime COLLATE utf8_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`uuid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

# 三、create flink table
```sql
set table.exec.resource.default-parallelism = 3;
set execution.checkpointing.interval=3s;
SET execution.type = streaming;
-- set execution.savepoint.path = 'hdfs://nameservice1/flink/checkpoints/';

-- create cdc table 
CREATE TABLE if not exists user_action (
  uuid       int,
  user_code  STRING,
  page       STRING,
  action_time TIMESTAMP(9),
  PRIMARY KEY (uuid) not ENFORCED
) WITH (
  'connector' = 'mysql-cdc',
  'hostname' = 'jz-desktop-06',
  'port' = '3306',
  'username' = 'root',
  'password' = 'Ioubuy123',
  'database-name' = 'kinodb',
  'table-name' = 'user_action'
);

-- create iceberg table(hadoop)
CREATE TABLE if not exists ods_user_action (
    uuid       int,
    user_code  STRING,
    page       STRING,
    action_time TIMESTAMP(9),
    PRIMARY KEY (uuid) not ENFORCED
) WITH (
    'connector'='iceberg',
    'catalog-name'='hadoop_prod',
    'catalog-type'='hadoop',
    'warehouse'='hdfs://nameservice1/kino/warehouse/iceberg/hadoop/',
    'format-version'='2'
);

CREATE TABLE if not exists dwd_user_action (
    uuid       int,
    user_code  STRING,
    page       STRING,
    action_time DATE,
    PRIMARY KEY (uuid) not ENFORCED
) WITH (
    'connector'='iceberg',
    'catalog-name'='hadoop_prod',
    'catalog-type'='hadoop',
    'warehouse'='hdfs://nameservice1/kino/warehouse/iceberg/hadoop/dwd/',
    'format-version'='2'
);
```

# 四、insert 
```sql
SET execution.type = streaming;
SET table.dynamic-table-options.enabled=true;

-- cdc 实时写 iceberg
insert into ods_user_action select * from user_action;

-- iceberg 实时过滤写 iceberg
insert into dwd_user_action 
select uuid,user_code,page,cast(DATE_FORMAT(action_time, 'yyyy-MM-dd') as Date) as action_time
    from ods_user_action /*+ OPTIONS('streaming'='true', 'monitor-interval'='1s')*/
    where user_code <> '' and action_time IS NOT NULL;
```

# 五、select
```sql
-- 流式查询
SET execution.type = streaming;
SET table.dynamic-table-options.enabled=true;
select * from ods_user_action /*+ OPTIONS('streaming'='true', 'monitor-interval'='1s')*/;
```

# 六、update mysql 
```sql
mysql> update user_action set page = './home/11' where uuid = 1;
```

在 flink webui 上，ods 实时过滤写 dwd 任务异常重启，异常日志如下：

```bash
2022-05-13 18:21:57,497 WARN  org.apache.flink.runtime.taskmanager.Task                    [] - Source: Iceberg table (hadoop_prod.default_database.ods_user_action) monitor (1/1)#15 (995e5a22e25aa14d0bdeeb2e9f8f8758) switched from RUNNING to FAILED with failure cause: java.lang.UnsupportedOperationException: Found overwrite operation, cannot support incremental data in snapshots (4843342586195727148, 5957089803228782689]
	at org.apache.iceberg.IncrementalDataTableScan.snapshotsWithin(IncrementalDataTableScan.java:121)
	at org.apache.iceberg.IncrementalDataTableScan.planFiles(IncrementalDataTableScan.java:73)
	at org.apache.iceberg.BaseTableScan.planTasks(BaseTableScan.java:204)
	at org.apache.iceberg.DataTableScan.planTasks(DataTableScan.java:30)
	at org.apache.iceberg.flink.source.FlinkSplitGenerator.tasks(FlinkSplitGenerator.java:86)
	at org.apache.iceberg.flink.source.FlinkSplitGenerator.createInputSplits(FlinkSplitGenerator.java:38)
	at org.apache.iceberg.flink.source.StreamingMonitorFunction.monitorAndForwardSplits(StreamingMonitorFunction.java:143)
	at org.apache.iceberg.flink.source.StreamingMonitorFunction.run(StreamingMonitorFunction.java:121)
	at org.apache.flink.streaming.api.operators.StreamSource.run(StreamSource.java:104)
	at org.apache.flink.streaming.api.operators.StreamSource.run(StreamSource.java:60)
	at org.apache.flink.streaming.runtime.tasks.SourceStreamTask$LegacySourceFunctionThread.run(SourceStreamTask.java:269)
```

Iceberg 社区对该 bug 的相关信息：

- [pull request1](https://github.com/apache/iceberg/projects/15)
- [pull request2](https://github.com/apache/iceberg/pull/4528)


# 七、总结
- iceberg v2 表才支持 delete & update, 默认 v1 不支持。
- 实时入湖如果涉及到变更数据，后续 Flink 的实时 ETL 会报错，目前社区已经在做相关 bug-fix。






