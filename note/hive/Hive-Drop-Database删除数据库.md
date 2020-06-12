删除一个数据库，默认情况下，hive不允许删除含有表的数据库，要先将数据库中的表清空才能drop，否则会报错

加入CASCADE关键字，可以强制删除一个数据库
```sql
hive (default)> drop database if exists table;
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. InvalidOperationException(message:Database table is not empty. One or more tables exist.)


hive (default)> drop database if exists table cascade;
```