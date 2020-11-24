
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