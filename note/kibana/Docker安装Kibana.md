
* [一、安装 Docker](#%E4%B8%80%E5%AE%89%E8%A3%85-docker)
* [二、下载 Kibana 镜像](#%E4%BA%8C%E4%B8%8B%E8%BD%BD-kibana-%E9%95%9C%E5%83%8F)
* [三、运行](#%E4%B8%89%E8%BF%90%E8%A1%8C)
* [四、访问](#%E5%9B%9B%E8%AE%BF%E9%97%AE)

---

# 一、安装 Docker
[CentOS7安装Docker](../../note/docker/CentOS7安装Docker.md)


# 二、下载 Kibana 镜像
```bash
[root@docker1 ~]# docker pull kibana:7.6.2
```

# 三、运行
```bash
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://自己的IP地址:9200 -p 5601:5601 -d kibana:7.6.2
```
进入容器修改相应内容
```bash
server.port: 5601
server.host: 0.0.0.0
elasticsearch.hosts: [ "http://自己的IP地址:9200" ]
i18n.locale: "Zh-CN"
```

# 四、访问
http://自己的IP地址:5601/app/kibana