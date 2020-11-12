


---


# 一、操作系统调优
操作系统调优主要从三个方面
- Linux 文件句柄数
- 最大虚拟内存
- Swap 虚拟内存

## 1.1 Linux 文件句柄数
Linux 默认设置是 1024, 该默认设置对于服务器进程来说太小, 适当调大, 通过修改 `limits.conf` 来增大
```bash
$ echo "* soft nofile 65535" >> /etc/security/limits.conf
$ echo "* hard nofile 65535" >> /etc/security/limits.conf
```

## 1.2 最大虚拟内存
最大虚拟内存(max_map_count), 定义了进程能拥有的内存区域
```bash
$ sysctl -w vm.max_map_count=102400
```

## 1.3 Swap 虚拟内存
Swap的作用类似 Windows 系统下的 "虚拟内存"。 当物理内存不足时, 拿出部分硬盘空间当Swap分区（虚拟成内存）使用,从而解决内存容量不足的情况

当HBase的数据发生交换时, 会发生IO操作, 降低读取效率, 所以应当避免 HBase 数据被交换

```bash
$ swapoff -a  # 临时关闭Swap
$ sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久关闭Swap
```

# 二、HBase 配置优化
## 2.1 HBase RegionServer 的优化
HBase的实时数据会首先写入 Memstore 内存中, 然后才会刷新到磁盘上, 同时HBase也会将经常被访问的热点Key进行缓存, 因此 HBase对JVM的内存要求比较高;

当操作系统有32GB内存, 建议给操作系统和堆外内存预留8GB内存暴涨系统运行稳定, 剩余24GB分配给HBase的JVM堆内存

```bash
$ hbase_regionserver_opts -Xmx24g -Xms24g -Xmn6g
```

## 2.2 G1垃圾回收器的配置
因为G1垃圾回收期能够有效的降低JVM Full GC的次数, 因此我们通过 `-XX:+UseG1GC` 开启G1垃圾回收器

需要注意的是, JDK1.8以上才支持G1垃圾回收期

## 2.3 HBase线程参数设置
1. hbase.regionserver.handler.count 参数
    
    表示RegionServer同时有多少个线程处理客户端请求, 默认为10
    - 对于单次Put、Get数据大(单条数据超过2MB)的情况下, 适中调节到[30-50]
    - 对于单次Put、Get数据小、TPS要求高的情况下, 调高配置到[100-200]

2. hbase.regionserver.thread.compaction.small 和 hbase.regionserver.thread.compaction.large
    用于负责处理所有的 compaction 请求
     - 当compact的文件总大小大于throttlePoint, 则将compaction请求分配给largeCompactions处理, 
     - 否则分配给smallCompactions处理
    
    `throttlePoint`参数可以通过: `hbase.regionserver.thread.compaction.throttle`调节, 该参数默认为`2.5GB`, 
    
    `hbase.regionserver.thread.compaction.small` 和 `hbase.regionserver.thread.compaction.large`默认都只有一个线程, 可以改成4/6

3. hbase.hregion.max.filesize参数
    - 如果该参数设置过小, 触发split几率变大, 系统的整体访问服务会出现不稳定现象；
    - 如果该参数设置过大, 则不太适合经常split和compaction, 因为做一次compaction和split会产生较长的停顿, 对于应用的读写性能冲击非常大
    - 在生产环境中高并发运行下, 最佳大小为5-10G
    - 同时建议关闭 major_compact 'table_name', 在非高峰时期的时候再去调用 major_compact进行HBase大合并, 这样可以在减少split的同时, 显著提升集群的性能和吞吐量

4. hfile.block.cache.size参数
    表示 `regionserver cache`的大小, 默认是0.2, 其含义是整个堆内存的多少比例作为regionserver的cache, 调大该值会提升查询性能, 当然也不宜过大, 如果 hbase主要是大量的查询, 写入频率不是很高的话, 调到0.5就足够了

5. hbase.hregion.memstore.flush.size参数
    表示一个regionserver的单个region memstore的大小, 默认是 64MB
    
    配置时需要参考平均每个regionserver管理的region数量, 如果每台regionserver管理的region不多的话, 可以适当调节到512MB, 
    
    - 该值如果设置的比较小, 则HBase会快速频繁的将memstore刷新到磁盘上, 
    - 如果设置的较大, 则客户端的请求会在memstore写满后在刷新到磁盘

6. hbase.regionserver.global.memstore.upperLimit 和 hbase.regionserver.global.memstore.lowerLimit
    这两个参数人别指定了一个 RegionServer上MemStore总共可以使用的堆内存的最大百分比和最小百分比,
     
    当memstore遇到upperLimit的时候MemStore中的数据会被刷新到硬盘, 直到遇到lowerLimit时停止刷盘, 
    
    - 如果场景是写入频繁的话可以适当调大该值(6/4), 
    - 如果是读取频繁的话, 可以适当调小该值, 把更多的内存让给查询缓存.
    
    
# 三、HBase 小技巧
1. rowKey 规则的影响
    因为HBase是基于RowKey将数据存储在不同的RegionServer上的, 如果RowKey指定的不合理, 很可能造成很多数据集中在某个RegionServer上, 而其他RegionServer上的数据量很少, 造成数据倾斜/某个key过热, 我们在设计过程中需要尽量保证 RowKey的随机性, 以保证所有数据都均衡的分布在每个节点上
    
2. 预创建分区    
    在创建HBase表时, 就预先根据可能的RowKey划分出来多个 Region而不是默认一个, 从而可以将后续的读取操作负载均衡到不同的 Region上, 避免热点现象

3. 启动数据压缩
    HBase集群常常要存储PB级别的数据, 这将造成服务器磁盘费用过高, 一般采用压缩算法来节省磁盘空间, 目前HBase默认支持的算法包括GZ、LZO、Snappy, 其中Snappy压缩算法压缩率和性能表现最优秀 
    
4. 监控报警
    安装类似Cloudera这样的管理工具, 对HBase的实时运行状态进行监控, 如果发生性能瓶颈或者系统稳定文问题及时早发现、早处理