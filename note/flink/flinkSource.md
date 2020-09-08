
* [前言](#%E5%89%8D%E8%A8%80)
* [一、从集合读取数据](#%E4%B8%80%E4%BB%8E%E9%9B%86%E5%90%88%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE)
* [二、从文件读取数据](#%E4%BA%8C%E4%BB%8E%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE)
* [三、从 kafka 读取数据](#%E4%B8%89%E4%BB%8E-kafka-%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE)
* [四、自定义 Source](#%E5%9B%9B%E8%87%AA%E5%AE%9A%E4%B9%89-source)
  * [4\.1 自定义随机产生 SensorReading对象 的 Source](#41-%E8%87%AA%E5%AE%9A%E4%B9%89%E9%9A%8F%E6%9C%BA%E4%BA%A7%E7%94%9F-sensorreading%E5%AF%B9%E8%B1%A1-%E7%9A%84-source)
  * [4\.2 自定义 MySQL Source](#42-%E8%87%AA%E5%AE%9A%E4%B9%89-mysql-source)

---
# 前言
Fink的编程模型中，主要有三块
- Source: 读取数据源
- Transform: 业务逻辑也就是在 Source 和 Sink 中间的操作。
- Sink: 写出结果

本篇主要介绍 Source 的四种情况:
- 从集合读取
- 从文件读取
- 从 Kafka 读取
- 自定义 Source(MySQL Source)

# 一、从集合读取数据
```scala 3
package day02

import Mode.SensorReading
import org.apache.flink.streaming.api.scala._

object readList {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env.fromCollection(List(
      SensorReading("sensor_1", 1547718199, 35.8),
      SensorReading("sensor_6", 1547718201, 15.4),
      SensorReading("sensor_7", 1547718202, 6.7),
      SensorReading("sensor_10", 1547718205, 38.1)
    ))
    stream.print.setParallelism(1)

    env.execute(readList.getClass.getName)
  }
}
```

# 二、从文件读取数据
src\main\resources\SensorReading
```text
sensor_1,1547718199,35.8
sensor_6,1547718201,15.4
sensor_7,1547718202,6.7
sensor_10,1547718205,38.1
```
```scala 3
package day02

import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment

object ReadSource {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val filePathFilter = "D:\\work\\kino\\FlinkTutorial\\src\\main\\resources\\SensorReading"
    val stream = env.readTextFile(filePathFilter)
    stream.print.setParallelism(4)
    env.execute(ReadSource.getClass.getName)
  }
}
```

# 三、从 kafka 读取数据
```scala 3
package day02

import java.util.Properties

import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka._

object ReadKafka {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    val properties = new Properties()
    properties.setProperty("bootstrap.servers", "hadoop1:9092")
    properties.setProperty("group.id", "consumer-group1")
    properties.setProperty("zookeeper.connect", "hadoop1:2181")
    properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
    properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
    properties.setProperty("auto.offset.reset", "earliest")

    val stream = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))

    stream.print()

    env.execute(ReadKafka.getClass.getName)
  }
}
```
启动 Kafka 生产者, 输入
```bash
> sensor_1,1547718199,35.8
> sensor_6,1547718201,15.4
> sensor_7,1547718202,6.7
> sensor_10,1547718205,38.1
```

# 四、自定义 Source
除了以上的 source 数据来源, 还可以自己定义 source. 自定义 source 只需要传入一个 `SourceFunction` 就可以, 具体如下:
## 4.1 自定义随机产生 SensorReading对象 的 Source
```scala 3
package day02

import Mode.SensorReading
import org.apache.flink.streaming.api.datastream.DataStreamSource
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment
import org.apache.flink.streaming.api.functions.source.SourceFunction

import scala.util.Random

object MySource1 {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val value: DataStreamSource[SensorReading] = env.addSource(new MySensorSource())
    value.print()
    env.execute(MySource1.getClass.getName)
  }

  class MySensorSource extends SourceFunction[SensorReading]{

    var running: Boolean = true

    override def run(ctx: SourceFunction.SourceContext[SensorReading]): Unit = {
      //初始化一个随机数发生器
      val random = new Random
      var curTemp = 1.to(10).map(
        i => ("sensor_" + i,65+random.nextGaussian()*20)
      )
      while (running){
        //更新温度值
        curTemp = curTemp.map(
          t => (t._1, t._2 + random.nextGaussian())
        )
        //获取当前时间戳
        val curTime = System.currentTimeMillis()
        curTemp.foreach(
          t => ctx.collect(SensorReading(t._1, curTime, t._2))
        )
        Thread.sleep(100)
      }
    }

    override def cancel(): Unit = {
      running = false
    }
  }
}
```
输出结果:
```scala 3
5> SensorReading(sensor_6,1599205319839,59.92642400729985)
6> SensorReading(sensor_7,1599205319839,66.19691445430901)
1> SensorReading(sensor_2,1599205319839,118.33056052942452)
4> SensorReading(sensor_5,1599205319839,69.19684948320257)
7> SensorReading(sensor_8,1599205319839,61.57564221552262)
2> SensorReading(sensor_3,1599205319839,79.93216211581978)
...
```


## 4.2 自定义 MySQL Source
1. 添加 pom 依赖
    ```xml
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>
    ```
2. 创建mysql表，作为读取的数据源
    ```sql
    CREATE TABLE `person` (
      id int(10) unsigned NOT NULL AUTO_INCREMENT,
      name varchar(260) NOT NULL DEFAULT '' COMMENT '姓名',
      age int(11) unsigned NOT NULL DEFAULT '0' COMMENT '年龄',
      sex tinyint(2) unsigned NOT NULL DEFAULT '2' COMMENT '0:女, 1男',
      email text COMMENT '邮箱',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8 COMMENT='人员定义';
    ```
   插入一些数据，作为数据源的内容：
   ```sql
   insert into person values
      (null, 'Johngo12', 12, 1, 'Source01@flink.com'),
      (null, 'Johngo13', 13, 0, 'Source02@flink.com'),
      (null, 'Johngo14', 14, 0, 'Source03@flink.com'),
      (null, 'Johngo15', 15, 0, 'Source04@flink.com'),
      (null, 'Johngo16', 16, 1, 'Source05@flink.com'),
      (null, 'Johngo17', 17, 1, 'Source06@flink.com'),
      (null, 'Johngo18', 18, 0, 'Source07@flink.com'),
      (null, 'Johngo19', 19, 0, 'Source08@flink.com'),
      (null, 'Johngo20', 20, 1, 'Source09@flink.com'),
      (null, 'Johngo21', 21, 0, 'Source10@flink.com'); 
   ```
3. 创建样例类
    ```scala 3
    case class Person(id:Int,name:String,age:Int,sex:Int,email:String)
    ```
4. 创建自定义Source类，继承 RichSourceFunction
    ```scala 3
    package day02
    
    import java.sql.{Connection, DriverManager, PreparedStatement, ResultSet}
    
    import Mode.{Person, SensorReading}
    import org.apache.flink.configuration.Configuration
    import org.apache.flink.streaming.api.functions.source.{RichSourceFunction, SourceFunction}
    import org.apache.flink.streaming.api.scala._
    
    object MySource2 {
      def main(args: Array[String]): Unit = {
        val env = StreamExecutionEnvironment.getExecutionEnvironment
        val mysqlStream = env.addSource(new MySqlSource)
        mysqlStream.print
        env.execute(MySource2.getClass.getName)
      }
    
      class MySqlSource extends RichSourceFunction[Person]{
    
        var running: Boolean = true
        var ps: PreparedStatement = null
        var conn: Connection = null
    
        /**
         * 与MySQL建立连接信息
         * @return
         */
        def getConnection():Connection = {
          var conn: Connection = null
          val DB_URL: String = "jdbc:mysql://docker:12345/kinodb"
          val USER: String = "root"
          val PASS: String = "123456"
    
          try{
            Class.forName("com.mysql.cj.jdbc.Driver")
            conn = DriverManager.getConnection(DB_URL, USER, PASS)
          } catch {
            case _: Throwable => println("due to the connect error then exit!")
          }
          conn
        }
    
        /**
         * open()方法初始化连接信息
         * @param parameters
         */
        override def open(parameters: Configuration): Unit = {
          super.open(parameters)
          conn = this.getConnection()
          val sql = "select * from kinodb.person"
          ps = this.conn.prepareStatement(sql)
        }
    
    
        override def run(ctx: SourceFunction.SourceContext[Person]): Unit = {
          val resSet:ResultSet = ps.executeQuery()
          while(running & resSet.next()) {
            ctx.collect(Person(
              resSet.getInt("id"),
              resSet.getString("name"),
              resSet.getInt("age"),
              resSet.getInt("sex"),
              resSet.getString("email")
            ))
          }
        }
    
        override def cancel(): Unit = {
          running = false
        }
    
        /**
         * 关闭连接信息
         */
        override def close(): Unit = {
          if(conn != null) {
            conn.close()
    
          }
          if(ps != null) {
            ps.close()
          }
        }
      }
    }
    ```
    输出结果:
    ```scala 3
    2> Person(17,Johngo19,19,0,Source08@flink.com)
    5> Person(12,Johngo14,14,0,Source03@flink.com)
    8> Person(15,Johngo17,17,1,Source06@flink.com)
    6> Person(13,Johngo15,15,0,Source04@flink.com)
    1> Person(16,Johngo18,18,0,Source07@flink.com)
    4> Person(11,Johngo13,13,0,Source02@flink.com)
    3> Person(10,Johngo12,12,1,Source01@flink.com)
    7> Person(14,Johngo16,16,1,Source05@flink.com)
    3> Person(18,Johngo20,20,1,Source09@flink.com)
    4> Person(19,Johngo21,21,0,Source10@flink.com)
    ```