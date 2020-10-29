


命令基本语法 | 功能描述
--- | ---
help | 显示所有操作命令
ls path [watch] | 使用 ls 命令来查看当前znode中所包含的内容
ls2 path [watch] | 查看当前节点数据并能看到更新次数等数据
create | 普通创建 </br> -s  含有序列 </br> -e  临时（重启或者超时消失）
get path [watch] | 获得节点的值
set | 设置节点的具体值
stat | 查看节点状态
delete | 删除节点
rmr | 递归删除节点


# 1．启动客户端
```bash
[root@hadoop1 zookeeper-3.6.2-bin]# bin/zkCli.sh
```
# 2．显示所有操作命令
```bash
[zk: localhost:2181(CONNECTED) 1] help
```
# 3．查看当前znode中所包含的内容
```bash
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
```
# 4．查看当前节点详细数据
```bash
[zk: localhost:2181(CONNECTED) 1] ls2 /
[zookeeper]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```
# 5．分别创建2个普通节点
```bash
[zk: localhost:2181(CONNECTED) 3] create /sanguo "jinlian"
Created /sanguo
[zk: localhost:2181(CONNECTED) 4] create /sanguo/shuguo "liubei"
Created /sanguo/shuguo
```
# 6．获得节点的值
```bash
[zk: localhost:2181(CONNECTED) 5] get /sanguo
jinlian
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000003
mtime = Wed Aug 29 00:03:23 CST 2018
pZxid = 0x100000004
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1
[zk: localhost:2181(CONNECTED) 6]
[zk: localhost:2181(CONNECTED) 6] get /sanguo/shuguo
liubei
cZxid = 0x100000004
ctime = Wed Aug 29 00:04:35 CST 2018
mZxid = 0x100000004
mtime = Wed Aug 29 00:04:35 CST 2018
pZxid = 0x100000004
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```
# 7．创建短暂节点
```bash
[zk: localhost:2181(CONNECTED) 7] create -e /sanguo/wuguo "zhouyu"
Created /sanguo/wuguo
（1）在当前客户端是能查看到的
[zk: localhost:2181(CONNECTED) 3] ls /sanguo 
[wuguo, shuguo]
（2）退出当前客户端然后再重启客户端
[zk: localhost:2181(CONNECTED) 12] quit
[kino@hadoop104 zookeeper-3.4.10]$ bin/zkCli.sh
（3）再次查看根目录下短暂节点已经删除
[zk: localhost:2181(CONNECTED) 0] ls /sanguo
[shuguo]
```
# 8．创建带序号的节点
```bash
1. 先创建一个普通的根节点/sanguo/weiguo
[zk: localhost:2181(CONNECTED) 1] create /sanguo/weiguo "caocao"
Created /sanguo/weiguo
2. 创建带序号的节点
[zk: localhost:2181(CONNECTED) 2] create -s /sanguo/weiguo/xiaoqiao "jinlian"
Created /sanguo/weiguo/xiaoqiao0000000000
[zk: localhost:2181(CONNECTED) 3] create -s /sanguo/weiguo/daqiao "jinlian"
Created /sanguo/weiguo/daqiao0000000001
[zk: localhost:2181(CONNECTED) 4] create -s /sanguo/weiguo/diaocan "jinlian"
Created /sanguo/weiguo/diaocan0000000002
如果原来没有序号节点，序号从0开始依次递增。如果原节点下已有2个节点，则再排序时从2开始，以此类推。
```
# 9．修改节点数据值
```bash
[zk: localhost:2181(CONNECTED) 6] set /sanguo/weiguo "simayi"
```
# 10．节点的值变化监听
```bash
1. 在hadoop104主机上注册监听/sanguo节点数据变化
[zk: localhost:2181(CONNECTED) 26] [zk: localhost:2181(CONNECTED) 8] get /sanguo watch
2. 在hadoop103主机上修改/sanguo节点的数据
[zk: localhost:2181(CONNECTED) 1] set /sanguo "xisi"
3. 观察hadoop104主机收到数据变化的监听
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged path:/sanguo
```
# 11．节点的子节点变化监听（路径变化）
```bash
1. 在hadoop104主机上注册监听/sanguo节点的子节点变化
[zk: localhost:2181(CONNECTED) 1] ls /sanguo watch
[aa0000000001, server101]
2. 在hadoop103主机/sanguo节点上创建子节点
[zk: localhost:2181(CONNECTED) 2] create /sanguo/jin "simayi"
Created /sanguo/jin
3. 观察hadoop104主机收到子节点变化的监听
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/sanguo
```
# 12．删除节点
```bash
[zk: localhost:2181(CONNECTED) 4] delete /sanguo/jin
```
# 13．递归删除节点
```bash
[zk: localhost:2181(CONNECTED) 15] rmr /sanguo/shuguo
```
# 14．查看节点状态
```bash
[zk: localhost:2181(CONNECTED) 17] stat /sanguo
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000011
mtime = Wed Aug 29 00:21:23 CST 2018
pZxid = 0x100000014
cversion = 9
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 1
```
