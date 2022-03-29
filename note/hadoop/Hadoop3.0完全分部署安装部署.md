

---


# 一、下载安装包
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
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop1:8020</value>
	</property>
	<!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/app/hadoop-3.1.3/data</value>
	</property>

	<!-- 配置HDFS网页登录使用的静态用户为root -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>root</value>
	</property>

	<!-- 配置该 root(superUser)允许通过代理访问的主机节点 -->
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
	</property>
	<!-- 配置该 root(superUser)允许通过代理用户所属组 -->
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
	</property>
	<!-- 配置该root(superUser)允许通过代理的用户-->
    <property>
        <name>hadoop.proxyuser.root.users</name>
        <value>*</value>
	</property>
</configuration>
```
② 配置 `hadoop-env.sh`
```bash
[root@hadoop1 hadoop]# vim hadoop-env.sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_131
```
③ 配置 `hdfs-site.xml`
```bash
[root@hadoop1 hadoop]# vim hdfs-site.xml
<configuration>
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>
<!-- nn web端访问地址-->
<property>
    <name>dfs.namenode.http-address</name>
    <value>hadoop1:9870</value>
</property>
<!-- 2nn web端访问地址-->
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop3:9868</value>
</property>
</configuration>
```
④ 配置 `yarn-env.sh`
```bash
[root@hadoop1 hadoop]# vim yarn-env.sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_131
```
⑤ 配置 `yarn-site.xml`
```bash
<configuration>
<!-- 指定MR走shuffle -->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<!-- 指定ResourceManager的地址-->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop1</value>
</property>
<!-- 环境变量的继承 -->
<property>
    <name>yarn.nodemanager.env-whitelist</name>
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
</property>
<!-- yarn容器允许分配的最大最小内存 -->
<property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>2048</value>
</property>
<property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>4096</value>
</property>

<!-- yarn容器允许管理的物理内存大小 -->
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>8092</value>
</property>

<!-- 关闭yarn对物理内存和虚拟内存的限制检查 -->
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
<!-- 开启日志聚集功能 -->
<property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
</property>
<!-- 设置日志聚集服务器地址 -->
<property>  
        <name>yarn.log.server.url</name>  
        <value>http://hadoop1:19888/jobhistory/logs</value>
</property>
<!-- 设置日志保留时间为7天 -->
<property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
</property>
</configuration>
```
⑥ 配置 `mapred-env.sh`
```bash
[root@hadoop1 hadoop]# vim mapred-env.sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_131
```
⑦ 配置 `mapred-site.xml`
```bash
[root@hadoop1 hadoop]# cp mapred-site.xml.template mapred-site.xml

[root@hadoop1 hadoop]# vim mapred-site.xml
<configuration>
<!-- 指定MR运行在Yarn上 -->
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
<!-- 历史服务器端地址 -->
<property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop1:10020</value>
</property>
<!-- 历史服务器web端地址 -->
<property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop1:19888</value>
</property>
</configuration>
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

## 5.4 启动 historyserver
```bash
./sbin/mr-jobhistory-daemon.sh start historyserver
```