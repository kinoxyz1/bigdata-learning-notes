



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
$ kubectl create configmap game-config --from-file=./ -n aaa
$ kubectl get cm -n aaa
NAME              DATA   AGE
game-config       2      8s
nginx-configmap   2      4d12h
$ kubectl describe cm/game-config -n aaa
Name:         game-config
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

$ kubectl get cm/game-config -n aaa -o yaml
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