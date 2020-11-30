






---
# 一、副本机制&水平扩展&水平收缩
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
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
在上面的 Yaml 文件中, 定义了一个Nginx镜像有三个副本(replicas: 3), 这个 Deployment 和 ReplicaSet 的关系如下图

![Deployment和ReplicaSet%20关系图](../../img/k8s/副本机制&水平扩容&滚动升级/Deployment和ReplicaSet%20关系图.png)

ReplicaSet 保证系统中 Pod 的个数永远等于指定的个数
- 当 Pod 的个数小于指定的个数时, 会自动创建一个新的 Pod, 这就是 "水平扩展"
- 当 Pod 的个数大于指定的个数时, 会自动销毁一个已经存在的 Pod, 这就是 "水平收缩"

触发 "水平扩展"、"水平收缩" 有如下指令:
```bash
$ kubectl scale deployment nginx-deployment --replicas=4
```
此时, Pod 的个数就应该为 4 个
```bash
[root@k8s-master k8s-yaml]# kubectl get pod 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-8q4q4   1/1     Running   0          13m
nginx-deployment-5bf87f5f59-lpxds   1/1     Running   0          13m
nginx-deployment-5bf87f5f59-nlpxm   1/1     Running   0          10m
nginx-deployment-5bf87f5f59-scm58   1/1     Running   0          13m
```

# 滚动更新
这里先停止上面启动的 Pod,
```bash
$ kubectl delete -f nginx-deployment.yaml
```

在创建 Pod 的命令上加一个参数: `--record`, 表示记录下每次操作的命令, 方便后面查看
```bash
$ kubectl apply -f nginx-deployment.yaml --record

$ kubectl get pod 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-5tj6z   1/1     Running   0          35s
nginx-deployment-5bf87f5f59-756wt   1/1     Running   0          35s
nginx-deployment-5bf87f5f59-spv8b   1/1     Running   0          35s
```

这里首先检查一下 nginx-deployment 创建后的状态信息
```bash
[root@k8s-master k8s-yaml]# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
```
在返回的结果中, 我们可以看到 3 个状态字段, 含义如下:
1. READY: 当前准备完成的 Pod 个数
2. UP-TO-DATE: 当前处于最新版本的 Pod 的个数，所谓最新版本指的是 Pod 的 Spec 部分与 Deployment 里 Pod 模板里定义的完全一致；
3. AVAILABLE: 当前已经可用的 Pod 的个数，即：既是 Running 状态，又是最新版本，并且已经处于 Ready（健康检查正确）状态的 Pod 的个数。

这 3 个状态字段中, 只有 AVAILABLE 是用户期望的最终状态

k8s 提供了一条指令, 可以 实时查看 Deployment 对象的变化状态:
```bash
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...deployment.apps/nginx-deployment successfully rolled out
```
在这个返回结果中，"2 out of 3 new replicas have been updated" 意味着已经有 2 个 Pod 进入了 UP-TO-DATE 状态。继续等待一会儿，我们就能看到这个 Deployment 的 3 个 Pod, 就进入到了 AVAILABLE 状态:
```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           27s
```

这时候还可以尝试查看一下这个 Deployment 所控制的 ReplicaSet
```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-5bf87f5f59   3         3         3       40s
```

如上所示, 在用户提交了一个 Deployment 对象后, Deployment Controller 就会立即创建一个 Pod 副本个数为 3 的 ReplicaSet, 这个 ReplicaSet 的名字, 由 Deployment 的名字和一个随机字符串共同组成


如果这个时候, 我们修改了 Deployment 的 Pod 模板, 就会触发 "滚动更新" 的操作, 这里我们修改 Nginx 的版本
```bash
$ kubectl edit deployment/nginx-deployment
...
containers:
  - image: nginx:1.8  # 从 1.7.9 -> 1.8
    imagePullPolicy: IfNotPresent
    name: nginx
...
```
在 `kubectl edit` 指令编辑完成后, 保存退出, k8s 就会立即触发 "滚动更新" 的操作

此时可以通过如下命令查看 nginx-deployment 的变化过程
```bash
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...deployment.extensions/nginx-deployment successfully rolled out
```

也可以通过 `kubectl describe pod ` 查看Deployment 的 Event, 可以看到 "滚动更新" 的过程
```bash
$ kubectl describe deployment nginx-deployment
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  113s  deployment-controller  Scaled up replica set nginx-deployment-5bf87f5f59 to 3
  Normal  ScalingReplicaSet  63s   deployment-controller  Scaled up replica set nginx-deployment-5f8c6846ff to 1
  Normal  ScalingReplicaSet  62s   deployment-controller  Scaled down replica set nginx-deployment-5bf87f5f59 to 2
  Normal  ScalingReplicaSet  62s   deployment-controller  Scaled up replica set nginx-deployment-5f8c6846ff to 2
  Normal  ScalingReplicaSet  61s   deployment-controller  Scaled down replica set nginx-deployment-5bf87f5f59 to 1
  Normal  ScalingReplicaSet  61s   deployment-controller  Scaled up replica set nginx-deployment-5f8c6846ff to 3
  Normal  ScalingReplicaSet  58s   deployment-controller  Scaled down replica set nginx-deployment-5bf87f5f59 to 0
```
像这样的, 将一个集群中正在运行的多个 Pod 版本, 交替地逐一升级的过程, 就是 "滚动更新"


在这个 "滚动更新" 过程完成之后，可以查看一下新、旧两个 ReplicaSet 的最终状态:
```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-5bf87f5f59   0         0         0       3m11s
nginx-deployment-5f8c6846ff   3         3         3       2m21s
``` 

## 滚动更新的好处
在滚动更新的时候, 旧的 Pod 会在新的 Pod 创建出来之前一直在线, 确保服务的连续性, 服务并不会受到太大的影响

k8s 为了进一步确保服务的连续性, Deployment Controller 还会确保 在任何时间窗口内, 只有指定比例的 Pod 处于离线状态, 同时, 它也会确保, 在任何时间窗口内, 只有指定比例的新的 Pod 被创建出来, 这两个比例都是可以配置的, 默认都是 DESIRED 值得 25%

所以, 在上面这个 Deployment 的例子中, 它有 3 个 Pod 副本, 那么控制器在 "滚动更新" 的过程中永远都会确保至少有 2 个 Pod 处于可用状态, 至多只有 4 个 Pod 同时存在于集群中。这个策略, 是 Deployment 对象的一个字段, 名叫 RollingUpdateStrategy, 如下所示:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
...
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

在上面这个 RollingUpdateStrategy 的配置中，maxSurge 指定的是除了 DESIRED 数量之外，在一次“滚动”中，Deployment 控制器还可以创建多少个新 Pod；而 maxUnavailable 指的是，在一次“滚动”中，Deployment 控制器可以删除多少个旧 Pod。

同时，这两个配置还可以用前面我们介绍的百分比形式来表示，比如：maxUnavailable=50%，指的是我们最多可以一次删除“50%*DESIRED 数量”个 Pod。

结合以上讲述，现在我们可以扩展一下 Deployment、ReplicaSet 和 Pod 的关系图了。
![Deployment和ReplicaSet%Deployment、ReplicaSet%20和%20Pod%20的关系图](../../img/k8s/副本机制&水平扩容&滚动升级/Deployment、ReplicaSet%20和%20Pod%20的关系图了.png)
