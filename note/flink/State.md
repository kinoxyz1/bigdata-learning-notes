



---
# 一、Flink State 相关概念
Flink 可以处理 **有界流** 和 **无界流**

有界流的典型代表如 MapReduce 和 Spark, 通常情况下, MapReduce 用于做离线计算, 比如计算昨天的销售总额, Spark 可以用来做 离线计算 和 实时计算(SparkStreaming), 只不过 Spark 的 实时计算 实际上是微批处理, 例如计算近一小时的销售总额。

MapReduce 和 Spark 所能处理的情况实际上都是一批数据, 例如 01:00 - 02:00 的数据, 也就是说, MapReduce 和 Spark 默认在 02:00 的时候, 所有 02:00 之前的数据都被搜集到了, 但是实际生产中, 计算一天销售总额总是让人头疼的, 因为在 24:00 的时候并不能保证数据全部都到了, 可能因为网络等原因, 造成 数据在第二天的 0点 0几分才入库, 那么计算当天的销售总额就不是一个可参考的有效结果。

因为 Spark 实时计算是微批处理, 所以 MapReduce 和 Spark 并不能保证实时性, 而 Flink 不仅可以进行像 Spark 那样的离线批处理, 还可以进行 流处理(即收集到一条数据处理一条), 所以 Flink 可以保证结果的时效性。

Flink 之所以比 MapReduce 和 Spark 优秀, 大部分取决于 Flink 的 State。 因为 Flink 可以对 **无界流** 进行处理(即收集到一条数据处理一条), 那么 Flink 在计算像 Spark 那样最近一小时的销售总额, 就是来一条数据计算一条, 下一条的数据 和 上一次结果进行累加, 对于这每一条数据进来产生的结果, 就是一个 State, Flink 可以通过 **状态后端** 对 State 进行存储 和 维护。

**状态后端** 在 Flink1.12 中, 可以有三种选择:  MemoryStateBacked、FsStateBacked、RocksDBStateBacked, 这个在下面会详细说到

当 Flink 程序因为特殊原因崩溃 或者 升级Flink 的时候, 再下一次重新启动 Flink 程序, 就可以通过 State 进行状态恢复, 要想确保恢复的正确性, 前提就是做到 State 的一致性, Flink 的容错机制就实际上是状态一致性的管理。

State 一致性就是像上面说的, 当 Flink 程序故障, 下次恢复的正确程度，也称为一致性级别(完全准确还是数据丢失还是重复计算), 关于一致性级别, 一共有几种情况:
1. at-most-once(最多一次): 数据有可能被丢失造成漏消费
2. at-last-once(最少一次): 数据有有可能重复造成重复计算, 但是一定不会造成数据丢失
3. exactly-once(精准一次): 数据不多不少刚刚好

说到这个 State 一致性, 这个是说了状态的几种情况, Flink 可以通过 addSource() 进行数据的读取, 也可以通过 addSink() 进行数据的写出, 即: 我们不仅需要考虑 Flink Source 的State一致性, 也需要考虑到 Flink Sink 的State一致性, 那这个 State 的一致性, 就得考虑这种 **端到端的State一致性**

**端到端的State一致性**, 往简单想的实现思路大概可以是, Flink 从 Queue 读取一条数据, 进行 Transform处理, Sink 写出, 写出后 Source 停止读取, 把 State 存储起来, 再开始处理下一条数据, 这种实现方式弊端不少, 在 Flink 内部是如何实现这种 端到端的精确一致性的?

![Flink端到端状态一致性](../../img/flink/Flink%20State/Flink%20端到端状态一致性.png)

上图就是 Flink 的处理大致过程, Flink 会进行如下的 State 存储

![Flink端到端状态一致性存储](../../img/flink/Flink%20State/Flink%20端到端状态一致性存储.png)

当 Flink 程序故障时, 就可以进行如下的 State 恢复

![Flink端到端状态一致性恢复](../../img/flink/Flink%20State/Flink%20端到端状态一致性恢复.png)

大概的顺序如下

![Flink端到端状态一致性过程](../../img/flink/Flink%20State/Flink%20端到端状态一致性过程.png)

因为 Flink 有了 State, 所以就能天然的保证 **端到端的 State 精确一致性**(exactly-once), 如上就是 Flink 的 容错机制了。

Flink 提供了 CheckPoint(检查点) 机制, CheckPoint 是 Flink 可靠性的基石, 它可以保证 Flink 因为某些原因故障, 能够恢复故障之前的某个状态, 简而言之, CheckPoint 是和 State 相辅相成的, 在 Flink 程序中, 可以设置 间隔多久进行一次 CheckPoint 生成 State 的 Snapshot。

# 二、Flink State 的分类
Flink 有两种基本类型的状态: Managed State & Raw State

|  |Managed State | Raw State 
----|----|----
| 状态管理方式 | Flink Runtime 托管, 自动存储, 自动恢复, 自动伸缩 | 用户自己管理
| 状态数据结构 | Flink 提供多种常用的数据结构, ListState/MapState 等 | 字节数组: byte[] |
| 使用场景 | 绝大多数 Flink 算子 | 所有算子 | 

# 三、Managed State 的分类
一共分为两类:
1. Keyed State(键控状态)
2. Operator State(算子状态)

| | Operator State | KeyedState |
|----|----|----|
|适用算子类型| 可用于所有算子: 常用于Source, 例如: FlinkKafkaConsumer| 只适用于课也得Stream上的算子|
|状态分配|一个算子的子任务对应一个状态| 一个Key对应一个 State: 一个算子会处理多个Key, 则访问相应的多个State|
|创建和访问方式|实现 CheckPointedFunction | 重写 RichFunction, 通过里面的 RuntimeContext访问|
|横向扩展| 并发改变时有多重重写分配方式可选: 均匀分配和合并后每个得到全量 | 并发改变, State随着 Key在实例间迁移 |
|支持的数据结构| ListState 和 BroadCastState | ValueState、ListState、MapState、ReduceState、AggregatingState |


# 四、Operator State 的使用
Operator State 的实际应用场景不如 keyed State 多, 它经常被用在 Source 或者 Sink 等算子上, 用来保存流入数据的偏移量 或 对输出数据做缓存, 以保证 Flink 应用的 Exactly-Once 语义

## 4.1 ListState
```java
package com.kino.flink.state.operator;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.runtime.state.FunctionInitializationContext;
import org.apache.flink.runtime.state.FunctionSnapshotContext;
import org.apache.flink.runtime.state.hashmap.HashMapStateBackend;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.checkpoint.CheckpointedFunction;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.concurrent.TimeUnit;

public class ListStateTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStateBackend(new HashMapStateBackend());
        // env.setStateBackend(new MemoryStateBackend());
        CheckpointConfig config = env.getCheckpointConfig();
        config.setCheckpointStorage("file:///checkpoint-dir");
        // 任务流取消和故障时会保留Checkpoint数据，以便根据实际需要恢复到指定的Checkpoint
        config.enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
        // 设置checkpoint的周期, 每隔1000 ms进行启动一个检查点
        config.setCheckpointInterval(1000);
        // 设置模式为exactly-once
        config.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        // 确保检查点之间有至少500 ms的间隔【checkpoint最小间隔】
        config.setMinPauseBetweenCheckpoints(500);
        // 检查点必须在一分钟内完成，或者被丢弃【checkpoint的超时时间】
        config.setCheckpointTimeout(6000);
        // 同一时间只允许进行一个检查点
        config.setMaxConcurrentCheckpoints(1);
        config.enableUnalignedCheckpoints();
        // 指定自动重启策略
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.of(0, TimeUnit.SECONDS)));

        env.socketTextStream("localhost", 9999)
                .map(new MyListState())
                .print();

        env.execute();
    }

    static class MyListState implements MapFunction<String, Long>, CheckpointedFunction {

        private Long count = 0L;
        private ListState<Long> listState;

        @Override
        public Long map(String value) throws Exception {
            Long aa = 10 / Long.parseLong(value);
            count++;
            return count;
        }

        /**
         * 做 CheckPoint 时被调用, 需要实现具体的 snapshot 逻辑, 执行 CheckPoint 保存哪些状态
         *
         * @param context
         * @throws Exception
         */
        @Override
        public void snapshotState(FunctionSnapshotContext context) throws Exception {
            System.out.println("MyListState.snapshotState, listState: " + count);
            listState.clear();
            listState.add(count);
        }

        /**
         * 初始化时被调用, 向 本地ListState 中填充数据, 每个子任务调用一次
         *
         * @param context
         * @throws Exception
         */
        @Override
        public void initializeState(FunctionInitializationContext context) throws Exception {
            System.out.println("MyListState.initializeState");

            ListStateDescriptor<Long> stateDescriptor = new ListStateDescriptor<Long>("state", Long.class);

            this.listState = context.getOperatorStateStore().getListState(stateDescriptor);

            if(context.isRestored()){
                for (Long aLong : listState.get()) {
                    this.count += aLong;
                    System.out.println("---> "+aLong);
                }
                listState.clear();
            }
        }
    }
}
```

## 4.2 BroadCastState 
```java
package com.kino.flink.state.operator;

import org.apache.flink.api.common.state.BroadcastState;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.common.state.ReadOnlyBroadcastState;
import org.apache.flink.streaming.api.datastream.BroadcastStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.BroadcastProcessFunction;
import org.apache.flink.util.Collector;

public class BroadcastStateTest extends BroadcastProcessFunction<String, String, String> {
    private static MapStateDescriptor<String, String> mapStateDescriptor;

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<String> dataStreamSource = env.socketTextStream("localhost", 9999);
        DataStreamSource<String> controlStreamSource = env.socketTextStream("localhost", 8888);

        mapStateDescriptor = new MapStateDescriptor<>("state", String.class, String.class);

        BroadcastStream<String> broadcastStream = controlStreamSource
                .broadcast(mapStateDescriptor);

        dataStreamSource.connect(broadcastStream)
                .process(new BroadcastStateTest())
                .print();
        env.execute();
    }

    @Override
    public void processElement(String value, ReadOnlyContext ctx, Collector<String> out) throws Exception {
        ReadOnlyBroadcastState<String, String> broadcastState = ctx.getBroadcastState(mapStateDescriptor);
        System.out.println(broadcastState.get("switch"));
        out.collect(broadcastState.get("switch"));
    }

    @Override
    public void processBroadcastElement(String value, Context ctx, Collector<String> out) throws Exception {
        BroadcastState<String, String> broadcastState = ctx.getBroadcastState(mapStateDescriptor);
        // 把值放入广播状态
        System.out.println(value);
        broadcastState.put("switch", value);
    }
}
```

# 五、Keyed State 的使用
Keyed State 是根据出入数据流中定义的 Key 来维护和访问的, Keyed State 只能用于 KeyedStream(keyBy 算子之后)

Flink 为每个键值维护一个状态实例, 并将具有相同键的所有数据, 都分区到一个算子任务中, 这个任务会维护和处理这个 Key 对应的状态。 当任务处理一条数据时, 它会自动将状态的访问范围限定为当前数据的 Key。因此, 具有相同 Key 的所有数据都会访问相同的状态.

## 5.1 ValueState
```java
package com.kino.flink.state.keyed;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;


public class ValueStateTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new AdLog(split[0], split[1], split[2], Double.parseDouble(split[3]));
                })
                .keyBy(AdLog -> (AdLog.getUuid()) + AdLog.getItemCode())
                .process(new MyValueState())
                .print();

        env.execute();
    }

    static class MyValueState extends KeyedProcessFunction<String, AdLog, String> {

        private ValueState<String> valueState;

        @Override
        public void open(Configuration parameters) throws Exception {
            valueState = getRuntimeContext().getState(new ValueStateDescriptor<String>("valueState", String.class));
        }

        @Override
        public void processElement(AdLog value, Context ctx, Collector<String> out) throws Exception {
            if (value.getBehavior().equals("add") && valueState.value() == null) {
                valueState.update(value.getItemCode());
                out.collect("用户:" + value.getUuid() + " 加购了: " + value.getItemCode() + "商品");
            } else if (value.getBehavior().equals("pay") && valueState.value() != null) {
                valueState.clear();
                out.collect("用户:" + value.getUuid() + " 支付了: " + value.getItemCode() + "商品");
            } else if (value.getBehavior().equals("del") && valueState.value() != null) {
                valueState.clear();
                out.collect("用户:" + value.getUuid() + " 取消了: " + value.getItemCode() + "商品");
            }
        }
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class AdLog {
    private String uuid;     // 用户唯一标识
    private String itemCode; // 商品code
    private String behavior; // 用户行为  add: 加购, pay: 支付, del: 删除
    private Double price;    // 价格
}
```

## 5.2 ListState
```java
package com.kino.flink.state.keyed;

import org.apache.commons.lang3.StringUtils;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;

import java.util.ArrayList;

public class ListStateTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new AdLog(split[0], split[1], split[2], Double.parseDouble(split[3]));
                })
                .keyBy(AdLog -> (AdLog.getUuid() + AdLog.getItemCode()))
                .process(new KeyedProcessFunction<String, AdLog, String>() {
                    private ListState<String> listState;

                    @Override
                    public void open(Configuration parameters) throws Exception {
                        listState = getRuntimeContext()
                                .getListState(new ListStateDescriptor<String>("listState", String.class));
                    }

                    // 求连续三次加购的用户和商品
                    @Override
                    public void processElement(AdLog value, Context ctx, Collector<String> out) throws Exception {
                        if (value.getBehavior().trim().equals("add")) {
                            listState.add(value.getItemCode());
                            ArrayList<String> strings = new ArrayList<>();
                            for (String s : listState.get()) {
                                strings.add(s);
                            }
                            out.collect(StringUtils.join(strings, ","));
                        } else {
                            listState.clear();
                        }
                    }
                })
                .print();
        env.execute();
    }
}
```
## 5.3 ReducingState
```java
package com.kino.flink.state.keyed;

import org.apache.flink.api.common.state.ReducingState;
import org.apache.flink.api.common.state.ReducingStateDescriptor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;

public class ReducingStateTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new AdLog(split[0], split[1], split[2], Double.parseDouble(split[3]));
                })
                .keyBy(AdLog -> (AdLog.getUuid() + AdLog.getItemCode()))
                .process(new KeyedProcessFunction<String, AdLog, String>() {
                    private ReducingState<Double> reducingState;

                    @Override
                    public void open(Configuration parameters) throws Exception {
                        reducingState = getRuntimeContext()
                                .getReducingState(
                                        new ReducingStateDescriptor<Double>(
                                                "reducingState",
                                                // new ReduceFunction<Double>() {
                                                //     @Override
                                                //     public Double reduce(Double value1, Double value2) throws Exception {
                                                //         return value1 + value2;
                                                //     }
                                                // },
                                                Double:: sum,   // 这两种写法都行
                                                Double.class));
                    }

                    // 计算每个用户支付事件的总金额
                    @Override
                    public void processElement(AdLog value, Context ctx, Collector<String> out) throws Exception {
                        if(value.getBehavior().equals("pay")){
                            reducingState.add(value.getPrice());
                            out.collect(reducingState.get().toString());
                        }
                    }
                })
                .print();

        env.execute();
    }
}
```
## 5.4 AggregationState
```java
package com.kino.flink.state.keyed;

import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.state.AggregatingState;
import org.apache.flink.api.common.state.AggregatingStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;

public class AggregatingStateTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new AdLog(split[0], split[1], split[2], Double.parseDouble(split[3]));
                })
                .keyBy(AdLog::getUuid)
                .process(new KeyedProcessFunction<String, AdLog, Double>() {
                    private AggregatingState<Integer, Double> aggregatingState;

                    @Override
                    public void open(Configuration parameters) throws Exception {
                        aggregatingState = getRuntimeContext().getAggregatingState(
                                new AggregatingStateDescriptor<Integer, Tuple2<Integer, Integer>, Double>(
                                        "aggregatingState",
                                        new AggregateFunction<Integer, Tuple2<Integer, Integer>, Double>() {
                                            @Override
                                            public Tuple2<Integer, Integer> createAccumulator() {
                                                return Tuple2.of(0, 1);
                                            }

                                            @Override
                                            public Tuple2<Integer, Integer> add(Integer value, Tuple2<Integer, Integer> accumulator) {
                                                return Tuple2.of(value + accumulator.f0, accumulator.f1 + 1);
                                            }

                                            @Override
                                            public Double getResult(Tuple2<Integer, Integer> accumulator) {
                                                return accumulator.f0 * 1D / accumulator.f1;
                                            }

                                            @Override
                                            public Tuple2<Integer, Integer> merge(Tuple2<Integer, Integer> a, Tuple2<Integer, Integer> b) {
                                                return Tuple2.of(a.f0 + b.f0, a.f1 + b.f1);
                                            }
                                        },
                                        Types.TUPLE(Types.DOUBLE, Types.DOUBLE)));
                    }

                    @Override
                    public void processElement(AdLog value, Context ctx, Collector<Double> out) throws Exception {
                        aggregatingState.add(Integer.parseInt(value.getPrice().toString()));
                        out.collect(aggregatingState.get());
                    }
                });
        env.execute();
    }
}
```
## 5.5 MapState
```java
package com.kino.flink.state.keyed;

import org.apache.flink.api.common.state.MapState;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;

public class MapStateTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new AdLog(split[0], split[1], split[2], Double.parseDouble(split[3]));
                })
                .keyBy(AdLog::getUuid)
                .process(new KeyedProcessFunction<String, AdLog, String>() {
                    //使用 MapState 实现数据去重
                    private MapState<String, AdLog> mapState;

                    @Override
                    public void open(Configuration parameters) throws Exception {
                        mapState = getRuntimeContext().getMapState(
                                new MapStateDescriptor<String, AdLog>("mapState", String.class, AdLog.class));
                    }

                    @Override
                    public void processElement(AdLog value, Context ctx, Collector<String> out) throws Exception {
                        if(mapState.get(value.getUuid()) != null){
                            mapState.put(value.getUuid(), value);
                            out.collect(value.getUuid());
                        }
                    }
                });
        env.execute();
    }
}
```

# 六、State Backend
在开头说过, State backend(状态后端) 是用来对 State 进行 存储、访问、维护的, Flink 每接收到一条数据, 有状态的算子任务都会读取和更新状态, 由于有效的状态访问对于处理数据的低延迟至关重要, 因此每个并行任务(子任务)都会在本地维护其状态, 以确保快速的访问状态.

状态后端的功能:
1. 本地的状态管理
2. 将 CheckPoint 写入远程存储


## 6.1 State Backend 分类
1. MemoryStateBackend
    - State 存储在 JobManager 的内存中
    - 具有 快速、低延迟、不稳定 的特点
    - 使用场景: 本地测试, JobManager不容易挂(很难保证), 不推荐生产中使用
2. FsStateBackend
    - 本地状态存储在 JobManager 内存中, CheckPoint 存储在文件系统中
    - 访问快, 具有内存级的访问速度 以及 更好的容错
    - 使用场景: Window、Join等, 可以在生产中使用
3. RocksDBStateBacked
    - State 存储在 RocksDB 数据库中
    - 需要 序列化 和 反序列化
    - 使用场景: 适用于超大状态的作业(one day 的 Window), 对读写性能要求不要的作业, 可以用于生产环境

## 6.2 State Backend 的 配置
### 6.2.1 全局
修改 flink-conf.yaml 文件
```yaml
# The backend that will be used to store operator state checkpoints if
# checkpointing is enabled.
#
# Supported backends are 'jobmanager', 'filesystem', 'rocksdb', or the
# <class-name-of-factory>.
#
# 打开此处的 注释, 可配置项如上
# state.backend: filesystem
```


### 6.2.2 代码
在上面 Operator State 的 List State 示例中, 有配置 State Backend 为 FsStateBackend 的代码, 需要加入依赖:
```pom
<!-- https://mvnrepository.com/artifact/org.apache.flink/flink-shaded-hadoop-2-uber -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-shaded-hadoop-2-uber</artifactId>
    <version>2.7.5-9.0</version>
    <scope>provided</scope>
</dependency>
```

如果要使用 RocksDBBackend, 则需要引入依赖:
```pom
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-statebackend-rocksdb_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
    <scope>provided</scope>
</dependency>

env.setStateBackend(new RocksDBStateBackend("hdfs://hadoop102:8020/flink/checkpoints/rocksdb"));
```

# 七、Flink 的容错机制
## 7.1 状态一致性的级别
在流处理中, 状态的一致性可以分为三个级别:
1. at-most-once(最多一次)
   当 作业故障之后, 计算的结果可能造成数据丢失;
2. at-last-once(最少一次)
   当 作业故障后, 数据有有可能重复造成**重复计算**, 但是一定**不会造成数据丢失**
3. exactly-once(精准一次)
   当 作业故障回复后, 得到的结果即不多不少刚刚是正确的
   
其实在分布式系统中, 最难的两个问题就是:
1. Exactly Once Message processing
2. 保证消息的顺序处理

https://zhuanlan.zhihu.com/p/77677075

因此在曾经很长一段时间内, at-last-once(最少一次)非常流行, 例如: Apache Storm, 因为 保证 exactly-once 的系统实现起来非常复杂。就例如我们上面说的, 保证exactly-once, 我们简单的实现就是一批数据在处理的时候, source 停止接受数据, 等待这一批数据被所有算子处理完成, 再让 source 开始接受数据, 这样就导致延迟很高, 如果使程序具有幂等性来保证 exactly-once, 实现起来非常复杂。

**Flink 之所以比其他 流处理 引擎优秀的原因, 一个是 Flink 可以基于 EventTime 的有状态计算, 另一个就是 Flink 既保证了 exactly-once, 又保证了 低延迟和高吞吐 的能力**

## 7.2 端到端的状态一致性
在 [Flink 官网](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/learn-flink/overview/#stream-processing) 中有说到：
>In Flink, applications are composed of streaming dataflows that may be transformed by user-defined operators. These dataflows form directed graphs that start with one or more sources, and end in one or more sinks.

![Flink 组成](../../img/flink/Flink%20State/Flink%20组成.png)

因为一个 Flink 程序由  Source/Transaction/Sink 三部分组成, 所以端到端的一致性, 最关切的就是这三部分谁的一致性最难保证(木桶原理)。

Flink 的 Source 段可以从外部源读取数据, 要想保证 Source 端的一致性, 就需要外部源可以重设数据的读取位置(需要能重复的重指定位置读), Flink 流式处理程序的 Source 大多都是 Kafka Source, Kafka 可以在读取的时候指定 offset。

Flink 的 Transaction（用户业务逻辑部分）, 有 State 和 CheckPoint 机制保证其一致性。

Flink 的 Sink 在作业故障恢复时, 数据不会重复写到外部系统, 目前 Sink 有两种实现方式:
1. 幂等写入(Idempotent)
   幂等操作就是说, 可以重复执行很多次程序, 但是指挥导致一次结果的更改, 更改后后面的重复操作就不生效了；
2. 事务性写入(Transactional)
   这种方式需要写 事务 来写入外部系统, 事务和CheckPoint绑定, 等 CheckPoint 真正完成的时候, 才把所有对应的记过写入到 Sink 系统中, 对于这样的事务性写入, 又有两种实现方式:
   - 预写日志(WAL)
   - 两阶段提交(2PC)

   ![Flink exactly-once](../../img/flink/Flink%20State/Flink%20exactly-once.png)



## 7.3 CheckPoint 
还是我们上面说的, 一个 Flink 程序要想保证一致性, 在做 CheckPoint 的时候, 简单的办法就是暂停应用, 处理完成 再做 CheckPoint, 完成后再恢复应用。

[再 Flink 中 CheckPoint 的 机制原理来自"Chandy-Lamport algorithm"算法(分布式快照算)的一种变体: 异步 barrier 快照（asynchronous barrier snapshotting）](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/learn-flink/fault_tolerance/#how-does-state-snapshotting-work)

当检查点协调器（作业管理器的一部分）指示任务管理器开始检查点时，它会让所有源记录它们的偏移量并将编号的 checkpoint barriers 插入到它们的流中。这些 barriers 流经作业图，指示每个检查点前后的流部分(意思是说, barriers 会像 watermark 那样, 一直向下游传递), 在 Flink 中, 同一时间可以有多个不同快照的 barriers, 这意味着, 可以并发的出现不同的快照




## 7.4 SavePoint




## 7.5 CheckPoint 和 SavePoint 的区别



## 7.6 故障恢复(手动和自动)