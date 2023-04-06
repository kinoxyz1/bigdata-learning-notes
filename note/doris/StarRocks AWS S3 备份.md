





# StarRocks AWS S3 备份

[官方文档](https://docs.starrocks.io/zh-cn/2.2/faq/Exporting_faq#%E9%98%BF%E9%87%8C%E4%BA%91oss%E5%A4%87%E4%BB%BD%E4%B8%8E%E8%BF%98%E5%8E%9F)

StarRocks支持备份数据到阿里云OSS/AWS S3（或者兼容S3协议的对象存储）等。假设有两个StarRocks集群，分别DB1集群和DB2集群，我们需要将DB1中的数据备份到阿里云OSS，然后在需要的时候恢复到DB2，备份及恢复大致流程如下：

## 1.1 创建仓库

```sql
-- 语法
CREATE REPOSITORY `repository_name` 
WITH BROKER `broker_name`
ON LOCATION "oss://存储桶名称/路径"
PROPERTIES
(
"fs.oss.accessKeyId" = "xxx",
"fs.oss.accessKeySecret" = "yyy",
"fs.oss.endpoint" = "oss-cn-beijing.aliyuncs.com"
);

-- 示例
CREATE REPOSITORY `tosbackup_20221208` 
WITH BROKER `broker1`
ON LOCATION "tos://starrocks/backup_20221208"
PROPERTIES
(
"fs.tos.access.key" = "xxxx",
"fs.tos.secret.key" = "xxxx",
"fs.tos.endpoint" = "https://s3-cn-beijing.com"
)

```

说明:

- DB1和DB2都需要创建，且创建的REPOSITORY**仓库名称要相同**，仓库查看：

  ```bash
  SHOW REPOSITORIES;
  ```

- `broker_name` 需要填写一个集群中的 broker 名称，BrokerName查看：

  ```bash
  SHOW BROKER;
  ```

- fs.oss.endpoint后的路径不需要带存储桶名。

## 1.2 备份数据表

在DB1中将需要进行备份的表，BACKUP到云端仓库。在DB1中执行SQL：

```sql
-- 语法
BACKUP SNAPSHOT [db_name].{snapshot_name}
TO `repository_name`
ON (
`table_name` [PARTITION (`p1`, ...)],
...
)
PROPERTIES ("key"="value", ...);

-- 示例
BACKUP SNAPSHOT starrocksdb.snapshot_starrocksdb_ads_rpt_hq_store_monthly
TO tosbackup_20221208
ON (ads_rpt_hq_store_monthly)
PROPERTIES ("type" = "full")
```

说明:

> ```plaintext
> PROPERTIES 目前支持以下属性：
> "type" = "full"：表示这是一次全量更新（默认）。
> "timeout" = "3600"：任务超时时间，默认为一天，单位秒。
> ```

StarRocks目前不支持全数据库的备份，我们需要在ON (……)指定需要备份的表或分区，这些表或分区将并行的进行备份。 查看正在进行中的备份任务（注意同时进行的备份任务只能有一个）：

```sql
-- 语法
SHOW BACKUP FROM db_name;

-- 示例
SHOW BACKUP FROM starrocksdb;
```

备份完成后，可以查看OSS中备份数据是否已经存在（不需要的备份需在OSS中删除）：

```sql
-- 语法
$ SHOW SNAPSHOT ON OSS仓库名; 

-- 示例
$ SHOW SNAPSHOT ON tosbackup_20221208; 

-- 可以加 where 条件, 如: where SNAPSHOT = {snapshot_name}
$ SHOW SNAPSHOT ON tosbackup_20221208 where SNAPSHOT = 'snapshot_starrocksdb_ads_rpt_hq_store_monthly';
Snapshot																				Timestamp		Status
snapshot_starrocksdb_ads_rpt_hq_store_monthly								ERROR: no snapshot

-- 当 Status 为 OK 时, 表示备份完成, 可以在你的对象存储上找到该备份文件了。
```

## 1.3 数据还原

在DB2中进行数据还原，DB2中不需要创建需恢复的表结构，在进行Restore操作过程中会自动创建。执行还原SQL：

```sql
-- 语法
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM `repository_name`
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);

-- 示例
RESTORE SNAPSHOT starrocksdb.snapshot_starrocksdb_ads_rpt_hq_store_monthly
FROM `tosbackup_20221208`
ON ( `snapshot_starrocksdb_ads_rpt_hq_store_monthly`) 
PROPERTIES
(
    "backup_timestamp"="2022-04-14-15-42-13-087"
);
```

说明:

- `db_name` 和 `snapshot_name` 是在 `步骤1.2` 中创建的 `SNAPSHOT` 名.
- `repository_name` 是在 `步骤1.2` 中创建的 `repository_name` 名.
- `table_name` 是要还原的表名.
- `"key"="value"` 填 备份时间, 例如执行 `SHOW SNAPSHOT ON tosbackup_20221208 where SNAPSHOT = 'snapshot_starrocksdb_ads_rpt_hq_store_monthly';` 可以看见表备份的时间, 需要原封不动 copy 过来。

查看还原进度：

```sql
SHOW RESTORE;
```





