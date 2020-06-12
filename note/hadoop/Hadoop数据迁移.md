* [一、需求说明](#%E4%B8%80%E9%9C%80%E6%B1%82%E8%AF%B4%E6%98%8E)
* [二、找需要迁移的数据](#%E4%BA%8C%E6%89%BE%E9%9C%80%E8%A6%81%E8%BF%81%E7%A7%BB%E7%9A%84%E6%95%B0%E6%8D%AE)
* [三、迁移脚本](#%E4%B8%89%E8%BF%81%E7%A7%BB%E8%84%9A%E6%9C%AC)
* [五、注意事项](#%E4%BA%94%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

---
# 一、需求说明
把A集群上的数据迁移到B集群上。
需要预先知道A集群主节点的ip地址，和B集群主节点的ip地址。

---
# 二、找需要迁移的数据
![在这里插入图片描述](../../img/hadoop/hadoop数据迁移/20200601113543893.png)
然后点击可以查看文件目录树，这里以进入 /user/hive/warehouse/transpass_std.db/s_phone_inter_trip/source=cm/start_city=440300/year=2019/month=03/day=12 文件夹为例
![在这里插入图片描述](../../img/hadoop/hadoop数据迁移/20200601113600597.png)
根据实际迁移需求，找到你所想要迁移的数据，然后查看另一台集群上是否已存在。不存在则记录要迁移数据的文件目录，已存在则不需要迁移。
![在这里插入图片描述](../../img/hadoop/hadoop数据迁移/20200601113627749.png)

---
# 三、迁移脚本

```bash
#!/bin/bash

hdfspath=$1
hadoop fs -mkdir -p $1
hadoop distcp -update -skipcrccheck hdfs://A节点IP:8020$1 hdfs://B节点IP:8020$1
```
代码说明
![在这里插入图片描述](../../img/hadoop/hadoop数据迁移/2020060111383873.png)

将脚本文件保存，这里保存的文件名为hdfs-cp.sh

在脚本文件目录下运行shell脚本，sh hdfs-cp.sh +传输文件路径，这里传输/user/hive/warehouse/transpass_std.db/s_phone_inter_trip/source=cm/start_city=440300/year=2019/month=03为例

```bash
[hdfs@bigdata-1 ~]$ sh hdfs-cp.sh /user/hive/warehouse/transpass_std.db/s_phone_inter_trip/source=cm/start_city=440300/year=2019/month=03
```
运行时间根据数据大小而定，出现job completed successfully 表示传输成功
![在这里插入图片描述](../../img/hadoop/hadoop数据迁移/20200601113932201.png)

---
# 五、注意事项
文件太大可能传输失败，这时需要分批传输，不要一次性传太大
官方数据迁移教程：http://hadoop.apache.org/docs/r1.0.4/cn/distcp.html