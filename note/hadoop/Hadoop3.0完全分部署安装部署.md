
* [一、集群规划](#%E4%B8%80%E9%9B%86%E7%BE%A4%E8%A7%84%E5%88%92)
* [二、下载安装包](#%E4%BA%8C%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85%E5%8C%85)
* [三、解压安装、配置](#%E4%B8%89%E8%A7%A3%E5%8E%8B%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE)
  * [3\.1 解压到指定目录](#31-%E8%A7%A3%E5%8E%8B%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9B%AE%E5%BD%95)
  * [3\.2 修改核心配置文件](#32-%E4%BF%AE%E6%94%B9%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
  * [3\.3 Hadoop3\.0 用root用户启动需增加如下配置](#33-hadoop30-%E7%94%A8root%E7%94%A8%E6%88%B7%E5%90%AF%E5%8A%A8%E9%9C%80%E5%A2%9E%E5%8A%A0%E5%A6%82%E4%B8%8B%E9%85%8D%E7%BD%AE)
* [四、分发 hadoop3\.0](#%E5%9B%9B%E5%88%86%E5%8F%91-hadoop30)
* [五、启动 hadoop](#%E4%BA%94%E5%90%AF%E5%8A%A8-hadoop)
  * [5\.2 启动集群](#52-%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4)
  * [5\.3 在 WebUI 中查看](#53-%E5%9C%A8-webui-%E4%B8%AD%E6%9F%A5%E7%9C%8B)

---
# 一、集群规划
节点  | IP| 服务 
---- | ---- | ----
hadoop1 | 192.168.220.30 | namenode、datanode、NodeManager
hadoop2 | 192.168.220.31 | datanode、ResourceManager、NodeManager
hadoop3 | 192.168.220.32 | datanode、SecondaryNameNode、NodeManager


# 二、下载安装包
https://archive.apache.org/dist/hadoop/common/


# 三、解压安装、配置
## 3.1 解压到指定目录
```bash
[root@hadoop1 opt]# tar -zxvf hadoop-3.1.1.tar.gz -C /usr/bigdata/
```
## 3.2 修改核心配置文件
① 配置 `core-site.xml`
```bash
[root@hadoop1 hadoop-3.1.1]# cd /usr/bigdata/hadoop-3.1.1/etc/hadoop/
[root@hadoop1 hadoop]# vim core-site.xml
<!-- 指定HDFS中NameNode的地址 -->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
	<name>hadoop.tmp.dir</name>
	<value>/usr/bigdata/hadoop-3.1.1/data/tmp</value>
</property>
```
② 配置 `hadoop-env.sh`
```bash
[root@hadoop1 hadoop]# vim hadoop-env.sh
export JAVA_HOME=/usr/java/jdk1.8.0_131
```
③ 配置 `hdfs-site.xml`
```bash
[root@hadoop1 hadoop]# vim hdfs-site.xml
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>
<!-- 指定Hadoop辅助名称节点主机配置 -->
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop3:50090</value>
</property>
```
④ 配置 `yarn-env.sh`
```bash
[root@hadoop1 hadoop]# vim yarn-env.sh
export JAVA_HOME=/usr/java/jdk1.8.0_131
```
⑤ 配置 `yarn-site.xml`
```bash
<!-- Reducer获取数据的方式 -->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<!-- 指定YARN的ResourceManager的地址 -->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop1</value>
</property>
```
⑥ 配置 `mapred-env.sh`
```bash
[root@hadoop1 hadoop]# vim mapred-env.sh
export JAVA_HOME=/usr/java/jdk1.8.0_131
```
⑦ 配置 `mapred-site.xml`
```bash
[root@hadoop1 hadoop]# cp mapred-site.xml.template mapred-site.xml

[root@hadoop1 hadoop]# vim mapred-site.xml
<!-- 指定MR运行在Yarn上 -->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```
⑧ 配置 `workers` 
```bash
[root@hadoop1 hadoop]# vim workers
hadoop1
hadoop2
hadoop3
```


## 3.3 Hadoop3.0 用root用户启动需增加如下配置

① 在 start-dfs.sh、stop-dfs.sh 的最上边添加如下配置
```bash
#!/usr/bin/env bash
HDFS_DATANODE_USER=root    
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```
② 在 start-yarn.sh、stop-yarn.sh 的最上边添加如下配置
```bash
#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```
③ 在 start-all.sh、stop-all.sh 的最上边添加如下配置
```bash
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

# 四、分发 hadoop3.0
```bash
[root@hadoop1 conf]$ scp -r /usr/bigdata/hadoop-3.1.1 root@hadoop2://usr/bigdata
[root@hadoop1 conf]$ scp -r /usr/bigdata/hadoop-3.1.1 root@hadoop3://usr/bigdata
```

# 五、启动 hadoop
##5.1 格式化NameNode
如果集群是第一次启动, 需要格式化NameNode(注意格式化之前,一定要先停止上次启动的所有namenode和datanode进程,然后再删除data和log数据）
```bash
[root@hadoop1 hadoop-3.1.1]$ bin/hdfs namenode -format
```

## 5.2 启动集群
```bash
[root@hadoop1 hadoop-3.1.1]$ sbin/start-all.sh
```

## 5.3 在 WebUI 中查看
http://192.168.220.30:9870