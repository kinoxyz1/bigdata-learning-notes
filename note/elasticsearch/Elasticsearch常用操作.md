


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
 
# 二、增加索引
```bash
PUT /movie_index
```

# 三、删除索引
```bash
DELETE movie_index
```

# 四、新增文档
```bash
PUT /movie_index/_doc/1
{
  "id": 1,
  "name":"operation red sea",
  "doubanScore": 8.5,
  "actorList": [
    {"id": 1, "name": "zhang yi"},
    {"id": 2, "name": "hai qing"},
    {"id": 3, "name": "zhang han yu"}
  ]
}
# 或者
POST /movie_index/_doc/1
{
  "id": 1,
  "name":"operation red sea",
  "doubanScore": 8.5,
  "actorList": [
    {"id": 1, "name": "zhang yi"},
    {"id": 2, "name": "hai qing"},
    {"id": 3, "name": "zhang han yu"}
  ]
}
```
注意:

- 新增文档需要注意 PUT 和 POST 的区别
- PUT 需要指定文档的 id, 否则会报错
- POST 指不指定都可以, 如果不指定, 会自动生成一个id
```bash
POST /movie_index/_doc
{
  "id": 1,
  "name":"operation red sea",
  "doubanScore": 8.5,
  "actorList": [
    {"id": 1, "name": "zhang yi"},
    {"id": 2, "name": "hai qing"},
    {"id": 3, "name": "zhang han yu"}
  ]
}
```

# 五、搜索 type 全部数据


# 六、查找指定 id 的 document 数据


# 七、修改 document


# 八、删除 document


# 九、按条件查询(全部)


# 十、按照字段的分词查询


# 十一、按照字段的属性查询


# 十二、按照短语查询


# 十三、模糊查询


# 十四、过滤(查询后过滤)


# 十五、过滤(查询前过滤)


# 十六、按范围过滤


# 十七、排序


# 十八、分页查询


# 十九、查询指定的字段


# 二十、聚合