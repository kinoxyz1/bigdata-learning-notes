



---
# 一、什么是 CEP
[官方说明](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/libs/cep/)

FlinkCEP 可以用来在无穷无尽的流中检测出特定的数据, 比如 检测异常登录等。



# 二、CEP 的使用
## 2.1 导入 CEP 相关依赖
```pom
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-cep_${scala.version}</artifactId>
    <version>${flink.version}</version>
</dependency>
```


## 2.2 基本步骤
1. 定义规则模式
2. 应用到Stream上
3. 获取匹配结果

## 2.3 步骤一 定义规则模式
```java
Pattern<Test, Test> orderEventPattern = Pattern.<Test>begin("events")
                .where(new SimpleCondition<Test>() {
                    @Override
                    public boolean filter(Test value) throws Exception {
                        return "add".equalsIgnoreCase(value.getEventType());
                    }
                })
                .followedBy("follow")
                .where(new SimpleCondition<Test>() {
                    @Override
                    public boolean filter(Test value) throws Exception {
                        return "pay".equalsIgnoreCase(value.getEventType());
                    }
                }).within(Time.seconds(5));
```

## 2.4 步骤二 应用到Stream上
```java
PatternStream<Test> patternStream = CEP.pattern(orderEventStringKeyedStream, orderEventPattern);
```

## 2.5 步骤三 获取匹配结果
```java
SingleOutputStreamOperator<String> result = patternStream.select(
                new OutputTag<String>("No Pay") {
                },
                new OrderPayTimeOutFunc(),
                new OrderPaySelectFunc());
        result.getSideOutput(new OutputTag<String>("No Pay") {}).print("Time Out");
```

## 2.6 完整示例
```java
package com.jz.app;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.jz.bean.OrderEvent;
import com.jz.common.StreamingConfig;
import com.jz.utils.DateTimeUtil;
import com.jz.utils.KafkaUtil;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.PatternTimeoutFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.OutputTag;

import java.util.List;
import java.util.Map;

/**
 * @author: kino
 * @date: 2021/7/6 9:51
 */
public class TestCEP1 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        WatermarkStrategy<Test> orderEventWatermark = WatermarkStrategy.<Test>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<Test>() {
                    @Override
                    public long extractTimestamp(Test element, long recordTimestamp) {
                        return element.getTime() * 1000L;
                    }
                });

        FlinkKafkaConsumer<String> kafkaSource = KafkaUtil.getKafkaSource(StreamingConfig.TOPIC, StreamingConfig.GROUPID);
        DataStreamSource<String> kafkaDS = env.addSource(kafkaSource);
        SingleOutputStreamOperator<Test> inputDS = kafkaDS.map(line -> {
            JSONObject obj = JSON.parseObject(line);
            return new Test(obj.getString("uuid"), obj.getString("eventType"), obj.getString("itemCode"), obj.getLong("time"));
        }).assignTimestampsAndWatermarks(orderEventWatermark);

//        SingleOutputStreamOperator<Test> inputDS = env.readTextFile("D:\\work\\jz-dmp\\dc_streaming\\src\\main\\resources\\input\\test.log")
//                .map(line -> {
//                    JSONObject obj = JSON.parseObject(line);
//                    return new Test(obj.getString("uuid"), obj.getString("eventType"), obj.getString("itemCode"), obj.getLong("time"));
//                })
//                .assignTimestampsAndWatermarks(orderEventWatermark);

        KeyedStream<Test, String> orderEventStringKeyedStream = inputDS.keyBy(line -> line.getUuid()+line.getItemCode());

        Pattern<Test, Test> orderEventPattern = Pattern.<Test>begin("events")
                .where(new SimpleCondition<Test>() {
                    @Override
                    public boolean filter(Test value) throws Exception {
                        return "add".equalsIgnoreCase(value.getEventType());
                    }
                })
                .followedBy("follow")
                .where(new SimpleCondition<Test>() {
                    @Override
                    public boolean filter(Test value) throws Exception {
                        return "pay".equalsIgnoreCase(value.getEventType());
                    }
                }).within(Time.seconds(5));

        PatternStream<Test> patternStream = CEP.pattern(orderEventStringKeyedStream, orderEventPattern);

        SingleOutputStreamOperator<String> result = patternStream.select(
                new OutputTag<String>("No Pay") {
                },
                new OrderPayTimeOutFunc(),
                new OrderPaySelectFunc());
        orderEventStringKeyedStream.print();

        result.print();
        result.getSideOutput(new OutputTag<String>("No Pay") {}).print("Time Out");

        env.execute();
    }

    public static class OrderPayTimeOutFunc implements PatternTimeoutFunction<Test, String> {
        @Override
        public String timeout(Map<String, List<Test>> map, long l) throws Exception {
            Test createEvent = map.get("events").get(0);
            String createTime = DateTimeUtil.ssTimeToString(createEvent.getTime(), "yyyy-MM-dd HH:mm:ss");
            String endTime = DateTimeUtil.ssTimeToString((l/1000), "yyyy-MM-dd HH:mm:ss");
            return "用户: " + createEvent.getUuid() + " 在 " + createTime + " 加购商品(" + createEvent.getItemCode() + ")" +
                    " 并在 " + StreamingConfig.PAY_OVER_TIME + " 分钟内没有支付, 超时时间是: " + endTime;
        }
    }

    public static class OrderPaySelectFunc implements PatternSelectFunction<Test, String> {
        @Override
        public String select(Map<String, List<Test>> pattern) throws Exception {
            Test createEvent = pattern.get("events").get(0);
            Test payEvent = pattern.get("follow").get(0);
            String createTime = DateTimeUtil.ssTimeToString(createEvent.getTime(), "yyyy-MM-dd HH:mm:ss");
            String endTime = DateTimeUtil.ssTimeToString(payEvent.getTime(), "yyyy-MM-dd HH:mm:ss");
            return "用户: " + createEvent.getUuid() + " 在 " + createTime + " 加购商品(" + createEvent.getItemCode() + ")" +
                    " 并在 " + StreamingConfig.PAY_OVER_TIME + " 分钟内完成了支付, 支付时间是: " + endTime;
        }
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class Test {
    String uuid;
    String eventType;
    String itemCode;
    Long time;
}
```

# 三、Pattern API
Pattern API 是用来定义要从无界流中提取**复杂模式**序列。

**每个复杂模式序列由多个简单模式组成**

关于 Pattern API 的官方说明
```text
The pattern API allows you to define complex pattern sequences that you want to extract from your input stream.

Each complex pattern sequence consists of multiple simple patterns, i.e. patterns looking for individual events with the same properties. From now on, we will call these simple patterns patterns, and the final complex pattern sequence we are searching for in the stream, the pattern sequence. You can see a pattern sequence as a graph of such patterns, where transitions from one pattern to the next occur based on user-specified conditions, e.g. event.getName().equals("end"). A match is a sequence of input events which visits all patterns of the complex pattern graph, through a sequence of valid pattern transitions.
```
注意:
1. 每个模式都必须有一个唯一的名字, 此名字在后面用来匹配事件
2. 模式的名称不能包含 ":" 字符
  

## 3.1 简单模式
简单模式(也直接称为模式)可以是单例模式或循环模式, 单例模式接收单个事件, 循环模式接收多个事件。

### 3.1.1 单例模式
```java
// 1. 定义模式
Pattern<Test, Test> pattern = Pattern
    .<Test>begin("start")
    .where(new SimpleCondition<Test>() {
        @Override
        public boolean filter(Test value) throws Exception {
            return "test_1".equals(value.getId());
        }
    });
// 2. 在流上应用模式
PatternStream<Test> testPS = CEP.pattern(testStream, pattern);
// 3. 获取匹配到的结果
testPS
    .select(new PatternSelectFunction<Test, String>() {
        @Override
        public String select(Map<String, List<Test>> pattern) throws Exception {
            return pattern.toString();
        }
    })
    .print();
```

### 3.1.2 循环模式
单例模式可以设置循环规则, 如: 固定次数循环、范围内的次数、一次或多次、多次及多次以上

1. 固定次数
```java
// 1. 定义模式
Pattern<Test, Test> pattern = Pattern
    .<Test>begin("start")
    .where(new SimpleCondition<Test>() {
        @Override
        public boolean filter(Test value) throws Exception {
            return "test_1".equals(value.getId());
        }
    });
// 2. 在流上应用模式, 期望循环 4 次
PatternStream<Test> testPS = CEP.pattern(testStream, pattern).times(4);
// 3. 获取匹配到的结果
testPS
    .select(new PatternSelectFunction<Test, String>() {
        @Override
        public String select(Map<String, List<Test>> pattern) throws Exception {
            return pattern.toString();
        }
    })
    .print();
```

2. 范围内的次数
```java
// 1. 定义模式
Pattern<Test, Test> pattern = Pattern
    .<Test>begin("start")
    .where(new SimpleCondition<Test>() {
        @Override
        public boolean filter(Test value) throws Exception {
            return "test_1".equals(value.getId());
        }
    });
// 2. 在流上应用模式, 期望循环 2、3、4 次
PatternStream<Test> testPS = CEP.pattern(testStream, pattern).times(2, 4);
// 3. 获取匹配到的结果
testPS
    .select(new PatternSelectFunction<Test, String>() {
        @Override
        public String select(Map<String, List<Test>> pattern) throws Exception {
            return pattern.toString();
        }
    })
    .print();
```

3. 一次或多次
```java
// 1. 定义模式
Pattern<Test, Test> pattern = Pattern
    .<Test>begin("start")
    .where(new SimpleCondition<Test>() {
        @Override
        public boolean filter(Test value) throws Exception {
            return "test_1".equals(value.getId());
        }
    });
// 2. 在流上应用模式, 期望循环 1 次或多次
PatternStream<Test> testPS = CEP.pattern(testStream, pattern).oneOrMore();
// 3. 获取匹配到的结果
testPS
    .select(new PatternSelectFunction<Test, String>() {
        @Override
        public String select(Map<String, List<Test>> pattern) throws Exception {
            return pattern.toString();
        }
    })
    .print();
```

4. 多次及多次以上
```java
// 1. 定义模式
Pattern<Test, Test> pattern = Pattern
    .<Test>begin("start")
    .where(new SimpleCondition<Test>() {
        @Override
        public boolean filter(Test value) throws Exception {
            return "test_1".equals(value.getId());
        }
    });
// 2. 在流上应用模式, 期望出现 2次 或 2次以上
PatternStream<Test> testPS = CEP.pattern(testStream, pattern).timesOrMore(2);
// 3. 获取匹配到的结果
testPS
    .select(new PatternSelectFunction<Test, String>() {
        @Override
        public String select(Map<String, List<Test>> pattern) throws Exception {
            return pattern.toString();
        }
    })
    .print();
```


## 3.2 模式条件
对于每个模式， 可以指定传入事件必须满足相应的条件, 才能被接受到模式中, 例如事件的某个值必须大于5, 或者大于之前接受事件的平均值.

可以通过 `pattern.where()`、`pattern.or()` 或 `pattern.until()` 方法指定事件属性的条件。这些可以是 `IterativeConditions` 或 `SimpleConditions`。

### 3.2.1 迭代条件(IterativeConditions)
最常见的条件类型, 可以**根据先前接受的事件的属性或它们的子集的统计数据指定接受后续事件的条件方式。**

示例: 接受模式名为: "middle" 的下一个事件, 如果它的名字以 "foo" 开头
```java
middle.oneOrMore()
  .subtype(SubEvent.class)
  .where(new IterativeCondition<SubEvent>() {
      @Override
      public boolean filter(SubEvent value, Context<SubEvent> ctx) throws Exception {
        if (!value.getName().startsWith("foo")) {
            return false;
        }

        double sum = value.getPrice();
        // 这里会匹配先前所有接受的事件
        for (Event event : ctx.getEventsForPattern("middle")) {
            sum += event.getPrice();
        }
        return Double.compare(sum, 5.0) < 0;
      }
    });
```

### 3.2.2 简单条件(SimpleConditions)
迭代条件 是依据先前接受到的事件的属性进行判断是否接受事件, 

简单条件 是根据事件本身的属性来决定是否接受事件。
```java
// 1. 定义模式
start
  // .subtype(): 将接受事件的类型限制为初始事件类型的子类型。      
  .subtype(SubEvent.class)
  .where(new SimpleCondition<Event>() {
  @Override
  public boolean filter(Event value) {
        return value.getName().startsWith("foo");
  }
});
```

### 3.2.3 组合条件
在复杂情况下, 迭代条件 和 简单条件 并不能满足业务, 很多场景下, 都需要将条件进行组合使用, 例如: 年龄 > 30 并且(或者) 年收入 > 30W 
```java
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.age > 30;
    }
}).or(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.Income > 300000;
    }
});

或者
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.age > 30;
    }
}).where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.Income > 300000;
        }
    });
```

### 3.2.4 停止条件
在循环模式中(oneOrMore() & oneOrMore().optional()), 可以指定停止的条件, 例如接收值大于5的事件, 当值的总和大于 50 结束.

```java
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.val > 30;
    }
}).oneOrMore().until(new IterativeCondition<Event>() {
    @Override
    public boolean filter(Event value, Context ctx) throws Exception {
        return value.sum > 50 // alternative condition
    }
});
```

## 3.3 组合模式(模式序列)
在上面介绍了 单例模式 和 循环模式, 一个规则可以出现一次或循环依次出现多次, 但是实际可能不全然如此, 例如需要计算用户点击购买之后是否在30分钟内支付了, 如果只是做连续匹配, 就会出现问题, 因为在点击购买之后支付之前, 一样可以进行其他事件的操作。

### 3.3.1 严格连续
严格连续是说, 期望用户定义的多个规则事件都是严格连续一个接一个的出现的, 中间不会出现任何其他的事件
```java
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.age > 30;
    }
}).next("end").where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.Income > 300000;
        }
    });
```


```java
// notNext 表示 事件A 之后一定不能是 事件B
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.age > 30;
    }
}).notNext("end").where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.Income > 300000;
        }
    });
```


### 3.3.2 松散连续
松散连续是说, 事件A 和 事件B 中间, 可以有事件C, A 和 B 事件不是一定要连续
```java
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.age > 30;
    }
}).followedBy("end").where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.Income > 300000;
        }
    });
```


```java
// notFollowedBy 是说, 不期望一个事件在 A 和 B 之间发生
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.age > 30;
    }
}).notFollowedBy("end").where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.Income > 300000;
        }
    });
```

### 3.3.3 非确定的松散连续
当事件为: {a, c, b, b} 时:
1. followedBy: 匹配的结果为: {a, b}
2. followedByAny: 匹配的结果为: {a, b}, {a, b}
```java
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.age > 30;
    }
}).followedByAny("end").where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.Income > 300000;
        }
    });
```

