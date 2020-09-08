


---
# 一、集群规划
节点  | IP| 服务 
---- | ---- | ----
hadoop1 | 192.168.220.30 | zookeeper
hadoop2 | 192.168.220.31 | zookeeper
hadoop3 | 192.168.220.32 | zookeeper


# 二、下载安装包
https://downloads.apache.org/zookeeper/


# 三、解压安装
将文件解压在指定目录中 
```bash
[root@hadoop1 opt]# pwd
/opt
[root@hadoop1 opt]# ll
总用量 496424
-rw-r--r--. 1 root root  16402010 9月   7 18:51 zookeeper-3.4.5.tar.gz
[root@hadoop1 opt]# tar -zxvf zookeeper-3.4.5.tar.gz -C /usr/bigdata
[root@hadoop1 opt]# cd /usr/bigdata/
[root@hadoop1 bigdata]# ll
总用量 2
drwxr-xr-x. 12  501 games 4096 9月   7 19:11 zookeeper-3.4.5
```


# 四、配置服务器编号
① 在 zookeeper 根目录下创建 `zkData` 目录
```bash
[root@hadoop1 zookeeper-3.4.5]$ mkdir zkData
```
② 在 上面创建的 `zkData` 目录下创建 `myid` 文件, 并添加相应的编号
```bash
[root@hadoop1 zkData]$ vim myid
1
```



# 五、修改 zoo.cfg 文件
① 重命名 `/usr/bigdata/zookeeper-3.4.5/conf` 这个目录下的 zoo_sample.cfg 为 zoo.cfg
```bash
[root@hadoop1 conf]$ mv zoo_sample.cfg zoo.cfg
```
② 编辑 zoo.cfg 文件
```bash
[root@hadoop1 conf]$ vim zoo.cfg
```
③ 修改数据存储路径配置、添加日志路径
```bash
dataDir=/usr/bigdata/zookeeper-3.4.5/zkData
dataLogDir=/usr/bigdata/zookeeper-3.4.5/log
```
④ 增加如下配置
```bash
#######################cluster##########################
server.1=hadoop1:2888:3888
server.2=hadoop2:2888:3888
server.3=hadoop3:2888:3888
```
配置参数解读: `server.A=B:C:D`

- A是一个数字, 表示这个是第几号服务器;
   - 集群模式下配置一个文件myid, 这个文件在dataDir目录下, 这个文件里面有一个数据就是A的值, Zookeeper启动时读取此文件, 拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
- B是这个服务器的ip地址; 
- C是这个服务器与集群中的Leader服务器交换信息的端口; 
- D是万一集群中的Leader服务器挂了, 需要一个端口来重新进行选举, 选出一个新的Leader, 而这个端口就是用来执行选举时服务器相互通信的端口。


# 六、分发 zookeeper
```bash
[root@hadoop1 conf]$ scp -r /usr/bigdata/zookeeper-3.4.5 root@hadoop2://usr/bigdata
[root@hadoop1 conf]$ scp -r /usr/bigdata/zookeeper-3.4.5 root@hadoop3://usr/bigdata
```

# 七、修改 hadoop2、hadoop3 的 myid
```bash
[root@hadoop2 zkData]$ vim myid
2

[root@hadoop3 zkData]$ vim myid
3
```


# 八、进群操作
## 8.1 分别启动Zookeeper
```bash
[root@hadoop1 zookeeper-3.4.5]$ bin/zkServer.sh start
[root@hadoop2 zookeeper-3.4.5]$ bin/zkServer.sh start
[root@hadoop3 zookeeper-3.4.5]$ bin/zkServer.sh start
```

## 8.2 查看状态
```bash
[root@hadoop1 zookeeper-3.4.5]# bin/zkServer.sh status
JMX enabled by default
Using config: /usr/bigdata/zookeeper-3.4.5/bin/../conf/zoo.cfg
Mode: followe
```

## 8.3 客户端连接
```bash
[root@hadoop1 zookeeper-3.4.5]# bin/zkCli.sh
```

