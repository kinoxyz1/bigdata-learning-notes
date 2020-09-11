

* [一、下载安装包](#%E4%B8%80%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85%E5%8C%85)
* [二、解压、修改配置文件](#%E4%BA%8C%E8%A7%A3%E5%8E%8B%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
* [二、测试](#%E4%BA%8C%E6%B5%8B%E8%AF%95)
  * [2\.1 需求](#21-%E9%9C%80%E6%B1%82)
* [2\.2 安装netcat工具](#22-%E5%AE%89%E8%A3%85netcat%E5%B7%A5%E5%85%B7)
  * [2\.3 创建配置文件](#23-%E5%88%9B%E5%BB%BA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
  * [2\.4 先开启flume监听端口](#24-%E5%85%88%E5%BC%80%E5%90%AFflume%E7%9B%91%E5%90%AC%E7%AB%AF%E5%8F%A3)
  * [2\.5 使用netcat工具向本机的44444端口发送内容](#25-%E4%BD%BF%E7%94%A8netcat%E5%B7%A5%E5%85%B7%E5%90%91%E6%9C%AC%E6%9C%BA%E7%9A%8444444%E7%AB%AF%E5%8F%A3%E5%8F%91%E9%80%81%E5%86%85%E5%AE%B9)
  * [2\.6 在Flume监听页面观察接收数据情况](#26-%E5%9C%A8flume%E7%9B%91%E5%90%AC%E9%A1%B5%E9%9D%A2%E8%A7%82%E5%AF%9F%E6%8E%A5%E6%94%B6%E6%95%B0%E6%8D%AE%E6%83%85%E5%86%B5)

----

# 一、下载安装包
http://flume.apache.org/download.html

# 二、解压、修改配置文件
```bash
[root@hadoop1 opt]# tar -zxvf apache-flume-1.9.0-bin.tar.gz -C /usr/bigdata/
[root@hadoop1 opt]# cd /usr/bigdata/
[root@hadoop1 bigdata]# mv apache-flume-1.9.0-bin/ flume-1.9.0
[root@hadoop1 flume-1.9.0]# mv conf/flume-env.sh.template conf/flume-env.sh
[root@hadoop1 flume-1.9.0]# vim conf/flume-env.sh
export JAVA_HOME=/usr/java/jdk1.8.0_131
```

# 二、测试
## 2.1 需求
使用Flume监听一个端口, 收集该端口数据, 并打印到控制台。

# 2.2 安装netcat工具
```bash
[root@hadoop1 flume-1.9.0]# yum install -y nc
```

## 2.3 创建配置文件
在flume目录下创建job文件夹并进入job文件夹, 在job文件夹下创建Flume Agent配置文件flume-netcat-logger.conf。
```bash
[root@hadoop1 flume-1.9.0]# mkdir job
[root@hadoop1 flume-1.9.0]# cd job
[root@hadoop1 job]# vim flume-netcat-logger.conf

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 1000

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## 2.4 先开启flume监听端口
```bash
[root@hadoop1 flume-1.9.0]# bin/flume-ng agent -c conf/ -n a1 -f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
```

## 2.5 使用netcat工具向本机的44444端口发送内容
```bash
[root@hadoop1 ~]# nc localhost 44444
1112222333444
OK
flume
OK
hallo
OK
```

## 2.6 在Flume监听页面观察接收数据情况
```bash
.....
2020-09-08 11:46:52,538 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:166)] Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
2020-09-08 11:47:07,553 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 31 31 31 32 32 32 32 33 33 33 34 34 34          1112222333444 }
2020-09-08 11:47:16,556 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 66 6C 75 6D 65                                  flume }
2020-09-08 11:47:17,045 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 68 61 6C 6C 6F                                  hallo }
....
```