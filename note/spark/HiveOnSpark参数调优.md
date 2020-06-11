* [前言](#%E5%89%8D%E8%A8%80)
* [executor参数](#executor%E5%8F%82%E6%95%B0)
  * [spark\.executor\.cores](#sparkexecutorcores)
  * [spark\.executor\.memory/spark\.yarn\.executor\.memoryOverhead](#sparkexecutormemorysparkyarnexecutormemoryoverhead)
  * [spark\.executor\.instances](#sparkexecutorinstances)
  * [spark\.dynamicAllocation\.enabled](#sparkdynamicallocationenabled)
* [Driver参数](#driver%E5%8F%82%E6%95%B0)
  * [spark\.driver\.cores](#sparkdrivercores)
  * [spark\.driver\.memory/spark\.driver\.memoryOverhead](#sparkdrivermemorysparkdrivermemoryoverhead)
* [Hive参数](#hive%E5%8F%82%E6%95%B0)
  * [hive\.auto\.convert\.join\.noconditionaltask\.size](#hiveautoconvertjoinnoconditionaltasksize)
  * [hive\.merge\.sparkfiles](#hivemergesparkfiles)
  
  
---

# 前言

Hive on Spark是指使用Spark替代传统MapReduce作为Hive的执行引擎，在HIVE-7292提出。Hive on Spark的效率比on MR要高不少，但是也需要合理调整参数才能最大化性能，本文简单列举一些调优项。为了符合实际情况，Spark也采用on YARN部署方式来说明。

----
# executor参数
## spark.executor.cores
该参数表示每个Executor可利用的CPU核心数。其值不宜设定过大，因为Hive的底层以HDFS存储，而HDFS有时对高并发写入处理不太好，容易造成race condition。根据我们的实践，设定在3~6之间比较合理。

假设我们使用的服务器单节点有32个CPU核心可供使用。考虑到系统基础服务和HDFS等组件的余量，一般会将YARN NodeManager的`yarn.nodemanager.resource.cpu-vcores`参数设为28，也就是YARN能够利用其中的28核，此时将spark.executor.cores设为4最合适，最多可以正好分配给7个Executor而不造成浪费。又假设`yarn.nodemanager.resource.cpu-vcores`为26，那么将spark.executor.cores设为5最合适，只会剩余1个核。

由于一个Executor需要一个YARN Container来运行，所以还需保证`spark.executor.cores`的值不能大于单个Container能申请到的最大核心数，即`yarn.scheduler.maximum-allocation-vcores`的值。



## spark.executor.memory/spark.yarn.executor.memoryOverhead
这两个参数分别表示每个Executor可利用的堆内内存量和堆外内存量。堆内内存越大，Executor就能缓存更多的数据，在做诸如map join之类的操作时就会更快，但同时也会使得GC变得更麻烦。Hive官方提供了一个计算Executor总内存量的经验公式，如下：

`yarn.nodemanager.resource.memory-mb * (spark.executor.cores / yarn.nodemanager.resource.cpu-vcores)`

其实就是按核心数的比例分配。在计算出来的总内存量中，80%~85%划分给堆内内存，剩余的划分给堆外内存。



假设集群中单节点有128G物理内存，`yarn.nodemanager.resource.memory-mb`（即单个NodeManager能够利用的主机内存量）设为120G，那么总内存量就是：120 * 1024 * (4 / 28) ≈ 17554MB。再按8:2比例划分的话，最终`spark.executor.memory`设为约13166MB，`spark.yarn.executor.memoryOverhead`设为约4389MB。



与上一节同理，这两个内存参数相加的总量也不能超过单个Container最多能申请到的内存量，即`yarn.scheduler.maximum-allocation-mb`。



## spark.executor.instances
该参数表示执行查询时一共启动多少个Executor实例，这取决于每个节点的资源分配情况以及集群的节点数。若我们一共有10台32C/128G的节点，并按照上述配置（即每个节点承载7个Executor），那么理论上讲我们可以将spark.executor.instances设为70，以使集群资源最大化利用。但是实际上一般都会适当设小一些（推荐是理论值的一半左右），因为Driver也要占用资源，并且一个YARN集群往往还要承载除了Hive on Spark之外的其他业务。



## spark.dynamicAllocation.enabled
上面所说的固定分配Executor数量的方式可能不太灵活，尤其是在Hive集群面向很多用户提供分析服务的情况下。所以更推荐将spark.dynamicAllocation.enabled参数设为true，以启用Executor动态分配。


---
# Driver参数
## spark.driver.cores
该参数表示每个Driver可利用的CPU核心数。绝大多数情况下设为1都够用。

## spark.driver.memory/spark.driver.memoryOverhead
这两个参数分别表示每个Driver可利用的堆内内存量和堆外内存量。根据资源富余程度和作业的大小，一般是将总量控制在512MB~4GB之间，并且沿用Executor内存的“二八分配方式”。例如，spark.driver.memory可以设为约819MB，`spark.driver.memoryOverhead`设为约205MB，加起来正好1G。


---
# Hive参数
绝大部分Hive参数的含义和调优方法都与on MR时相同，但仍有两个需要注意。



## hive.auto.convert.join.noconditionaltask.size
我们知道，当Hive中做join操作的表有一方是小表时，如果`hive.auto.convert.join`和`hive.auto.convert.join.noconditionaltask`开关都为true（默认即如此），就会自动转换成比较高效的map-side join。而`hive.auto.convert.join.noconditionaltask.size`


但是Hive on MR下统计表的大小时，使用的是数据在磁盘上存储的近似大小，而Hive on Spark下则改用在内存中存储的近似大小。由于HDFS上的数据很有可能被压缩或序列化，使得大小减小，所以由MR迁移到Spark时要适当调高这个参数，以保证map join正常转换。一般会设为100~200MB左右，如果内存充裕，可以更大点。



## hive.merge.sparkfiles
小文件是HDFS的天敌，所以Hive原生提供了合并小文件的选项，在on  MR时是`hive.merge.mapredfiles`，但是on Spark时会改成hive.merge.sparkfiles，注意要把这个参数设为true。至于小文件合并的阈值参数，即`hive.merge.smallfiles.avgsize`与`hive.merge.size.per.task`都没有变化