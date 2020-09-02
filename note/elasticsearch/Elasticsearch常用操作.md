


---

具体操作是在 Kibana WebUI 中查询 ES 数据

启动 Kibana 和 ElasticSearch, 在浏览器输入: http://自己的IP地址:5601/app/kibana

![Kibana WebUI Search Es](../../img/elasticsearch/常用操作/Kibana%20WebUI.png)

# 一、查看 ES 中的索引
```bash
GET /_cat/indices?v

结果
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_task_manager_1   Y_WipHlMRXWifwjdlynP9A   1   0          2            0     39.8kb         39.8kb
green  open   .apm-agent-configuration AWBEbuU9TeeQgNJmWszQdA   1   0          0            0       283b           283b
green  open   .kibana_1                BVGE3McUS-ikx1B-SseaYQ   1   0         19            3     20.7kb         20.7kb
```
![查看 ES 中的索引](../../img/elasticsearch/常用操作/查看%20ES%20中的索引.png)

>表头说明:

表头 | 说明
---- | ----
health | green(集群完整) yellow(单点正常、集群不完整) red(单点不正常)
status | 是否能使用
index | 索引名
uuid | 索引统一编号
pri | 主节点几个
rep |从节点几个
docs.count | 文档数
docs.deleted | 文档被删了多少
store.size | 整体占空间大小
pri.store.size | 主节点占
 