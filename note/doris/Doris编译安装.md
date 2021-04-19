






---
# 一、编译方式
官方提供了两种编译方式:
1. 官方提供了集成了编译环境的 Docker 镜像
2. 手动安装环境进行编译

因为服务器的准备环境难调, 所以选择用下载 Docker 镜像, 在容器中编译

# 二、使用 Docker 镜像编译
## 2.1 下载镜像
需要先部署 Docker, 参考 Docker 部分的笔记
```bash
$ docker pull apachedoris/doris-dev:build-env-1.2
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
$ docker run -it -v /app/doris/.m2:/root/.m2 -v /app/doris/incubator-doris-DORIS-0.12.0-release/:/root/incubator-doris-DORIS-0.12.0-release/ apachedoris/doris-dev:build-env-1.2
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
保证磁盘还有 50G 可用空间
### 2.5.1 编译 fe 和 be
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
进入编译好的 output 目录中去, 编写 fe 的Dockerfile
```bash
$ vim Dockerfile_fe
FROM centos:7.2.1511
RUN mkdir /app/doris/ -p
# copy jdk and palo binary
COPY jdk1.8.0_281/ /usr/java/jdk1.8.0_281/
COPY fe/ /app/doris/fe/
# set java home
ENV JAVA_HOME //usr/java/jdk1.8.0_281/
ENV PATH $PATH:$JAVA_HOME/bin
RUN echo javac
# set fe port: http/thrift/mysql/bdbje
EXPOSE 8030 9020 9030 9010
# fe log and meta-data
VOLUME "/app/doris/fe/conf" "/app/doris/fe/log" "/app/doris/fe/palo-meta"
WORKDIR /app/doris/fe/
CMD "bin/start_fe.sh"
```
构建fe镜像, 创建并配置镜像映射文件doris-meta和conf, 启动容器
```bash
docker build -t doris/fe:0.12.0 -f Dockerfile_fe  .
docker run -itd --name doris_fe_node1 -p 9030:9030 -p 8030:8030 doris/fe:0.12.0
```

# 四、be 镜像制作
进入编译好的 output 目录中去, 编写 be 的Dockerfile
```bash
$ vim Dockerfile_be
FROM centos:7.2.1511
RUN mkdir /home/palo/run/ -p
COPY be/ /home/palo/run/be/
EXPOSE 9060 9070 8040 9050
VOLUME ["/home/palo/run/be/conf", "/home/palo/run/be/log", "/home/palo/run/be/data/"]
WORKDIR /home/palo/run/be/

# 我编译的时候有报错句柄问题, 所以加了如下内容
RUN echo "* soft nofile 204800"  >> /etc/security/limits.conf
RUN echo "* hard nofile 204800"  >> /etc/security/limits.conf
RUN echo "* soft nproc 204800"  >> /etc/security/limits.conf
RUN echo "* hard nproc 204800 "  >> /etc/security/limits.conf
RUN echo   fs.file-max = 6553560  >> /etc/sysctl.conf
RUN cat /etc/security/limits.conf

# 我编译的时候有异常, log下显示是没有/home/disk1/palo 文件, 所以这里一次性创建一点
RUN mkdir -p /home/disk1/palo
RUN mkdir -p /home/disk2/palo
RUN mkdir -p /home/disk3/palo
RUN mkdir -p /home/disk4/palo
RUN mkdir -p /home/disk5/palo

CMD "bin/start_be.sh" 
```

```bash
docker build -t doris/be:0.12.0 -f Dockerfile_be .
docker run -itd --name doris_be1 doris/be:0.12.0
docker run -itd --name doris_be2 doris/be:0.12.0
docker run -itd --name doris_be3 doris/be:0.12.0
```

# 五、BE 加入到集群中
选择一台装有 mysql 客户端的机器, 登录doris: `mysql -h<宿主机ip> -P9030 -uroot -p`
```bash
# IP 需要填容器的, 可以使用 docker inspect <CONTAINER ID> | grep IPAddress 查看
$ ALTER SYSTEM ADD BACKEND "172.17.0.4:9050";
```

