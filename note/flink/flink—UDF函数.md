





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
        val filterDS = inputDS.filter(new MyFilterFunction())
        filterDS.print
        env.execute(this.getClass.getName)
    }
}

class MyFilterFunction() extends FilterFunction[SensorReading]{
    override def filter(value: SensorReading): Boolean = {
        //将传感器温度小于30°的过滤掉
        value.temperature > 30
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


# 二、匿名函数



# 三、富函数