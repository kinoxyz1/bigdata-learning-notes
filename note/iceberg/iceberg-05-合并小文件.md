







---
# 一、引入 maven 依赖
```xml
    <properties>
        <spark.version>3.2.1</spark.version>
        <scala.version>2.12.10</scala.version>
        <log4j.version>1.2.17</log4j.version>
        <slf4j.version>1.7.22</slf4j.version>
        <iceberg.version>0.13.1</iceberg.version>
        <scala.binary.version>2.12</scala.binary.version>
        <flink.version>1.13.6</flink.version>
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
      
				<!-- Flink 的依赖引入 -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-table-common -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-common</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-table-api-java -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-table-api-java-bridge -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-table-planner -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-table-planner-blink -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner-blink_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.iceberg</groupId>
            <artifactId>iceberg-flink-runtime</artifactId>
            <version>0.12.1</version>
<!--            <version>1.13-0.14.0-SNAPSHOT</version>-->
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.3</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-clients -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
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

# 二、JavaCode
## 2.1 压缩(合并小文件)
```java
package com.kino.iceberg;

import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.data.RowData;
import org.apache.hadoop.conf.Configuration;
import org.apache.iceberg.Snapshot;
import org.apache.iceberg.Table;
import org.apache.iceberg.actions.RewriteDataFilesActionResult;
import org.apache.iceberg.catalog.Catalog;
import org.apache.iceberg.catalog.Namespace;
import org.apache.iceberg.catalog.TableIdentifier;
import org.apache.iceberg.flink.CatalogLoader;
import org.apache.iceberg.flink.TableLoader;
import org.apache.iceberg.flink.actions.Actions;
import org.apache.iceberg.flink.sink.FlinkSink;
import org.apache.iceberg.flink.source.FlinkSource;

import java.util.HashMap;
import java.util.concurrent.TimeUnit;

public class FlinkSample1 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
				compress(env, "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA");
    }
  
    // 压缩(重写文件操作)
    public static void compress(StreamExecutionEnvironment env, String hadoopPath) throws Exception {
        TableLoader tableLoader = TableLoader.fromHadoopTable(hadoopPath);
        tableLoader.open();
        Table table = tableLoader.loadTable();
        Actions.forTable(env, table)
            .rewriteDataFiles()
            .maxParallelism(1)
            .targetSizeInBytes(128 * 1024 * 1024)   // 128M
            .execute();
    }
}
```

压缩完成之后，在 data 目录中会产生一个新的 parquet 文件，放到 Spark 中读取
```java
scala> spark.read.parquet("hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/data/00000-0-b29d4cb9-641a-471d-9b78-d50b01c21ba1-00001.parquet").show
+----+----+
|  id|name|
+----+----+
|1002|李四|
|1003|王五|
|1004|  CC|
|1005|  BB|
|1006|  AA|
+----+----+
```

## 2.2 删除过期快照
查看快照
```sql
spark-sql> select * from hadoop_prod.iceberg_db.testA.snapshots;
committed_at	        snapshot_id	        parent_id	        operation	manifest_list	                                                                                                                                    summary
2022-05-17 10:55:59.624	5975152508767258906	NULL                    append	    hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-5975152508767258906-1-2b2b448b-7f3c-4906-8552-d8ffa0f22478.avro	{"added-data-files":"1","added-files-size":"702","added-records":"1","changed-partition-count":"1","flink.job-id":"7c3b89ef8a838dac7e32aea4ba5118f9","flink.max-committed-checkpoint-id":"9223372036854775807","total-data-files":"1","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"702","total-position-deletes":"0","total-records":"1"}
2022-05-17 11:49:49.222	8240004834922072738	5975152508767258906	append	    hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-8240004834922072738-1-7516ba25-57dd-493c-889f-44b92d55f4fc.avro	{"added-data-files":"1","added-files-size":"692","added-records":"2","changed-partition-count":"1","flink.job-id":"c6fbbfea7f87ac9dc6a54b2787959f69","flink.max-committed-checkpoint-id":"9223372036854775807","total-data-files":"2","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"1394","total-position-deletes":"0","total-records":"3"}
2022-05-17 12:50:12.107	3048447316468867114	8240004834922072738	append	    hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-3048447316468867114-1-98cfdc64-2a72-4944-bf31-a493a4e78680.avro	{"added-data-files":"1","added-files-size":"672","added-records":"3","changed-partition-count":"1","flink.job-id":"8f892c609ba9e21c980f1dc11fae89ef","flink.max-committed-checkpoint-id":"9223372036854775807","total-data-files":"3","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"2066","total-position-deletes":"0","total-records":"6"}
2022-05-17 13:09:02.755	7752151405581972398	3048447316468867114	delete	    hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-7752151405581972398-1-c02a6578-f98e-4b44-b150-eba7031295c0.avro	{"changed-partition-count":"1","deleted-data-files":"1","deleted-records":"1","removed-files-size":"702","spark.app.id":"local-1652759895862","total-data-files":"2","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"1364","total-position-deletes":"0","total-records":"5"}
2022-05-17 13:41:50.895	7616118909735383993	7752151405581972398	replace	    hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-7616118909735383993-1-38158f1b-e0c0-4d7b-a419-86f86a5df541.avro	{"added-data-files":"1","added-files-size":"711","added-records":"5","changed-partition-count":"1","deleted-data-files":"2","deleted-records":"5","removed-files-size":"1364","total-data-files":"1","total-delete-files":"0","total-equality-deletes":"0","total-files-size":"711","total-position-deletes":"0","total-records":"5"}
Time taken: 0.094 seconds, Fetched 5 row(s)
```

删除快照
```java
// 删除 过期快照
Snapshot snapshot = table.currentSnapshot();
long old = snapshot.timestampMillis() - TimeUnit.MINUTES.toSeconds(5); // 删除前 5 分钟的快照
table
  .expireSnapshots()
  .expireOlderThan(old)
  .commit();
```

再次查看快照, 只剩最后一个了
```sql
spark-sql> select * from hadoop_prod.iceberg_db.testA.history;
made_current_at	snapshot_id	parent_id	is_current_ancestor
2022-05-17 13:41:50.895	7616118909735383993	7752151405581972398	true
Time taken: 0.059 seconds, Fetched 1 row(s)
```

查看数据
```sql
spark-sql> select * from hadoop_prod.iceberg_db.testA;
id	name
1002	李四
1003	王五
1004	CC
1005	BB
1006	AA
Time taken: 0.09 seconds, Fetched 5 row(s)
```

当删除过期快照后, 可以看到 metadata file 1-3已经被删除, 但是 4-6 依然存在
```bash
[root@jz-desktop-01 iceberg]# hdfs dfs -ls /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata
Found 9 items
-rw-r--r--   3 kino supergroup       5871 2022-05-17 15:21 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/1ef07e3a-639c-4455-9b7c-7d31a316ac20-m0.avro
-rw-r--r--   3 kino supergroup       5896 2022-05-17 15:21 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/1ef07e3a-639c-4455-9b7c-7d31a316ac20-m1.avro
-rw-r--r--   3 kino supergroup       5817 2022-05-17 15:21 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/1ef07e3a-639c-4455-9b7c-7d31a316ac20-m2.avro
-rw-r--r--   3 kino supergroup       3831 2022-05-17 15:21 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-6204093385299882330-1-1ef07e3a-639c-4455-9b7c-7d31a316ac20.avro
-rw-r--r--   3 root supergroup       4290 2022-05-17 15:19 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v4.metadata.json
-rw-r--r--   3 root supergroup       5141 2022-05-17 15:20 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v5.metadata.json
-rw-r--r--   3 kino supergroup       6042 2022-05-17 15:21 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v6.metadata.json
-rw-r--r--   3 kino supergroup       2707 2022-05-17 15:22 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v7.metadata.json
-rw-r--r--   3 kino supergroup          1 2022-05-17 15:22 /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/version-hint.text
```

这是因为在 snap-6204093385299882330-1-1ef07e3a-639c-4455-9b7c-7d31a316ac20.avro 依然有相应的引用

```bash
[root@jz-desktop-01 iceberg]# hdfs dfs -cat /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v7.metadata.json
{
  "format-version" : 1,
  "table-uuid" : "ed58956d-4d5b-4df5-849b-30a6d9864dc6",
  "location" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA",
  "last-updated-ms" : 1652772145192,
  "last-column-id" : 2,
  "schema" : {
    "type" : "struct",
    "schema-id" : 0,
    "fields" : [ {
      "id" : 1,
      "name" : "id",
      "required" : false,
      "type" : "long"
    }, {
      "id" : 2,
      "name" : "name",
      "required" : false,
      "type" : "string"
    } ]
  },
  "current-schema-id" : 0,
  "schemas" : [ {
    "type" : "struct",
    "schema-id" : 0,
    "fields" : [ {
      "id" : 1,
      "name" : "id",
      "required" : false,
      "type" : "long"
    }, {
      "id" : 2,
      "name" : "name",
      "required" : false,
      "type" : "string"
    } ]
  } ],
  "partition-spec" : [ ],
  "default-spec-id" : 0,
  "partition-specs" : [ {
    "spec-id" : 0,
    "fields" : [ ]
  } ],
  "last-partition-id" : 999,
  "default-sort-order-id" : 0,
  "sort-orders" : [ {
    "order-id" : 0,
    "fields" : [ ]
  } ],
  "properties" : {
    "owner" : "root",
    "write.metadata.delete-after-commit.enabled" : "true",
    "write.metadata.previous-versions-max" : "3"
  },
  "current-snapshot-id" : 6204093385299882330,
  "snapshots" : [ {
    "snapshot-id" : 6204093385299882330,
    "parent-snapshot-id" : 8382357312601294571,
    "timestamp-ms" : 1652772094373,
    "summary" : {
      "operation" : "replace",
      "added-data-files" : "1",
      "deleted-data-files" : "5",
      "added-records" : "5",
      "deleted-records" : "5",
      "added-files-size" : "711",
      "removed-files-size" : "3313",
      "changed-partition-count" : "1",
      "total-records" : "5",
      "total-files-size" : "711",
      "total-data-files" : "1",
      "total-delete-files" : "0",
      "total-position-deletes" : "0",
      "total-equality-deletes" : "0"
    },
    "manifest-list" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-6204093385299882330-1-1ef07e3a-639c-4455-9b7c-7d31a316ac20.avro",
    "schema-id" : 0
  } ],
  "snapshot-log" : [ {
    "timestamp-ms" : 1652772094373,
    "snapshot-id" : 6204093385299882330
  } ],
  "metadata-log" : [ {
    "timestamp-ms" : 1652771993834,
    "metadata-file" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v4.metadata.json"
  }, {
    "timestamp-ms" : 1652772017183,
    "metadata-file" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v5.metadata.json"
  }, {
    "timestamp-ms" : 1652772094373,
    "metadata-file" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/v6.metadata.json"
  } ]
}
```

其中的 manifest-list 又引用了三个 manifest-file，所以还有三个 manifest-file 未被删除，manifest-file 中又指向了真正的 parquet 文件，此时删除过期 snapshot 后，parquet 文件被删除了，但是没有完全被删除。

```bash
[root@jz-desktop-01 iceberg]# hdfs dfs -get /kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/snap-6204093385299882330-1-1ef07e3a-639c-4455-9b7c-7d31a316ac20.avro

[root@jz-desktop-01 iceberg]# java -jar avro-tools-1.11.0.jar tojson --pretty snap-6204093385299882330-1-1ef07e3a-639c-4455-9b7c-7d31a316ac20.avro
22/05/17 15:27:21 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
{
  "manifest_path" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/1ef07e3a-639c-4455-9b7c-7d31a316ac20-m2.avro",
  "manifest_length" : 5817,
  "partition_spec_id" : 0,
  "added_snapshot_id" : {
    "long" : 6204093385299882330
  },
  "added_data_files_count" : {
    "int" : 1
  },
  "existing_data_files_count" : {
    "int" : 0
  },
  "deleted_data_files_count" : {
    "int" : 0
  },
  "partitions" : {
    "array" : [ ]
  },
  "added_rows_count" : {
    "long" : 5
  },
  "existing_rows_count" : {
    "long" : 0
  },
  "deleted_rows_count" : {
    "long" : 0
  }
}
{
  "manifest_path" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/1ef07e3a-639c-4455-9b7c-7d31a316ac20-m1.avro",
  "manifest_length" : 5896,
  "partition_spec_id" : 0,
  "added_snapshot_id" : {
    "long" : 6204093385299882330
  },
  "added_data_files_count" : {
    "int" : 0
  },
  "existing_data_files_count" : {
    "int" : 0
  },
  "deleted_data_files_count" : {
    "int" : 3
  },
  "partitions" : {
    "array" : [ ]
  },
  "added_rows_count" : {
    "long" : 0
  },
  "existing_rows_count" : {
    "long" : 0
  },
  "deleted_rows_count" : {
    "long" : 3
  }
}
{
  "manifest_path" : "hdfs://nameservice1/kino/warehouse/iceberg/hadoop/iceberg_db/testA/metadata/1ef07e3a-639c-4455-9b7c-7d31a316ac20-m0.avro",
  "manifest_length" : 5871,
  "partition_spec_id" : 0,
  "added_snapshot_id" : {
    "long" : 6204093385299882330
  },
  "added_data_files_count" : {
    "int" : 0
  },
  "existing_data_files_count" : {
    "int" : 0
  },
  "deleted_data_files_count" : {
    "int" : 2
  },
  "partitions" : {
    "array" : [ ]
  },
  "added_rows_count" : {
    "long" : 0
  },
  "existing_rows_count" : {
    "long" : 0
  },
  "deleted_rows_count" : {
    "long" : 2
  }
}
```


## 2.3 删除孤立数据(有 bug)
需要设置 spark 参数
```bash
vim conf/spark-defaults.conf
spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions
```

```sql
// 删除孤立数据
CALL hadoop_prod.system.remove_orphan_files(table => 'iceberg_db.testA', dry_run => true)
```


## 2.4 删除无效 metadata 
```sql
目前只能在建表语句中指定
spark-sql> create table testA( 
  id bigint,
  name string
) using iceberg
TBLPROPERTIES (
--   'write.distribution-mode' = 'hash',
  'write.metadata.delete-after-commit.enabled' = 'true',
  'write.metadata.previous-versions-max' = 3
);

insert into testA values(1001,' 张三');
insert into testA values(1002, '李四'),(1003, '王五');
insert into testA values(1004, 'CC'),(1005, 'BB'),(1006, 'AA');
delete from testA where id = 1001;
```
- `write.distribution-mode`: 定义写入数据的分布
    - none: 无操作；
    - hash: 按分区键散列分布；
    - range: 按分区键或排序键分配；
- `write.metadata.delete-after-commit.enabled`: 每次操作后是否删除旧的 metadata file
- `write.metadata.previous-versions-max`: 要保留的旧 metadata file 数量






















