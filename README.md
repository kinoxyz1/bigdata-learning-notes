# bigdata-learning-notes

# 大数据学习笔记

## 大数据

<details>
<summary>Zookeeper</summary>

* [Zookeeper 实现分布式锁](note/zookeeper/zookeeper实现分布式锁.md)
</details>

<!--分界线-->

<details>
<summary>Hadoop</summary>

* [Hadoop 基准测试](note/hadoop/Hadoop基准测试.md)
* [Hadoop数据迁移](note/hadoop/Hadoop数据迁移.md)
</details>

<!--分界线-->

<details>
<summary>Hive</summary>

* [Hive beeline连接](note/hive/Hive-beeline连接.md)
* [Hive 导出 csv 文件](note/hive/Hive导出csv文件.md)
* [Hive drop database删除数据库](note/hive/Hive-Drop-Database删除数据库.md)
* [Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient](note/hive/Hive异常1.md)
* [Hive DDL 数据定义](note/hive/Hive-DDL数据定义.md)
* [Hive 查询](note/hive/Hive查询.md)
</details>

<!--分界线-->

<details>
<summary>Kafka</summary>

* [kafka stop 脚本有时候不起作用的原因](note/kafka/kafka-stop脚本有时候不起作用的原因.md)
</details>

<!--分界线-->

<details>
<summary>Sentry</summary>

* [CDH安全之Sentry权限管理](note/sentry/CDH安全之Sentry权限管理.md)
* [hue: There are currently no roles defined](note/sentry/sentry异常1.md)
</details>

<!--分界线-->

<details>
<summary>Phoenix</summary>

* [CDH 平台安装 Apache Phoenix](note/phoenix/CDH平台安装Apache-Phoenix.md)
</details>

<!--分界线-->

<details>
<summary>CDH</summary>

* [Centos7.7 CDH6.2.1 安装教程](note/cdh/Centos7.7-CDH6.2.1安装教程.md)
* [CDH 安装 Hue 连接不上MySQL](note/cdh/CDH安装Hue连接不上MySQL.md)
* [centos 克隆后要做的操作](note/cdh/centos克隆后要做的操作.md)
* [CDH 查询 Hive执行过的SQL语句](note/cdh/CDH查询Hive执行过的SQL语句.md)
</details>

<!--分界线-->

<details>
<summary>Scala</summary>

* [scala 模式匹配](note/scala/scala模式匹配match.md)
* [scala 隐式转换](note/scala/scala隐式转换.md)
* [scala 的排序](note/scala/scala的排序.md)
* [scala 集合的 map 映射、高阶函数使用、集合的压平、 集合的过滤、集合的简化、集合的折叠、集合的扫描、集合的拉链、集合的迭代器、集合的分组](note/scala/scala集合的map映射等.md)
* [scala API](note/scala/scala-API.md)
* [scala 数组](note/scala/scala数组.md)
* [scala 给类取别名](note/scala/scala给类取别名.md)
* [scala 的 trait](note/scala/scala的trait.md)
* [scala 包声明和包导入](note/scala/scala包声明和包导入.md)
* [scala类和对象](note/scala/scala类和对象.md)
* [scala 值调用、名调用和控制抽象](note/scala/scala值调用、名调用和控制抽象.md)
* [scala 闭包和柯里化](note/scala/scala闭包和柯里化.md)
* [scala 高阶函数(高阶算子)](note/scala/scala高阶函数(高阶算子).md)
* [scala 流程控制](note/scala/scala流程控制.md)
* [scala 变量和数据类型](note/scala/scala变量和数据类型.md)
* [scala 部分应用函数与偏函数](note/scala/scala部分应用函数与偏函数.md)
</details>

<!--分界线-->

<details>
<summary>Spark</summary>

* [Spark 常用 API](note/spark/Spark常用API.md)
* [Hive on Spark 参数调优](note/spark/HiveOnSpark参数调优.md)
* [Spark Streaming 的 reduceByKeyAndWindow 窗口函数](note/spark/SparkStreaming的reduceByKeyAndWindow窗口函数.md)
* [Spark 任务停止后自动重启](note/spark/Spark任务停止后自动重启.md)
* [Spark源码之-CDH6下Spark2.4写Hive分区表异常](note/spark/Spark源码之-CDH6下Spark2.4写Hive分区表异常.md)
* Spark 内核
  * [Spark 内核概述]
  * [Spark Shuffle解析](note/spark/spark-memory/SparkShuffle解析.md)
  * [Spark 内存管理](note/spark/spark-memory/Spark内存管理.md)

* Spark 性能优化和故障处理
  * [Spark 性能优化](note/spark/spark-performance/Spark性能优化.md)

</details>

<!--分界线-->

<details>
<summary>Canal</summary>

* [使用 canal 实时监控 mysql 并读取到 Kafka(scala 版)](note/canal/使用canal实时监控mysql并读取到Kafka-scala版.md)
</details>

<!--分界线-->

---

## 大数据运维
<details>
<summary>Zabbix</summary>
  
* [Centos7.7 安装 Zabbix](note/zabbix/Centos7.7安装Zabbix.md)
  * 编译源码安装zabbix4.4
    * [Centos7.7 编译源码安装使用 Zabbix(zabbix-server)](note/zabbix/Centos7.7编译源码安装使用Zabbix(zabbix-server).md)
    * [Centos7.7 编译源码安装使用 Zabbix(zabbix-agent)](note/zabbix/Centos7.7编译源码安装使用Zabbix(zabbix-agent).md)
  * 二进制文件安装使用 Zabbix5.0
    * [Centos7.7 二进制文件安装使用 Zabbix5.0(zabbix-server)](note/zabbix/Centos7.7二进制文件安装使用Zabbix5.0(zabbix-server).md)
    * [Centos7.7 二进制文件安装使用 Zabbix5.0(zabbix-agent)](note/zabbix/Centos7.7二进制文件安装使用Zabbix5.0(zabbix-agent).md)
* [Zabbix5.0 中文乱码](note/zabbix/Zabbix5.0中文乱码.md)
* [Zabbix: 添加被监控主机、创建主机、监控项、触发器、图形和模板](note/zabbix/Zabbix添加被监控主机、创建主机、监控项、触发器、图形和模板.md)
* [Zabbix: 自定义邮件告警](note/zabbix/Zabbix自定义邮件告警.md)
* [CentOS7安装 docker](note/docker/CentOS7安装docker.md)
  </details>

<!--分界线-->

---

## Linux 相关
<details>
<summary>Linux</summary>
  
* Linux 用户管理
  * [vi和vim的使用](note/linux/Linux用户管理/vi和vim的使用.md)
  * [开机、重启和用户登录注销](note/linux/Linux用户管理/开机、重启和用户登录注销.md)
  * [用户管理](note/linux/Linux用户管理/用户管理.md)


* [Linux 集群分发脚本](note/linux/Linux集群分发脚本.md)
* [Linux下卸载 MySQL](note/linux/Linux下卸载MySQL.md)
* [Linux Swap分区](note/linux/Linux-Swap分区.md)
* [Linux扩展/删除swap分区](note/linux/Linux扩展-删除swap分区.md)
* [kill pid 和 kill -9 pid 的区别](note/linux/kill-pid.md)
* [This account is currently not available（用户当前不可用）](note/linux/用户当前不可用.md)
  </details>  

<!--分界线-->

---

## Centos 相关
<details>
<summary>Centos</summary>
  
* [Centos7系统更换yum源镜像为国内镜像](note/centos/Centos7系统更换yum源镜像为国内镜像.md)
  </details>  