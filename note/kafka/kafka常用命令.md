

* [一、查看当所有 topic](#%E4%B8%80%E6%9F%A5%E7%9C%8B%E5%BD%93%E6%89%80%E6%9C%89-topic)
* [二、创建topic](#%E4%BA%8C%E5%88%9B%E5%BB%BAtopic)
* [三、删除topic](#%E4%B8%89%E5%88%A0%E9%99%A4topic)
* [四、发送消息](#%E5%9B%9B%E5%8F%91%E9%80%81%E6%B6%88%E6%81%AF)
* [五、消费消息](#%E4%BA%94%E6%B6%88%E8%B4%B9%E6%B6%88%E6%81%AF)
* [六、查看某个Topic的详情](#%E5%85%AD%E6%9F%A5%E7%9C%8B%E6%9F%90%E4%B8%AAtopic%E7%9A%84%E8%AF%A6%E6%83%85)
* [七、修改分区数](#%E4%B8%83%E4%BF%AE%E6%94%B9%E5%88%86%E5%8C%BA%E6%95%B0)


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