







# zookeeper
```bash
docker run -d -p 2181:2181 --name zookeeper1 zookeeper

docker inspect zookeeper1
找到IPAddress拿到ip
```

# kafka
```bash

# KAFKA_BROKER_ID：配置broker_id，在kafka集群中是唯一的
# KAFKA_ZOOKEEPER_CONNECT：配置连接 zookeeper 的 ip 和 port
# KAFKA_LISTENERS：配置 kafka 监听的端口
docker run -d --name kafka1 -p 9092:9092 \
  -e KAFKA_BROKER_ID=0 \
  -e KAFKA_ZOOKEEPER_CONNECT=172.17.0.3:2181 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092 \
  -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
  wurstmeister/kafka
  
docker exec -it kafka1 bash
cd /opt/kafka/bin
# 创建主题
./kafka-topics.sh --create --zookeeper 172.17.0.3:2181 --replication-factor 1 --partitions 1 --topic test

# 生产
# --broker-list：指定 kafka 的 ip 和 port
# --topic：指定消息发送到哪个主题
./kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test
{"id": "1", "name": "kino1"}


# 消费
# --bootstrap-server：指定 kafka 的 ip 和 port
# --topic：指定订阅的主题
./kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test
```

# redis
```bash
docker run -d -p 6379:6379 \
-v /Users/kino/app/data/redis/redis.conf:/etc/redis/redis.conf \
-v /Users/kino/app/data/redis/data:/data \
--name redis redis redis-server /etc/redis/redis.conf
```
