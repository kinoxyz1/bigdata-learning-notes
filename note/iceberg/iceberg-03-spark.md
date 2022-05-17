




---
# 一、环境准备
## 1.1 download spark

```bash
wget https://dlcdn.apache.org/spark/spark-3.2.1/spark-3.2.1-bin-hadoop2.7.tgz
```

## 1.2 download iceberg jar

```bash
wget https://repo1.maven.org/maven2/org/apache/iceberg/iceberg-spark-runtime-3.2_2.12/0.13.1/iceberg-spark-runtime-3.2_2.12-0.13.1.jar
```

## 1.3 install

```bash
## 解压 spark
tar -zxvf spark-3.2.1-bin-hadoop2.7.tgz
## cp iceberg jar to sparkJarsPath
cp iceberg-spark-runtime-3.2_2.12-0.13.1.jar $SPARK_HOME/jars
```

## 1.4 edit spark config
```yaml
vim conf/spark-defaults.conf
spark.sql.catalog.hive_prod = org.apache.iceberg.spark.SparkCatalog
spark.sql.catalog.hive_prod.type = hive
spark.sql.catalog.hive_prod.uri = thrift://jz-desktop-10:9083

spark.sql.catalog.hadoop_prod = org.apache.iceberg.spark.SparkCatalog
spark.sql.catalog.hadoop_prod.type = hadoop
spark.sql.catalog.hadoop_prod.warehouse = hdfs://nameservice1/kino/warehouse/iceberg/hadoop/

spark.sql.catalog.catalog-name.type = hadoop
spark.sql.catalog.catalog-name.default-namespace = kinodb
spark.sql.catalog.catalog-name.uri = thrift://jz-desktop-10:9083
spark.sql.catalog.catalog-name.warehouse= hdfs://nameservice1/kino/warehouse/iceberg/hadoop
```

# 二、Spark Operation
## 2.1 insert 
```sql
-- 连接 sparksql client
./bin/spark-sql --hiveconf hive.cli.print.header=true

-- 进入 catalog(在 spark-default.yaml 文件中配置的, 通过 show catalog 看不见)
use hadoop_prod;

-- 创建 database
create database kinodb;

-- 创建表(分区表)
create table testA(
  id bigint, name string, age int,
  dt string
) USING iceberg
PARTITIONED by(dt);

-- insert 
insert into testA values(1,'张三',18,'2021-06-21');
```

## 2.2 insert overwrite
```sql
-- insert overwrite(覆盖重刷)
insert overwrite testA values(2,'李四',20,'2021-06-21');
```

## 2.3 动态覆盖
```sql
-- 动态覆盖
vim spark-defaults.conf 
spark.sql.sources.partitionOverwriteMode=dynamic

create table hadoop_prod.kinodb.testB( id bigint,
  name string, age int,
  dt string
) USING iceberg
PARTITIONED by(dt);

insert into hadoop_prod.kinodb.testA values(1,'张三',18,'2021-06-22');

insert overwrite testB select * from testA;
```

## 2.4 静态覆盖
```sql
-- 静态覆盖(手动指定分区)
insert overwrite testB Partition(dt='2021-06-26') select id,name,age from testA;
```

## 2.5 delete
```sql
-- delete(delete 不会真正删除 hdfs 上的文件)
delete from testB where dt >='2021-06-21' and dt <='2021-06-26';
```

## 2.6 历史快照
```sql
-- history
select * from hadoop_prod.kinodb.testB.history;
2022-05-16 15:59:40.991	618425031136003513	NULL	true
2022-05-16 16:00:29.255	6028470489417111241	618425031136003513	true
2022-05-16 16:01:34.305	6242810283652646314	6028470489417111241	true
Time taken: 0.084 seconds, Fetched 3 row(s)

-- snapshots(每次操作后的快照记录)
select * from hadoop_prod.kinodb.testB.snapshots;
committed_at	snapshot_id	parent_id	operation	manifest_list	summary
2022-05-16 15:59:40.991	618425031136003513	NULL	overwrite	hdfs://nameservice1/kino/warehouse/iceberg/hadoop/kinodb/testB/metadata/snap-618425031136003513-1-4015889c-c275-409a-836b-d209f024cdc3.avro	{"added-data-files":"2","added-files-size":"2407","added-records":"2","changed-partition-count":"2","replace-partitions":"true","spark.app.id":"local-1652687931341","total-data-files":"2","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"2407","total-position-deletes":"0","total-records":"2"}
2022-05-16 16:00:29.255	6028470489417111241	618425031136003513	overwrite	hdfs://nameservice1/kino/warehouse/iceberg/hadoop/kinodb/testB/metadata/snap-6028470489417111241-1-bbaccd86-bb9e-4c4b-b2c5-19da704aa3ad.avro	{"added-data-files":"1","added-files-size":"1250","added-records":"2","changed-partition-count":"1","replace-partitions":"true","spark.app.id":"local-1652687931341","total-data-files":"3","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"3657","total-position-deletes":"0","total-records":"4"}
2022-05-16 16:01:34.305	6242810283652646314	6028470489417111241	delete	hdfs://nameservice1/kino/warehouse/iceberg/hadoop/kinodb/testB/metadata/snap-6242810283652646314-1-1d86751b-f8e1-436b-9c9a-cd7a4059f7db.avro	{"changed-partition-count":"3","deleted-data-files":"3","deleted-records":"4","removed-files-size":"3657","spark.app.id":"local-1652687931341","total-data-files":"0","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"0","total-position-deletes":"0","total-records":"0"}
Time taken: 1.064 seconds, Fetched 3 row(s)

-- 根据 snapshots id 关联查询
select * from hadoop_prod.kinodb.testB.history a join hadoop_prod.kinodb.testB.snapshots b on   a.snapshot_id=b.snapshot_id;

-- 查询执行 snapshots id 的记录(可以用于刷回误删除数据)
spark.read.option("snapshot-id","618425031136003513").format("iceberg").load("/kino/warehouse/iceberg/hadoop/kinodb/testB").show
+---+----+---+----------+
| id|name|age|        dt|
+---+----+---+----------+
|  2|李四| 20|2021-06-21|
|  1|张三| 18|2021-06-22|
+---+----+---+----------+
```
## 2.7 隐藏分区 - days
```sql
create table hadoop_prod.kinodb.testC(
  id bigint, 
  name string, 
  ts timestamp
) using iceberg
partitioned by (days(ts));

insert into hadoop_prod.kinodb.testC values(1,'张三',cast(1624773600 as timestamp)),(2,'李四',cast(1624860000 as timestamp));

select * from hadoop_prod.kinodb.testC;
```

## 2.8 隐藏分区 - years
```sql
drop table hadoop_prod.kinodb.testC; 
create table hadoop_prod.kinodb.testC(
  id bigint, 
  name string, 
  ts timestamp
) using iceberg
partitioned by (years(ts));

insert into hadoop_prod.kinodb.testC values(1,'张三',cast(1624860000 as timestamp)),(2,'李四',cast(1593324000 as timestamp));

select * from hadoop_prod.kinodb.testC;
```

## 2.9 隐藏分区 - months
```sql
drop table hadoop_prod.kinodb.testC; 
create table hadoop_prod.kinodb.testC(
  id bigint, 
  name string, 
  ts timestamp
) using iceberg
partitioned by (months(ts));

insert into hadoop_prod.kinodb.testC values(1,'张三',cast(1624860000 as timestamp)),(2,'李四',cast(1622181600 as timestamp));

select * from hadoop_prod.kinodb.testC;
```

## 2.10 隐藏分区 - hours(有 bug)
```sql
drop table hadoop_prod.kinodb.testC; 
create table hadoop_prod.kinodb.testC(
  id bigint, 
  name string, 
  ts timestamp
) using iceberg
partitioned by (hours(ts));

insert into hadoop_prod.kinodb.testC values(1,'张三',cast(1622181600 as timestamp)),(2,'李四',cast(1622178000 as timestamp));

select * from hadoop_prod.kinodb.testC;

hdfs dfs -ls /kino/warehouse/iceberg/hadoop/kinodb/testC/data
Found 2 items
drwxr-xr-x   - root supergroup          0 2022-05-16 16:27 /kino/warehouse/iceberg/hadoop/kinodb/testC/data/ts_hour=2021-05-28-05
drwxr-xr-x   - root supergroup          0 2022-05-16 16:27 /kino/warehouse/iceberg/hadoop/kinodb/testC/data/ts_hour=2021-05-28-06

-- 修改时区(无效)
vim spark-defaults.conf 
spark.sql.session.timeZone=GMT+8

insert into hadoop_prod.kinodb.testC values(1,'张三',cast(1622181600 as timestamp)),(2,'李四',cast(1622178000 as timestamp));
```

## 2.11 bucket 函数
https://iceberg.apache.org/spec/#partition-transforms

```sql
drop table hadoop_prod.kinodb.testC; 
create table hadoop_prod.kinodb.testC(
  id bigint, 
  name string, 
  ts timestamp
) using iceberg
partitioned by (bucket(16,id));

insert into hadoop_prod.kinodb.testC values(1,'张 1',cast(1622152800 as timestamp)),(1,'李 1',cast(1622178000 as timestamp)), (2,'张 2',cast(1622152800 as timestamp)),(3,'李 2',cast(1622178000 as timestamp)), (4,'张 3',cast(1622152800 as timestamp)),(6,'李 3',cast(1622178000 as timestamp)), (5,'张 4',cast(1622152800 as timestamp)),(8,'李 4',cast(1622178000 as timestamp)); 

insert into hadoop_prod.kinodb.testC values(9,'张 5',cast(1622152800 as timestamp)),(10,'李 5',cast(1622178000 as timestamp)), (11,'张 6',cast(1622152800 as timestamp)),(12,'李 6',cast(1622178000 as timestamp)); 

insert into hadoop_prod.kinodb.testC values(13,'张 7',cast(1622152800 as timestamp)),(14,'李 7',cast(1622178000 as timestamp)),(15,'张 8',cast(1622152800 as timestamp)),(16,'李 8',cast(1622178000 as timestamp)); 

insert into hadoop_prod.kinodb.testC values(17,'张 9',cast(1622152800 as timestamp)),(18,'李 9',cast(1622178000 as timestamp)),(18,'张 10',cast(1622152800 as timestamp)),(20,'李 10',cast(1622178000 as timestamp)); 

insert into hadoop_prod.kinodb.testC values(1001,'张 9',cast(1622152800 as timestamp)),(1003,'李 9',cast(1622178000 as timestamp)),(1002,'张 10',cast(1622152800 as timestamp)),(1004,'李 10',cast(1622178000 as timestamp));

select *from hadoop_prod.kinodb.testC;
```

## 2.12 truncate
```sql
drop table hadoop_prod.kinodb.testC; 
create table hadoop_prod.kinodb.testC(
  id bigint, 
  name string, 
  ts timestamp
) using iceberg
partitioned by (truncate(4,id));

insert into hadoop_prod.kinodb.testC values(1,'张 1',cast(1622152800 as timestamp)),(1,'李 1',cast(1622178000 as timestamp)), (2,'张 2',cast(1622152800 as timestamp)),(3,'李 2',cast(1622178000 as timestamp)), (4,'张 3',cast(1622152800 as timestamp)),(6,'李 3',cast(1622178000 as timestamp)), (5,'张 4',cast(1622152800 as timestamp)),(8,'李 4',cast(1622178000 as timestamp)); 

select * from hadoop_prod.kinodb.testC;


insert into hadoop_prod.kinodb.testC values(100,'张 1',cast(1622152800 as timestamp));

UPDATE hadoop_prod.kinodb.testC SET name = '11111' where id = 100;
```

# 三、DataFream API
上传 `core-site.xml`、`hdfs-site.xml`、`mapred-sit.xml`、`yarn-site.xml`

maven 依赖
```xml
<properties>
        <spark.version>3.2.1</spark.version>
        <scala.version>2.12.10</scala.version>
        <log4j.version>1.2.17</log4j.version>
        <slf4j.version>1.7.22</slf4j.version>
        <iceberg.version>0.13.1</iceberg.version>
    </properties>


    <dependencies>
        <!-- Spark 的依赖引入 -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-hive_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <!-- 引入Scala -->
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.iceberg</groupId>
            <artifactId>iceberg-spark3-runtime</artifactId>
            <version>${iceberg.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.iceberg</groupId>
            <artifactId>iceberg-spark3-extensions</artifactId>
            <version>${iceberg.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.1</version>
                <executions>
                    <execution>
                        <id>compile-scala</id>
                        <goals>
                            <goal>add-source</goal>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile-scala</id>
                        <goals>
                            <goal>add-source</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

java code
```java
package com.kino.iceberg;

import org.apache.spark.SparkConf;
import org.apache.spark.sql.DataFrameStatFunctions;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class SparkSample1 {
    public static void main(String[] args) {
        SparkConf sparkConf = new SparkConf();
        sparkConf.set("spark.sql.catalog.hadoop_prod", "org.apache.iceberg.spark.SparkCatalog");
        sparkConf.set("spark.sql.catalog.hadoop_prod.type", "hadoop");
        sparkConf.set("spark.sql.catalog.hadoop_prod.warehouse", "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/");
        sparkConf.set("spark.sql.catalog.catalog-name.type", "hadoop");
        sparkConf.set("spark.sql.catalog.catalog-name.default-namespace", "kinodb");
        sparkConf.set("spark.sql.sources.partitionOverwriteMode", "dynamic");
        sparkConf.set("spark.sql.session.timeZone", "GMT+8");
        sparkConf.setMaster("local[*]").setAppName("table_operations");

        SparkSession sparkSession = SparkSession.builder().config(sparkConf).getOrCreate();
        readTable(sparkSession);

    }

    /**
     * 读取表
     * @param sparkSession
     */
    public static void readTable(SparkSession sparkSession) {
        sparkSession.table("hadoop_prod.kinodb.testA").show();
        sparkSession.read().format("iceberg").load("hadoop_prod.kinodb.testA").show();
        sparkSession.read().format("iceberg").load("/kino/warehouse/iceberg/hadoop/kinodb/testC").show(); // 路径到表就行，不要到具体文件
    }

    /**
     * 读取快照表
     * Streaming Reads: Iceberg supports processing incremental data in spark structured streaming jobs which starts from a historical timestamp:
     */
    public static void readSnapshots(SparkSession sparkSession) {
        sparkSession.read()
            .option("stream-from-timestamp", "1652687707000") //毫秒时间戳，查询比该值时间更早的快照
            .option("streaming-skip-delete-snapshots", "true")
            .option("streaming-skip-overwrite-snapshots", "true")
            .format("iceberg")
            .load("hadoop_prod.kinodb.testA")
            .show();
    }

    /**
     * 写入数据并且创建表
     * @param sparkSession
     * @param table
     * @throws TableAlreadyExistsException
     */
    public static void writeTable(SparkSession sparkSession, String table) throws TableAlreadyExistsException {
        ArrayList<People> aList = new ArrayList<>();
        aList.add(new People(1L, "张三", "2022-01-01"));
        aList.add(new People(2L, "张四", "2022-01-02"));
        aList.add(new People(3L, "张五", "2022-01-03"));
        Dataset<Row> dataFrame = sparkSession.createDataFrame(aList, People.class);
        dataFrame
            .writeTo(table)
            .partitionedBy(new Column("dt"))
            .create();
    }

    /**
     * append 写
     * @param sparkSession
     * @param table
     * @throws NoSuchTableException
     */
    public static void append(SparkSession sparkSession, String table) throws NoSuchTableException {
        ArrayList<People> aList = new ArrayList<>();
        aList.add(new People(1L, "张三", "2022-01-01"));
        aList.add(new People(2L, "张四", "2022-01-02"));
        aList.add(new People(3L, "张五", "2022-01-03"));
        Dataset<Row> dataFrame = sparkSession.createDataFrame(aList, People.class);
        // DataFrameWriteV2 API
        dataFrame.writeTo(table).append();
        // DataFrameWriteV1 API
        dataFrame.write().format("iceberg").mode("append").save(table);
    }

    /**
     * overwrite 覆盖写
     * @param sparkSession
     * @param table
     * @throws NoSuchTableException
     */
    public static void overwrite(SparkSession sparkSession, String table) throws NoSuchTableException {
        ArrayList<People> aList = new ArrayList<>();
        aList.add(new People(1L, "张三", "2022-01-01"));
        aList.add(new People(2L, "张四", "2022-01-02"));
        aList.add(new People(3L, "张五", "2022-01-03"));
        Dataset<Row> dataFrame = sparkSession.createDataFrame(aList, People.class);
        // 只覆盖刷新分区数据
        dataFrame.writeTo(table).overwritePartitions();
        Column column = new Column("dt");
        column.equals("2022-01-03");
        dataFrame.writeTo(table).overwrite(column);
    }
}
```











