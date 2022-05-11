




---


# 备份
```sql
-- 查看 broker 信息
show broker;

-- broker_addr:port: 192.168.1.146:8000
-- ALTER SYSTEM ADD BROKER broker1 "<broker_addr:port>";

-- 创建 REPOSITORY(repository_name: tosbackup_20220510)
CREATE REPOSITORY `<repository_name>`
WITH BROKER `broker1`
ON LOCATION "tos://starrocks/<backup_path>"
PROPERTIES
(
"fs.tos.access.key" = "<对象存储秘钥key>",
"fs.tos.secret.key" = "<对象存储秘钥value>",
"fs.tos.endpoint" = "https://ip:port"
);

-- 备份表 （Snapshot示例: snap_shot_<db_name>_<table_name>）
BACKUP SNAPSHOT `<dbname>`.`<Snapshot>`
TO <repository_name>
ON (<table_name>)
PROPERTIES ("type" = "full");

-- 查看一个数据库的备份详情
show backup;
```


# 还原
```sql
-- 查看 REPOSITORIES, 还原库没有备份库中的REPOSITORIES就要再创建
show REPOSITORIES;

-- 在还原数据库，创建 REPOSITORY(和 备份创建的一致)
CREATE REPOSITORY `<repository_name>`
WITH BROKER `broker1`
ON LOCATION "tos://starrocks/xxx"
PROPERTIES
(
"fs.tos.access.key" = "<对象存储秘钥key>",
"fs.tos.secret.key" = "<对象存储秘钥value>",
"fs.tos.endpoint" = "https://ip:port"
);

-- 在备份数据，查看备份信息
show SNAPSHOT on <repository_name>; -- where SNAPSHOT = 'xxx';


-- 在还原数据库，还原
RESTORE SNAPSHOT `<dbname>`.`<Snapshot>`
FROM `<repository_name>`
ON ( `<table_name>` as `<target_table_name>`) 
PROPERTIES
(
    "backup_timestamp"="<Timestamp>"
);

-- 查询还原详情
show restore;
```

# 官方文档

https://docs.starrocks.com/zh-cn/main/administration/Backup_and_restore


# 博客
https://zhuanlan.zhihu.com/p/394875165





























