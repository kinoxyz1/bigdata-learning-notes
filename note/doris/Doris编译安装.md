






---
# 一、编译方式
官方提供了两种编译方式:
1. 官方提供了集成了编译环境的 Docker 镜像
2. 手动安装环境进行编译

因为服务器的环境难调, 所以选择用下载 Docker 镜像, 在容器中编译

# 二、使用 Docker 镜像编译
## 2.1 下载镜像
需要先部署 Docker, 参考 Docker 部分的笔记
```bash
$ docker pull apachedoris/doris-dev:build-env
```
不同的 Doris 版本，需要下载对应的镜像版本


| 镜像版本   | commit id  | doris 版本  |
| ------ | ------------ |------------ |
apachedoris/doris-dev:build-env	before | ff0dd0d | 0.8.x, 0.9.x
apachedoris/doris-dev:build-env-1.1	|ff0dd0d	|0.10.x, 0.11.x
apachedoris/doris-dev:build-env-1.2|4ef5a8c|0.12.x, 0.13
apachedoris/doris-dev:build-env-1.3|	ad67dd3|	0.14.x 或更新版本

doris 0.14.0 版本仍然使用apachedoris/doris-dev:build-env-1.2 编译，之后的代码将使用apachedoris/doris-dev:build-env-1.3

## 2.2 运行镜像
建议以挂载本地 Doris 源码目录的方式运行镜像，这样编译的产出二进制文件会存储在宿主机中，不会因为镜像退出而消失。

同时，建议同时将镜像中 maven 的 .m2 目录挂载到宿主机目录，以防止每次启动镜像编译时，重复下载 maven 的依赖库。
```bash
$ docker run -it -v /your/local/.m2:/root/.m2 -v /your/local/incubator-doris-DORIS-0.12.0-release/:/root/incubator-doris-DORIS-0.12.0-release/ apachedoris/doris-dev:build-env
```

## 2.3 下载源码
下载地址: https://dist.apache.org/repos/dist/dev/incubator/doris/
```bash
$ wget https://dist.apache.org/repos/dist/dev/incubator/doris/0.12.0-rc01/apache-doris-0.12.0-incubating-src.tar.gz
```

## 2.4 编译前的准备
因为一些仓库的问题, 需要调整 pom 文件

1. 修改 fe/pom.xml
    ```bash
    <url>https://repo.spring.io/plugins-release/</url>
    修改为
    <url>https://repository.cloudera.com/artifactory/ext-release-local</url>
    
    <url>https://repository.cloudera.com/content/repositories/third-party/</url>
    修改为
    <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    ```

## 2.5 编译 
### 2.5.1 编译 fe 和 be
保证磁盘还有 50G 可用空间
```bash
$ sh build
```
### 2.5.2 编译 broker
```bash
$ cd fs_brokers/apache_hdfs_broker/
$ sh build
```

期间可能有的 jar 下不下来, 多试几次就好了

# 三、fe 镜像制作
进入编译好的 output 目录中去
```bash
$ vim Dockerfile_fe
FROM primetoninc/jdk:1.8
MAINTAINER Doris <doris@apache.org>

#RUN yum install net-tools -y

COPY fe /opt/fe

WORKDIR /opt/fe
RUN mkdir doris-meta

EXPOSE 8030 9030

ENTRYPOINT ["/opt/fe/bin/start_fe.sh"]
```
构建fe镜像, 创建并配置镜像映射文件doris-meta和conf, 启动容器
```bash
docker build -t doris/fe:0.12.0 -f Dockerfile_fe  .
docker run -itd \
  --name fe_1 \
  -p 8030:8030 \
  -p 9030:9030 \
  -v /app/doris/fe/conf:/opt/fe/conf \
  -v /app/doris/fe/log:/opt/fe/log \
  -v /app/doris/fe/doris-meta:/opt/fe/doris-meta \
  fe:1.0.0
```

# 四、be 镜像制作
```bash
$ vim Dockerfile_be
FROM primetoninc/jdk:1.8
MAINTAINER Doris <doris@apache.org>

COPY be /opt/be

WORKDIR /opt/be
RUN mkdir storage

EXPOSE 9050

ENTRYPOINT ["/opt/be/bin/start_be.sh"]
```

```bash
docker build -t doris/be:0.12.0 -f Dockerfile_be .
docker run -itd \
  --name be_1 \
  -p 9051:9050 \
  -v /app/doris/be/conf:/opt/be/conf \
  -v /app/doris/be/storage:/opt/be/storage \
  be:1.0.0
docker run -itd \
  --name be_2 \
  -p 9152:9050 \
  -v /app/doris/be/conf:/opt/be/conf \
  -v /app/doris/be/storage:/opt/be/storage \
  be:1.0.0
docker run -itd \
  --name be_3 \
  -p 9253:9050 \
  -v /app/doris/be/conf:/opt/be/conf \
  -v /app/doris/be/storage:/opt/be/storage \
  be:1.0.0
```