

* [下载 Phoenix](#%E4%B8%8B%E8%BD%BD-phoenix)
* [解压](#%E8%A7%A3%E5%8E%8B)
* [拷贝指定 jar到 HBase 的lib目录](#%E6%8B%B7%E8%B4%9D%E6%8C%87%E5%AE%9A-jar%E5%88%B0-hbase-%E7%9A%84lib%E7%9B%AE%E5%BD%95)
* [在 CDG \- hbase配置中加入相关参数](#%E5%9C%A8-cdg---hbase%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%8A%A0%E5%85%A5%E7%9B%B8%E5%85%B3%E5%8F%82%E6%95%B0)
* [将 hdfs 和 hbase 相关配置文件拷贝到 Phoenix/bin目录下](#%E5%B0%86-hdfs-%E5%92%8C-hbase-%E7%9B%B8%E5%85%B3%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E6%8B%B7%E8%B4%9D%E5%88%B0-phoenixbin%E7%9B%AE%E5%BD%95%E4%B8%8B)
* [重启 Hbase即可连接Phoenix](#%E9%87%8D%E5%90%AF-hbase%E5%8D%B3%E5%8F%AF%E8%BF%9E%E6%8E%A5phoenix)

---

# 下载 Phoenix

http://phoenix.apache.org/download.html

---
# 解压
```bash
[root@bigdata-5 ~]# tar -zxvf apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz 
apache-phoenix-5.0.0-HBase-2.0-bin/
apache-phoenix-5.0.0-HBase-2.0-bin/bin/
apache-phoenix-5.0.0-HBase-2.0-bin/examples/
......
```

---
# 拷贝指定 jar到 HBase 的lib目录
phoenix-core-<version>.jar
htrace-core-3.1.0-incubating.jar
```bash
[root@bigdata-5 phoenix]# cp 上述两个.jar /opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/hbase/lib/
```
---
# 在 CDG - hbase配置中加入相关参数
![在这里插入图片描述](../../img/phoenix/安装phoenix/20200302202254805.png)
![在这里插入图片描述](../../img/phoenix/安装phoenix/20200302202303806.png)

hbase-site.xml 的 HBase 服务高级配置代码段（安全阀）
```xml
phoenix.schema.isNamespaceMappingEnabled
true
命名空间开启

hbase.regionserver.wal.codec
org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec
二级索引
```

hbase-site.xml 的 HBase 客户端高级配置代码段（安全阀）
```xml
phoenix.schema.isNamespaceMappingEnabled
true
```

---
# 将 hdfs 和 hbase 相关配置文件拷贝到 Phoenix/bin目录下

![在这里插入图片描述](../../img/phoenix/安装phoenix/2020030218082641.png)

---
# 重启 Hbase即可连接Phoenix