
# 一、环境准备
## 1.1 准备 Spark 运行环境
请先[下载Spark](http://spark.apache.org/downloads.html "下载Spark"), Spark版本请选择 >= 2.x.x。下载解压后，不需要做任何配置即可提交Spark deploy-mode = local模式的任务。 如果你期望任务运行在Standalone集群或者Yarn、Mesos集群上，请参考Spark官网的 [Spark部署文档](http://spark.apache.org/docs/latest/cluster-overview.html "Spark部署文档")。

## 1.2 准备 Hive 运行环境
Hive 的安装参考其他博客即可

## 1.3 配置好 Spark On Hive 的环境
① 将 hive-site.xml、core-site.xml、hdfs-site.xml 拷贝一份到 spark 的 conf 目录下

## 1.4 准备 WaterDrop
WaterDrop 的安装参考博客

# 二、配置文件
## 2.1 准备工作
在 Hive 中创建一张表, 并写入几条数据
```sql
CREATE TABLE `default.s_vehicle_gps_taxi`(
  `vehicle_id` string, 
  `loc_time` bigint
  )
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat';

insert into default.s_vehicle_gps_taxi("粤B0C2M5",1572560572);
insert into default.s_vehicle_gps_taxi("粤B0C2R7",1572618705);
insert into default.s_vehicle_gps_taxi("粤B0C3P0",1572588217);
insert into default.s_vehicle_gps_taxi("粤B0C4P0",1572596602);
insert into default.s_vehicle_gps_taxi("粤B0C5M2",1572603587);
insert into default.s_vehicle_gps_taxi("粤B0C7M7",1572566285);
insert into default.s_vehicle_gps_taxi("粤B0C7R9",1572564798);
insert into default.s_vehicle_gps_taxi("粤B0C9M5",1572588185);
insert into default.s_vehicle_gps_taxi("粤B0C9Q2",1572548412);
insert into default.s_vehicle_gps_taxi("粤B0H1K0",1572612227);
insert into default.s_vehicle_gps_taxi("粤B0H1K6",1572564441);
insert into default.s_vehicle_gps_taxi("粤B0H8K1",1572541497);
insert into default.s_vehicle_gps_taxi("粤B0H9K5",1572550667);
insert into default.s_vehicle_gps_taxi("粤B0K0A6",1572581648);
insert into default.s_vehicle_gps_taxi("粤B0K5A0",1572584053);
insert into default.s_vehicle_gps_taxi("粤B0K61U",1572540875);
insert into default.s_vehicle_gps_taxi("粤B0K61U",1572577852);
insert into default.s_vehicle_gps_taxi("粤B0M3D6",1572599081);
insert into default.s_vehicle_gps_taxi("粤B0Q1S1",1572618836);
insert into default.s_vehicle_gps_taxi("粤B0Q2S0",1572590496);
```

```xml
vim hive2hdfs.conf
spark {
  # Waterdrop defined streaming batch duration in seconds
  spark.streaming.batchDuration = 5
  spark.app.name = "Waterdrop-Hive-To-Hdfs"
  spark.ui.port = 13000
  spark.sql.catalogImplementation = "hive"
}

input {
  hive {
    pre_sql = "select vehicle_id,loc_time from default.s_vehicle_gps_taxi limit 10"
    result_table_name = "s_vehicle_gps_taxi_kino"
  }
}

filter {
  remove {
    source_field = ["loc_time"]
  }
}

output {
  hdfs {
    path = "hdfs:///data/logs-taxi-${now}"
    serializer = "text"
    path_time_format = "yyyy.MM.dd"
  }
}
```
执行:
```bash
/opt/software/waterdrop/waterdrop/bin/start-waterdrop.sh --master local[4] --deploy-mode client --config /opt/software/waterdrop/waterdrop/config/hive2hdfs.conf
```

# 三、验证结果
查看 HDFS 上路径(hdfs:///data/logs-taxi-${now})下是否有文件
```bash
[root@kino100 /]# hadoop fs -ls /data/logs-taxi-2020.07.14/
Found 2 items
-rw-r--r--   1 root supergroup          0 2020-07-14 13:13 /data/logs-taxi-2020.07.14/_SUCCESS
-rw-r--r--   1 root supergroup        100 2020-07-14 13:13 /data/logs-taxi-2020.07.14/part-00000-70da7e80-67e3-4b67-8133-85a2eea04ab6-c000.txt

[root@kino100 /]# hadoop fs -cat /data/logs-taxi-2020.07.14/part-00000-70da7e80-67e3-4b67-8133-85a2eea04ab6-c000.txt
粤B0C1Q9
粤B0C2R7
粤B0C4P5
粤B0C4R2
粤B0C5M2
粤B0C7M6
粤B0C7N5
粤B0C7R9
粤B0C8P7
粤B0C8P7
```