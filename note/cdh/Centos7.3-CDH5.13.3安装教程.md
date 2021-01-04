



---
# 一、下载
CM:
[下载1](http://archive.cloudera.com/cm5/cm/5/)
[下载2](http://archive.cloudera.com/cm5/installer/5.13.0/)

CDH:
[下载1](https://archive.cloudera.com/cdh5/parcels/5.13.3/)
[下载2](http://archive.cloudera.com/cdh5/parcels/5.13/)

KAFKA:
[下载1](https://archive.cloudera.com/kafka/parcels/3.1.0/)
[下载2](http://archive.cloudera.com/csds/kafka/)

# 二、安装前准备
## 2.1 配置 hosts
(ps: 所有节点执行)

在台机器上输入: `vim /etc/hosts`

```bash
192.168.1.221 bigdata001
192.168.1.222 bigdata002
192.168.1.223 bigdata003
```

## 2.2 卸载自带的jdk
(ps: 所有节点执行)

```bash
[root@kino-cdh01 ~]# rpm -qa |grep jdk
java-1.8.0-openjdk-headless-1.8.0.222.b03-1.el7.x86_64
java-1.7.0-openjdk-headless-1.7.0.221-2.6.18.1.el7.x86_64
copy-jdk-configs-3.3-10.el7_5.noarch
java-1.8.0-openjdk-1.8.0.222.b03-1.el7.x86_64
java-1.7.0-openjdk-1.7.0.221-2.6.18.1.el7.x86_64

[root@kino-cdh01 ~]# rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.222.b03-1.el7.x86_64
[root@kino-cdh01 ~]# rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.221-2.6.18.1.el7.x86_64
[root@kino-cdh01 ~]# rpm -e --nodeps copy-jdk-configs-3.3-10.el7_5.noarch
[root@kino-cdh01 ~]# rpm -e --nodeps java-1.8.0-openjdk-1.8.0.222.b03-1.el7.x86_64
[root@kino-cdh01 ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.221-2.6.18.1.el7.x86_64

```
## 2.3 卸载自带的 mariadb
(ps: 所有节点执行)

```bash
[root@kino-cdh01 ~]# rpm -qa | grep -i mariadb | xargs rpm -e --nodeps
```
删除残留文件

```bash
[root@kino-cdh01 ~]# find / -name mysql | xargs rm -rf
[root@kino-cdh01 ~]# find / -name my.cnf | xargs rm -rf
[root@kino-cdh01 ~]# cd /var/lib/
[root@kino-cdh01 ~]# rm -rf mysql/
```

## 2.4 关闭防火墙
(ps: 所有节点执行)

```bash
[root@kino-cdh01 ~]# systemctl stop firewalld.service
[root@kino-cdh01 ~]# systemctl disable firewalld.service
```

## 2.5 关闭 SELINUX
三台都关闭SELINUX，编辑/etc/selinux/config配置文件，把SELINUX的值改为disabled

```bash
[root@kino-cdh01 ~]# vim /etc/selinux/config


SELINUX=disabled
```

## 2.5 配置免密登录
(ps: 所有节点执行)

```bash
ssh-keygen -t rsa  # 直接回车

ssh-copy-id kino-cdh01   # 输入 yes, 输入密码 
ssh-copy-id kino-cdh02   # 输入 yes, 输入密码 
ssh-copy-id kino-cdh03   # 输入 yes, 输入密码 
```



## 2.6 安装jdk
(ps: 所有节点都要安装)

```bash
[root@kino-cdh01 ~]# mkdir /usr/java
```
将放在服务器上的 jdk-8u181-linux-x64.tar.gz 解压到 /usr/java 目录下

```bash
[root@kino-cdh01 ~]# tar -zxvf /opt/software/jdk-8u181-linux-x64.tar.gz -C /usr/java/
```
将  /usr/java 分发到其他服务器

```bash
[root@kino-cdh01 ~]# scp -r /usr/java root@bigdata2:/usr/java
[root@kino-cdh01 ~]# scp -r /usr/java root@bigdata3:/usr/java
```
配置 JAVA_HOME 环境变量(所有的主机都需要)
```bash
[root@kino-cdh01 ~]# cat >> /etc/profile << EOF
> #JAVA_HOME
> export JAVA_HOME=/usr/java/jdk1.8.0_181
> export PATH=$PATH:$JAVA_HOME/bin
> EOF
[root@kino-cdh01 ~]# source /etc/profile
[root@kino-cdh01 ~]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

## 2.7 时钟同步
```bash
1、所有机器安装ntp ：yum -y install ntp

2、CM节点配置时钟与自己同步：vim /etc/ntp.conf，删除其他server，加入：
server  127.127.1.0     # local clock
fudge   127.127.1.0 stratum 10

3、其他非CM节点，同步CM节点的时间，vim /etc/ntp.conf，加入：
server xxx.xxx.xxx.xx

4、重启所有机器的ntp服务
systemctl restart ntpd或者service ntpd restart
systemctl status ntpd或者service ntpd status

5、验证同步
所有节点执行ntpq –p，左边出现*号表示同步成功。

6、若不成功;
/usr/sbin/ntpdate stdtime.gov.hk 
ntpdate xxx.xxx.xxx.xxx
手动同步时间
```

## 2.8 http服务
```bash
yum -y install httpd
systemctl start httpd 或service httpd start
```
---
# 三、在线安装 MariaDB
(ps: 主节点执行)
## 3.1 安装 MariaDB
[CentOS7安装MariaDB](../MariaDB/CentOS7安装MariaDB.md)

## 3.2 查看 MariaDB 状态
```bash
[root@hadoop1 html]# systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2020-09-04 10:32:23 CST; 1h 19min ago
 Main PID: 2069 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─2069 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─2243 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock

9月 04 10:32:21 hadoop1 systemd[1]: Starting MariaDB database server...
9月 04 10:32:21 hadoop1 mariadb-prepare-db-dir[2034]: Database MariaDB is probably initialized in /var/lib/mysql already, nothing is done.
9月 04 10:32:21 hadoop1 mariadb-prepare-db-dir[2034]: If this is not the case, make sure the /var/lib/mysql is empty before running mariadb-prepare-db-dir.
9月 04 10:32:21 hadoop1 mysqld_safe[2069]: 200904 10:32:21 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
9月 04 10:32:21 hadoop1 mysqld_safe[2069]: 200904 10:32:21 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
9月 04 10:32:23 hadoop1 systemd[1]: Started MariaDB database server.
```

## 3.3 设置root可以远程登录

```bash
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'MariaDB 密码' WITH GRANT OPTION;

mysql> FLUSH PRIVILEGES;

mysql> exit;
```
## 3.9 设置MySql忽略大小写：
用root登录，打开并修改 `/etc/my.cnf`; 在 `[mysqld]` 节点下，加入一行： `lower_case_table_names=1`

重启MySql服务：`systemctl restart mariadb`

## 3.10 修改MySQL中文乱码
[修改MySQL中文乱码](../../note/MySQL/修改MySQL中文乱码.md)

## 3.11 为 CM 安装mysql驱动
下载地址: https://dev.mysql.com/downloads/connector/j/

将 `mysql-connector-java-5.1.27-bin.jar` 拷贝到 新创建的 `/usr/share/java` 路径下，并重命名为 `mysql-connector-java.jar`

```bash
[root@kino-cdh01 mysql]# tar -zxvf mysql-connector-java-5.1.27.tar.gz

[root@kino-cdh01 mysql]# cp mysql-connector-java-5.1.27-bin.jar /usr/share/java

[root@kino-cdh01 mysql]# cd /usr/share/java

[root@kino-cdh01 java]# mv mysql-connector-java-5.1.27-bin.jar mysql-connector-java.jar

[root@kino-cdh01 java]# ll /usr/share/java
总用量 2216
lrwxrwxrwx. 1 root root      23 2月   4 20:47 icedtea-web.jar -> ../icedtea-web/netx.jar
lrwxrwxrwx. 1 root root      25 2月   4 20:47 icedtea-web-plugin.jar -> ../icedtea-web/plugin.jar
-rw-r--r--. 1 root root   62891 6月  10 2014 jline.jar
-rw-r--r--. 1 root root 1079759 8月   2 2017 js.jar
-rw-r--r--. 1 root root 1007505 5月   9 00:34 mysql-connector-java.jar
-rw-r--r--. 1 root root   18387 8月   2 2017 rhino-examples.jar
lrwxrwxrwx. 1 root root       6 2月   4 20:47 rhino.jar -> js.jar
-rw-r--r--. 1 root root   92284 3月   6 2015 tagsoup.jar
```
将该驱动发到每一台服务器

```bash
[root@kino-cdh01 java]# scp -r /usr/share/java/mysql-connector-java.jar root@kino-cdh02:/usr/share/java/
mysql-connector-java.jar 				100%  984KB  25.3MB/s   00:00    

[root@kino-cdh01 java]# scp -r /usr/share/java/mysql-connector-java.jar root@kino-cdh03:/usr/share/java/
mysql-connector-java.jar     			100%  984KB  30.7MB/s   00:00 
```
---
# 四、安装
## 4.1 解压 & 分发
```bash
mkdir -p /opt/cloudera
tar zxvf /opt/cloudera-manager-centos7-cm5.13.3_x86_64.tar.gz -C /opt/cloudera/

scp -r /opt/cloudera root@bigdata002:/opt
scp -r /opt/cloudera root@bigdata003:/opt
```


## 4.2 创建cloudera-scm用户
所有节点执行
```bash
useradd --system --home=/opt/cloudera/cm-5.13.3/run/cloudera-scm-server/ --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm
```

## 4.3 为Cloudera Manager 建立数据库 并初始化
所有节点执行
```bash
#创建/usr/share/java/ 用于后期cdh启动部署
mkdir -p /usr/share/java/
cp mysql-connector-java.jar /usr/share/java/
cp mysql-connector-java.jar /opt/cloudera/cm-5.13.3/share/cmf/lib/
```

## 4.4 登录Mysql创建数据库，创建用户并授权
```bash
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hive DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```

## 4.5 初始脚本配置数据库scm_prepare_database.sh
主节点执行
```bash
/opt/cloudera/cm-5.13.3/share/cmf/schema/scm_prepare_database.sh mysql scm root Kino123#!
```

## 4.6 配置从节点cloudera-manger-agent指向主节点服务器文件
所有节点执行
```bash
vim /opt/cloudera/cm-5.13.3/etc/cloudera-scm-agent/config.ini
server_host=bigdata001
```

## 4.7上传cdh包
```bash
mkdir -p /opt/cloudera/parcel-repo
chown cloudera-scm:cloudera-scm /opt/cloudera/parcel-repo
mv CDH-5.13.3-1.cdh5.13.3.p0.2-el7.parcel.sha1 CDH-5.13.3-1.cdh5.13.3.p0.2-el7.parcel.sha
```

## 4.8 创建parcels目录
所有节点执行
```bash
mkdir -p /opt/cloudera/parcels
```

# 五、启动CM
## 5.1 启动主节点cloudera-scm-server
主节点执行
```bash
/opt/cloudera/cm-5.13.3/etc/init.d/cloudera-scm-server start
tail -f /opt/cloudera/cm-5.13.3/log/cloudera-scm-server/cloudera-scm-server.log
```
## 5.2 启动cloudera-scm-agent服务
所有节点执行
```bash
/opt/cloudera/cm-5.13.3/etc/init.d/cloudera-scm-agent start
tail -f /opt/cloudera/cm-5.13.3/log/cloudera-scm-agent/cloudera-scm-agent.out
```

## 5.3 登录
http://bigdata001:7180



## 安装 kafka
将 第一步 KAFKA 下载1 的内容 copy 到 parcel-repo

将 第一步 KAFKA 下载2 的内容 copy 到 /opt/cloudera/csd

webui 上 -> 检查新 Parcel
