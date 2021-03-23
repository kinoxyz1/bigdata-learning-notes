





--- 


# 一、容器建行检查
在 Kubernetes 中可以为每个 Pod 定义一个健康检查的 "探针"(Probe), 这样 Kubernetes 就会根据这个 Probe 的返回值决定这个容器的状态, 而不是直接以容器镜像是否运行作为依据, 这种机制是生产环境中保证应用是否健康存活的重要手段.

```bash
$ vim liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: test-liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
在这个例子中, 定义了一个容器: busybox, 容器在启动后, 会在 /tmp 目录下创建healthy 文件夹, 以此作为自己正在正常运行的标志, 30s 之后, 再将这个容器删除掉。

同时, 在这个 yaml 文件中定义了一个 livenessProbe(健康检查), 它的类型是 exec, 这意味着会在容器启动后, 在容器中执行一条: `cat /tmp/healthy` 的命令, 如果这个文件存在, 这条命令的返回值就是 0, Pod 就会任务这个容器不仅已经启动, 而且是建行的, 这个健康检查, 在容器启动 5s 后开始执行(initialDelaySeconds: 5), 每 5s 执行一次(periodSeconds: 5).

创建Pod:
```bash
$ kubectl apply -f liveness.yaml
```
查看 Pod 的状态:
```bash
$ kubectl get pod 
NAME                 READY   STATUS    RESTARTS   AGE
test-liveness-exec   1/1     Running   0          8s
```
可以看到, 这个 Pod 通过了健康检查, 就进入到 Running 状态了

等 30s 之后, 查看该 Events 的信息, 会发现有一个异常
```bash
Events:
  Type     Reason     Age              From                Message
  ----     ------     ----             ----                -------
  Normal   Scheduled  <unknown>        default-scheduler   Successfully assigned default/test-liveness-exec to k8s-node2
  Normal   Pulling    46s              kubelet, k8s-node2  Pulling image "busybox"
  Normal   Pulled     40s              kubelet, k8s-node2  Successfully pulled image "busybox"
  Normal   Created    40s              kubelet, k8s-node2  Created container liveness
  Normal   Started    40s              kubelet, k8s-node2  Started container liveness
  Warning  Unhealthy  1s (x2 over 6s)  kubelet, k8s-node2  Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```
这时候, /tmp/healthy 已经不存在了, 所以是不健康的了, 这时候再查看 Pod 的状态, 会发现这个容器状态还是 Running 状态, 并没有进入 Failed 状态
```bash
$ kubectl get pod 
NAME                 READY   STATUS    RESTARTS   AGE
test-liveness-exec   1/1     Running   1          3m55s
```
此时 RESTARTS 从 0 变成了 1, 显然这个容器已经被 Kubernetes 重启了, 在这个过程中, Pod 保持 Running 不变。

需要注意的是, Kubernetes 中没有 docker 中的 stop, 所以虽然是 Restart, 但实际上却是重新创建了 容器

# 二、恢复机制 
Kubernetes 在 容器发生异常时, 会重新创建, 这个机制就是 Kubernetes 里的 **Pod 恢复机制**, 也叫: `restartPolicy`

`restartPolicy` 是 Pod 的 Spec 部分的一个重要字段(pod.spec.restartPolicy), 默认值是 Always: 在任何时候这个容器发生了异常, 一定要重新创建.

在 Kubernetes 中, Pod 的恢复永远都是发生在当前节点上的, 不会跑到别的节点上去, 一旦一个 Pod 与一个节点(Node) 绑定, 除非这个绑定发生了变化(pod.spec.node 字段被修改), 否则它永远都不会离开这个节点, 这这也就意味着, 如果这个宿主机宕机了, 这个 Pod 也不会主动迁移到其他节点上去. 如果想要 Pod 出现在其他节点上, 就必须使用 Deployment 这样的控制器来管理 Pod

`restartPolicy` 一共有三种恢复策略:
1. Always: 在任何情况, 只要容器不在运行, 就自动重启容器;
2. OnFailure: 在容器运行异常时, 才重启容器;
3. Never: 不重启容器;

这三种恢复策略需根据实际情况选择, 例如如果是计算一批数据的总和, 计算完成后状态就变成了 Succeeded, 这时候如果再用 `restartPolicy=Always` 强制重启 Pod, 就没有任何意义; 如果需要关心容器的运行时状态, 比如日志等, 就需要将 `restartPolicy` 设置为 Never, 因为容器一旦被自动创建, 这些内容就有可能丢失。

在 Kubernetes 中, 官方将 restartPolicy 和 Pod 里容器的状态, 以及 Pod 状态对应的关系, [分了很多情况](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#example-states), 这里是张磊老师说的两种设计原理:
1. 只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器(Always和OnFailure), 那么这个 Pod 就会保持 Running 状态, 并进行容器重启, 否则, Pod 就会进去 Failed 状态;
2. 对于包含多个容器的 Pod, 只有他里面所有的容器都进入异常状态后, Pod 才会进入 Failed 状态。在此之前, Pod 都是 Running 状态, 此时 Pod 的 READY 字段会显示正常容器的个数, 比如:
```bash
$ kubectl get pod 
NAME                                                         READY   STATUS    RESTARTS   AGE
mssql-2019-7b6c4c4b8-92v52                                   0/1     Running   204        16h
```
所以, 如果只有一个 Pod 里面只有一个容器, 然后这个容器异常退出了, 那么, 只有当 `restartPolicy=Never` 时, 这个 Pod 才会进入 Failed 状态, 其他情况下, 由于 Kubernetes 都可以重启这个容器, 所以 Pod 的状态保持 Running 不变.

如果这个 Pod 中有多个容器, 仅有一个容器异常退出, 它就始终保持 Running 状态, 哪怕及时 `restartPolicy=Never`, 只有当其中所有容器都异常退出后, 这个 Pod 才会进入 Failed 状态。


# 三、livenessProbe 类型
livenessProbe 除了 exec 类型以外, 还支持 HTTP 和 TCP 这两种类型的请求
```bash
...
livenessProbe:
     httpGet:
       path: /healthz
       port: 8080
       httpHeaders:
       - name: X-Custom-Header
         value: Awesome
       initialDelaySeconds: 3
       periodSeconds: 3
```
```bash
...
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```