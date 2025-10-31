



# pg_dump 备份

pg_dump 会生成一个 sql 文件, 通常情况下可以对所有的数据库进行备份, 但是需要注意的是, 必须具备所有要备份的数据库的读权限, 如果要备份这个库, 可以使用超级管理员运行

## 备份

```bash
# 基本命令, 具体可以使用 man 查看帮助
$ pg_dump -X dbname > dumpfile 

# 示例
$ pg_dump -d "$DATABASE" -h "$HOST" -p "$PORT" -U "$USER" -f "/backup/$(basename "$backup_file")"
```

## 恢复
```bash
$ psql -X --set ON_ERROR_STOP=on dbname < dumpfile
```
默认情况下，当遇到 SQL 错误时，psql 脚本会继续执行。您可能希望使用 ON_ERROR_STOP 变量设置来更改该行为，并在发生 SQL 错误时使 psql 以退出状态 3 退出




























