




--- 
# 一、下载
[官方网址](https://dolphinscheduler.apache.org/zh-cn/)

[官方下载地址](https://mirrors.tuna.tsinghua.edu.cn/apache/incubator/dolphinscheduler/1.3.3/apache-dolphinscheduler-incubating-1.3.3-dolphinscheduler-bin.tar.gz)

# 二、安装
## 2.1 解压
```bash
$ tar -zxvf apache-dolphinscheduler-incubating-1.3.3-dolphinscheduler-bin.tar.gz -C /usr/bigdata/
$ cd /usr/bigdata/
$ mv apache-dolphinscheduler-incubating-1.3.3-dolphinscheduler-bin/ dolphinscheduler-incubating-1.3.3
```

## 2.2 创建用户
所有部署 dolphinscheduler 的机器上都要执行
```bash
$ useradd dolphinscheduler
$ echo "dolphinscheduler123" | passwd --stdin dolphinscheduler
$ echo 'dolphinscheduler  ALL=(ALL)  NOPASSWD: NOPASSWD: ALL' >> /etc/sudoers
$ sed -i 's/Defaults    requirett/#Defaults    requirett/g' /etc/sudoers
```

## 2.3 配置 hosts, 配置 ssh 免密登录
所有机器执行
```bash
$ vim /etc/hosts
192.168.220.51 hadoop1
192.168.220.52 hadoop2
192.168.220.53 hadoop3

$ ssh-keygen -t rsa
$ ssh-copy-id hadoop1
$ ssh-copy-id hadoop2
$ ssh-copy-id hadoop3
```

## 2.4 初始化数据库 
dolphinscheduler 默认使用的是 PostgreSQL, 可以使用 MySQL, 只需添加对应的驱动包到 dolphinscheduler 的 lib 目录下即可
```bash
$ cp /usr/bigdata/hive-3.1.2-bin/lib/mysql-connector-java-5.1.48.jar /usr/bigdata/dolphinscheduler-incubating-1.3.3/lib/
```
进入数据库创建 dolphinscheduler 所需数据库
```bash
$ mysql -uroot -pKino123.
mysql> CREATE DATABASE dolphinscheduler DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
mysql> GRANT ALL PRIVILEGES ON dolphinscheduler.* TO 'dolphinscheduler'@'%' IDENTIFIED BY 'Kino123.';
mysql> GRANT ALL PRIVILEGES ON dolphinscheduler.* TO 'dolphinscheduler'@'localhost' IDENTIFIED BY 'Kino123.';
mysql> flush privileges;
```

## 2.5 创建表并且导入基础数据
```bash
$ cd /usr/bigdata/dolphinscheduler-incubating-1.3.3/conf/
$ vim datasource.properties
```
将postgresql的配置注释，并添加 Mariadb 地址
```bash
# postgresql
#spring.datasource.driver-class-name=org.postgresql.Driver
#spring.datasource.url=jdbc:postgresql://localhost:5432/dolphinscheduler
#spring.datasource.username=test
#spring.datasource.password=test

# Mariadb
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://hadoop2:3306/dolphinscheduler?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true 
spring.datasource.username=dolphinscheduler
spring.datasource.password=Kino123.
```
执行导入脚本, 执行之前确保有 JAVA_HOME 
```bash
$ echo $JAVA_HOME
/usr/java/jdk1.8.0_202
$ sh ./script/create-dolphinscheduler.sh
...
11:56:08.523 [main] INFO org.apache.dolphinscheduler.dao.upgrade.shell.CreateDolphinScheduler - upgrade DolphinScheduler finished
11:56:08.523 [main] INFO org.apache.dolphinscheduler.dao.upgrade.shell.CreateDolphinScheduler - create DolphinScheduler success
```

## 2.6 修改运行参数
### ① 环境变量
```bash
$ vim conf/env/dolphinscheduler_env.sh 
export HADOOP_HOME=/usr/bigdata/hadoop-3.2.1
export HADOOP_CONF_DIR=/usr/bigdata/hadoop-3.2.1/etc/hadoop
#export SPARK_HOME1=/opt/soft/spark1
#export SPARK_HOME2=/opt/soft/spark2
#export PYTHON_HOME=/opt/soft/python
export JAVA_HOME=/usr/java/jdk1.8.0_202
export HIVE_HOME=/usr/bigdata/hive-3.1.2-bin
#export FLINK_HOME=/opt/soft/flink
#export DATAX_HOME=/opt/soft/datax/bin/datax.py

export PATH=$HADOOP_HOME/bin:$SPARK_HOME1/bin:$SPARK_HOME2/bin:$PYTHON_HOME:$JAVA_HOME/bin:$HIVE_HOME/bin:$PATH:$FLINK_HOME/bin:$DATAX_HOME:$PATH
```

### ② 将jdk软链到/usr/bin/java下
```bash
$ ln -s /usr/java/jdk1.8.0_202/bin/java /usr/bin/java
```

### ③ 修改 conf/config/install_config.conf各参数
```bash
$ vim conf/config/install_config.conf
dbtype="mysql"
dbhost="hadoop1:3306"
username="dolphinscheduler"
dbname="dolphinscheduler"
password="123"
zkQuorum="hadoop1:2181,hadoop2:2181,hadoop3:2181"

# ds安装目录 不同于/usr/bigdata/dolphinscheduler-incubating-1.3.3
installPath="/usr/bigdata/dolphinscheduler-incubating-1.3.3/ds"

deployUser="dolphinscheduler"
mailServerHost="smtp.qq.com"
mailServerPort="25"

# sender,配置了和 mailUser 一样就行
mailSender="416595168@qq.com"

# user
mailUser="416595168@qq.com"

#邮箱密码
mailPassword="Rm416595168.."

#starttlsEnable和sslEnable不能同时为true
starttlsEnable="true"
sslEnable="false"
sslTrust="smtp.qq.com"

resourceStorageType="HDFS"
defaultFS="hdfs://hadoop1:8020"

#resourcemanager HA对应的地址
yarnHaIps="hadoop1"

#因为使用了resourcemaanger Ha所以保持默认,如果是单resourcemanager配置对应ip
singleYarnIp="yarnIp1"

#资源上传根路径，支持hdfs和s3
resourceUploadPath="/data/dolphinscheduler"

hdfsRootUser="hdfs"

#需要部署ds的机器
ips="hadoop1,hadoop2,hadoop3"
sshPort="22"

#指定master
masters="hadoop101"

#指定workers，并且可以指定组名，default为默认组名
workers="hadoop102:default,hadoop103:default"

#报警服务器地址
alertServer="hadoop103"

#后台api服务器地址
apiServers="hadoop102"
```

### ④ 待定参数
如果需要将资源上传到HDFS功能，并且开启了NAMENODE HA则需要将配置文件复制到 /usr/bigdata/dolphinscheduler-incubating-1.3.3/conf下。非NAMENODE HA则可以忽略
```bash
$ cp /usr/bigdata/hadoop-3.2.1/etc/hadoop/core-site.xml /usr/bigdata/dolphinscheduler-incubating-1.3.3/conf/
$ cp /usr/bigdata/hadoop-3.2.1/etc/hadoop/hdfs-site.xml /usr/bigdata/dolphinscheduler-incubating-1.3.3/conf/
```

### ⑤ 一键部署
切换用户, 执行脚本
```bash
$ chown -R dolphinscheduler:dolphinscheduler dolphinscheduler-incubating-1.3.3
$ su dolphinscheduler
$ cd dolphinscheduler-incubating-1.3.3/
$ pwd
/usr/bigdata/dolphinscheduler-incubating-1.3.3
```
因为 alert 服务需要访问mysql，所以 hadoop2 上也需要mysql驱动包
```bash
$ scp -r /usr/bigdata/hive-3.1.2-bin/lib/mysql-connector-java-5.1.48 root@hadoop2:/usr/share/java
```
一键部署
```bash
$ sh install.sh
```
脚本完成后，会启动一下5个服务
```bash
[root@hadoop1 dolphinscheduler-incubating-1.3.3]# xcall.sh jps
================current host is hadoop1=================
--> excute command "jps"
2532 ResourceManager
2085 DataNode
4261 MasterServer
4486 Jps
1655 QuorumPeerMain
2695 NodeManager
4347 LoggerServer
1919 NameNode
4303 WorkerServer
================current host is hadoop2=================
--> excute command "jps"
3440 MasterServer
3584 ApiApplicationServer
1586 NodeManager
3490 WorkerServer
1351 QuorumPeerMain
1463 DataNode
3533 LoggerServer
3773 Jps
================current host is hadoop3=================
--> excute command "jps"
1680 NodeManager
1362 QuorumPeerMain
2802 WorkerServer
1556 SecondaryNameNode
2968 Jps
2889 AlertServer
1466 DataNode
2844 LoggerServer
excute successfully !
```
- MasterServer ----- master服务
- WorkerServer ----- worker服务
- LoggerServer ----- logger服务
- ApiApplicationServer ----- api服务
- AlertServer ----- alert服务

### ⑥ 访问
http://hadoop1:12345/dolphinscheduler     端口在 conf/config/install_config.conf 中设置

