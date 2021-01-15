



---

# 一、Window 说明
Flink 流式计算是处理无限数据集的引擎, 而Window 则是将这个无限的数据集切分成有限大小的块(buckets/桶) , 从而对这些块(buckets/桶)进行计算操作

# 二、Window 类型
Window 可以将无限的数据集切分成有限大小的块, 按什么类型切分可以分为如下几类:
1. CountWindow: 按照指定的数据条数生成一个 Window, 和时间无关
2. TimeWindow: 按照时间生成 Window
   - 滚动窗口(Tumbling Windows): 将数据依据固定的窗口长度对数据进行切分
     
     **特点:** 时间对齐, 窗口长度固定, 没有重叠
    
     **场景:** 适合 BI 统计等(做每个时间段的聚合计算)

     ![滚动窗口](../../img/flink/window/滚动窗口.png)     

   - 滑动窗口(Sliding Windows): 由固定的窗口长度和滑动间隔组成

     特点: 时间对齐, 窗口长度固定, 可以有重叠
    
     场景: 对最近一个时间段内的统计(最近5分钟的失败率决定报警)

     ![滑动窗口](../../img/flink/window/滑动窗口.png)

   - 会话窗口(Session Windows): 由一系列事件组合一个指定时间长度的 timeout 间隙组成, 类似 web 应用的 session, 也就是一段时间没有接收到新数据就会生成新的窗口
    
     特点: 时间五对其

     ![会话窗口](../../img/flink/window/会话窗口.png)
    

# 三、Window API - TimeWindow(时间窗口)
TimeWindow 是将指定时间范围内的所有数据组成一个window, 一次对一个 window 里面的所有数据进行计算

## 3.1 滚动窗口(Tumbling Windows)
Flink 默认的时间窗口根据 Processing Time 进行窗口划分, 将 Flink 获取到的数据根据进入 Flink 的时间划分到不同的窗口中
```scala
package com.kino.windows

import com.kino.mode.SensorReading
import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer011

import java.util.Properties

/**
 * create by kino on 2021/1/15
 * 滚动窗口(Tumbling Window)
 */
object TumblingWindowTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置并行度
    env.setParallelism(1)

    val props = new Properties()
    props.put("bootstrap.servers","bigdata001:9092")
    props.put("group.id","consumer-group1")
    props.put("zookeeper.connect","bigdata001:2181")
    props.put("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer")
    props.put("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer")
    props.put("auto.offset.reset","earliest")
    // 设置 Kafka Source
    var inputStream = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), props))
    // 将读取到的数据源转换成 样例类
    val dataStream = inputStream.map(x => {
      val splits = x.split(",")
      SensorReading(splits(0).toString, splits(1).toLong, splits(2).toDouble)
    })
    // 将样例类转换成 元祖, 然后做 keyBy, 设置 15s 一个窗口
    val outputSteam: DataStream[(String, Double)] = dataStream
      .map(x => (x.id, x.temperature))
      .keyBy(_._1)
      .timeWindow(Time.seconds(15))
      .reduce((x1, x2) => (x1._1, x1._2.min(x2._2)))

    outputSteam.print()
    env.execute(this.getClass.getName)
  }
}
```