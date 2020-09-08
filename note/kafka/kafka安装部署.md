
* [一、下载](#%E4%B8%80%E4%B8%8B%E8%BD%BD)
* [二、集群规划](#%E4%BA%8C%E9%9B%86%E7%BE%A4%E8%A7%84%E5%88%92)
* [三、集群部署](#%E4%B8%89%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2)
  * [3\.1 解压到指定目录](#31-%E8%A7%A3%E5%8E%8B%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9B%AE%E5%BD%95)
  * [3\.2 在 /usr/bigdata/kafka\_2\.11\-2\.2\.1 目录下创建 logs文件夹](#32-%E5%9C%A8-usrbigdatakafka_211-221-%E7%9B%AE%E5%BD%95%E4%B8%8B%E5%88%9B%E5%BB%BA-logs%E6%96%87%E4%BB%B6%E5%A4%B9)
  * [3\.3 修改配置文件](#33-%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
  * [3\.4 配置环境变量](#34-%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
  * [3\.5 分发 kafka](#35-%E5%88%86%E5%8F%91-kafka)
  * [3\.6 配置分发过去机器的环境变量和 broker\.id](#36-%E9%85%8D%E7%BD%AE%E5%88%86%E5%8F%91%E8%BF%87%E5%8E%BB%E6%9C%BA%E5%99%A8%E7%9A%84%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E5%92%8C-brokerid)
  * [3\.7 启动集群](#37-%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4)
  * [3\.8 关闭集群](#38-%E5%85%B3%E9%97%AD%E9%9B%86%E7%BE%A4)

----
# 一、下载
http://kafka.apache.org/downloads

# 二、集群规划
节点  | IP| 服务 
---- | ---- | ----
hadoop1 | 192.168.220.30 | kafka
hadoop2 | 192.168.220.31 | kafka
hadoop3 | 192.168.220.32 | kafka


# 三、集群部署
## 3.1 解压到指定目录
```bash
[root@hadoop1 opt]# tar -zxvf kafka_2.11-2.2.1.tgz -C /usr/bigdata/
```

## 3.2 在 /usr/bigdata/kafka_2.11-2.2.1 目录下创建 logs文件夹
```bash
[root@hadoop1 kafka_2.11-2.2.1]# mkdir logs
```

## 3.3 修改配置文件
```bash
[root@hadoop1 kafka_2.11-2.2.1]# cd config/
[root@hadoop1 config]# vim server.properties

#broker的全局唯一编号，不能重复
broker.id=1
#删除topic功能使能
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘IO的现成数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka运行日志存放的路径	
log.dirs=/usr/bigdata/kafka_2.11-2.2.1/logs
#topic在当前broker上的分区个数
num.partitions=1
#用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment文件保留的最长时间，超时将被删除
log.retention.hours=168
#配置连接Zookeeper集群地址
zookeeper.connect=hadoop1:2181,hadoop2:2181,hadoop3:2181
```

## 3.4 配置环境变量
```bash
[root@hadoop1 kafka_2.11-2.2.1]# vim /etc/profile
KAFKA_HOME=/usr/bigdata/kafka_2.11-2.2.1
PATH=$PATH:$KAFKA_HOME/bin

[root@hadoop1 config]# source /etc/profile
```

## 3.5 分发 kafka
```bash
[root@hadoop1 config]# scp -r /usr/bigdata/kafka_2.11-2.2.1 root@hadoop2://usr/bigdata/
[root@hadoop1 config]# scp -r /usr/bigdata/kafka_2.11-2.2.1 root@hadoop3://usr/bigdata/
```

## 3.6 配置分发过去机器的环境变量和 broker.id
```bash
[root@hadoop2 ~]# vim /etc/profile
KAFKA_HOME=/usr/bigdata/kafka_2.11-2.2.1
PATH=$PATH:$KAFKA_HOME/bin
[root@hadoop2 config]# source /etc/profile

[root@hadoop3 ~]# vim /etc/profile
KAFKA_HOME=/usr/bigdata/kafka_2.11-2.2.1
PATH=$PATH:$KAFKA_HOME/bin
[root@hadoop3 config]# source /etc/profile

[root@hadoop2 ~]# cd $KAFKA_HOME
[root@hadoop2 kafka_2.11-2.2.1]# vim config/server.properties
broker.id=2

[root@hadoop3 ~]# cd $KAFKA_HOME
[root@hadoop3 kafka_2.11-2.2.1]# vim config/server.properties
broker.id=3
```


## 3.7 启动集群
```bash
[root@hadoop1 kafka_2.11-2.2.1]$ bin/kafka-server-start.sh config/server.properties &
[root@hadoop2 kafka_2.11-2.2.1]$ bin/kafka-server-start.sh config/server.properties &
[root@hadoop3 kafka_2.11-2.2.1]$ bin/kafka-server-start.sh config/server.properties &
```

## 3.8 关闭集群
```bash
[root@hadoop1 kafka_2.11-2.2.1]$ bin/kafka-server-stop.sh stop
[root@hadoop2 kafka_2.11-2.2.1]$ bin/kafka-server-stop.sh stop
[root@hadoop3 kafka_2.11-2.2.1]$ bin/kafka-server-stop.sh stop
```


