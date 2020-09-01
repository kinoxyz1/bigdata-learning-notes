
* [一、安装 Docker](#%E4%B8%80%E5%AE%89%E8%A3%85-docker)
* [二、下载 Elasticsearch 镜像](#%E4%BA%8C%E4%B8%8B%E8%BD%BD-elasticsearch-%E9%95%9C%E5%83%8F)
* [三、创建网络](#%E4%B8%89%E5%88%9B%E5%BB%BA%E7%BD%91%E7%BB%9C)
* [四、查看镜像](#%E5%9B%9B%E6%9F%A5%E7%9C%8B%E9%95%9C%E5%83%8F)
* [五、运行](#%E4%BA%94%E8%BF%90%E8%A1%8C)
* [六、浏览器查看](#%E5%85%AD%E6%B5%8F%E8%A7%88%E5%99%A8%E6%9F%A5%E7%9C%8B)

---

# 一、安装 Docker
[CentOS7安装Docker](../../note/docker/CentOS7安装Docker.md)

# 二、下载 Elasticsearch 镜像
```bash
[root@docker1 ~]# docker pull elasticsearch:7.6.2
```

# 三、创建网络
如果需要安装kibana等其他，需要创建一个网络，名字任意取，让他们在同一个网络，使得es和kibana通信
```bash
[root@docker1 ~]# docker network create esnet
```

# 四、查看镜像
```bash
[root@docker1 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
elasticsearch       7.6.2               f29a1ee41030        5 months ago        791MB
```

# 五、运行
```bash
[root@docker1 ~]# docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 --network esnet -e "discovery.type=single-node"  -d elasticsearch:7.6.2

# docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 --network esnet -e "discovery.type=single-node" -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins -v /mydata/elasticsearch/logs:/usr/share/elasticsearch/logs -d elasticsearch:7.6.2
```

# 六、浏览器查看
在浏览器中输入: http://IP:9200/, 可以看到如下结果
```json
{
  "name" : "313a91cdee55",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "dbW0TIPDRHaVfq56Cpoyhg",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```