
* [一、Volume 概述](#%E4%B8%80volume-%E6%A6%82%E8%BF%B0)
* [二、 Volume 使用](#%E4%BA%8C-volume-%E4%BD%BF%E7%94%A8)
  * [2\.1 声明 YAML 文件内的 Volume 配置](#21-%E5%A3%B0%E6%98%8E-yaml-%E6%96%87%E4%BB%B6%E5%86%85%E7%9A%84-volume-%E9%85%8D%E7%BD%AE)


---
有需要温习 Volume 的小伙伴移步: [Docker容器数据卷](../docker/Docker容器数据卷.md)


# 一、Volume 概述
Volume 是 Pod 中能够被多个容器访问的共享目录。

Kubernetes 的 Volume 定义在 Pod 上, 它被一个 Pod 中的多个容器挂载到具体的文件目录下。

Volume 与 Pod 的生命周期相同, 但与容器的生命周期不相关, 当容器终止或重启时, Volume 中的数据也不会丢失。

要使用 volume, pod 需要指定 volume 的类型和内容（字段）和 映射到容器的位置（字段）。

Kubernetes 支持多种类型的 Volume,包括
 - emptyDir
 - hostPath
 - gcePersistentDisk
 - awsElasticBlockStore 
 - nfs
 - iscsi
 - flocker
 - glusterfs
 - rbd
 - cephfs
 - gitRepo
 - secret
 - persistentVolumeClaim
 - downwardAPI
 - azureFileVolume
 - azureDisk
 - vsphereVolume
 - Quobyte
 - PortworxVolume
 - ScaleIO
 
 
# 二、 Volume 使用
## 2.1 声明 YAML 文件内的 Volume 配置
在 [k8s常用操作命令](k8s常用操作命令.md) 中的 create 命令, 创建了一个 Nginx YAML, 内容如下:
```yaml
[root@k8s-master ~]# vim nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

我们在这个 Deployment 为基础, 尝试声明一个 Volume

在 Kubernetes 中, Volume 是属于 Pod 对象的一部分。所以, 我们就需要修改这个 YAML 文件里的 template.spec 字段, 如下所示:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-vol
      volumes:
      - name: nginx-vol
        emptyDir: {}
```

① 可以看到, 我们在 Deployment 的 Pod 模板部分添加了一个 volumes 字段, 定义了这个 Pod 声明的所有 Volume。它的名字叫作 nginx-vol, 类型是 emptyDir。

② emptyDir 类型其实就等同于 Docker 的隐式 Volume 参数, 即: 不显式声明宿主机目录的 Volume。所以, Kubernetes 也会在宿主机上创建一个临时目录, 这个目录将来就会被绑定挂载到容器所声明的 Volume 目录上。

③ EmptyDir 类型的 volume 创建于 pod 被调度到某个宿主机上的时候, 而同一个 pod 内的容器都能读写 EmptyDir 中的同一个文件。

④ 一旦这个 pod 离开了这个宿主机, EmptyDir 中的数据就会被永久删除。

⑤ 所以目前 EmptyDir 类型的 volume 主要用作临时空间, 比如 Web 服务器写日志或者 tmp 文件需要的临时目录。

Pod 中的容器, 使用的是 volumeMounts 字段来声明自己要挂载哪个 Volume, 并通过 mountPath 字段来定义容器内的 Volume 目录, 比如：/usr/share/nginx/html

当然, Kubernetes 也提供了显式的 Volume 定义, 它叫作 hostPath。比如下面的这个 YAML 文件:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-vol
      volumes:
        - name: nginx-vol
          hostPath: 
            path:  "/var/data"
```
这样, 容器 Volume 挂载的宿主机目录, 就变成了 /var/data。

使用 `kubectl apply` 指令, 更新这个 Deployment
```bash
[root@k8s-master yaml]# kubectl apply -f nginx-deployment.yaml
```

通过 `kubectl get` 指令，查看两个 Pod 被逐一更新的过程：
```bash
[root@k8s-master yaml]# kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-5f8c6846ff-492d7   1/1     Running             0          44m
nginx-deployment-5f8c6846ff-l2prp   1/1     Running             0          45m
nginx-deployment-9754ccbdf-5spwg    0/1     ContainerCreating   0          19s
```

从返回结果中, 我们可以看到, 新旧两个 Pod, 被交替创建、删除, 最后剩下的就是新版本的 Pod。

然后, 你可以使用 `kubectl describe` 查看一下最新的 Pod, 就会发现 Volume 的信息已经出现在了 Container 描述部分:
```bash
...
Containers:
  nginx:
    Container ID:   docker://07b4f89248791c2aa47787e3da3cc94b48576cd173018356a6ec8db2b6041343
    Image:          nginx:1.8
    ...
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from nginx-vol (rw)
...
Volumes:
  nginx-vol:
    Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
```




---
在之前的笔记中, 有介绍过 volume 存放/共享容器中的数据, 在 k8s 中, 还存在几种特殊的 volume, 它们存在的意义不是为了存放/共享容器中的数据。**这些特殊的 volume 的作用是为容器提供预先定义好的数据**, 所以从容器的角度来看, 这些 volume 里的信息就是仿佛**被 k8s 投射进容器的一样**

目前为止, k8s 支持的特殊 volume 一共有四种:
1. Secret
2. ConfigMap
3. Downward API
4. ServiceAccountToken

# 一、Secret 作用
讲加密数据存在 etcd 里面, 让 Pod 容器以挂载 Volume 方式进行访问

使用场景: 数据库的凭证


# 二、创建 secret 加密数据
```yaml
$ vim Secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm

$ kubectl apply -f Secret.yaml 
secret/mysecret created
$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-ch88s   kubernetes.io/service-account-token   3      9d
mysecret              Opaque                                2      6s
```

# 三、以变量的形式挂载到 pod 容器中
```bash
$ vim secret-var.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password

$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          29s

$ kubectl exec -it mypod bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

root@mypod:/# echo $SECRET_USERNAME
admin
root@mypod:/# echo $SECRET_PASSWORD
1f2d1e2e67df
```


# 四、以 volume 形式挂载到 pod 容器中
```bash
$ vim secret-vol.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret

$ kubectl apply -f secret-vol.yaml 
pod/mypod created

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          10s

$ kubectl exec -it mypod bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

root@mypod:/# ls /etc/fo
fonts/ foo/   

root@mypod:/# ls /etc/fo
fonts/ foo/   

root@mypod:/# ls /etc/foo/
password  username

root@mypod:/# cat /etc/foo/password
1f2d1e2e67dfroot@mypod:/# cat /etc/foo/username 
adminroot@mypod:/# 
```







---

在之前的笔记中, 有介绍过 volume 存放/共享容器中的数据, 在 k8s 中, 还存在几种特殊的 volume, 它们存在的意义不是为了存放/共享容器中的数据。**这些特殊的 volume 的作用是为容器提供预先定义好的数据**, 所以从容器的角度来看, 这些 volume 里的信息就是仿佛**被 k8s 投射进容器的一样**

目前为止, k8s 支持的特殊 volume 一共有四种:
1. Secret
2. ConfigMap
3. Downward API
4. ServiceAccountToken


# 一、ConfigMap 作用
和 Secret 类似, ConfigMap 的作用是将不加密数据存储到 etcd 中, 让 Pod 以变量的形式或者 Volume 挂载到容器中.

场景: 配置文件


# 二、创建 ConfigMap
```bash
# 1. 准备 redis.properties 配置文件, 如下:
redis.host=127.0.0.1
redis.port=6379
redis.password=123456

# 2. 创建
$ kubectl create configmap redis-config --from-file=redis.properties 
configmap/redis-config created

$ kubectl get cm
NAME           DATA   AGE
redis-config   1      5s

$ kubectl describe cm redis-config
Name:         redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis.properties:
----
redis.host=127.0.0.1
redis.port=6379
redis.password=123456


Events:  <none>
```

# 三、以 Volume 的形式挂载到 Pod 中去
```bash
$ vim cm.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: busybox
      image: busybox
      command: [ "/bin/sh","-c","cat /etc/config/redis.properties" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: redis-config
  restartPolicy: Never

$ kubectl apply -f cm.yaml 
pod/mypod created

$ kubectl get pod
NAME    READY   STATUS      RESTARTS   AGE
mypod   0/1     Completed   0          6s

$ kubectl logs mypod
redis.host=127.0.0.1
redis.port=6379
redis.password=123456
```

# 四、以 变量 的形式挂载到 Pod 中去
```bash
$ vim myconfig.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfig
  namespace: default
data:
  special.level: info
  special.type: hello

$ kubectl apply -f myconfig.yaml 
configmap/myconfig created

$ kubectl get cm
NAME           DATA   AGE
myconfig       2      3s
redis-config   1      5m8s

$ vim config-var.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: busybox
      image: busybox
      command: [ "/bin/sh", "-c", "echo $(LEVEL) $(TYPE)" ]
      env:
        - name: LEVEL
          valueFrom:
            configMapKeyRef:
              name: myconfig
              key: special.level
        - name: TYPE
          valueFrom:
            configMapKeyRef:
              name: myconfig
              key: special.type
  restartPolicy: Never


$ kubectl apply -f config-var.yaml 
pod/mypod created

$ kubectl get pods
NAME    READY   STATUS              RESTARTS   AGE
mypod   0/1     ContainerCreating   0          3s

$ kubectl logs mypod
info hello

```


# Downward API
作用: 让 Pod 里的容器能够直接获取到这个 **Pod 对象本身的信息**

例如:
```yaml
$ vim downward-vol.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-downwardapi-volume
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
spec:
  containers:
    - name: client-container
      image: busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      projected:
        sources:
        - downwardAPI:
            items:
              - path: "labels"
                fieldRef:
                  fieldPath: metadata.labels
```
在这个 Yaml 文件中, 声明了一个 projected 类型的 volume, 数据源是 Downward API, 这个 Downward API column 声明了要暴露 Pod 的 metadata.labels 信息给容器

通过这样的声明, 当前 Pod 的 labels 字段的值, 就会被 k8s 自动挂载称为容器里面的 /etc/podinfo/labels 文件

当启动这个容器的时候, 会不断的打印出 /etc/podinfo/labels 里的内容, 可以通过 kubectl logs 查看
```bash
$ kubectl apply -f downward-vol.yaml
$ kubectl logs test-downwardapi-volume
```

到目前为止 Downward API 支持的字段如下: 
```yaml
1. 使用fieldRef可以声明使用:
spec.nodeName - 宿主机名字
status.hostIP - 宿主机IP
metadata.name - Pod的名字
metadata.namespace - Pod的Namespace
status.podIP - Pod的IP
spec.serviceAccountName - Pod的Service Account的名字
metadata.uid - Pod的UID
metadata.labels['<KEY>'] - 指定<KEY>的Label值
metadata.annotations['<KEY>'] - 指定<KEY>的Annotation值
metadata.labels - Pod的所有Label
metadata.annotations - Pod的所有Annotation

2. 使用resourceFieldRef可以声明使用:
容器的CPU limit
容器的CPU request
容器的memory limit
容器的memory request
```

需要注意的是, Downward API 能够获取到的信息, 一定是 Pod 里的容器进程启动之前就能够确定下来的, 如果你想获取容器进程的 Pid, 则通过 Downward API 获取不到


# 四、补充
需要注意的是, 通过 Secret/ConfigMap 的方式挂载到容器里, 一旦其对应的 Etcd 里的数据被更新, 这些 volume 里的内容也会被更新, 但是存在一定的延迟, 如果以环境变量的方式挂载, 则不会自动更新
