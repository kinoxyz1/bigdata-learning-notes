


---
# 一、环境
16.04.1-Ubuntu

# 二、下载
[confluent 官网](https://www.confluent.io/)

[confluent 下载地址](https://www.confluent.io/download)

# 三、解压
```bash
$ tar -zxvf confluent-6.0.1.tar.gz
$ cd confluent-6.0.1
总用量 36
drwxr-xr-x  8 bencom bencom 4096 11月 21 05:30 ./
drwxrwxr-x  4 bencom bencom 4096 12月 24 11:25 ../
drwxr-xr-x  3 bencom bencom 4096 11月 21 04:26 bin/
drwxr-xr-x 16 bencom bencom 4096 11月 21 04:25 etc/
drwxr-xr-x  3 bencom bencom 4096 11月 21 04:22 lib/
drwxr-xr-x  3 bencom bencom 4096 11月 21 04:26 libexec/
-rw-r--r--  1 bencom bencom  871 11月 21 05:30 README
drwxr-xr-x  7 bencom bencom 4096 11月 21 04:26 share/
drwxr-xr-x  2 bencom bencom 4096 11月 21 05:30 src/
```

# 四、配置 zookeeper、kafka
在data下创建两个目录
```bash
$ bencom@jz-desktop-03:/app/bigdata-app/confluent-6.0.1$ mkdir 
$ mkdir data
$ cd data/
$ mkdir zkdata
$ mkdir kafkadata
```
修改 zookeeper.properties 文件
```bash
$ vim etc/kafka/zookeeper.properties
# 修改 dataDir 为 上面创建的 zkdata 路径
$ dataDir=/app/bigdata-app/confluent-6.0.1/data/zkdata
```
修改server.properties文件
```bash
$ vim etc/kafka/server.properties
$ port=9099
$ log.dirs=/app/bigdata-app/confluent-6.0.1/data/kafkadata
```

# 五、启动 Confluent Platform
创建启动脚本
```bash
$ vim bin/start-all.sh
export PATH=/app/bigdata-app/confluent-6.0.1/bin:$PATH

export CONFLUENT_CURRENT=/app/bigdata-app/confluent-6.0.1/data
export CONFLUENT_HOME=/app/bigdata-app/confluent-6.0.1

export ZOOKEEPER_HEAP_OPTS="-Xmx256m -Xms256m"
export KAFKA_HEAP_OPTS="-Xmx2G -Xms512m"
export SCHEMAREGISTRY_HEAP_OPTS="-Xmx256m -Xms256m"
export KAFKAREST_HEAP_OPTS="-Xmx256m -Xms256m"
export CONNECT_HEAP_OPTS="-Xmx2G -Xms2G"

#confluent local start connect
confluent local services connect start
#connect-web-start
```
赋予执行权限
```bash
$ chmod 775 bin/start-all.sh
```
启动
```bash
$ bin/start-all.sh
The local commands are intended for a single-node development environment only,
NOT for production usage. https://docs.confluent.io/current/cli/index.html

Using CONFLUENT_CURRENT: /app/bigdata-app/confluent-6.0.1/data/confluent.798278
ZooKeeper is [UP]
Starting Kafka
Kafka is [UP]
Starting Schema Registry
Schema Registry is [UP]
Starting Connect
Connect is [UP]
```
查看 kafka topic
```bash
$ bin/kafka-topics --zookeeper localhost:2181 --list
__consumer_offsets
_confluent-license
_confluent-telemetry-metrics
_confluent_balancer_api_state
_confluent_balancer_broker_samples
_confluent_balancer_partition_samples
# 查看 topic 详细信息
$ bin/kafka-topics --zookeeper localhost:2181 --describe --topic _confluent-telemetry-metrics
# 消费
$ bin/kafka-console-consumer --bootstrap-server localhost:9099 --from-beginning --topic _confluent-telemetry-metrics
```

# 六、配置 Confluent-connectors
[下载 connectors](https://www.confluent.io/hub/?_ga=2.151811257.1872375735.1607484271-1088345547.1606377268)

创建目录
```bash
$ mkdir -p plugins/libs
```
上传下载好的jar
```bash
$ ll plugins/libs
drwxr-xr-x 6 root  root  4096 Dec 22 15:09 confluentinc-kafka-connect-jdbc-10.0.1
drwxr-xr-x 2 kafka kafka 4096 Dec  1 13:48 debezium-connector-db2
drwxr-xr-x 2 kafka kafka 4096 Dec  1 13:48 debezium-connector-mongodb
drwxr-xr-x 2 kafka kafka 4096 Dec  1 13:48 debezium-connector-mysql
drwxr-xr-x 2 kafka kafka 4096 Dec  1 13:48 debezium-connector-oracle
drwxr-xr-x 2 kafka kafka 4096 Dec  1 13:48 debezium-connector-postgres
drwxr-xr-x 2 kafka kafka 4096 Dec  1 13:48 debezium-connector-sqlserver
drwxr-xr-x 2 kafka kafka 4096 Dec  1 13:48 debezium-connector-vitess
drwxr-xr-x 2 root  root  4096 Feb 22  2019 kudusink
```
查看
```bash
$ curl -H "Accept:application/json" http://192.168.1.141:8083/connectors
[]
```

# 七、启动 WebUI
```bash
$ bin/control-center-start etc/confluent-control-center/control-center.properties
```
