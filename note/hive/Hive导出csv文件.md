

```sql
hive -e "set hive.cli.print.header=true; SELECT * FROM default.s_transit_people_ic_cache " | sed 's/[\t]/,/g'  > outputData.csv
```
参数说明:
	- `set hive.cli.print.header=true`：输出表头
	- `sed 's/[\t]/,/g`：将\t替换成,
	- `> outputData.csv`：输出到 outputData.csv 文件中