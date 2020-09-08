




----
# 一、查看当所有 topic
```bash
[root@hadoop1 kafka_2.11-2.2.1]# bin/kafka-topics.sh --zookeeper hadoop1:2181 --list
```

# 二、创建topic
```bash
[root@hadoop1 kafka_2.11-2.2.1]# bin/kafka-topics.sh --zookeeper hadoop1:2181 --create --replication-factor 3 --partitions 1 --topic first
```
选项说明：
- --topic: 定义topic名
- --replication-factor: 定义副本数
- --partitions: 定义分区数


# 三、删除topic
```bash
[root@hadoop1 kafka_2.11-2.2.1]# bin/kafka-topics.sh --zookeeper hadoop1:2181 --delete --topic first
```


# 四、发送消息
```bash
[root@hadoop1 kafka_2.11-2.2.1]# bin/kafka-console-producer.sh --broker-list hadoop1:9092 --topic first
```

# 五、消费消息
```bash
[root@hadoop1 kafka_2.11-2.2.1]# bin/kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --from-beginning --topic first
```

# 六、查看某个Topic的详情
```bash
[root@hadoop1 kafka_2.11-2.2.1]# bin/kafka-topics.sh --zookeeper hadoop1:2181 --describe --topic first
```

# 七、修改分区数
```bash
[root@hadoop1 kafka_2.11-2.2.1]# bin/kafka-topics.sh --zookeeper hadoop1:2181 --alter --topic first --partitions 6
```