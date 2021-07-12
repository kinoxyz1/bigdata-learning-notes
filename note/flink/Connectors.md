



---
Connectors 是数据进出 Flink 的一套接口实现, 可以实现 Flink 与各个数据库(存储系统)的连接

当然, 数据进出 Flink 不仅仅局限于 Connectors, 对应的还有:
1. Async I/O: [异步访问外部数据存储的异步 I/O API](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/connectors/datastream/overview/#%e5%bc%82%e6%ad%a5-io)
2. Queryable State: [当多读少些时, 外部应用程序从 Flink 拉取需要的数据, 而不是 Flink 把大量的数据推入外部系统](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/connectors/datastream/overview/#%e5%8f%af%e6%9f%a5%e8%af%a2%e7%8a%b6%e6%80%81)


# 一、Source
Flink 可以从很多不同的数据源来获取数据, 将获取到的数据交由 Flink 进行对应的 ETL 处理, Flink 获取数据的来源称之为数据源(Source)
## 1.1 从Java集合服务数据
导入依赖:
```xml
<properties>
    <flink.version>1.12.0</flink.version>
    <java.version>1.8</java.version>
    <scala.binary.version>2.11</scala.binary.version>
    <slf4j.version>1.7.30</slf4j.version>
</properties>

<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.16</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-java</artifactId>
    <version>${flink.version}</version>
</dependency>
```
准备User类:
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
class User {
    String uuid;
    String name;
    String sex;
    Integer age;
}
```
示例:
```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import java.util.Arrays;
import java.util.List;

public class ListSource {
    public static void main(String[] args) throws Exception {
        // 批处理环境
        // ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        // 流处理环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // 设置并行度
        env.setParallelism(1);

        List<User> userList = Arrays.asList(
                new User("uuid_001", "001", "男", 20),
                new User("uuid_002", "002", "女", 21),
                new User("uuid_003", "003", "女", 10),
                new User("uuid_004", "004", "男", 25)
        );
        env.fromCollection(userList).print("list source");
        env.execute();
    }
}
```
输出结果:
```java
list source> User(uuid=uuid_001, name=001, sex=男, age=20)
list source> User(uuid=uuid_002, name=002, sex=女, age=21)
list source> User(uuid=uuid_003, name=003, sex=女, age=10)
list source> User(uuid=uuid_004, name=004, sex=男, age=25)
```

## 1.2 从文件中读取数据
准备数据文件(创建 input.txt), 内容如下:
```text
"uuid_001", "001", "男", 20
"uuid_002", "002", "女", 21
"uuid_003", "003", "女", 10
"uuid_004", "004", "男", 25
```
示例:
```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class FileSource {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.readTextFile("src/main/resources/input.txt").print("file source");
        env.execute();
    }
}
```
输出结果:
```text
file source> "uuid_001", "001", "男", 20
file source> "uuid_002", "002", "女", 21
file source> "uuid_003", "003", "女", 10
file source> "uuid_004", "004", "男", 25
```
说明:
1. 参数可以是目录也可以是文件
2. 路径可以是相对路径也可以是绝对路径
3. 相对路径是从系统属性user.dir获取路径: idea下是project的根目录, standalone模式下是集群节点根目录
4. 也可以从hdfs目录下读取, 使用路径:hdfs://...., 由于Flink没有提供hadoop相关依赖, 需要pom中添加相关依赖:
```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>3.1.3</version>
    <scope>provided</scope>
</dependency>
```

## 1.3 从socket读取数据
示例:
```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class SocketSource {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        
        env.socketTextStream("localhost", 9999).print("socket source");
        env.execute();
    }
}
```

## 1.4 从kafka读取数据
添加依赖:
```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.11</artifactId>
    <version>1.12.0</version>
</dependency>
```
示例:
```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import java.util.Properties;

public class KafkaSource {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "flink_source_kafka_01");
        properties.setProperty("auto.offset.reset", "latest");
        env
           .addSource(new FlinkKafkaConsumer<String>("flink", new SimpleStringSchema(), properties))
           .print("kafka source");
        env.execute();
    }
}
```

## 1.4 自定义source
大多数情况下, 前面几种方式已经满足需要, 如果在特殊情况下, flink 还能提供自定义数据源的方式

flink 自定义数据源需要实现 `SourceFunction`, 具体示例如下:

```java
package com.kino.flink.d01;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.io.*;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class CustomSource {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.addSource(new MySource("localhost", 9999)).print("custom source");
        env.execute();
    }
}

class MySource implements SourceFunction<User> {

    private String host;
    private Integer port;
    private volatile Boolean isRunning = true;
    private Socket socket;

    public MySource() {}

    public MySource(String host, Integer port) {
        this.host = host;
        this.port = port;
    }

    @Override
    public void run(SourceContext<User> ctx) throws Exception {
        // 1. 和服务器创建连接
        Socket socket = new Socket(host, port);
        // 2. 发送的信息
        InputStream os = socket.getInputStream();
        InputStreamReader isr = new InputStreamReader(os, StandardCharsets.UTF_8);
        BufferedReader read = new BufferedReader(isr);
        String line = null;
        while (isRunning && (line = read.readLine()) != null) {
            String[] split = line.split(",");
            ctx.collect(new User(split[0],split[1],split[2],Integer.parseInt(split[3].toString())));
        }
    }

    @Override
    public void cancel() {
        isRunning = false;
        try {
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

# 二、Transform
Transform 是 Flink 中进行算子转换的, 转换算子可以把一个或者多个 DataStream 转成一个或多个 DataStream。

[Flink1.3 所有算子](https://ci.apache.org/projects/flink/flink-docs-release-1.13/zh/docs/dev/datastream/operators/overview/#%e7%ae%97%e5%ad%90)

说明: 在 Flink1.3 中

## 2.1 map
`DataStream` -> `DataStream`

取一个元素并产生一个元素。将输入流的值加倍的映射函数:

示例一: lambda 表达式
```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class MapTransform1 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env
          .fromElements(1,2,3,4,5)
          .map(line -> line * 2)
          .print("MapTransform1");

        env.execute();
    }
}
```

示例二: 重写 MapFunction
```java
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class MapTransform2 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .map(new MyMapFunction())
           .print("MapTransform2");

        env.execute();
    }

    private static class MyMapFunction implements MapFunction<Integer, Integer> {
        @Override
        public Integer map(Integer value) throws Exception {
            return value * 2;
        }
    }
}
```

示例三: 重写 RichMapFunction
```java
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class MapTransform3 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .map(new MyRichMapFunction())
           .print("MapTransform3");

        env.execute();
    }

    /**
     * Flink 所有的函数都有 Rich 版本
     * Rich Function 和常规 函数的不同在于, Rich Function 可以获取运行上下文
     *    拥有一些生命周期方法, 可以实现更复杂的功能
     */
    private static class MyRichMapFunction extends RichMapFunction<Integer, Integer> {
        RuntimeContext runtimeContext = null;
        @Override
        public void open(Configuration parameters) throws Exception {
            runtimeContext = this.getRuntimeContext();
            System.out.println("MyRichMapFunction.open");
        }

        @Override
        public Integer map(Integer value) throws Exception {
            System.out.println("task name: "+runtimeContext.getTaskName());
            return value * 2;
        }

        @Override
        public void close() throws Exception {
            System.out.println("MyRichMapFunction.close");
        }
    }
}
```
输出结果:
```text
MyRichMapFunction.open
task name: Source: Collection Source -> Map -> Sink: Print to Std. Out
MapTransform3> 2
task name: Source: Collection Source -> Map -> Sink: Print to Std. Out
MapTransform3> 4
task name: Source: Collection Source -> Map -> Sink: Print to Std. Out
MapTransform3> 6
task name: Source: Collection Source -> Map -> Sink: Print to Std. Out
MapTransform3> 8
task name: Source: Collection Source -> Map -> Sink: Print to Std. Out
MapTransform3> 10
MyRichMapFunction.close
```
说明:
1. 默认生命周期方法, 初始化方法, 在每个并行度上只会被调用一次, 而且先被调用
2. 默认生命周期方法, 最后一个方法, 做一些清理工作, 在每个并行度上只调用一次, 而且是最后被调用
3. getRuntimeContext() 方法提供了函数的 RuntimeContext 的一些信息, 如并行度、任务名、state状态等。

## 2.2 flatMap
`DataStream `-> `DataStream`

取一个元素并产生零个、一个或多个元素。将句子拆分为单词的 flatmap 函数:

示例一: lambda 表达式:
```java
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class FlatMapTransform1 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .flatMap((Integer  value, Collector<Integer > out) -> {
               out.collect(value * value);
               out.collect(value * value * value);
           }).returns(Types.INT)
           .print("FlatMapTransform1");

        env.execute();
    }
}
```
示例二: 重写 FlatMapFunction
```java
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class FlatMapTransform2 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .flatMap(new MyFlatMapFunction())
           .print(FlatMapTransform2.class.getSimpleName());

        env.execute();
    }

    private static class MyFlatMapFunction implements FlatMapFunction<Integer, String> {
        @Override
        public void flatMap(Integer value, Collector<String> collector) throws Exception {
            collector.collect(String.valueOf(value * value));
            collector.collect(String.valueOf(value * value * value));
        }
    }
}
```

## 2.3 filter
`DataStream` -> `DataStream`

为每个元素计算一个布尔函数，并**保留那些函数返回 true 的元素**。过滤掉零值的过滤器:

示例一: lambda 表达式:
```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class FilterTransform1 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .filter(line -> (line > 3))
           .print(FilterTransform1.class.getSimpleName());

        env.execute();
    }
}
```
示例二: 重写 FilterFunction
```java
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class FilterTransform2 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .filter(new MyFilterFunction())
           .print(FilterTransform2.class.getSimpleName());

        env.execute();
    }

    static class MyFilterFunction implements FilterFunction<Integer> {
        @Override
        public boolean filter(Integer value) throws Exception {
            return value > 3;
        }
    }
}
```

## 2.4 KeyBy
`DataStream` -> `KeyedStream`

在逻辑上将流划分为不相交的分区。所有具有**相同键的记录都分配到同一个分区**。在内部，**keyBy()是通过哈希分区实现的**。

注意: 什么值不可以作为KeySelector的Key:
- 没有覆写hashCode方法的POJO, 而是依赖Object的hashCode. 因为这样分组没有任何的意义: 每个元素都会得到一个独立无二的组.  实际情况是:可以运行, 但是分的组没有意义.
- 任何类型的数组

示例一: lambda 表达式:
```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class KeyByTransform1 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .keyBy(line -> (line % 2 == 0 ? "even" : "odd"))
           .print(KeyByTransform1.class.getSimpleName());

        env.execute();
    }
}
```

示例二: 实现 KeySelector:
```java
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class KeyByTransform2 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .keyBy(new MyKeyByFunction())
           .print(KeyByTransform2.class.getSimpleName());

        env.execute();
    }

    private static class MyKeyByFunction implements KeySelector<Integer, String> {
        @Override
        public String getKey(Integer integer) throws Exception {
            return integer % 2 == 0 ? "even" : "odd";
        }
    }
}
```

## 2.5 shuffle
`DataStream` -> `DataStream`

把流中的元素随机打乱. 对同一个组数据, 每次只需得到的结果都不同.

```java
public class ShuffleTransform {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.fromElements(1,2,3,4,5)
           .shuffle()
           .print(ShuffleTransform.class.getSimpleName());

        env.execute();
    }
}
```

## 2.6 connect
连接两个流, 两个流被 connect 之后, 只是被放在了同一个流里面, 内部任然保持各自的数据和形式不发生任何变化, 两个流式互相独立的。

`DataStream[A]`, `DataStream[B]` -> `DataStream[A,B]`

示例:
```java
import org.apache.flink.streaming.api.datastream.ConnectedStreams;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class ConnectTransform {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Integer> ds1 = env.fromElements(1, 2, 3, 4, 5);
        DataStreamSource<Integer> ds2 = env.fromElements(6, 7, 8, 9, 10);

        ConnectedStreams<Integer, Integer> connect = ds1.connect(ds2);
        connect.getFirstInput().print("first");
        connect.getSecondInput().print("second");

        env.execute();
    }
}
```
注意:
1. 两个流中存储的数据类型可以不同
2. 只是机械的合并在一起, 内部仍然是分离的2个流
3. 只能2个流进行connect, 不能有第3个参与

## 2.7 union
对两个或者两个以上的DataStream进行union操作，产生一个包含所有DataStream元素的新DataStream

```java
public class UnionTransform {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Integer> ds1 = env.fromElements(1, 2, 3, 4, 5);
        DataStreamSource<Integer> ds2 = env.fromElements(6, 7, 8, 9, 10, 11, 12);
        DataStreamSource<Integer> ds3 = env.fromElements(13, 14, 15, 16, 17, 18, 19, 20, 21);

        ds1
           .union(ds2)
           .union(ds3)
           .print(UnionTransform.class.getSimpleName());

        env.execute();
    }
}
```
connect与 union 区别:
1. union之前两个流的类型必须是一样，connect可以不一样
2. connect只能操作两个流，union可以操作多个。


## 2.8 sum/min/max/minBy/maxBy
KeyedStream的每一个支流做聚合。执行完成后，会将聚合的结果合成一个流返回，所以结果都是DataStream

说明:
1. 如果流中存储的是POJO或者scala的样例类, 参数使用字段名
2. 如果流中存储的是元组, 参数就是位置(基于0...)

`KeyedStream` -> `SingleOutputStreamOperator`

示例:
```java
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironme;

public class OtherTransform {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        KeyedStream<Integer, String> keyedStream = env.fromElements(1, 2, 3, 4, 5)
           .keyBy(line -> (line % 2 == 0 ? "even" : "odd"));

        keyedStream.sum(0).print("sum");
        keyedStream.min(0).print("min");
        keyedStream.max(0).print("max");
        // minBy 和 maxBy 可以指定出现相同值的时候, 其他字段是否取第一个, true: 取第一个, false: 取最后一个
        keyedStream.maxBy(0, false).print("maxBy");
        keyedStream.minBy(0, true).print("minBy");

        env.execute();
    }
}
```

## 2.9 reduce
一个 KeyedStream 的聚合操作, 合并当前的元素和上次聚合的结果, 产生一个新的值, 返回的流中包含每一次聚合的结果, 而不是只返回最后一次聚合的最终结果

`KeyedStream` -> `SingleOutputStreamOperator`

示例一:
```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import java.util.ArrayList;

public class ReduceTransform1 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        ArrayList<User> users = new ArrayList<>();
        users.add(new User("abcdefg", 12345.11, 24));
        users.add(new User("ffffaa", 14321.12, 24));
        users.add(new User("fewaff", 5312.67, 23));
        users.add(new User("htehtff", 54365.73, 23));
        env.fromCollection(users)
           .keyBy(User::getAge)
           .reduce((user, t1) -> {
               return new User(user.getUuid(), user.getSalary() + t1.getSalary(), user.getAge() + t1.getAge());
           })
           .print();

        env.execute();
    }
}
```

示例二:
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import java.util.ArrayList;

public class ReduceTransform {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        ArrayList<User> users = new ArrayList<>();
        users.add(new User("abcdefg", 12345.11, 24));
        users.add(new User("ffffaa", 14321.12, 24));
        users.add(new User("fewaff", 5312.67, 23));
        users.add(new User("htehtff", 54365.73, 23));
        KeyedStream<User, Integer> keyedStream = env.fromCollection(users)
           .keyBy(User::getAge);

        keyedStream.reduce(new ReduceFunction<User>() {
            @Override
            public User reduce(User user, User t1) throws Exception {
                return new User(user.getUuid(), user.getSalary() + t1.getSalary(), user.getAge() + t1.getAge());
            }
        }).print();
        env.execute();
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class User {
    String uuid;
    Double salary;
    int age;
}
```



# 三、Sink