

* [前言](#%E5%89%8D%E8%A8%80)
* [一、函数类(Function Classes)](#%E4%B8%80%E5%87%BD%E6%95%B0%E7%B1%BBfunction-classes)
* [二、匿名函数](#%E4%BA%8C%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0)
* [三、富函数](#%E4%B8%89%E5%AF%8C%E5%87%BD%E6%95%B0)



---
# 前言
该篇介绍 Flink 实现 UDF函数 -- 更细粒度的控制流

# 一、函数类(Function Classes)
Flink 暴露了酥油的 udf 函数的接口(实现方式为接口或抽象类), 例如 MapFunction、FilterFunction、ProcessFunction等等.

下面列举一个 FilterFunction 接口的例子:

```scala 3
package day03

import Mode.SensorReading
import org.apache.flink.api.common.functions.FilterFunction
import org.apache.flink.streaming.api.scala._

object UDF {
    def main(args: Array[String]): Unit = {
        import Mode.SensorReading
        val env = StreamExecutionEnvironment.getExecutionEnvironment
        val inputDS = env.readTextFile("D:\\work\\kino\\FlinkTutorial\\src\\main\\resources\\SensorReading")
            .map(x => {
                val splitArrays = x.split(",")
                SensorReading(splitArrays(0).toString,splitArrays(1).toLong,splitArrays(2).toDouble)
            })
        //使用自定义的 Filter 函数
        val filterDS = inputDS.filter(new MyFilterFunction("30"))
        filterDS.print
        env.execute(this.getClass.getName)
    }
}

class MyFilterFunction(condition: String) extends FilterFunction[SensorReading]{
    override def filter(value: SensorReading): Boolean = {
        //将传感器温度小于30°的过滤掉
        value.temperature > condition.toInt
    }
}
```
SensorReading 文件的内容如下:
```text
sensor_1,1547718199,35.8
sensor_6,1547718201,15.4
sensor_7,1547718202,6.7
sensor_10,1547718205,38.1
```
输出的结果:
```scala 3
1> SensorReading(sensor_1,1547718199,35.8)
6> SensorReading(sensor_10,1547718205,38.1)
```


可以点进去 FilterFunction, FilterFunction 继承 Function, 按 ctrl+alt+b 查看 Function 的所有实现类

![Function](../../img/flink/UDF/Function.png)


# 二、匿名函数
```scala 3
val filterDS = inputDS.filter(_.temperature > 30)
```


# 三、富函数
"富函数" 是 DataStream API 提供的一个函数类的接口, 所有 Flink 函数类都有其 Rich 版本. 

它与常规函数的不同在于, 可以获取运行环境的上下文, 并拥有一些生命周期方法, 所以可以实现更复杂的功能.

- RichMapFunction
- RichFlatMapFunction
- RichFilterFunction
- …

Rich Function 有一个生命周期的概念。典型的生命周期方法有:
- open() 方法是 rich function 的初始化方法, 当一个算子例如 map 或者 filter 被调用之前 open()会被调用.
- close() 方法是生命周期中的最后一个调用的方法, 做一些清理工作.
- getRuntimeContext() 方法提供了函数的 RuntimeContext 的一些信息, 例如函数执行的并行度, 任务的名字, 以及 state 状态

```scala 3
package day03

import Mode.SensorReading
import org.apache.flink.api.common.functions.RichFlatMapFunction
import org.apache.flink.streaming.api.scala._
import org.apache.flink.util.Collector
import org.apache.flink.configuration.Configuration

object MyFlatMapRichFunctionTest {
    def main(args: Array[String]): Unit = {
        val env = StreamExecutionEnvironment.getExecutionEnvironment
        val inputDS = env.readTextFile("D:\\work\\kino\\FlinkTutorial\\src\\main\\resources\\SensorReading")

        //使用自定义的 Filter 函数
        inputDS.flatMap(new MyFlatMapRichFunction()).print()
        env.execute(this.getClass.getName)
    }
}

class MyFlatMapRichFunction extends RichFlatMapFunction[String, SensorReading] {
    var startTime: Long = _

    override def open(parameters: Configuration): Unit = {
        startTime = System.currentTimeMillis()
    }

    override def flatMap(value: String, out: Collector[SensorReading]) = {
        val splitArrays = value.split(",")
        out.collect(SensorReading(s"$startTime-"+splitArrays(0).toString+s"-${System.currentTimeMillis()}", splitArrays(1).toLong, splitArrays(2).toDouble))
    }

    override def close(): Unit = {
        //做一些清理工作, 比如关闭连接池
    }
}
```
输出的结果:
```scala 3
7> SensorReading(1599820160812-sensor_10-1599820161038,1547718205,38.1)
2> SensorReading(1599820160812-sensor_1-1599820161036,1547718199,35.8)
3> SensorReading(1599820160812-sensor_6-1599820161037,1547718201,15.4)
5> SensorReading(1599820160812-sensor_7-1599820161036,1547718202,6.7)
```