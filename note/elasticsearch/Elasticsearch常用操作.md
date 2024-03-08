
* [一、查看 ES 中的索引](#%E4%B8%80%E6%9F%A5%E7%9C%8B-es-%E4%B8%AD%E7%9A%84%E7%B4%A2%E5%BC%95)
* [二、增加索引](#%E4%BA%8C%E5%A2%9E%E5%8A%A0%E7%B4%A2%E5%BC%95)
* [三、删除索引](#%E4%B8%89%E5%88%A0%E9%99%A4%E7%B4%A2%E5%BC%95)
* [四、新增文档](#%E5%9B%9B%E6%96%B0%E5%A2%9E%E6%96%87%E6%A1%A3)
* [五、搜索 type 全部数据](#%E4%BA%94%E6%90%9C%E7%B4%A2-type-%E5%85%A8%E9%83%A8%E6%95%B0%E6%8D%AE)
* [六、查找指定 id 的 document 数据](#%E5%85%AD%E6%9F%A5%E6%89%BE%E6%8C%87%E5%AE%9A-id-%E7%9A%84-document-%E6%95%B0%E6%8D%AE)
* [七、修改 document](#%E4%B8%83%E4%BF%AE%E6%94%B9-document)
  * [7\.1 整个 document 替换](#71-%E6%95%B4%E4%B8%AA-document-%E6%9B%BF%E6%8D%A2)
  * [7\.2 只修改某个字段](#72-%E5%8F%AA%E4%BF%AE%E6%94%B9%E6%9F%90%E4%B8%AA%E5%AD%97%E6%AE%B5)
* [八、删除 document](#%E5%85%AB%E5%88%A0%E9%99%A4-document)
* [九、按条件查询(全部)](#%E4%B9%9D%E6%8C%89%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2%E5%85%A8%E9%83%A8)
* [十、按照字段的分词查询](#%E5%8D%81%E6%8C%89%E7%85%A7%E5%AD%97%E6%AE%B5%E7%9A%84%E5%88%86%E8%AF%8D%E6%9F%A5%E8%AF%A2)
* [十一、按照字段的属性查询](#%E5%8D%81%E4%B8%80%E6%8C%89%E7%85%A7%E5%AD%97%E6%AE%B5%E7%9A%84%E5%B1%9E%E6%80%A7%E6%9F%A5%E8%AF%A2)
* [十二、按照短语查询](#%E5%8D%81%E4%BA%8C%E6%8C%89%E7%85%A7%E7%9F%AD%E8%AF%AD%E6%9F%A5%E8%AF%A2)
* [十三、模糊查询](#%E5%8D%81%E4%B8%89%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2)
* [十四、过滤(查询后过滤)](#%E5%8D%81%E5%9B%9B%E8%BF%87%E6%BB%A4%E6%9F%A5%E8%AF%A2%E5%90%8E%E8%BF%87%E6%BB%A4)
* [十五、过滤(查询前过滤)](#%E5%8D%81%E4%BA%94%E8%BF%87%E6%BB%A4%E6%9F%A5%E8%AF%A2%E5%89%8D%E8%BF%87%E6%BB%A4)
* [十六、按范围过滤](#%E5%8D%81%E5%85%AD%E6%8C%89%E8%8C%83%E5%9B%B4%E8%BF%87%E6%BB%A4)
* [十七、排序](#%E5%8D%81%E4%B8%83%E6%8E%92%E5%BA%8F)
* [十八、分页查询](#%E5%8D%81%E5%85%AB%E5%88%86%E9%A1%B5%E6%9F%A5%E8%AF%A2)
* [十九、查询指定的字段](#%E5%8D%81%E4%B9%9D%E6%9F%A5%E8%AF%A2%E6%8C%87%E5%AE%9A%E7%9A%84%E5%AD%97%E6%AE%B5)
* [二十、每个演员参演了多少部电影](#%E4%BA%8C%E5%8D%81%E6%AF%8F%E4%B8%AA%E6%BC%94%E5%91%98%E5%8F%82%E6%BC%94%E4%BA%86%E5%A4%9A%E5%B0%91%E9%83%A8%E7%94%B5%E5%BD%B1)
* [二十一、每个演员参演电影的平均分是多少，并按评分排序](#%E4%BA%8C%E5%8D%81%E4%B8%80%E6%AF%8F%E4%B8%AA%E6%BC%94%E5%91%98%E5%8F%82%E6%BC%94%E7%94%B5%E5%BD%B1%E7%9A%84%E5%B9%B3%E5%9D%87%E5%88%86%E6%98%AF%E5%A4%9A%E5%B0%91%E5%B9%B6%E6%8C%89%E8%AF%84%E5%88%86%E6%8E%92%E5%BA%8F)

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

# 索引删除后不会释放磁盘，需要 forcemerge 
POST _forcemerge?only_expunge_deletes=true
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
POST /movie_index/_doc/2
{
  "id":2,
  "name":"operation meigong river",
  "doubanScore":8.0,
  "actorList":[  
    {"id":3,"name":"zhang han yu"}
  ]
}

POST /movie_index/_doc/3
{
  "id":3,
  "name":"incident red sea",
  "doubanScore":5.0,
  "actorList":[  
    {"id":4,"name":"zhang chen"}
  ]
}
# 或者
POST /movie_index/_doc/1
{
  "id": 1,
  "name":"operation red sea",
  "doubanScore": 8.5,
  "actorList": [
    {"id": 1,"name": "zhang yi"},
    {"id": 2,"name": "hai qing"},
    {"id": 3,"name": "zhang han yu"}
  ]
}
POST /movie_index/_doc/2
{
  "id":2,
  "name":"operation meigong river",
  "doubanScore":8.0,
  "actorList":[  
    {"id":3,"name":"zhang han yu"}
  ]
}
POST /movie_index/_doc/3
{
  "id":3,
  "name":"incident red sea",
  "doubanScore":5.0,
  "actorList":[  
    {"id":4,"name":"zhang chen"}
  ]
}
```
注意:

- 新增文档需要注意 PUT 和 POST 的区别
- PUT 需要指定文档的 id, 否则会报错
- POST 指不指定都可以, 如果不指定, 会自动生成一个id
- 如果之前没建过 index 或者 type, es 会自动创建。
```bash
POST /movie_index/_doc
{
  "id": 1,
  "name":"operation red sea",
  "doubanScore": 8.5,
  "actorList": [
    {"id": 1,"name": "zhang yi"},
    {"id": 2,"name": "hai qing"},
    {"id": 3,"name": "zhang han yu"}
  ]
}
POST /movie_index/_doc
{
  "id":2,
  "name":"operation meigong river",
  "doubanScore":8.0,
  "actorList":[  
    {"id":3,"name":"zhang han yu"}
  ]
}
POST /movie_index/_doc
{
  "id":3,
  "name":"incident red sea",
  "doubanScore":5.0,
  "actorList":[  
    {"id":4,"name":"zhang chen"}
  ]
}
```

# 五、搜索 type 全部数据
```bash
GET movie_index/_search
```

# 六、查找指定 id 的 document 数据
```bash
GET /movie_index/_doc/MjSpTXQBR8s2ISKoGB6H
```

# 七、修改 document
## 7.1 整个 document 替换
```bash
PUT movie_index/_doc/MjSpTXQBR8s2ISKoGB6H
{
  "id":"3",
  "name":"incident red sea",
  "doubanScore":"8.0",
  "actorList":[  
    {"id":"1","name":"zhang chen"}
  ]
}
```
## 7.2 只修改某个字段
```bash
POST movie_index/_doc/MjSpTXQBR8s2ISKoGB6H/_update
{
  "doc": {
    "doubanScore": "8.1"
  }
}
```

# 八、删除 document
```bash
DELETE movie_index/_doc/MjSpTXQBR8s2ISKoGB6H
```

# 九、按条件查询(全部)
```bash
GET movie_index/_search
{
  "query": {
    "match_all": {}
  }
}
```

# 十、按照字段的分词查询
```bash
GET movie_index/_search
{
  "query": {
    "match": {
      "name": "sea"
    }
  }
}
```
查询到的结果如下(注意看中间的注释):
```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      ## 搜索到的记录总数, 等于下面 hits 的记录数
      "value" : 2,
      ## 搜索的关系是 eq(等于)
      "relation" : "eq"
    },
    ## 结果中最高的那个权重(类似百度关键字搜索到的第一条信息就是权重最高的, ps: 除去买广告...)
    "max_score" : 0.4700036,   
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 0.4700036,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NjTfTXQBR8s2ISKotR5J",
        "_score" : 0.4700036,
        "_source" : {
          "id" : 3,
          "name" : "incident red sea",
          "doubanScore" : 5.0,
          "actorList" : [
            {
              "id" : 4,
              "name" : "zhang chen"
            }
          ]
        }
      }
    ]
  }
}
```

# 十一、按照字段的属性查询
```bash
GET movie_index/_search
{
  "query": {
    "match": {
      "actorList.name": "zhang"
    }
  }
}
```
查询到的结果如下:
```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 0.16786805,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NjTfTXQBR8s2ISKotR5J",
        "_score" : 0.16786805,
        "_source" : {
          "id" : 3,
          "name" : "incident red sea",
          "doubanScore" : 5.0,
          "actorList" : [
            {
              "id" : 4,
              "name" : "zhang chen"
            }
          ]
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 0.15162274,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NTTfTXQBR8s2ISKoqh75",
        "_score" : 0.14874382,
        "_source" : {
          "id" : 2,
          "name" : "operation meigong river",
          "doubanScore" : 8.0,
          "actorList" : [
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      }
    ]
  }
}
```

# 十二、按照短语查询
按照短语查询的意思是指, 匹配某个 field 的整个内容, 不再利用分词技术
```bash
GET movie_index/_search
{
  "query": {
    "match_phrase": {
      "name": "operation red"
    }
  }
}
```
查询到的结果如下:
```json
{
  "took" : 17,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.9400072,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 0.9400072,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      }
    ]
  }
}
```
说明: 搞不清楚可以和 `按照字段的分词查询` 对比一下, 例如上述结果和如下语句结果对比:
```bash
GET movie_index/_search
{
  "query": {
    "match": {
      "name": "operation red"
    }
  }
}
```

# 十三、模糊查询
校正匹配分词, 当一个单词都无法准确匹配, es 通过一种算法对非常接近的单词也给与一定的评分, 能够查询出来, 但是消耗更多的性能.
```bash
GET movie_index/_search
{
  "query": {
    "fuzzy": {
      "name": "red"
    }
  }
}
```
查询到的结果如下:
```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.4700036,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 0.4700036,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NjTfTXQBR8s2ISKotR5J",
        "_score" : 0.4700036,
        "_source" : {
          "id" : 3,
          "name" : "incident red sea",
          "doubanScore" : 5.0,
          "actorList" : [
            {
              "id" : 4,
              "name" : "zhang chen"
            }
          ]
        }
      }
    ]
  }
}
```

# 十四、过滤(查询后过滤)
```bash
GET movie_index/_search
{
  "query": {
    "match": {
      "name": "red"
    }
  },
  "post_filter": {
    "term": {
      "actorList.id": "3"
    }
  }
}
```
查询到的结果如下:
```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.4700036,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 0.4700036,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      }
    ]
  }
}
```

# 十五、过滤(查询前过滤)
```bash
GET movie_index/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": 
            {"actorList.id": "3"}},
        {  "term": 
            {"actorList.id": "1"}
        }
      ],
      "must": [
        {
          "match": {
            "actorList.name": "yi"
          }
        }
      ]
    }
  }
}
```
查询到的结果如下:
```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.7505475,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 0.7505475,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      }
    ]
  }
}
```

# 十六、按范围过滤
```bash
GET movie_index/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "doubanScore": {
            "gt": 5,
            "lt": 9
          }
        }
      }
    }
  }
}
```
查询到的结果如下:
```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 0.0,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NTTfTXQBR8s2ISKoqh75",
        "_score" : 0.0,
        "_source" : {
          "id" : 2,
          "name" : "operation meigong river",
          "doubanScore" : 8.0,
          "actorList" : [
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      }
    ]
  }
}
```


# 十七、排序
```bash
GET movie_index/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "doubanScore": {
        "order": "asc"
      }
    }
  ]
}
```
查询到的结果如下:
```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NjTfTXQBR8s2ISKotR5J",
        "_score" : null,
        "_source" : {
          "id" : 3,
          "name" : "incident red sea",
          "doubanScore" : 5.0,
          "actorList" : [
            {
              "id" : 4,
              "name" : "zhang chen"
            }
          ]
        },
        "sort" : [
          5.0
        ]
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NTTfTXQBR8s2ISKoqh75",
        "_score" : null,
        "_source" : {
          "id" : 2,
          "name" : "operation meigong river",
          "doubanScore" : 8.0,
          "actorList" : [
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        },
        "sort" : [
          8.0
        ]
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : null,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        },
        "sort" : [
          8.5
        ]
      }
    ]
  }
}
```

# 十八、分页查询
```bash
GET movie_index/_search
{
  "query": {
    "match_all": {}
  },
  "from": 1,
  "size": 1
}
```
查询到的结果如下:
```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NTTfTXQBR8s2ISKoqh75",
        "_score" : 1.0,
        "_source" : {
          "id" : 2,
          "name" : "operation meigong river",
          "doubanScore" : 8.0,
          "actorList" : [
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      }
    ]
  }
}
```

# 十九、查询指定的字段
```bash
GET movie_index/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["name", "doubanScore"]
}
```
查询到的结果如下:
```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 1.0,
        "_source" : {
          "doubanScore" : 8.5,
          "name" : "operation red sea"
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NTTfTXQBR8s2ISKoqh75",
        "_score" : 1.0,
        "_source" : {
          "doubanScore" : 8.0,
          "name" : "operation meigong river"
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NjTfTXQBR8s2ISKotR5J",
        "_score" : 1.0,
        "_source" : {
          "doubanScore" : 5.0,
          "name" : "incident red sea"
        }
      }
    ]
  }
}
```


# 二十、每个演员参演了多少部电影
```bash
GET movie_index/_search
{
  ## 显示 0 条记录
  "size": 0, 
  "aggs": {
    "groupby_actor": {
      "terms": {
        "field": "actorList.name.keyword",
        ## 聚合后的记录显示 10 条
        "size": 10
      }
    }
  }
}
```
查询到的结果如下:
```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "groupby_actor" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "zhang han yu",
          "doc_count" : 2
        },
        {
          "key" : "hai qing",
          "doc_count" : 1
        },
        {
          "key" : "zhang chen",
          "doc_count" : 1
        },
        {
          "key" : "zhang yi",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

# 二十一、每个演员参演电影的平均分是多少，并按评分排序
```bash
GET movie_index/_search
{ 
  "aggs": {
    "groupby_actor_id": {
      "terms": {
        "field": "actorList.name.keyword" ,
        "order": {
          "avg_score": "desc"
          }
      },
      "aggs": {
        "avg_score":{
          "avg": {
            "field": "doubanScore" 
          }
        }
       }
    } 
  }
}
```
查询到的结果如下:
```json
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NDTfTXQBR8s2ISKoox5W",
        "_score" : 1.0,
        "_source" : {
          "id" : 1,
          "name" : "operation red sea",
          "doubanScore" : 8.5,
          "actorList" : [
            {
              "id" : 1,
              "name" : "zhang yi"
            },
            {
              "id" : 2,
              "name" : "hai qing"
            },
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NTTfTXQBR8s2ISKoqh75",
        "_score" : 1.0,
        "_source" : {
          "id" : 2,
          "name" : "operation meigong river",
          "doubanScore" : 8.0,
          "actorList" : [
            {
              "id" : 3,
              "name" : "zhang han yu"
            }
          ]
        }
      },
      {
        "_index" : "movie_index",
        "_type" : "_doc",
        "_id" : "NjTfTXQBR8s2ISKotR5J",
        "_score" : 1.0,
        "_source" : {
          "id" : 3,
          "name" : "incident red sea",
          "doubanScore" : 5.0,
          "actorList" : [
            {
              "id" : 4,
              "name" : "zhang chen"
            }
          ]
        }
      }
    ]
  },
  "aggregations" : {
    "groupby_actor_id" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "hai qing",
          "doc_count" : 1,
          "avg_score" : {
            "value" : 8.5
          }
        },
        {
          "key" : "zhang yi",
          "doc_count" : 1,
          "avg_score" : {
            "value" : 8.5
          }
        },
        {
          "key" : "zhang han yu",
          "doc_count" : 2,
          "avg_score" : {
            "value" : 8.25
          }
        },
        {
          "key" : "zhang chen",
          "doc_count" : 1,
          "avg_score" : {
            "value" : 5.0
          }
        }
      ]
    }
  }
}
```
