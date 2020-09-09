


---
# 一、下载 Spark

1．官网地址

http://spark.apache.org/

2．文档查看地址

https://spark.apache.org/docs/2.1.1/

3．下载地址

https://spark.apache.org/downloads.html


# 二、Local 模式
① 解压 安装包
```bash
[root@hadoop1 opt]# tar -zxvf spark-3.0.1-bin-hadoop2.7.tgz -C /usr/bigdata/
```
② 运行官方案例
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.12-3.0.1.jar \
100
```

# 三、Standalone模式
构建一个由Master+Slave构成的Spark集群，Spark运行在集群中。

## 3.1 进入spark安装目录下的conf文件夹
修改配置文件名称
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]# mv slaves.template slaves
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]# mv spark-env.sh.template spark-env.sh
```

## 3.2 修改slave文件，添加work节点
```bash
[root@hadoop1 conf]$ vim slaves
hadoop1
hadoop2
hadoop3
```

## 3.3 修改spark-env.sh文件，添加如下配置
```bash
[root@hadoop1 conf]$ vim spark-env.sh
SPARK_MASTER_HOST=hadoop1
SPARK_MASTER_PORT=7077
```

## 3.4 分发 Spark
```bash
[root@hadoop1 conf]$ scp -r /usr/bigdata/spark-3.0.1-bin-hadoop2.7 root@hadoop2://usr/bigdata
[root@hadoop1 conf]$ scp -r /usr/bigdata/spark-3.0.1-bin-hadoop2.7 root@hadoop3://usr/bigdata
```

##  3.5 启动 Spark
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ sbin/start-all.sh
```
打开浏览器查看 WebUI: hadoop1:8080

注意：如果遇到 “JAVA_HOME not set” 异常，可以在sbin目录下的spark-config.sh 文件中加入如下配置:
```bash
export JAVA_HOME=XXXX
```

## 3.6 运行官方案例
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop1:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.12-3.0.1.jar \
100
```

## 3.7 配置 JobHistoryServer
① 修改spark-default.conf.template名称
```bash
[root@hadoop1 conf]$ mv spark-defaults.conf.template spark-defaults.conf
```
② 修改spark-default.conf文件，开启Log
```bash
[root@hadoop1 conf]$ vi spark-defaults.conf
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://hadoop1:9000/directory
```
注意: HDFS上的目录需要提前存在。

③ 修改spark-env.sh文件，添加如下配置
```bash
[root@hadoop1 conf]$ vi spark-env.sh

export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 
-Dspark.history.retainedApplications=30 
-Dspark.history.fs.logDirectory=hdfs://hadoop1:9000/directory"
```
参数描述：
- spark.eventLog.dir：Application在运行过程中所有的信息均记录在该属性指定的路径下
- spark.history.ui.port=18080  WEBUI访问的端口号为18080
- spark.history.fs.logDirectory=hdfs://hadoop1:9000/directory配置了该属性后，在start-history-server.sh时就无需再显式的指定路径，Spark History Server页面只展示该指定路径下的信息
- spark.history.retainedApplications=30指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

④ 分发配置文件
```bash
[root@hadoop1 conf]$ scp -r spark-defaults.conf /usr/bigdata/spark-3.0.1-bin-hadoop2.7/conf
[root@hadoop1 conf]$ scp -r spark-env.sh /usr/bigdata/spark-3.0.1-bin-hadoop2.7/conf
```

⑤ 再次运行官方案例
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop1:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.12-3.0.1.jar \
100
```

⑥ 查看历史服务

hadoop1:18080



# 四、Yarn模式
## 4.1 修改配置文件
① 修改hadoop配置文件yarn-site.xml,添加如下内容
```bash
[root@hadoop1 hadoop]# vim yarn-site.xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
</property>
```
② 修改spark-env.sh，添加如下配置
```bash
[root@hadoop1 conf]$ vi spark-env.sh

YARN_CONF_DIR=/usr/bigdata/hadoop-3.1.1/etc/hadoop
```

## 4.2 分发配置文件
```bash
[atguigu@hadoop102 conf]$ scp -r /usr/bigdata/hadoop-3.1.1/etc/hadoop/etc/hadoop/yarn-site.xml root@hadoop2://usr/bigdata/hadoop-3.1.1/etc/hadoop/etc/hadoop/
[atguigu@hadoop102 conf]$ scp -r /usr/bigdata/hadoop-3.1.1/etc/hadoop/etc/hadoop/yarn-site.xml root@hadoop3://usr/bigdata/hadoop-3.1.1/etc/hadoop/etc/hadoop/
```

## 4.3 执行一个程序
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.12-3.0.1.jar \
100
```

## 4.4 配置日志
① 修改配置文件spark-defaults.conf, 添加如下内容
```bash
spark.yarn.historyServer.address=hadoop1:18080
spark.history.ui.port=18080
```

② 重启spark历史服务
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ sbin/stop-history-server.sh 
stopping org.apache.spark.deploy.history.HistoryServer
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ sbin/start-history-server.sh 
```

③ 提交任务到Yarn执行
```bash
[root@hadoop1 spark-3.0.1-bin-hadoop2.7]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.12-3.0.1.jar \
100
```

④ Web页面查看日志

hadoop1:8088

hadoop1:18080