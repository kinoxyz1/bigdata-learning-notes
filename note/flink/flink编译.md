




---
# clone code
git clone https://github.com/apache/flink.git

# checkout branch
```bash
git checkout -b release-1.12.7 release-1.12.7
```

# build code
```bash
mvn -T2C clean package -DskipTests -Dfast
```
看见以下信息即为成功
```bash
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for Flink : 1.12.7:
[INFO] 
[INFO] Flink : Tools : Force Shading ...................... SUCCESS [  2.311 s]
[INFO] Flink : ............................................ SUCCESS [  2.816 s]
[INFO] Flink : Annotations ................................ SUCCESS [  4.738 s]
[INFO] Flink : Test utils : ............................... SUCCESS [  1.637 s]
[INFO] Flink : Test utils : Junit ......................... SUCCESS [  6.083 s]
[INFO] Flink : Metrics : .................................. SUCCESS [  1.637 s]
[INFO] Flink : Metrics : Core ............................. SUCCESS [  3.432 s]
[INFO] Flink : Core ....................................... SUCCESS [ 25.479 s]
[INFO] Flink : Java ....................................... SUCCESS [  5.242 s]
[INFO] Flink : Queryable state : .......................... SUCCESS [  0.864 s]
[INFO] Flink : Queryable state : Client Java .............. SUCCESS [  2.034 s]
[INFO] Flink : FileSystems : .............................. SUCCESS [  0.866 s]
[INFO] Flink : FileSystems : Hadoop FS .................... SUCCESS [  3.178 s]
[INFO] Flink : Runtime .................................... SUCCESS [01:07 min]
[INFO] Flink : Scala ...................................... SUCCESS [01:54 min]
[INFO] Flink : FileSystems : Mapr FS ...................... SUCCESS [  1.620 s]
[INFO] Flink : FileSystems : Hadoop FS shaded ............. SUCCESS [ 14.245 s]
[INFO] Flink : FileSystems : S3 FS Base ................... SUCCESS [  1.773 s]
[INFO] Flink : FileSystems : S3 FS Hadoop ................. SUCCESS [  7.990 s]
[INFO] Flink : FileSystems : S3 FS Presto ................. SUCCESS [  9.911 s]
[INFO] Flink : FileSystems : Swift FS Hadoop .............. SUCCESS [ 18.282 s]
[INFO] Flink : FileSystems : OSS FS ....................... SUCCESS [  8.510 s]
[INFO] Flink : FileSystems : Azure FS Hadoop .............. SUCCESS [ 10.566 s]
[INFO] Flink : Optimizer .................................. SUCCESS [  2.628 s]
[INFO] Flink : Connectors : ............................... SUCCESS [  0.857 s]
[INFO] Flink : Connectors : File Sink Common .............. SUCCESS [  1.622 s]
[INFO] Flink : Streaming Java ............................. SUCCESS [  8.865 s]
[INFO] Flink : Clients .................................... SUCCESS [  2.516 s]
[INFO] Flink : Test utils : Utils ......................... SUCCESS [  1.341 s]
[INFO] Flink : Runtime web ................................ SUCCESS [02:12 min]
[INFO] Flink : Test utils : Connectors .................... SUCCESS [  1.559 s]
[INFO] Flink : Connectors : Base .......................... SUCCESS [  2.003 s]
[INFO] Flink : Connectors : Files ......................... SUCCESS [  2.276 s]
[INFO] Flink : Examples : ................................. SUCCESS [  0.925 s]
[INFO] Flink : Examples : Batch ........................... SUCCESS [ 33.547 s]
[INFO] Flink : Connectors : Hadoop compatibility .......... SUCCESS [ 11.783 s]
[INFO] Flink : State backends : ........................... SUCCESS [  1.844 s]
[INFO] Flink : State backends : RocksDB ................... SUCCESS [  2.736 s]
[INFO] Flink : Tests ...................................... SUCCESS [ 51.493 s]
[INFO] Flink : Streaming Scala ............................ SUCCESS [01:04 min]
[INFO] Flink : Connectors : HCatalog ...................... SUCCESS [ 12.266 s]
[INFO] Flink : Table : .................................... SUCCESS [  1.634 s]
[INFO] Flink : Table : Common ............................. SUCCESS [  4.498 s]
[INFO] Flink : Table : API Java ........................... SUCCESS [  4.293 s]
[INFO] Flink : Table : API Java bridge .................... SUCCESS [  2.652 s]
[INFO] Flink : Table : API Scala .......................... SUCCESS [ 39.987 s]
[INFO] Flink : Table : API Scala bridge ................... SUCCESS [ 18.313 s]
[INFO] Flink : Table : SQL Parser ......................... SUCCESS [ 20.732 s]
[INFO] Flink : Libraries : ................................ SUCCESS [  0.865 s]
[INFO] Flink : Libraries : CEP ............................ SUCCESS [  4.859 s]
[INFO] Flink : Table : Planner ............................ SUCCESS [02:56 min]
[INFO] Flink : Table : SQL Parser Hive .................... SUCCESS [ 11.806 s]
[INFO] Flink : Table : Runtime Blink ...................... SUCCESS [  9.499 s]
[INFO] Flink : Table : Planner Blink ...................... SUCCESS [03:34 min]
[INFO] Flink : Formats : .................................. SUCCESS [  0.866 s]
[INFO] Flink : Formats : Json ............................. SUCCESS [  2.595 s]
[INFO] Flink : Connectors : Elasticsearch base ............ SUCCESS [  4.608 s]
[INFO] Flink : Connectors : Elasticsearch 5 ............... SUCCESS [ 15.717 s]
[INFO] Flink : Connectors : Elasticsearch 6 ............... SUCCESS [  4.113 s]
[INFO] Flink : Connectors : Elasticsearch 7 ............... SUCCESS [  4.083 s]
[INFO] Flink : Connectors : HBase base .................... SUCCESS [  9.282 s]
[INFO] Flink : Connectors : HBase 1.4 ..................... SUCCESS [  7.576 s]
[INFO] Flink : Connectors : HBase 2.2 ..................... SUCCESS [ 12.122 s]
[INFO] Flink : Formats : Hadoop bulk ...................... SUCCESS [  2.584 s]
[INFO] Flink : Formats : Orc .............................. SUCCESS [  3.794 s]
[INFO] Flink : Formats : Orc nohive ....................... SUCCESS [  3.657 s]
[INFO] Flink : Formats : Avro ............................. SUCCESS [  9.274 s]
[INFO] Flink : Formats : Parquet .......................... SUCCESS [ 16.973 s]
[INFO] Flink : Formats : Csv .............................. SUCCESS [  2.588 s]
[INFO] Flink : Connectors : Hive .......................... SUCCESS [  5.081 s]
[INFO] Flink : Connectors : JDBC .......................... SUCCESS [  2.627 s]
[INFO] Flink : Connectors : RabbitMQ ...................... SUCCESS [  1.929 s]
[INFO] Flink : Connectors : Twitter ....................... SUCCESS [  2.125 s]
[INFO] Flink : Connectors : Nifi .......................... SUCCESS [  0.556 s]
[INFO] Flink : Connectors : Cassandra ..................... SUCCESS [  6.686 s]
[INFO] Flink : Metrics : JMX .............................. SUCCESS [  1.238 s]
[INFO] Flink : Connectors : Kafka ......................... SUCCESS [ 14.052 s]
[INFO] Flink : Connectors : Google PubSub ................. SUCCESS [  1.352 s]
[INFO] Flink : Connectors : Kinesis ....................... SUCCESS [ 15.564 s]
[INFO] Flink : Connectors : SQL : Elasticsearch 6 ......... SUCCESS [  7.624 s]
[INFO] Flink : Connectors : SQL : Elasticsearch 7 ......... SUCCESS [  9.319 s]
[INFO] Flink : Connectors : SQL : HBase 1.4 ............... SUCCESS [  8.056 s]
[INFO] Flink : Connectors : SQL : HBase 2.2 ............... SUCCESS [ 13.777 s]
[INFO] Flink : Connectors : SQL : Hive 1.2.2 .............. SUCCESS [  4.762 s]
[INFO] Flink : Connectors : SQL : Hive 2.2.0 .............. SUCCESS [  9.362 s]
[INFO] Flink : Connectors : SQL : Hive 2.3.6 .............. SUCCESS [  9.066 s]
[INFO] Flink : Connectors : SQL : Hive 3.1.2 .............. SUCCESS [ 10.773 s]
[INFO] Flink : Connectors : SQL : Kafka ................... SUCCESS [  0.997 s]
[INFO] Flink : Connectors : SQL : Kinesis ................. SUCCESS [  8.338 s]
[INFO] Flink : Formats : Avro confluent registry .......... SUCCESS [  2.487 s]
[INFO] Flink : Formats : Sequence file .................... SUCCESS [  0.813 s]
[INFO] Flink : Formats : Compress ......................... SUCCESS [  1.776 s]
[INFO] Flink : Formats : SQL Orc .......................... SUCCESS [  2.295 s]
[INFO] Flink : Formats : SQL Parquet ...................... SUCCESS [  0.750 s]
[INFO] Flink : Formats : SQL Avro ......................... SUCCESS [  3.289 s]
[INFO] Flink : Formats : SQL Avro Confluent Registry ...... SUCCESS [  7.052 s]
[INFO] Flink : Examples : Streaming ....................... SUCCESS [ 31.104 s]
[INFO] Flink : Examples : Table ........................... SUCCESS [ 29.000 s]
[INFO] Flink : Examples : Build Helper : .................. SUCCESS [  0.695 s]
[INFO] Flink : Examples : Build Helper : Streaming Twitter  SUCCESS [  0.714 s]
[INFO] Flink : Examples : Build Helper : Streaming State machine SUCCESS [  0.701 s]
[INFO] Flink : Examples : Build Helper : Streaming Google PubSub SUCCESS [  3.407 s]
[INFO] Flink : Container .................................. SUCCESS [  0.728 s]
[INFO] Flink : Queryable state : Runtime .................. SUCCESS [  1.999 s]
[INFO] Flink : Mesos ...................................... SUCCESS [01:12 min]
[INFO] Flink : Kubernetes ................................. SUCCESS [ 11.766 s]
[INFO] Flink : Yarn ....................................... SUCCESS [  3.922 s]
[INFO] Flink : Libraries : Gelly .......................... SUCCESS [  5.814 s]
[INFO] Flink : Libraries : Gelly scala .................... SUCCESS [ 38.170 s]
[INFO] Flink : Libraries : Gelly Examples ................. SUCCESS [ 18.752 s]
[INFO] Flink : External resources : ....................... SUCCESS [  1.633 s]
[INFO] Flink : External resources : GPU ................... SUCCESS [  1.620 s]
[INFO] Flink : Metrics : Dropwizard ....................... SUCCESS [  1.168 s]
[INFO] Flink : Metrics : Graphite ......................... SUCCESS [  0.466 s]
[INFO] Flink : Metrics : InfluxDB ......................... SUCCESS [  1.261 s]
[INFO] Flink : Metrics : Prometheus ....................... SUCCESS [  1.169 s]
[INFO] Flink : Metrics : StatsD ........................... SUCCESS [  1.169 s]
[INFO] Flink : Metrics : Datadog .......................... SUCCESS [  4.544 s]
[INFO] Flink : Metrics : Slf4j ............................ SUCCESS [  1.169 s]
[INFO] Flink : Libraries : CEP Scala ...................... SUCCESS [ 25.075 s]
[INFO] Flink : Table : Uber ............................... SUCCESS [  5.072 s]
[INFO] Flink : Table : Uber Blink ......................... SUCCESS [  8.706 s]
[INFO] Flink : Python ..................................... SUCCESS [ 21.271 s]
[INFO] Flink : Table : SQL Client ......................... SUCCESS [  5.541 s]
[INFO] Flink : Libraries : State processor API ............ SUCCESS [  2.653 s]
[INFO] Flink : ML : ....................................... SUCCESS [  1.844 s]
[INFO] Flink : ML : API ................................... SUCCESS [  1.346 s]
[INFO] Flink : ML : Lib ................................... SUCCESS [  0.851 s]
[INFO] Flink : ML : Uber .................................. SUCCESS [  0.139 s]
[INFO] Flink : Scala shell ................................ SUCCESS [ 24.477 s]
[INFO] Flink : Dist ....................................... SUCCESS [ 12.287 s]
[INFO] Flink : Yarn Tests ................................. SUCCESS [  4.659 s]
[INFO] Flink : E2E Tests : ................................ SUCCESS [  1.709 s]
[INFO] Flink : E2E Tests : CLI ............................ SUCCESS [  4.255 s]
[INFO] Flink : E2E Tests : Parent Child classloading program SUCCESS [  4.228 s]
[INFO] Flink : E2E Tests : Parent Child classloading lib-package SUCCESS [  4.132 s]
[INFO] Flink : E2E Tests : Dataset allround ............... SUCCESS [  4.133 s]
[INFO] Flink : E2E Tests : Dataset Fine-grained recovery .. SUCCESS [  4.134 s]
[INFO] Flink : E2E Tests : Datastream allround ............ SUCCESS [  5.317 s]
[INFO] Flink : E2E Tests : Batch SQL ...................... SUCCESS [  4.222 s]
[INFO] Flink : E2E Tests : Stream SQL ..................... SUCCESS [  4.222 s]
[INFO] Flink : E2E Tests : Distributed cache via blob ..... SUCCESS [  4.233 s]
[INFO] Flink : E2E Tests : High parallelism iterations .... SUCCESS [ 11.297 s]
[INFO] Flink : E2E Tests : Stream stateful job upgrade .... SUCCESS [  2.262 s]
[INFO] Flink : E2E Tests : Queryable state ................ SUCCESS [  5.774 s]
[INFO] Flink : E2E Tests : Local recovery and allocation .. SUCCESS [  4.224 s]
[INFO] Flink : E2E Tests : Elasticsearch 5 ................ SUCCESS [  9.125 s]
[INFO] Flink : E2E Tests : Elasticsearch 6 ................ SUCCESS [  6.580 s]
[INFO] Flink : Quickstart : ............................... SUCCESS [  3.517 s]
[INFO] Flink : Quickstart : Java .......................... SUCCESS [  3.071 s]
[INFO] Flink : Quickstart : Scala ......................... SUCCESS [  3.071 s]
[INFO] Flink : E2E Tests : Quickstart ..................... SUCCESS [  4.207 s]
[INFO] Flink : E2E Tests : Confluent schema registry ...... SUCCESS [  5.950 s]
[INFO] Flink : E2E Tests : Stream state TTL ............... SUCCESS [  5.509 s]
[INFO] Flink : E2E Tests : SQL client ..................... SUCCESS [  4.453 s]
[INFO] Flink : E2E Tests : File sink ...................... SUCCESS [  4.863 s]
[INFO] Flink : E2E Tests : State evolution ................ SUCCESS [  4.663 s]
[INFO] Flink : E2E Tests : RocksDB state memory control ... SUCCESS [  2.157 s]
[INFO] Flink : E2E Tests : Common ......................... SUCCESS [  5.010 s]
[INFO] Flink : E2E Tests : Metrics availability ........... SUCCESS [  1.958 s]
[INFO] Flink : E2E Tests : Metrics reporter prometheus .... SUCCESS [  2.010 s]
[INFO] Flink : E2E Tests : Heavy deployment ............... SUCCESS [  8.003 s]
[INFO] Flink : E2E Tests : Connectors : Google PubSub ..... SUCCESS [  3.746 s]
[INFO] Flink : E2E Tests : Streaming Kafka base ........... SUCCESS [  2.846 s]
[INFO] Flink : E2E Tests : Streaming Kafka ................ SUCCESS [  6.809 s]
[INFO] Flink : E2E Tests : Plugins : ...................... SUCCESS [  2.249 s]
[INFO] Flink : E2E Tests : Plugins : Dummy fs ............. SUCCESS [  1.563 s]
[INFO] Flink : E2E Tests : Plugins : Another dummy fs ..... SUCCESS [  1.584 s]
[INFO] Flink : E2E Tests : TPCH ........................... SUCCESS [  3.563 s]
[INFO] Flink : E2E Tests : Streaming Kinesis .............. SUCCESS [ 11.239 s]
[INFO] Flink : E2E Tests : Elasticsearch 7 ................ SUCCESS [  6.585 s]
[INFO] Flink : E2E Tests : Common Kafka ................... SUCCESS [  2.318 s]
[INFO] Flink : E2E Tests : TPCDS .......................... SUCCESS [  4.040 s]
[INFO] Flink : E2E Tests : Netty shuffle memory control ... SUCCESS [  2.902 s]
[INFO] Flink : E2E Tests : Python ......................... SUCCESS [  8.838 s]
[INFO] Flink : E2E Tests : HBase .......................... SUCCESS [  3.846 s]
[INFO] Flink : State backends : Heap spillable ............ SUCCESS [  1.277 s]
[INFO] Flink : Contrib : .................................. SUCCESS [  0.857 s]
[INFO] Flink : Contrib : Connectors : Wikiedits ........... SUCCESS [  1.088 s]
[INFO] Flink : FileSystems : Tests ........................ SUCCESS [  2.814 s]
[INFO] Flink : Docs ....................................... SUCCESS [  2.648 s]
[INFO] Flink : Walkthrough : .............................. SUCCESS [  3.513 s]
[INFO] Flink : Walkthrough : Common ....................... SUCCESS [  1.320 s]
[INFO] Flink : Walkthrough : Datastream Java .............. SUCCESS [  3.073 s]
[INFO] Flink : Walkthrough : Datastream Scala ............. SUCCESS [  3.071 s]
[INFO] Flink : Tools : CI : Java .......................... SUCCESS [  6.671 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  11:46 min (Wall Clock)
[INFO] Finished at: 2023-04-18T00:02:27+08:00
[INFO] ------------------------------------------------------------------------
```


# flink on k8s 远程调试
debug session: 在启动命令中添加: `-Ddebug -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5100`, 5100 是debug端口
```bash
    spec:
      containers:
      - args:
        - native-k8s
        - $JAVA_HOME/bin/java -classpath $FLINK_CLASSPATH -Ddebug -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5100
          -Xmx1073741824 -Xms1073741824 -XX:MaxMetaspaceSize=268435456 -Duser.timezone=GMT+08
          org.apache.flink.kubernetes.entrypoint.KubernetesSessionClusterEntrypoint
          -D jobmanager.memory.off-heap.size=134217728b -D jobmanager.memory.jvm-overhead.min=201326592b
          -D jobmanager.memory.jvm-metaspace.size=268435456b -D jobmanager.memory.heap.size=1073741824b
          -D jobmanager.memory.jvm-overhead.max=201326592b
```

debug jobmanager 和 taskmanager: 在 flink-conf.yaml 中添加:
```bash
    env.java.opts.jobmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005"
    env.java.opts.taskmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5006"
    env.java.opts.client: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5008"
```
后续就是正常debug java 项目的操作了。