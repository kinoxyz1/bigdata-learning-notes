



----

# 一、volume 类型
k8s 中有很多种类型的 volume: 
- awsElasticBlockStore
- azureDisk
- azureFile
- cephfs
- cinder
- configMap
- downwardAPI
- emptyDir
- fc (光纤通道)
- flocker （已弃用）
- gcePersistentDisk
- gitRepo (已弃用)
- glusterfs
- hostPath
- iscsi
- local
- nfs
- persistentVolumeClaim
- portworxVolume
- projected
- quobyte
- rbd
- scaleIO （已弃用）
- secret
- storageOS
- vsphereVolume

详细可以查看官方文档
[volume types](https://kubernetes.io/zh/docs/concepts/storage/volumes/#volume-types)


# 二、emptyDir
emptyDir 不会保存数据到磁盘上, pod 被移除掉的时候 emptyDir 也会被永久删掉
```yaml
$ vi emptydir.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx
  namespace: aaa
  labels:
    app:  nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app:  nginx
    spec:
      containers:
      - name:  nginx
        image:  nginx
        ports:
        - containerPort:  80
          name:  nginx
        volumeMounts:
        - name: my-empty-dir
          mountPath: /opt/
      volumes:
        - name: my-empty-dir
          emptyDir: {}
      restartPolicy: Always

$ kubectl apply -f emptydir.yaml
$ kubectl exec -it nginx-59f8b57fc8-vqntf -n aaa bash
> echo 111 > /opt/1.txt
> cat /opt/1.txt
111
```
删除容器
```yaml
$ kubectl delete pod/nginx-59f8b57fc8-vqntf -n aaa
# 等待容器重启, 再查看容器中的 /opt 目录下是否有 1.txt 文件
$ kubectl exec -it nginx-59f8b57fc8-7mz9b -n aaa ls /opt

```

# 三、hostPath
```yaml
$ vim hostpath.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: aaa
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeName: jz-desktop-02
      containers:
        - name: nginx
          image: nginx
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
          ports:
            - containerPort: 80
              name: nginx
          volumeMounts:
            - name: localtime
              mountPath: /opt
      volumes:
        - name: localtime
          hostPath:
            path: /home/kino/hostpath
      restartPolicy: Always

$ kubectl apply -f hostpath.yaml

$ kubectl exec -it nginx-86bcd5dd95-4bx5x -n aaa ls /opt
a1.yaml
```
删掉pod重新创建后再看 /opt 目录下是否有文件
```yaml
$ kubectl delete pod/nginx-86bcd5dd95-4bx5x -n aaa
$ kubectl exec -it nginx-86bcd5dd95-hqxvt -n aaa ls /opt
a1.yaml
```

> 需要注意的是: </br>
> - hostPath 仅仅会将 pod 所在机器的指定 path 和 容器mountPath 进行绑定, 如果每次 pod 运行在不同服务器上, 那容器内部将会是空目录


# 四、configMap
configMap 保存的是键值对数据(非秘密性), Pod 可以将 ConfigMap 的数据使用在 环境变量、命令行参数、volume中。

configMap 保存的数据不能超过 1M, 如果超过该大小, 应该是用挂载数据卷或其他方式。

## 4.1 根据目录创建 configMap 
将一个目录下的所有配置文件创建成一个configMap: `--from-file`
```bash
$ mkdir configmap
$ wget https://kubernetes.io/examples/configmap/game.properties -O configmap/game.properties
$ wget https://kubernetes.io/examples/configmap/ui.properties -O configmap/ui.properties

$ cat *
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

# 创建 configmap
$ kubectl create configmap game-config --from-file=./ -n storage
$ kubectl get cm -n storage
NAME              DATA   AGE
game-config       2      8s
nginx-configmap   2      4d12h
$ kubectl describe cm/game-config -n storage
Name:         game-config
Namespace:    storage
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>

$ kubectl get cm/game-config -n storage -o yaml
apiVersion: v1
data:
  game.properties: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
kind: ConfigMap
metadata:
  creationTimestamp: "2021-09-22T15:43:11Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:game.properties: {}
        f:ui.properties: {}
    manager: kubectl-create
    operation: Update
    time: "2021-09-22T15:43:11Z"
  name: game-config
  namespace: aaa
  resourceVersion: "91076286"
  selfLink: /api/v1/namespaces/aaa/configmaps/game-config
  uid: 0f145467-e712-4183-af48-6c2ebac46fc4
```

## 4.2 基于文件创建 configMap
将一个或多个配置文件创建成configMap 对象: `--from-file`
```bash
$ kubectl create configmap game-config-2 --from-file=game.properties --from-file=ui.properties -n aaa
$ kubectl describe cm/game-config-2 -n aaa
Name:         game-config-2
Namespace:    aaa
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>

```

## 4.3 configMap 的使用
```yaml
# 创建一个 configMap 对象
$ vim configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
  namespace: aaa
data:
  # 类属性键；每一个键都映射到一个简单的值
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # 类文件键
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true   

---
# 在 pod 中使用 configMap
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
  namespace: aaa
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # 定义环境变量
        - name: PLAYER_INITIAL_LIVES # 请注意这里和 ConfigMap 中的键名是不一样的
          valueFrom:
            configMapKeyRef:
              name: game-demo           # 这个值来自 ConfigMap
              key: player_initial_lives # 需要取值的键
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
        - name: config
          mountPath: "/config"
          readOnly: true
  volumes:
    # 你可以在 Pod 级别设置卷，然后将其挂载到 Pod 内的容器中
    - name: config
      configMap:
        # 提供你想要挂载的 ConfigMap 的名字
        name: game-demo
        # 来自 ConfigMap 的一组键，将被创建为文件
        items:
          - key: "game.properties"
            path: "game.properties"
          - key: "user-interface.properties"
            path: "user-interface.properties"

# 查看容器中的环境变量 PLAYER_INITIAL_LIVES 和 UI_PROPERTIES_FILE_NAME
$ kubectl exec -it pod/configmap-demo-pod -n aaa sh
/ # echo $PLAYER_INITIAL_LIVES
3
/ # echo $UI_PROPERTIES_FILE_NAME
user-interface.properties

# 查看挂载到 /config 目录下的文件是否存在
/ # ls -l /config/
total 0
lrwxrwxrwx    1 root     root            22 Sep 23 10:09 game.properties -> ..data/game.properties
lrwxrwxrwx    1 root     root            32 Sep 23 10:09 user-interface.properties -> ..data/user-interface.properties
# 查看文件的内容
/ # cat /config/game.properties 
enemy.types=aliens,monsters
player.maximum-lives=5
/ # cat /config/user-interface.properties 
color.good=purple
color.bad=yellow
allow.textmode=true
```

当 configMap 的值被修改后, pod 中的值不会被自动修改, 需要重启 pod 加载 configMap 的内容
```bash
$ vim configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
  namespace: aaa
data:
  # 类属性键；每一个键都映射到一个简单的值
  player_initial_lives: "1"
  ui_properties_file_name: "user-interface.properties111"

  # 类文件键
  game.properties: |
    enemy.types=aliens,monsters1
    player.maximum-lives=51
  user-interface.properties: |
    color.good=purple1
    color.bad=yellow1
    allow.textmode=true1   
    
# 不重启 pod 查看环境变量是否更新了
$ kubectl exec -it pod/configmap-demo-pod -n aaa sh
/ # echo $PLAYER_INITIAL_LIVES 
3
/ # cat /config/game.properties 
enemy.types=aliens,monsters
player.maximum-lives=5

# 删除 pod 
$ kubectl delete pod/configmap-demo-pod -n aaa sh
# 重新创建
$ kubectl apply -f deploy.yaml
# 进入容器
$ kubectl exec -it pod/configmap-demo-pod -n aaa sh
# 查看环境变量的值和挂载上的文件的内容
/ # echo $UI_PROPERTIES_FILE_NAME
user-interface.properties111
/ # echo $PLAYER_INITIAL_LIVES
1
/ # cat /config/game.properties 
enemy.types=aliens,monsters1
player.maximum-lives=51
/ # cat /config/user-interface.properties 
color.good=purple1
color.bad=yellow1
allow.textmode=true1
```

## 4.4 不可变更的 configMap
当 configMap 被设置为不可变更之后,k8s 会关闭对 configMap 的监视操作, 降低对 kube-apiserver 的压力提升集群性能

configMap 的不可变更是不可逆的, 要想更改 configMap 的内容, 只能把当前不可变更的 configMap 删除掉重新创建

```bash
# 修改上面的 configmap.yaml 
$ vim configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
  namespace: aaa
data:
  # 类属性键；每一个键都映射到一个简单的值
  player_initial_lives: "1"
  ui_properties_file_name: "user-interface.properties111"

  # 类文件键
  game.properties: |
    enemy.types=aliens,monsters1
    player.maximum-lives=51
  user-interface.properties: |
    color.good=purple1
    color.bad=yellow1
    allow.textmode=true1   
immutable: true

# 修改 configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
  namespace: aaa
data:
  # 类属性键；每一个键都映射到一个简单的值
  player_initial_lives: "100"
  ui_properties_file_name: "user-interface.properties111"

  # 类文件键
  game.properties: |
    enemy.types=aliens,monsters1
    player.maximum-lives=51
  user-interface.properties: |
    color.good=purple1
    color.bad=yellow1
    allow.textmode=true1   
immutable: true

# 发布修改后的 configMap, 发现已经不可以修改了
$ kubectl apply -f configmap.yaml 
The ConfigMap "game-demo" is invalid: data: Forbidden: field is immutable when `immutable` is set
```

## 4.5 configMap 实操1
创建一个 nginx 容器, 增加 两个文件: index.html(自定义首页)/index1.html(自定义页面), 并且增加环境变量GOOD=kino
```bash
# 在文件夹中创建三个文件
echo "<h1>status: 200 no.1</h1>" > index.html
echo "<h1>status: 200 no.2</h1>" > index1.html

# 创建 configMap
kubectl create configmap nginx-config-1 --from-file=./ -n storage
或者
kubectl create configmap nginx-config-1 --from-file=index.html --from-file=index1.html -n storage

# 编辑 nginx 的 yaml 文件
vim configmap-01.yaml
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config-2
  namespace: storage
data:
  name: kino
  color.good: purple
  color.bad: yellow
  allow.textmode: true
  how.nice.to.look: fairlyNice

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  namespace: storage
spec:
  selector:
    matchLabels:
      app: my-nginx
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GOOD
          valueFrom:
            configMapKeyRef:
              name: nginx-config-2
              key: name
        ports:
        - containerPort:  80
          name:  my-nginx
        volumeMounts:
        - name: my-nginx-config-1
          mountPath: /usr/share/nginx/html/
      volumes:
        - name: my-nginx-config-1
          configMap:
            name: nginx-config-1
      restartPolicy: Always

# apply 
kubectl apply -f configmap-01.yaml
# 查看pod、cm
kubectl get pod,cm -n storage -o wide                                                                                                                                         Mon Jan 17 23:26:26 2022

NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
pod/my-nginx-784c7f4666-njn4l   1/1     Running   0          3m45s   10.244.1.8   flink02   <none>           <none>

NAME                       DATA   AGE
configmap/nginx-config-1   2      6m45s
configmap/nginx-config-2   5      3m45s

# 访问 两个 html 页面
curl 10.244.1.8
<h1>status: 200 no.1</h1>

curl 10.244.1.8/index1.html
<h1>status: 200 no.2</h1>

# 查看 GOOD 变量
kubectl exec -it pod/my-nginx-784c7f4666-njn4l -n storage bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
root@my-nginx-784c7f4666-njn4l:/# echo $GOOD
kino
```

## 4.6 configMap 实操2
自定义 my.cnf 文件, 修改字符集、连接数, 部署运行 mysql
```bash
vim mysql.yaml
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: mysql-config
  namespace: storage
data:
  my.cnf: |-
    [mysql]
    default-character-set=utf8

    [client]
    default-character-set=utf8

    [mysqld]
    # 字符集
    init_connect='SET NAMES utf8'
    # 最大连接数
    max_connections=1000
    # binlog
    log-bin=mysql-bin
    binlog-format=ROW
    # 忽略大小写
    lower_case_table_names=2
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: storage
spec:
  selector:
    matchLabels:
      app: mysql
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
        ports:
        - containerPort:  80
          name:  mysql
        volumeMounts:
        - name: config
          mountPath: /etc/mysql/conf.d
      volumes:
        - name: config
          configMap:
            name: mysql-config
      restartPolicy: Always

# 登录mysql
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8mb3                        |
| character_set_connection | utf8mb3                        |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8mb3                        |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.00 sec)

mysql> show variables like '%max_connections%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| max_connections        | 1000  |
| mysqlx_max_connections | 100   |
+------------------------+-------+
2 rows in set (0.00 sec)
```

## 4.7 configMap 实操3-subPath

[subPath官方文档](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#using-subpath)

在 k8s 中，volume 不能挂载到其他 volume 上，也不能与其他卷有硬链接。

不过可以使用 `volumeMounts.subPath` 属性指定所引用的卷内的子路径，而不是其根路径。

例如: 创建一个 nginx 容器，index.html 使用nginx 默认的，另外再挂载一个 login.html 进 `/usr/share/nginx/html/login.html`中

```yaml
# 创建login.html
$ echo "<h1>login.html</h1>" > login.html

# 创建 configmap
$ kubectl create configmap login-file --from-file=login.html -n kino

# 创建 nginx deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  namespace: kino
spec:
  selector:
    matchLabels:
      app: my-nginx
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      nodeName: jz-desktop-04
      containers:
      - name: my-nginx
        image: nginx
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort:  80
          name:  my-nginx
        volumeMounts:
        - mountPath: /usr/share/nginx/html/login.html
          name: login-volume
          subPath: index.html
      restartPolicy: Always
      volumes:
      - name: login-volume
        configMap:
          name: login-file
          items:
          - key: "login.html"
            path: "login.html"
            
# 查看 /usr/share/nginx/html 下的文件是否被覆盖
root@my-nginx-8545c54cf6-kh58p:/usr/share/nginx/html# ls -l
total 12
-rw-r--r-- 1 root root  497 Jul 19 14:05 50x.html
-rw-r--r-- 1 root root   15 Oct  9 14:56 index.html
drwxrwxrwx 2 root root 4096 Oct  9 14:54 login.html   # subPath 挂载进来的
```







# 五、NFS

[centos7 nfs安装部署](../../note/软件部署/centos/nfs.md)

## 5.1 使用 nfs 挂载
```bash
vim nginx-nfs.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: storage
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort:  80
          name: nginx
        volumeMounts:
        - name: nfs-volume
          mountPath: /usr/share/nginx/html/
      volumes:
        - name: nfs-volume
          nfs:
            server: 192.168.156.60
            path: /app/nfs
      restartPolicy: Always
 
 kubectl apply -f nginx-nfs.yaml
 
 # 在 /app/nfs 目录下创建 index.html 文件
 echo "hello nfs volume" > index.html
 
 
 # 访问 nginx 
 kubectl get pod -n storage -o wide
 pod/nginx-5d59d8d7fd-v744j      1/1     Running            0          3m44s   10.244.2.11   flink03   <none>           <none>

curl 10.244.2.11
hello nfs volume
```


# 六、ceph 
https://ceph.io/
## 6.1、基本概念

Ceph可以有

- Ceph对象存储：键值存储，其接口就是简单的GET,PUT,DEL等。如七牛，阿里云oss等
- Ceph块设备：**AWS的EBS**，**青云的云硬盘**和**阿里云的盘古系统**，还有**Ceph的RBD**(RBD是Ceph面向块存储的接口)
- Ceph文件系统：它比块存储具有更丰富的接口，需要考虑目录、文件属性等支持，实现一个支持并行化的文件存储应该是最困难的。

一个Ceph存储集群需要

- 至少一个Ceph监视器、Ceph管理器、Ceph OSD（对象存储守护程序）
- 需要运行Ceph文件系统客户端，则需要部署 Ceph Metadata Server。
- **Monitors**:  [Ceph Monitor](https://docs.ceph.com/en/latest/glossary/#term-Ceph-Monitor) (`ceph-mon`) 监视器：维护集群状态信息
   - 维护集群状态的映射，包括监视器映射，管理器映射，OSD映射，MDS映射和CRUSH映射。
   - 这些映射是Ceph守护程序相互协调所必需的关键群集状态。
   - 监视器还负责管理守护程序和客户端之间的身份验证。
   - 通常至少需要三个监视器才能实现冗余和高可用性。
- **Managers**: [Ceph Manager](https://docs.ceph.com/en/latest/glossary/#term-Ceph-Manager) 守护进程(`ceph-mgr`) : 负责跟踪运行时指标和Ceph集群的当前状态
   - Ceph Manager守护进程（ceph-mgr）负责跟踪运行时指标和Ceph集群的当前状态
   - 包括存储利用率，当前性能指标和系统负载。
   - Ceph Manager守护程序还托管基于python的模块，以管理和公开Ceph集群信息，包括基于Web的Ceph Dashboard和REST API。
   - 通常，至少需要两个管理器才能实现高可用性。
- **Ceph OSDs**: [Ceph OSD](https://docs.ceph.com/en/latest/glossary/#term-Ceph-OSD) (对象存储守护进程, `ceph-osd`) 【存储数据】
   - 通过检查其他Ceph OSD守护程序的心跳来存储数据，处理数据复制，恢复，重新平衡，并向Ceph监视器和管理器提供一些监视信息。
   - 通常至少需要3个Ceph OSD才能实现冗余和高可用性。
- **MDSs**: [Ceph Metadata Server](https://docs.ceph.com/en/latest/glossary/#term-Ceph-Metadata-Server) (MDS, `ceph-mds`ceph元数据服务器)
   -  存储能代表 [Ceph File System](https://docs.ceph.com/en/latest/glossary/#term-Ceph-File-System) 的元数据(如：Ceph块设备和Ceph对象存储不使用MDS).
   -  Ceph元数据服务器允许POSIX文件系统用户执行基本命令（如ls，find等），而不会给Ceph存储集群带来巨大负担


## 6.2 Rook
### 6.2.1、基本概念
Rook是云原生平台的存储编排工具

Rook工作原理如下：

![rook 工作原理](../../img/k8s/PV和PVC/rook-architecture.png)

Rook架构如下:

![rook架构图](../../img/k8s/PV和PVC/rook架构图.png)

RGW：为Restapi Gateway

### 6.2.2、operator是什么

k8s中operator+CRD（CustomResourceDefinitions【k8s自定义资源类型】），可以快速帮我们部署一些有状态应用集群，如redis，mysql，Zookeeper等。

Rook的operator是我们k8s集群和存储集群之间进行交互的解析器



CRD：CustomResourceDefinitions (自定义资源)；如：Itdachang

operator：这个能处理自定义资源类型

## 6.3 部署
https://rook.io/docs/rook/v1.6/ceph-quickstart.html

### 6.3.1、查看前提条件

- Raw devices (no partitions or formatted filesystems)； 原始磁盘，无分区或者格式化
- Raw partitions (no formatted filesystem)；原始分区，无格式化文件系统


```bash
fdisk -l 
找到自己挂载的磁盘
如： /dev/vdc

# 查看满足要求的
lsblk -f

#云厂商都这么磁盘清0
dd if=/dev/zero of=/dev/vdc bs=1M status=progress


# NAME                  FSTYPE      LABEL UUID                                   MOUNTPOINT
# vda
# └─vda1                LVM2_member       >eSO50t-GkUV-YKTH-WsGq-hNJY-eKNf-3i07IB
# ├─ubuntu--vg-root   ext4              c2366f76-6e21-4f10-a8f3-6776212e2fe4   /
# └─ubuntu--vg-swap_1 swap              9492a3dc-ad75-47cd-9596-678e8cf17ff9   [SWAP]
# vdb
```
vdb 是可用的

### 6.3.2、部署&修改operator
```bash
git clone --single-branch --branch v1.6.11 https://github.com/rook/rook.git
cd cluster/examples/kubernetes/ceph
## vim operator.yaml
## 建议修改以下的东西。在operator.yaml里面
## ROOK_CSI_CEPH_IMAGE: "registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/cephcsi:v3.3.1"
## ROOK_CSI_REGISTRAR_IMAGE: "registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/csi-node-driver-registrar:v2.0.1"
## ROOK_CSI_RESIZER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/csi-resizer:v1.0.1"
## ROOK_CSI_PROVISIONER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/csi-provisioner:v2.0.4"
## ROOK_CSI_SNAPSHOTTER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/csi-snapshotter:v4.0.0"
## ROOK_CSI_ATTACHER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/csi-attacher:v3.0.2"

kubectl create -f crds.yaml -f common.yaml -f operator.yaml #注意修改operator镜像

kubectl -n rook-ceph get pod
```

### 6.3.3 部署集群
修改`cluster.yaml`使用我们指定的磁盘当做存储节点即可
```bash
...
    image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ceph-ceph:v15.2.11  # 修改镜像
...
  mon:
    # Set the number of mons to be started. Must be an odd number, and is generally recommended to be 3.
    count: 3
    # The mons should be on unique nodes. For production, at least 3 nodes are recommended for this reason.
    # Mons should only be allowed on the same node for test environments where data loss is acceptable.
    allowMultiplePerNode: false
  mgr:
    # When higher availability of the mgr is needed, increase the count to 2.
    # In that case, one mgr will be active and one in standby. When Ceph updates which
    # mgr is active, Rook will update the mgr services to match the active mgr.
    count: 2
    modules:
      # Several modules should not need to be included in this list. The "dashboard" and "monitoring" modules
      # are already enabled by other settings in the cluster CR.
      - name: pg_autoscaler
        enabled: true
  # enable the ceph dashboard for viewing cluster status
...
  storage: # cluster level storage configuration and selection
    useAllNodes: false
    useAllDevices: false
    #deviceFilter:
    config:
      osdsPerDevice: "3" # this value can be overridden at the node or device level
    nodes:
    - name: "k8s-master099"
      devices: # specific devices to use for storage can be specified for each node
      - name: "vdb"
    - name: "k8s-master100"
      devices:
      - name: "vdb"
    - name: "k8s-master101"
      devices:
      - name: "vdb"
```

### 6.3.4、部署dashboard
https://www.rook.io/docs/rook/v1.6/ceph-dashboard.html

前面的步骤，已经自动部署了。

```bash
kubectl -n rook-ceph get service
#查看service


#为了方便访问我们改为nodePort。应用nodePort文件


#获取访问密码
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo

#默认账号 admin
4/qt]e5wad_HY:0&V.ba
```
暴露 ingress 提供访问
```bash
vim dashboard-ingress-https.yaml
#
# This example is for Kubernetes running an ngnix-ingress
# and an ACME (e.g. Let's Encrypt) certificate service
#
# The nginx-ingress annotations support the dashboard
# running using HTTPS with a self-signed certificate
#
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rook-ceph-mgr-dashboard
  namespace: rook-ceph # namespace:cluster
  annotations:
    kubernetes.io/ingress.class: "nginx"
    # kubernetes.io/tls-acme: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/server-snippet: |
      proxy_ssl_verify off;
spec:
  # tls:
  #   - hosts:
  #       - rook.kinoxyz.com
  #     secretName: rook-tls-secret
  rules:
    - host: rook.kinoxyz.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: rook-ceph-mgr-dashboard
                port:
                  number: 8443
                  
                  
kubectl apply -f dashboard-ingress-https.yaml
```
访问 https://rook.kinoxyz.com 即可

### 6.3.5 说明
```bash
## 部署完的组件如下
NAME                                                     READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-4kctq                                   3/3     Running     0          48m
csi-cephfsplugin-7fnrm                                   3/3     Running     0          48m
csi-cephfsplugin-provisioner-5fc67679b6-6kbgs            6/6     Running     0          48m
csi-cephfsplugin-provisioner-5fc67679b6-s5z9b            6/6     Running     0          48m
csi-cephfsplugin-wrqvt                                   3/3     Running     0          48m
csi-rbdplugin-75pph                                      3/3     Running     0          48m
csi-rbdplugin-provisioner-675764ff49-gqh4k               6/6     Running     0          48m
csi-rbdplugin-provisioner-675764ff49-x74t4               6/6     Running     0          48m
csi-rbdplugin-slzzd                                      3/3     Running     0          48m
csi-rbdplugin-xq82s                                      3/3     Running     0          48m
rook-ceph-crashcollector-k8s-master099-6998b6d8f-nb4rm   1/1     Running     0          33m
rook-ceph-crashcollector-k8s-master100-7c9c8df98-t8d8g   1/1     Running     0          46m
rook-ceph-crashcollector-k8s-master101-fd444f5dc-kl7pk   1/1     Running     0          33m
rook-ceph-mds-myfs-a-5575d8cc8f-tt2w8                    1/1     Running     0          33m
rook-ceph-mds-myfs-b-69f74567c5-bp7zv                    1/1     Running     0          33m
rook-ceph-mgr-a-5d7c4dd494-gkcc9                         2/2     Running     0          46m
rook-ceph-mgr-b-65fb676b5-nhwp6                          2/2     Running     0          46m
rook-ceph-mon-a-867d4f65fc-r4ggj                         1/1     Running     0          47m
rook-ceph-mon-b-5c64d58fb7-xdrsj                         1/1     Running     0          47m
rook-ceph-mon-c-787f48598-2v7n9                          1/1     Running     0          46m
rook-ceph-operator-bfdc879fd-wjm7k                       1/1     Running     0          48m
rook-ceph-osd-0-678696f975-rfftx                         1/1     Running     0          46m
rook-ceph-osd-1-75fcc48dc7-d6j2v                         1/1     Running     0          46m
rook-ceph-osd-2-5497cbdb45-cf7rd                         1/1     Running     0          46m
rook-ceph-osd-3-5f6cbd76f6-whw54                         1/1     Running     0          46m
rook-ceph-osd-4-68dfc99778-r88np                         1/1     Running     0          46m
rook-ceph-osd-5-7978cb5f7f-hkxd9                         1/1     Running     0          46m
rook-ceph-osd-6-6b48fc54cf-zl8k8                         1/1     Running     0          46m
rook-ceph-osd-7-54f5554468-8rl47                         1/1     Running     0          46m
rook-ceph-osd-8-56f8c9486-xq2gn                          1/1     Running     0          46m
rook-ceph-osd-prepare-k8s-master099-p44ll                0/1     Completed   0          44m
rook-ceph-osd-prepare-k8s-master100-lsz5s                0/1     Completed   0          44m
rook-ceph-osd-prepare-k8s-master101-k2tdf                0/1     Completed   0          44m
```

## 6.4 卸载
```bash
# rook集群的清除，
##1、 delete -f 之前的yaml

##2、 再执行如下命令
kubectl -n rook-ceph get cephcluster
kubectl -n rook-ceph patch cephclusters.ceph.rook.io rook-ceph -p '{"metadata":{"finalizers": []}}' --type=merge

##3、 清除每个节点的 /var/lib/rook 目录


## 顽固的自定义资源删除；
kubectl -n rook-ceph patch cephblockpool.ceph.rook.io replicapool -p '{"metadata":{"finalizers": []}}' --type=merge
```

## 6.5 实战
### 6.5.1 块存储(RDB)
```bash
cd rook/cluster/examples/kubernetes/ceph/csi/rbd
kubectl apply -f storageclass.yaml
```
### 6.5.2 案例
```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-nginx
  namespace: default
spec:
  selector:
    matchLabels:
      app: sts-nginx # has to match .spec.template.metadata.labels
  serviceName: "sts-nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: sts-nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: sts-nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 20Mi
---
apiVersion: v1
kind: Service
metadata:
  name: sts-nginx
  namespace: default
spec:
  selector:
    app: sts-nginx
  type: ClusterIP
  ports:
  - name: sts-nginx
    port: 80
    targetPort: 80
    protocol: TCP
```
> 测试： 创建sts、修改nginx数据、删除sts、重新创建sts。他们的数据丢不丢，共享不共享

### 6.5.3 文件存储(CephFS)
常用 文件存储。 RWX模式；如：10个Pod共同操作一个地方

https://rook.io/docs/rook/v1.6/ceph-filesystem.html

```bash
cd rook/cluster/examples/kubernetes
kubectl apply -f filesystem.yaml
cd rook/cluster/examples/kubernetes/ceph/csi/cephfs
kubectl apply -f storageclass.yaml
```

### 6.5.4 案例
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx-deploy
  namespace: default
  labels:
    app:  nginx-deploy
spec:
  selector:
    matchLabels:
      app: nginx-deploy
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app:  nginx-deploy
    spec:
      containers:
      - name:  nginx-deploy
        image:  nginx
        volumeMounts:
        - name: localtime
          mountPath: /etc/localtime
        - name: nginx-html-storage
          mountPath: /usr/share/nginx/html
      volumes:
        - name: localtime
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
        - name: nginx-html-storage
          persistentVolumeClaim:
            claimName: nginx-pv-claim
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pv-claim
  labels:
    app:  nginx-deploy
spec:
  storageClassName: rook-cephfs
  accessModes:
    - ReadWriteMany  ##如果是ReadWriteOnce将会是什么效果
  resources:
    requests:
      storage: 10Mi
```
> 测试，创建deploy、修改页面、删除deploy，新建deploy是否绑定成功，数据是否在。


# 七、PV、PVC
[官方 PV、PVC 实战](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

[PV、PVC 的官方文档](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#access-modes)

## 7.1 持久卷（PersistentVolume ）

- 持久卷（PersistentVolume，PV）是集群中的一块存储，可以由管理员事先供应，或者 使用[存储类（Storage Class）](https://kubernetes.io/zh/docs/concepts/storage/storage-classes/)来动态供应。
- 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样，也是使用 卷插件来实现的，只是它们拥有独立于使用他们的Pod的生命周期。
- 此 API 对象中记述了存储的实现细节，无论其背后是 NFS、iSCSI 还是特定于云平台的存储系统。

### 7.1.1 示例
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-0001
  namespace: storage
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  nfs:
    path: /app/nfs
    server: 192.168.156/60
```
参数说明:
1. capacity.storage: [该 PV 的容量](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#capacity)
2. volumeMode: [卷模式, 默认为 Filesystem](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#volume-mode)
3. accessModes: [访问模式](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#access-modes)
    - ReadWriteOnce: 可以被一个节点以只读的方式挂载;
    - ReadOnlyMany: 可以被多个节点以只读的方式挂载;
    - ReadWriteMany: 可以被多个节点以读写的方式挂载;
    - ReadWriteOncePod: 卷可以被单个 Pod 以只读的方式挂载。如果需要集群中只有一个 Pod 可以读写该 PVC, 就需要用此访问模式.
4. persistentVolumeReclaimPolicy: [回收策略](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#reclaim-policy)
5. storageClassName: [用于和 PVC 匹配](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#class)


## 7.2 持久卷申请（PersistentVolumeClaim，PVC）

- 表达的是用户对存储的请求
- 概念上与 Pod 类似。 Pod 会耗用节点资源，而 PVC 申领会耗用 PV 资源。
- Pod 可以请求特定数量的资源（CPU 和内存）；同样 PVC 申领也可以请求特定的大小和访问模式 （例如，可以要求 PV 卷能够以 ReadWriteOnce、ReadOnlyMany 或 ReadWriteMany 模式之一来挂载，参见[访问模式](https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/#access-modes)）。

### 7.2.1 示例
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
  namespace: storage
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
```

## 7.3 PV、PVC 实操
使用 PV、PVC、ConfigMap 部署一个单机版的 MySQL
```bash
# 1. 创建 my.cnf 文件并 create 为 configmap
cat >> my.cnf << EOF
[mysql]
default-character-set=utf8

[client]
default-character-set=utf8

[mysqld]
# 字符集
default-character-set=utf8
init_connect='SET NAMES utf8'
# 最大连接数
max_connections=1000
# binlog
log-bin=mysql-bin
binlog-format=ROW
# 忽略大小写
lower_case_table_names=2
EOF

kubectl create configmap mysql-config --from-file=my.cnf -n storage

# 2. 创建 PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mysql-1
  namespace: storage
spec:
  capacity:
    storage: 50Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: my-nfs
  nfs:
    path: /app/nfs
    server: 192.168.156/60
    
# 3. 创建 PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-mysql-1
  namespace: storage
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 50Gi
  storageClassName: my-nfs

# 4. 创建 deploy 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-pv
  namespace: storage
  labels:
    app: mysql-pv
spec:
  selector:
    matchLabels:
      app: mysql-pv
  replicas: 1
  template:
    metadata:
      labels:
        app:  mysql-pv
    spec:
      containers:
      - name: mysql-pv
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
        ports:
        - containerPort:  80
          name: mysql-pv
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
        - name: my-config
          mountPath: /etc/mysql/conf.d
        - name: time-zone
          mountPath: /etc/localtime
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: pvc-mysql-1
        - name: my-config
          configMap: 
            name: mysql-config
        - name: time-zone
          hostPath: 
            path: /etc/localtime
      restartPolicy: Always

```

## 7.4 动态存储类（Storage Class）

- 尽管 PersistentVolumeClaim 允许用户消耗抽象的存储资源，常见的情况是针对不同的 问题用户需要的是具有不同属性（如，性能）的 PersistentVolume 卷。
- 集群管理员需要能够提供不同性质的 PersistentVolume，并且这些 PV 卷之间的差别不 仅限于卷大小和访问模式，同时又不能将卷是如何实现的这些细节暴露给用户。
- 为了满足这类需求，就有了 *存储类（StorageClass）* 资源。

[官方部署文档](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

### 7.4.1 部署
```bash
# 创建存储类
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
# provisioner: 指定供应商的名字
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            # 改成自己的 NFS 地址
            - name: NFS_SERVER
              value: 192.168.156.60
            - name: NFS_PATH
              value: /app/nfs
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.156.60
            path: /app/nfs
            
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```


### 7.4.2 设置默认
方式一:
```bash
## 创建了一个存储类
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  # 设置为默认存储类
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

方式二: 
```bash
kubectl patch storageclass managed-nfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

### 7.4.3 案例
```yaml

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: mysql-config
  namespace: storage
data:
  my.cnf: |
    [mysql]
    default-character-set=utf8
    [client]
    default-character-set=utf8
    [mysqld]
    default-character-set=utf8
    init_connect='SET NAMES utf8'
    max_connections=1000
    log-bin=mysql-bin
    binlog-format=ROW
    lower_case_table_names=2
---
# 3. 创建 PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: storage
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 50Gi
  # storageClassName: my-nfs

# 4. 创建 deploy 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-pv
  namespace: storage
  labels:
    app: mysql-pv
spec:
  selector:
    matchLabels:
      app: mysql-pv
  replicas: 1
  template:
    metadata:
      labels:
        app:  mysql-pv
    spec:
      containers:
      - name: mysql-pv
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
        ports:
        - containerPort:  80
          name: mysql-pv
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
        - name: my-config
          mountPath: /etc/mysql/conf.d
        - name: time-zone
          mountPath: /etc/localtime
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
        - name: my-config
          configMap: 
            name: mysql-config
        - name: time-zone
          hostPath: 
            path: /etc/localtime
      restartPolicy: Always
```
查看 pv/pvc 
```bash
$ kubectl get pv,pvc -A -owide
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS          REASON   AGE    VOLUMEMODE
persistentvolume/pvc-bc727051-8426-4249-8e13-8414eb5279f0   50Gi       RWO            Delete           Bound    storage/mysql-pvc   managed-nfs-storage            4m4s   Filesystem

NAMESPACE   NAME                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE    VOLUMEMODE
storage     persistentvolumeclaim/mysql-pvc   Bound    pvc-bc727051-8426-4249-8e13-8414eb5279f0   50Gi       RWO            managed-nfs-storage   4m4s   Filesystem

```


# 八、扩容 pv(ceph)
https://www.cnblogs.com/evescn/p/16929463.html

查看信息
```bash
# 查看要扩容的容器的磁盘挂载情况
$ kubectl exec -it gitlab-56b96d5594-d4zkd -n gitlab-new -- df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         879G  581G  254G  70% /
tmpfs            64M     0   64M   0% /dev
tmpfs            63G     0   63G   0% /sys/fs/cgroup
/dev/nvme0n1p2  879G  581G  254G  70% /etc/hosts
shm              64M     0   64M   0% /dev/shm
/dev/rbd6        20G   16G   4G   91% /home/git/data
tmpfs            63G   12K   63G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs            63G     0   63G   0% /proc/acpi
tmpfs            63G     0   63G   0% /proc/scsi
tmpfs            63G     0   63G   0% /sys/firmware

# 查看 deploy 使用的 pvc
$ kubectl get deploy gitlab -n gitlab-new -o yaml
...
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: gitlab-new-gitlab-data
          
# 查看 pvc 绑定的 pv
$ kubectl get pvc gitlab-new-gitlab-data -n gitlab-new -o yaml
...
  storageClassName: rbd
  volumeMode: Filesystem
  volumeName: pvc-93a0f3a3-0157-486c-ac63-008fb2ae6a36
  
# 查看 pv 的信息
$ kubectl get pv pvc-93a0f3a3-0157-486c-ac63-008fb2ae6a36 -o yaml
## 看 image 和 pool
...
  rbd:
    image: kubernetes-dynamic-pvc-dae1a1b2-9a06-11ec-91e7-365e0976bf6e
    keyring: /etc/ceph/keyring
    monitors:
    - 192.168.1.80:6789
    - 192.168.1.163:6789
    - 192.168.1.182:6789
    pool: kube
    secretRef:
      name: ceph-secret
      namespace: kube-system
```
修改 sc 
```bash
$ kubectl edit sc rbd 
# 加进去
allowVolumeExpansion: true  
```
修改pv的大小
```bash
$ kubectl edit pvc gitlab-new-gitlab-data -n gitlab-new
...
  resources:
    requests:
      storage: 50Gi    <--- 修改成指定的大小
```
修改pvc的大小
```bash
$ kubectl edit pvc gitlab-new-gitlab-data -n gitlab-new
...
  resources:
    requests:
      storage: 50Gi    <--- 修改成指定的大小
```
修改rbd的大小
```bash
# 登录到 pod 所在的机器执行以下命令
# 查看当前机器所有的rbd
$ rbd showmapped
id pool image                                                       snap device
0  kube kubernetes-dynamic-pvc-01d4ed95-ebc1-11ec-aa29-c6c1f9206139 -    /dev/rbd0
1  kube kubernetes-dynamic-pvc-c0da3843-8948-11ec-88db-d25b11a75b57 -    /dev/rbd1
5  kube kubernetes-dynamic-pvc-2eabc27f-d7f6-11ec-badd-8a131a0f411c -    /dev/rbd5
6  kube kubernetes-dynamic-pvc-dae1a1b2-9a06-11ec-91e7-365e0976bf6e -    /dev/rbd6
7  kube kubernetes-dynamic-pvc-7442f7fc-ebc0-11ec-aa29-c6c1f9206139 -    /dev/rbd7
8  kube kubernetes-dynamic-pvc-740322a4-ebc0-11ec-aa29-c6c1f9206139 -    /dev/rbd8

# 查看 rbd 的大小, info 是上面查的 pool 的值, kubernetes-dynam... 是上面查的 image 的只
$ rbd -p kube info kubernetes-dynamic-pvc-dae1a1b2-9a06-11ec-91e7-365e0976bf6e
rbd image 'kubernetes-dynamic-pvc-dae1a1b2-9a06-11ec-91e7-365e0976bf6e':
	size 20 GiB in 12800 objects
	order 22 (4 MiB objects)
	id: fbab6b8b4567
	block_name_prefix: rbd_data.fbab6b8b4567
	format: 2
	features: layering
	op_features:
	flags:
	create_timestamp: Wed Mar  2 16:57:57 2022
	
# 修改 rbd 的大小
$ rbd -p kube resize kubernetes-dynamic-pvc-dae1a1b2-9a06-11ec-91e7-365e0976bf6e --size 50G
调整大小
Resizing image: 100% complete...done.

# 查看 rbd 的大小
rbd -p kube info kubernetes-dynamic-pvc-dae1a1b2-9a06-11ec-91e7-365e0976bf6e
	size 50 GiB in 12800 objects
	order 22 (4 MiB objects)
	id: fbab6b8b4567
	block_name_prefix: rbd_data.fbab6b8b4567
	format: 2
	features: layering
	op_features:
	flags:
	create_timestamp: Wed Mar  2 16:57:57 2022
```
查看pv和pvc的大小
```bash
$ kubectl get pv,pvc -n gitlab-new |grep gitlab-new-gitlab-data
persistentvolume/pvc-93a0f3a3-0157-486c-ac63-008fb2ae6a36   50Gi       RWO            Delete           Bound    gitlab-new/gitlab-new-gitlab-data               rbd                     636d
persistentvolumeclaim/gitlab-new-gitlab-data          Bound    pvc-93a0f3a3-0157-486c-ac63-008fb2ae6a36   50Gi       RWO            rbd            636d
```
查看容器中的挂载
```bash
$ kubectl exec -it gitlab-56b96d5594-d4zkd -n gitlab-new -- df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         879G  581G  254G  70% /
tmpfs            64M     0   64M   0% /dev
tmpfs            63G     0   63G   0% /sys/fs/cgroup
/dev/nvme0n1p2  879G  581G  254G  70% /etc/hosts
shm              64M     0   64M   0% /dev/shm
/dev/rbd6        50G   16G   34G  32% /home/git/data
tmpfs            63G   12K   63G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs            63G     0   63G   0% /proc/acpi
tmpfs            63G     0   63G   0% /proc/scsi
tmpfs            63G     0   63G   0% /sys/firmware
```

