

* [一、安装](#%E4%B8%80%E5%AE%89%E8%A3%85)
  * [1\.1 网址](#11-%E7%BD%91%E5%9D%80)
  * [1\.2 本地 RPM 包安装](#12-%E6%9C%AC%E5%9C%B0-rpm-%E5%8C%85%E5%AE%89%E8%A3%85)
    * [1\.2\.1 上传5个文件(在 \.\./package/ClickHouse/安装包/\* 下)到 /opt/software/](#121-%E4%B8%8A%E4%BC%A05%E4%B8%AA%E6%96%87%E4%BB%B6%E5%9C%A8-packageclickhouse%E5%AE%89%E8%A3%85%E5%8C%85-%E4%B8%8B%E5%88%B0-optsoftware)
    * [1\.2\.2 分别安装这 5个 rpm文件](#122-%E5%88%86%E5%88%AB%E5%AE%89%E8%A3%85%E8%BF%99-5%E4%B8%AA-rpm%E6%96%87%E4%BB%B6)
    * [1\.2\.3 启动ClickServer](#123-%E5%90%AF%E5%8A%A8clickserver)
    * [1\.2\.4 使用 client 连接 server](#124-%E4%BD%BF%E7%94%A8-client-%E8%BF%9E%E6%8E%A5-server)
  * [1\.3 Yum 安装](#13-yum-%E5%AE%89%E8%A3%85)
  * [1\.4 启动](#14-%E5%90%AF%E5%8A%A8)
    * [1\.4\.1 服务端启动](#141-%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%90%AF%E5%8A%A8)
    * [1\.4\.2 客户端启动](#142-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%90%AF%E5%8A%A8)
  * [1\.5 分布式集群安装](#15-%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E5%AE%89%E8%A3%85)
    * [1\.5\.1 在 kino101, kino102 上面执行之前的所有步骤](#151-%E5%9C%A8-kino101-kino102-%E4%B8%8A%E9%9D%A2%E6%89%A7%E8%A1%8C%E4%B9%8B%E5%89%8D%E7%9A%84%E6%89%80%E6%9C%89%E6%AD%A5%E9%AA%A4)
    * [1\.5\.2 三台机器修改配置文件config\.xml](#152-%E4%B8%89%E5%8F%B0%E6%9C%BA%E5%99%A8%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6configxml)
    * [1\.5\.3 在三台机器的 etc 目录下新建 metrika\.xml 文件](#153-%E5%9C%A8%E4%B8%89%E5%8F%B0%E6%9C%BA%E5%99%A8%E7%9A%84-etc-%E7%9B%AE%E5%BD%95%E4%B8%8B%E6%96%B0%E5%BB%BA-metrikaxml-%E6%96%87%E4%BB%B6)
    * [1\.5\.4 三台机器启动ClickServer](#154-%E4%B8%89%E5%8F%B0%E6%9C%BA%E5%99%A8%E5%90%AF%E5%8A%A8clickserver)
    
---

# 一、安装
## 1.1 网址

官网：https://clickhouse.yandex/

下载地址：http://repo.red-soft.biz/repos/clickhouse/stable/el6/

## 1.2 本地 RPM 包安装
### 1.2.1 上传5个文件(在 ../package/ClickHouse/安装包/* 下)到 `/opt/software/`
```bash
[root@kino100 software]# ls
clickhouse-client-1.1.54236-4.el6.x86_64.rpm      
clickhouse-server-1.1.54236-4.el6.x86_64.rpm
clickhouse-compressor-1.1.54236-4.el6.x86_64.rpm  
clickhouse-server-common-1.1.54236-4.el6.x86_64.rpm
clickhouse-debuginfo-1.1.54236-4.el6.x86_64.rpm
```
### 1.2.2 分别安装这 5个 rpm文件
```bash
[root@kino100 software]# rpm -ivh *.rpm
Preparing...                ########################################### [100%]
   1:clickhouse-server-commo########################################### [ 20%]
   2:clickhouse-server      ########################################### [ 40%]
   3:clickhouse-client      ########################################### [ 60%]
   4:clickhouse-debuginfo   ########################################### [ 80%]
   5:clickhouse-compressor  ########################################### [100%]
```
### 1.2.3 启动ClickServer
前台启动：
```bash
[root@kino100 software]# clickhouse-server --config-file=/etc/clickhouse-server/config.xml
```
后台启动：
```bash
[root@kino100 software]# nohup clickhouse-server --config-file=/etc/clickhouse-server/config.xml  >null 2>&1 &
[1] 2696
```

### 1.2.4 使用 client 连接 server
```bash
[root@kino100 software]# clickhouse-client 
ClickHouse client version 1.1.54236.
Connecting to localhost:9000.
Connected to ClickHouse server version 1.1.54236.

:)
```

## 1.3 Yum 安装
首先, 需要添加官方存储库: 
```bash
sudo yum install yum-utils
sudo rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
sudo yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/stable/x86_64
```
如果想要安装最新版本, 将 `stable` 替换为 `testing`(建议在测试环境中使用)

然后运行一下命令:
```bash
sudo yum install clickhouse-server clickhouse-client
```

## 1.4 启动
### 1.4.1 服务端启动
可以运行如下命令在后台启动服务:
```bash
sudo service clickhouse-server start
```
可以在 `/var/log/clickhouse-server/` 目录中查看日志。

如果服务没有启动，请检查配置文件: `/etc/clickhouse-server/config.xml`。

你也可以在控制台中直接启动服务:
```bash
clickhouse-server --config-file=/etc/clickhouse-server/config.xml
```

在这种情况下，日志将被打印到控制台中，这在开发过程中很方便。

如果配置文件在当前目录中，你可以不指定`–config-file`参数。它默认使用`./config.xml`。

### 1.4.2 客户端启动
```bash
clickhouse-client
```

默认情况下它使用’default’用户无密码的与localhost:9000服务建立连接。

客户端也可以用于连接远程服务，例如:
```bash
clickhouse-client --host=example.com
```

检查系统是否工作:
```bash
milovidov@hostname:~/work/metrica/src/src/Client$ ./clickhouse-client
ClickHouse client version 0.0.18749.
Connecting to localhost:9000.
Connected to ClickHouse server version 0.0.18749.

:) SELECT 1

SELECT 1

┌─1─┐
│ 1 │
└───┘

1 rows in set. Elapsed: 0.003 sec.

:)
```


## 1.5 分布式集群安装
### 1.5.1 在 kino101, kino102 上面执行之前的所有步骤
### 1.5.2 三台机器修改配置文件config.xml
```bash
[root@kino100 ~]# vim /etc/clickhouse-server/config.xml

<listen_host>::</listen_host>
<!-- <listen_host>::1</listen_host> -->
<!-- <listen_host>127.0.0.1</listen_host> -->

[root@kino101 ~]# vim /etc/clickhouse-server/config.xml

<listen_host>::</listen_host>
<!-- <listen_host>::1</listen_host> -->
<!-- <listen_host>127.0.0.1</listen_host> -->

[root@hadoop104 ~]# vim /etc/clickhouse-server/config.xml

<listen_host>::</listen_host>
<!-- <listen_host>::1</listen_host> -->
<!-- <listen_host>127.0.0.1</listen_host> -->
```


### 1.5.3 在三台机器的 etc 目录下新建 metrika.xml 文件
```bash
[root@kino100 ~]# vim /etc/metrika.xml
```
添加如下内容：
```xml
<yandex>
<clickhouse_remote_servers>
    <perftest_3shards_1replicas>
        <shard>
             <internal_replication>true</internal_replication>
            <replica>
                <host>hadoop102</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <replica>
                <internal_replication>true</internal_replication>
                <host>hadoop103</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <internal_replication>true</internal_replication>
            <replica>
                <host>hadoop104</host>
                <port>9000</port>
            </replica>
        </shard>
    </perftest_3shards_1replicas>
</clickhouse_remote_servers>


<zookeeper-servers>
  <node index="1">
    <host>kino100</host>
    <port>2181</port>
  </node>

  <node index="2">
    <host>kino101</host>
    <port>2181</port>
  </node>
  <node index="3">
    <host>kino102</host>
    <port>2181</port>
  </node>
</zookeeper-servers>


<macros>
    <replica>kino100</replica>
</macros>

<networks>
   <ip>::/0</ip>
</networks>


<clickhouse_compression>
<case>
  <min_part_size>10000000000</min_part_size>
                                             
  <min_part_size_ratio>0.01</min_part_size_ratio>                                                                                                                                       
  <method>lz4</method>
</case>
</clickhouse_compression>

</yandex>
```
注意：上面 <macros> 需要根据机器不同去修改
### 1.5.4 三台机器启动ClickServer
首先在三台机器开启Zookeeper
前台启动：
```bash
[root@kino software]# clickhouse-server --config-file=/etc/clickhouse-server/config.xml
```
后台启动：
```bash
[root@kino software]# nohup clickhouse-server --config-file=/etc/clickhouse-server/config.xml  >null 2>&1 &
[1] 2696
```