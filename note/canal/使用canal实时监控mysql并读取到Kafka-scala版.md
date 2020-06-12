

---
# 一、canal 安装和启动



## 1.1下载 canal

```bash
wget https://github.com/alibaba/canal/releases/download/canal-1.1.2/canal.deployer-1.1.2.tar.gz
```



## 1.2 解压

```bash
# 要先执行此步骤创建 canal 文件
mkdir /opt/module/canal

tar -zxvf canal.deployer-1.1.2.tar.gz -C /opt/module/canal
```

## 1.3 配置

1.  `conf/canal.properties`：canal 的通用配置， 主要关注下 canal.prot，默认是 11111

	![在这里插入图片描述](../../img/canal/canal监控mysql/20191009212518580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0phdmFfUm9hZF9GYXI=,size_16,color_FFFFFF,t_70)

2.  `conf/example/instance.properties`：instance.properties是针对要追踪的 mysql 的实例配置

    **我在每处需要更改的地方增加了  `<-----` 标记**

    ```bash
    #################################################
    ## mysql serverId , v1.0.26+ will autoGen
    # slaveId 必须配置, 不能和 mysql 的 id 重复
    canal.instance.mysql.slaveId=100  # <-----
    
    # enable gtid use true/false
    canal.instance.gtidon=false
    
    # position info
    # mysql 的位置信息
    canal.instance.master.address=hadoop102:3306 # <-----
    canal.instance.master.journal.name=
    canal.instance.master.position=
    canal.instance.master.timestamp=
    canal.instance.master.gtid=
    
    # rds oss binlog
    canal.instance.rds.accesskey=
    canal.instance.rds.secretkey=
    canal.instance.rds.instanceId=
    
    # table meta tsdb info
    canal.instance.tsdb.enable=true
    #canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
    #canal.instance.tsdb.dbUsername=canal
    #canal.instance.tsdb.dbPassword=canal
    
    #canal.instance.standby.address =
    #canal.instance.standby.journal.name =
    #canal.instance.standby.position =
    #canal.instance.standby.timestamp =
    #canal.instance.standby.gtid=
    
    # username/password
    # 用户名和密码
    canal.instance.dbUsername=root # <-----
    canal.instance.dbPassword=aaa # <-----
    canal.instance.connectionCharset = UTF-8
    canal.instance.defaultDatabaseName =
    # enable druid Decrypt database password
    canal.instance.enableDruid=false
    #canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==
    
    # table regex
    canal.instance.filter.regex=.*\\..*
    # table black regex
    canal.instance.filter.black.regex=
    
    # mq config
    canal.mq.topic=example # <-----
    canal.mq.partition=0
    # hash partition config
    #canal.mq.partitionsNum=3
    #canal.mq.partitionHash=mytest.person:id,mytest.role:id
    #################################################
    
    ```

## 1.4 启动canal

```bash
bin/startup.sh
```

## 1.5 查看日志

canal 的日志在 `canal/logs/`  下

```bash
/opt/module/canal/logs >> ls
canal	example
```

## 1.6 关闭canal

```bash
bin/stop.sh
```

---
# 二、从 canal 读取数据到Kafka

## 2.1 创建 maven 工程

添加依赖:

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/com.alibaba.otter/canal.client -->
    <!--canal 客户端, 从 canal 服务器读取数据-->
    <dependency>
        <groupId>com.alibaba.otter</groupId>
        <artifactId>canal.client</artifactId>
        <version>1.1.2</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients -->
    <!-- kafka 客户端 -->
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>0.11.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.kino.dw</groupId>
        <artifactId>gmall-common</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

```

## 2.2 从 canal 服务读取数据的客户端

我们通过：`CanalConnectors` 工具类获取到 canal 连接，通过： `connector.get(100)` 获取到一个 `Message`，我们需要的数据就是从 `Message` 获取而来，具体关系有些复杂，详看下面的解释：


-   `Message`：get一次获取一个Message, 一个Message表示一批数据, 看成多条sql语句执行的结果
-   `Entry`：一个Message封装多个Entry, 一个Entry可以看出一条sql语句执行的多行结果
-   `StoreValue`：一个 Entry封装一个 storeValue, 可以看到是是数据序列化形式
-   `RowChange`：从StoreValue里面解析出来的数据类型RowData 一个rowChage会有多个RowData, 一个RowData封装了一行数据
-   `Column`：列

```scala
package com.kino.dw.gamallcanal

import java.net.InetSocketAddress

import com.alibaba.otter.canal.client.{CanalConnector, CanalConnectors}
import com.alibaba.otter.canal.protocol.CanalEntry.{EntryType, RowChange}
import com.alibaba.otter.canal.protocol.{CanalEntry, Message}
import com.kino.dw.gamallcanal.util.CanalHandler
import com.google.protobuf.ByteString

/**
  * @author kino
  * @date 2019/10/9 19:25
  * @version 1.0.0
  */
object CanalClient {
    def main(args: Array[String]): Unit = {
        // 1. 创建能连接到 Canal 的连接器对象
        val connector: CanalConnector = CanalConnectors.newSingleConnector(new InetSocketAddress("hadoop102", 11111), "example", "", "")
        // 2. 连接到 Canal
        connector.connect()
        // 3. 监控指定的表的数据的变化
        connector.subscribe("gmall.order_info")
        while (true) {
            // 4. 获取消息  (一个消息对应 多条sql 语句的执行)
            val msg: Message = connector.get(100) // 一次最多获取 100 条 sql
            // 5. 个消息对应多行数据发生了变化, 一个 entry 表示一条 sql 语句的执行
            val entries: java.util.List[CanalEntry.Entry] = msg.getEntries
            import scala.collection.JavaConversions._
            if (entries.size() > 0) {
                // 6. 遍历每行数据
                for (entry <- entries) {
                    // 7. EntryType.ROWDATA 只对这样的 EntryType 做处理
                    if (entry.getEntryType == EntryType.ROWDATA) {
                        // 8. 获取到这行数据, 但是这种数据不是字符串, 所以要解析
                        val value: ByteString = entry.getStoreValue
                        val rowChange: RowChange = RowChange.parseFrom(value)
                        // 9.定义专门处理的工具类: 参数 1 表名, 参数 2 事件类型(插入, 删除等), 参数 3: 具体的数据
                        CanalHandler.handle(entry.getHeader.getTableName, rowChange.getEventType, rowChange.getRowDatasList)
                    }
                }
                
            } else {
                println("没有抓取到数据...., 2s 之后重新抓取")
                Thread.sleep(2000)
            }
        }
        
    }
}
```

## 2.3 专门处理数据的工具类： `CanalHandler`

```scala
package com.kino.dw.canal.utils

import java.util

import com.alibaba.fastjson.JSONObject
import com.alibaba.otter.canal.protocol.CanalEntry
import com.alibaba.otter.canal.protocol.CanalEntry.EventType

/**
  * @author kino
  * @date 2019/10/9 19:33
  * @version 1.0.0
  */
object CanalHandler {
    def handler(tableName: String, rowDataList: util.List[CanalEntry.RowData], eventType: CanalEntry.EventType) = {
        import scala.collection.JavaConversions._
        if(tableName == "order_info" && eventType == EventType.INSERT && !rowDataList.isEmpty){
            for(rowData <- rowDataList){
                val jsonObj = new JSONObject()
                val columns: util.List[CanalEntry.Column] = rowData.getAfterColumnsList // 一行数据所有的列
                for(column <- columns){
                    // 此处读取到的数据 类似于 poi 读取 excel，挨个读取一行中的某个列
                    val key: String = column.getName
                    val value: String = column.getValue
                    jsonObj.put(key, value)
                }
                // 写入到kafja
                println(jsonObj.toJSONString)
            }
        }
    }
}
```

