



---
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

