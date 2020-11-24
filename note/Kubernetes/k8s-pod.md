
* [一、Pod基本概念](#%E4%B8%80pod%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)
* [二、Pod存在的意义](#%E4%BA%8Cpod%E5%AD%98%E5%9C%A8%E7%9A%84%E6%84%8F%E4%B9%89)
* [三、Pod实现机制](#%E4%B8%89pod%E5%AE%9E%E7%8E%B0%E6%9C%BA%E5%88%B6)
  * [例子一: Tomcat 和 War 包部署](#%E4%BE%8B%E5%AD%90%E4%B8%80-tomcat-%E5%92%8C-war-%E5%8C%85%E9%83%A8%E7%BD%B2)
* [四、Pos镜像拉取策略](#%E5%9B%9Bpos%E9%95%9C%E5%83%8F%E6%8B%89%E5%8F%96%E7%AD%96%E7%95%A5)
* [五、Pod资源限制](#%E4%BA%94pod%E8%B5%84%E6%BA%90%E9%99%90%E5%88%B6)
* [六、Pod重启机制](#%E5%85%ADpod%E9%87%8D%E5%90%AF%E6%9C%BA%E5%88%B6)
* [七、Pod健康检查](#%E4%B8%83pod%E5%81%A5%E5%BA%B7%E6%A3%80%E6%9F%A5)
* [八、节点亲和性](#%E5%85%AB%E8%8A%82%E7%82%B9%E4%BA%B2%E5%92%8C%E6%80%A7)
* [九、label 对Pod调度的影响](#%E4%B9%9Dlabel-%E5%AF%B9pod%E8%B0%83%E5%BA%A6%E7%9A%84%E5%BD%B1%E5%93%8D)
* [十、污点和污点容忍](#%E5%8D%81%E6%B1%A1%E7%82%B9%E5%92%8C%E6%B1%A1%E7%82%B9%E5%AE%B9%E5%BF%8D)
  * [10\.1 查看节点污染情况](#101-%E6%9F%A5%E7%9C%8B%E8%8A%82%E7%82%B9%E6%B1%A1%E6%9F%93%E6%83%85%E5%86%B5)
  * [10\.2 为节点添加污点](#102-%E4%B8%BA%E8%8A%82%E7%82%B9%E6%B7%BB%E5%8A%A0%E6%B1%A1%E7%82%B9)
  * [10\.3 污点容忍](#103-%E6%B1%A1%E7%82%B9%E5%AE%B9%E5%BF%8D)

---
# 一、Pod基本概念
Pod 是 k8s 中可以创建和管理的最小单元

Pod 是在 k8s 上运行容器化应用的资源对象, 其他的资源对象都是用来支撑或者扩展 Pod 对象功能的, 例如:
- 控制器对象是用来管控 Pod 对象的
- Service 或者 Ingress 资源对象是用来暴露 Pod 引用对象的
- PersistentVolume 资源对象是用来为 Pod 提供存储

k8s 不会直接处理容器本身, 而是处理 Pod

Pod 是由一个或多个 container 组成的


# 二、Pod存在的意义
假设我们的 k8s 集群上有两个几点: node1 有 3G 可用内存, node2 有 2.5G 可用内存, 这是需要用 Docker Swarm 来运行一个程序(包含3个容器, 每个都需要1G 内存, 3个容器互相依赖, 且必须运行在同一台机器上), 当这三个容器进入 Swarm 的调度队列, 然后这三个容器先后被调度到 node2 上(完全有可能的, 因为 第1个容器仅需要1G内存), 当最后一个容器被调度时, 集群上的可用内存仅有 0.5G 了, 但是这个程序中的3个容器存在互相依赖的约束

在 k8s 中, 会将这三个容器组成一个 Pod, k8s 在调度的时候, 自然就会去选择可用内存等于 3G 的 node1 节点进行绑定, 根本不会考虑 node2




# 三、Pod实现机制
Pod 中的所有容器, 会共享同一个 network namespace, 并且可以声明共享一个 volume  

在 k8s 中, Pod 的实现需要使用一个中间容器, 这个容器叫做: Infra容器. 在这个 Pod 中, Infra 容器永远都是第一个被创建的容器, 而其他用户定义的容器, 则通过 Join NetWork NameSpace 的方式, 与 Infra 容器关联在一起, 关系如下图:

![Infra关系](../../img/k8s/pod/Infra关系.png)

如图中所示, 这个 Pod 中有两个用户容器 A 和 B, 还有一个 Infra 容器, 在 k8s 中, Infra 容器永远处于 "暂停状态", 且占用极少的资源

在 Infra 容器初始了 Network NameSpace 之后, 用户容器就可以加入到 Infra 容器的 Network NameSpace 当中, 这也就意味着, 对于 Pod 里的容器 A 和 容器B来说:
1. 他们可以直接用 localhost 来进行通信
2. 他们看到的网络设备跟 Infra 容器看到的完全一样
3. 一个 Pod 只有一个 IP 地址, 也就是这个 Pod 的 Network NameSpace 对应的 IP 地址
4. 其他的所有网络资源, 都是一个 Pod 一份, 并且被该 Pod 中的所有容器共享
5. Pod 的生命周期只跟 Infra 容器一致, 而与 容器A 和 容器B 无关

对于容一个 Pod 里的所有用户容器来说, 他们的进出流量, 也可以认为都是通过 Infra 容器完成的。


对于 volume 的共享, K8s 只需要把所有的 volume 的定义都设计在 Pod 层级即可, 例如下面的例子:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  restartPolicy: Never
  volumes:
  - name: shared-data
    hostPath:      
      path: /data
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
```
在这个例子中, nginx-container 和 debian-container 都声明挂载了 shared-data 这个 volume, 而 shared-data 是 hostPath 类型, 所以, 它对应宿主机上的目录就是 /data, 而这个目录, 就同时被绑定进了这两个容器中, 这样的话 nginx-container 容器就可以从 /usr/share/nginx/html 目录中读取到 debian-container 生成的 index.html 文件了

## 例子一: Tomcat 和 War 包部署
有一个 JavaWeb 应用的 War 包, 需要被放在 Tomcat 的 webapps 目录下运行起来

我们可以把 Tomcat 和 War 包分别做成两个镜像, 然后把这两个镜像作为一个容器
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javaweb-2
spec:
  initContainers:
  - image: geektime/sample:v2
    name: war
    command: ["cp", "/sample.war", "/app"]
    volumeMounts:
    - mountPath: /app
      name: app-volume
  containers:
  - image: geektime/tomcat:7.0
    name: tomcat
    command: ["sh","-c","/root/apache-tomcat-7.0.42-v2/bin/start.sh"]
    volumeMounts:
    - mountPath: /root/apache-tomcat-7.0.42-v2/webapps
      name: app-volume
    ports:
    - containerPort: 8080
      hostPort: 8001 
  volumes:
  - name: app-volume
    emptyDir: {}
```
`spec.initContainers`: 会比 spec.containers 定义的用户容器先启动, 并且 Init Container 容器会按顺序逐一启动, 直到所有的 Init Container 容器全部启动完成后, 用户容器才会启动

这个例子声明了一个名为 app-volume 的 volume, 声明了两个容器: war 和 tomcat 都挂载声明的 volume, 首先将 war 容器初始化, 并且将 宿主机根目录下的 /sample.war cp 到容器的 /app 目录下; 然后初始化 tomcat 容器, 执行 command 中的命令, 并且暴露 8080 端口

# 四、Pos镜像拉取策略
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec: 
  comtainers:
    - name: nginx
    inage: nginx:1.14
    imagePullPolicy: Always
```
imagePullPolicy有三个参数可选:
- IfNotPresent: 默认值, 镜像在宿主机上不存在时才拉取
- Always: 每次创建 pod 都会重新拉取一次镜像
- Never: Pod 永远不会主动拉取这个镜像


# 五、Pod资源限制
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  comtainers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
- resources.requests:  要求Container的内存和cpu必须有这么大
- resources.limits: 要求Container的内存最大为这么大

# 六、Pod重启机制
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: dns-test
spec:
  containers:
    - name: busybox
      image: busybox:1.28.4
      args:
      - /bin/sh
      - -c
      - sleep 36000
    restartPolicy: Never
```
restartPolicy有三个可选的参数:
- Always: 当容器终止退出后, 总是重启容器, 默认策略
- OnFailure: 当容器异常退出(退出状态码非0)时, 才重启容器
- Never: 当容器种植退出, 从不重启容器


# 七、Pod健康检查
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy
    livenessProbe(readinessProbe):
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
- livenessProbe(存活检查): 如果检查失败, 将杀死容器, 根据 Pod 的 restartPolicy 来操作.
- readinessProbe(就绪检查): 如果检查失败, k8s 会把该Pod从 service endpoints中剔除

Probe支持以下三种检查方式:
- httpGet: 发送 Http 请求, 返回200-400范围状态码为成功
- exec: 执行 Shell 命令返回状态码是0为成功
- tcpSocket: 发起 TCP Socket 建立成功

# 八、节点亲和性
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env_role
            operator: In
            values:
            - dev
            - test
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference: 
            matchExpressions:
            - key: group
              operator: In
              values:
              - otherprod
  containers:
  - name: webdemo
    image: nginx
```
- requiredDuringSchedulingIgnoredDuringExecution: 硬亲和性, 约束条件必须满足, 否则一直等待到条件满足
- preferredDuringSchedulingIgnoredDuringExecution: 软亲和性, 尝试满足, 不保证约束条件满足
  - weight: 权重
  - operator: 操作符
    - In
    - NotIn
    - Exists
    - Gt
    - Lt
    - DoesNotExists
    
# 九、label 对Pod调度的影响
[对节点创建 label](k8s-label.md)

```bash
spex: 
  nodeSelector"
    env_role: dev
  containers:
  - name: nginx
    image: nginx:1.15
```

# 十、污点和污点容忍
污点(Taint): 节点不做普通分配调度, 是node属性

和 `九、label 对Pod调度的影响`、 `八、节点亲和性` 类似

## 10.1 查看节点污染情况
```bash
$ kubectl describe node master | grep Taint
Taints:             node.kubernetes.io/unreachable:NoExecute
```
污点值有如下三个:
- NoSchedule: 一定不被调度
- PreferNoSchedule: 尽量不被调度
- NoExecute: 不会调度, 并且会驱除 Node 已有 Pod

## 10.2 为节点添加污点
```bash
# 添加污点
$ kubectl taint node k8s-master <key>=<value>:<effect>
$ kubectl taint node k8s-master kino=test:NoSchedule
node/k8s-master tainted

# 查看污点
$ kubectl describe node k8s-master | grep Taint
Taints:             kino=test:NoSchedule
```

## 10.3 污点容忍
```yaml
spec: 
  tolerations:
  - key: "ey"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: webdemo
    image: nginx
```