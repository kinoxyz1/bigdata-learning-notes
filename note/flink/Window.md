





---
# 一、Window 概述
在流处理应用用, 我们处理的数据往往都是无穷无尽的, 通常情况下我们不可能等到所有数据到齐后再计算, Flink 可以接收到一条数据就处理一条, 但是在很多场景下, 我们需要计算一些聚合类的操作, 例如: 计算最近一小时的销售总额、计算最近一小时用户活跃量等等, 此时我们就需要用到 Window 将无穷无尽的流切割成有限的快, 从而得到业务需要的指标结果。

# 二、Window 分类
## 2.1 时间驱动的 Window
可以分为:
1. 滚动窗口(Tumbling Window): 窗口长度固定, 窗口之间不重叠, 每个事件仅属于一个窗口;
2. 滑动窗口(Sliding Window): 窗口长度固定, 窗口之间有重叠, 一个事件可能属于多个窗口;
3. 会话窗口(Session Window): 窗口长度固定, 窗口之间不重叠, 每个事件仅属于一个窗口;
4. 全局窗口(Global Window): 窗口不会被自动触发, 需要用户写 定时器 触发计算;

### 2.1.1 滚动窗口(Tumbling Window)
![滚动窗口](../../img/flink/window/滚动窗口.png)

```java
package com.kino.flink.windows.time;

import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

public class TumblingWindowTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new Tuple2<String, Integer>(split[0], Integer.parseInt(split[1]));
                })
                .keyBy(k -> k.f0)
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                // 1. ReduceFunction
                // .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                //     @Override
                //     public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                //         return new Tuple2<String, Integer>(value1.f0, value1.f1 + value2.f1);
                //     }
                // })
                // 2. AggregateFunction, 三个泛型, 1: 输入 2: 中间结果 3: 输出结果
                // .aggregate(new AggregateFunction<Tuple2<String, Integer>, Integer, String>() {
                //     // 初始化累加器
                //     @Override
                //     public Integer createAccumulator() {
                //         return 0;
                //     }
                //
                //     // 累加逻辑
                //     @Override
                //     public Integer add(Tuple2<String, Integer> value, Integer accumulator) {
                //         return value.f1 + accumulator;
                //     }
                //
                //     // 获取结果
                //     @Override
                //     public String getResult(Integer accumulator) {
                //         return accumulator.toString();
                //     }
                //
                //     // 累加器的合并: 只有会话窗口才会调用
                //     @Override
                //     public Integer merge(Integer a, Integer b) {
                //         return a + b;
                //     }
                // })
                // 3. ProcessWindowFunction 三个泛型 1: 输入  2: 输出 3: key  4. Window Type
                .process(new ProcessWindowFunction<Tuple2<String, Integer>, Integer, String, TimeWindow>() {
                    /**
                     *
                     * @param s         key
                     * @param context   上下文对象
                     * @param elements  这个窗口中的所有元素
                     * @param out       收集器, 用于向下游传递
                     * @throws Exception
                     */
                    @Override
                    public void process(String s, Context context, Iterable<Tuple2<String, Integer>> elements, Collector<Integer> out) throws Exception {
                        int sum = 0;
                        for (Tuple2<String, Integer> element : elements) {
                            sum = sum + element.f1;
                        }
                        System.out.println("该窗口中一共有: " + elements.spliterator().estimateSize() + " 个元素.");
                        out.collect(sum);
                    }
                })
                .print();
        env.execute();
    }
}
```

### 2.1.2 滑动窗口(Sliding Window)
![滑动窗口](../../img/flink/window/滑动窗口.png)

```java
package com.kino.flink.windows.time;

import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

public class SlidingWindowTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new Tuple2<String, Integer>(split[0], Integer.parseInt(split[1]));
                })
                .keyBy(k -> k.f0)
                .window(SlidingEventTimeWindows.of(Time.seconds(5), Time.seconds(2)))
                .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                        return new Tuple2<String, Integer>(value1.f0, value1.f1 + value2.f1);
                    }
                })
                .print();
        env.execute();
    }
}
```

### 2.1.3 会话窗口(Session Window)
![会话窗口](../../img/flink/window/会话窗口.png)

```java
package com.kino.flink.windows.time;

import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.ProcessingTimeSessionWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

public class SessionWindowTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new Tuple2<String, Integer>(split[0], Integer.parseInt(split[1]));
                })
                .keyBy(k -> k.f0)
                .window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
                .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                        return new Tuple2<String, Integer>(value1.f0, value1.f1 + value2.f1);
                    }
                })
                .print();
        env.execute();
    }
}
```


### 2.1.4 全局窗口(Global Window)
![全局窗口](../../img/flink/window/全局窗口.png)

```java
package com.kino.flink.windows.time;

import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.GlobalWindows;
import org.apache.flink.streaming.api.windowing.triggers.Trigger;
import org.apache.flink.streaming.api.windowing.triggers.TriggerResult;
import org.apache.flink.streaming.api.windowing.windows.GlobalWindow;

public class GlobalWindowTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new Tuple2<String, Integer>(split[0], Integer.parseInt(split[1]));
                })
                .keyBy(k -> k.f0)
                .window(GlobalWindows.create())
                .trigger(new Trigger<Tuple2<String, Integer>, GlobalWindow>() {
                    private static final long serialVersionUID = 1L;
                    ValueStateDescriptor<Integer> stateDescriptor = new ValueStateDescriptor<>("total", Integer.class);

                    /**
                     * 每个事件进入窗口都会进行计算
                     * @param element       事件
                     * @param timestamp     进入窗口时间戳
                     * @param window        窗口类型
                     * @param ctx           Trigger 上下文对象
                     * @return
                     * @throws Exception
                     */
                    @Override
                    public TriggerResult onElement(Tuple2<String, Integer> element, long timestamp, GlobalWindow window, TriggerContext ctx) throws Exception {
                        ValueState<Integer> sumState = ctx.getPartitionedState(stateDescriptor);
                        if (null == sumState.value()) {
                            sumState.update(0);
                        }
                        sumState.update(element.f1 + sumState.value());
                        if (sumState.value() >= 2) {
                            //这里可以选择手动处理状态
                            //  默认的trigger发送是TriggerResult.FIRE 不会清除窗口数据
                            return TriggerResult.FIRE_AND_PURGE;
                        }
                        return TriggerResult.CONTINUE;
                    }

                    /**
                     * 根据事件进入窗口的 Process Time 进行计算
                     * @param time
                     * @param window
                     * @param ctx
                     * @return
                     * @throws Exception
                     */
                    @Override
                    public TriggerResult onProcessingTime(long time, GlobalWindow window, TriggerContext ctx) throws Exception {
                        return TriggerResult.CONTINUE;
                    }

                    /**
                     * 根据时间进入窗口的 Event Time 进行计算
                     * @param time
                     * @param window
                     * @param ctx
                     * @return
                     * @throws Exception
                     */
                    @Override
                    public TriggerResult onEventTime(long time, GlobalWindow window, TriggerContext ctx) throws Exception {
                        return TriggerResult.CONTINUE;
                    }

                    /**
                     * 窗口的清除
                     * @param window
                     * @param ctx
                     * @throws Exception
                     */
                    @Override
                    public void clear(GlobalWindow window, TriggerContext ctx) throws Exception {
                        System.out.println("清理窗口状态  窗口内保存值为" + ctx.getPartitionedState(stateDescriptor).value());
                        ctx.getPartitionedState(stateDescriptor).clear();
                    }
                })
                .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                        return new Tuple2<>(value1.f0, value1.f1 + value2.f1);
                    }
                })
                .print();
        env.execute();
    }
}
```

## 2.2 数据驱动的 Window
按时间个数生成 Window, 一个窗口长度为5的滚动窗口意思是说: 这个窗口只收集 5 个事件, 超过就产生新的窗口;

可以分为:
1. 滚动窗口: 窗口长度固定, 窗口之间不重叠, 一个事件仅属于一个窗口;
2. 滑动窗口: 窗口长度固定, 窗口之间有重叠, 一个事件可能属于多个窗口;


### 2.2.1 滚动窗口
```java
package com.kino.flink.windows.count;

import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TumblingCountTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new Tuple2<String, Integer>(split[0], Integer.parseInt(split[1]));
                })
                .keyBy(k -> k.f0)
                .countWindow(5)
                .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                        return new Tuple2<>(value1.f0, value1.f1 + value2.f1);
                    }
                })
                .print();
        env.execute();
    }
}
```

### 2.2.2 滑动窗口
```java
package com.kino.flink.windows.count;

import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class SlidingCountTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new Tuple2<String, Integer>(split[0], Integer.parseInt(split[1]));
                })
                .keyBy(k -> k.f0)
                .countWindow(5, 2)
                .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                        return new Tuple2<>(value1.f0, value1.f1 + value2.f1);
                    }
                })
                .print();
        env.execute();
    }
}
```

# 三、Window Function
.window() 方法是用来定义一个窗口, 窗口的计算需要 Window Function 指定如何计算, 当窗口关闭时, Window Function 就去计算窗口中的数据了;

## 3.1 Window Function 分类
1. ReduceFunction(增量聚合函数)
2. AggregateFunction(增量聚合函数)
3. ProcessWindowFunction(全窗口函数): 不高效, 执行这个函数前, 窗口中的数据被缓存在内存中, 在实现方法中, 得到的是窗口数据的 迭代器

### 3.2 三种 Window Function 示例代码
```java
package com.kino.flink.windows.time;

import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

public class TumblingWindowTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment().setParallelism(1);

        env.socketTextStream("localhost", 9999)
                .map(line -> {
                    String[] split = line.split(",");
                    return new Tuple2<String, Integer>(split[0], Integer.parseInt(split[1]));
                })
                .keyBy(k -> k.f0)
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                // 1. ReduceFunction
                // .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                //     @Override
                //     public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                //         return new Tuple2<String, Integer>(value1.f0, value1.f1 + value2.f1);
                //     }
                // })
                // 2. AggregateFunction, 三个泛型, 1: 输入 2: 中间结果 3: 输出结果
                // .aggregate(new AggregateFunction<Tuple2<String, Integer>, Integer, String>() {
                //     // 初始化累加器
                //     @Override
                //     public Integer createAccumulator() {
                //         return 0;
                //     }
                //
                //     // 累加逻辑
                //     @Override
                //     public Integer add(Tuple2<String, Integer> value, Integer accumulator) {
                //         return value.f1 + accumulator;
                //     }
                //
                //     // 获取结果
                //     @Override
                //     public String getResult(Integer accumulator) {
                //         return accumulator.toString();
                //     }
                //
                //     // 累加器的合并: 只有会话窗口才会调用
                //     @Override
                //     public Integer merge(Integer a, Integer b) {
                //         return a + b;
                //     }
                // })
                // 3. ProcessWindowFunction 三个泛型 1: 输入  2: 输出 3: key  4. Window Type
                .process(new ProcessWindowFunction<Tuple2<String, Integer>, Integer, String, TimeWindow>() {
                    /**
                     *
                     * @param s         key
                     * @param context   上下文对象
                     * @param elements  这个窗口中的所有元素
                     * @param out       收集器, 用于向下游传递
                     * @throws Exception
                     */
                    @Override
                    public void process(String s, Context context, Iterable<Tuple2<String, Integer>> elements, Collector<Integer> out) throws Exception {
                        long windowCount = 0L;
                        int sum = 0;
                        for (Tuple2<String, Integer> element : elements) {
                            windowCount++;
                            sum = sum + element.f1;
                        }
                        System.out.println("该窗口中一共有: " + elements.spliterator().estimateSize() + " 个元素.");
                        out.collect(sum);
                    }
                })
                .print();
        env.execute();
    }
}
```

# 四、KeyBy 对 Window 的影响
在选择 Window 之前, 还需要考虑是否有做 .keyBy() 操作, 如果没有做 .keyBy(), 则只能选择 .windowAll() 方法, 如果做了 .keyBy(), 则可以选择 .window() 方法
```java
.windowAll(TumblingProcessingTimeWindows.of(Time.seconds(10)))
```

作用于 keyedStream 上面的 Window, 窗口计算被并行的运用在多个 Task上, 可以理解为每个 Task 都有自己单独的 Window

作用于 不是 keyedStream 上面的 Window, 其并行度只能是1, 如果设置了多并行度, 最终也只会在其中一个 Task 上执行























