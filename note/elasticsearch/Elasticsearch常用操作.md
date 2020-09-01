


---

具体操作在 Kibana WebUI 中查询 ES 数据
![Kibana WebUI Search Es](../../img/elasticsearch/常用操作/Kibana%20WebUI.png)

# 一、查看 ES 中的索引
```bash
GET /_cat/indices?v

结果
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_task_manager_1   Y_WipHlMRXWifwjdlynP9A   1   0          2            0     39.8kb         39.8kb
green  open   .apm-agent-configuration AWBEbuU9TeeQgNJmWszQdA   1   0          0            0       283b           283b
yellow open   movie_index              ONKrA22AT06d9FxLZVgmXA   1   1          3            0      5.4kb          5.4kb
green  open   .kibana_1                BVGE3McUS-ikx1B-SseaYQ   1   0         19            3     20.7kb         20.7kb
```

![查看 ES 中的索引](../../img/elasticsearch/常用操作/查看%20ES%20中的索引.png)