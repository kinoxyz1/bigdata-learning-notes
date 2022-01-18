* [create](#create)
* [get](#get)
* [describe](#describe)
  * [查看这个节点(Node)对象的详细信息、状态和事件(Event)](#%E6%9F%A5%E7%9C%8B%E8%BF%99%E4%B8%AA%E8%8A%82%E7%82%B9node%E5%AF%B9%E8%B1%A1%E7%9A%84%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF%E7%8A%B6%E6%80%81%E5%92%8C%E4%BA%8B%E4%BB%B6event)
  * [查看 node、pod 等详细信息](#%E6%9F%A5%E7%9C%8B-nodepod-%E7%AD%89%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF)
* [replace](#replace)
* [apply](#apply)
* [exec](#exec)
* [delete](#delete)
* [label](#label)
  * [查看 label](#%E6%9F%A5%E7%9C%8B-label)
  * [修改 label](#%E4%BF%AE%E6%94%B9-label)
  * [删除 label](#%E5%88%A0%E9%99%A4-label)
* [扩容/缩容](#%E6%89%A9%E5%AE%B9%E7%BC%A9%E5%AE%B9)
* [弹性伸缩](#%E5%BC%B9%E6%80%A7%E4%BC%B8%E7%BC%A9)
* [导出 Yaml 文件](#%E5%AF%BC%E5%87%BA-yaml-%E6%96%87%E4%BB%B6)
* [应用升级](#%E5%BA%94%E7%94%A8%E5%8D%87%E7%BA%A7)
* [查看升级版本](#%E6%9F%A5%E7%9C%8B%E5%8D%87%E7%BA%A7%E7%89%88%E6%9C%AC)
* [回滚到上一版本](#%E5%9B%9E%E6%BB%9A%E5%88%B0%E4%B8%8A%E4%B8%80%E7%89%88%E6%9C%AC)
* [回滚到指定版本](#%E5%9B%9E%E6%BB%9A%E5%88%B0%E6%8C%87%E5%AE%9A%E7%89%88%E6%9C%AC)
* [查看回滚状态](#%E6%9F%A5%E7%9C%8B%E5%9B%9E%E6%BB%9A%E7%8A%B6%E6%80%81)
* [污点相关](#%E6%B1%A1%E7%82%B9%E7%9B%B8%E5%85%B3)
  * [1\.1 添加污点:](#11-%E6%B7%BB%E5%8A%A0%E6%B1%A1%E7%82%B9)
  * [1\.2 查看污点](#12-%E6%9F%A5%E7%9C%8B%E6%B1%A1%E7%82%B9)
  * [1\.3 删除污点](#13-%E5%88%A0%E9%99%A4%E6%B1%A1%E7%82%B9)
  * [1\.4 污点容忍度](#14-%E6%B1%A1%E7%82%B9%E5%AE%B9%E5%BF%8D%E5%BA%A6)
    * [① 等值判断](#-%E7%AD%89%E5%80%BC%E5%88%A4%E6%96%AD)
    * [② 存在性判断](#-%E5%AD%98%E5%9C%A8%E6%80%A7%E5%88%A4%E6%96%AD)
* [存储库相关](#%E5%AD%98%E5%82%A8%E5%BA%93%E7%9B%B8%E5%85%B3)
  * [1\. 添加存储库](#1-%E6%B7%BB%E5%8A%A0%E5%AD%98%E5%82%A8%E5%BA%93)
  * [2\. 查看存储库](#2-%E6%9F%A5%E7%9C%8B%E5%AD%98%E5%82%A8%E5%BA%93)
  * [3 移除存储库](#3-%E7%A7%BB%E9%99%A4%E5%AD%98%E5%82%A8%E5%BA%93)
  
  
  
---

# 一、Pod 相关



# 二、deploy 相关
## 2.1 根据 deploy 暴露 NodePort
```bash
kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
```


# create
语法:
```bash
4 kubectl create -f yaml文件名
```
示例:
```bash
# create 
$ kubectl create -f nginx-deployment.yaml

# kubectl get 命令检查这个 YAML 运行起来的状态是不是与我们预期的一致
$ kubectl get pods -l app=nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-pkkrf   1/1     Running   0          4m14s
nginx-deployment-5bf87f5f59-sdvqd   1/1     Running   0          4m14s
```
nginx-deployment.yaml
```yaml
$ vim nginx-deployment.yaml
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


# get
① 查看节点状态
```bash
$ kubectl get nodes
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    master   2d4h   v1.18.0
k8s-node1    Ready    <none>   2d4h   v1.18.0
k8s-node2    Ready    <none>   2d4h   v1.18.0
```
② 查看指定 pod
```bash
$ kubectl get pods -l app=nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-pkkrf   1/1     Running   0          4m14s
nginx-deployment-5bf87f5f59-sdvqd   1/1     Running   0          4m14s
```
`kubectl get` 指令的作用: 就是从 Kubernetes 里面获取（GET）指定的 API 对象

可以看到, 在这里我还加上了一个 -l 参数, 即获取所有匹配 app: nginx(spec.template.metadata.labels的值) 标签的 Pod。

需要注意的是, 在命令行中, 所有 key-value 格式的参数, 都使用 “=” 而非 “:” 表示

③ 检查节点上各个系统 Pod 的状态
```bash
$ kubectl get pods -n kube-system
NAME                                 READY   STATUS    RESTARTS   AGE
coredns-7ff77c879f-7jvnh             1/1     Running   1          2d4h
coredns-7ff77c879f-kh7j9             1/1     Running   1          2d4h
etcd-k8s-master                      1/1     Running   1          2d4h
kube-apiserver-k8s-master            1/1     Running   1          2d4h
kube-controller-manager-k8s-master   1/1     Running   1          2d4h
kube-flannel-ds-9r9xv                1/1     Running   1          2d4h
kube-flannel-ds-n6x9j                1/1     Running   1          2d4h
kube-flannel-ds-wbjrv                1/1     Running   1          2d4h
kube-proxy-5lj6m                     1/1     Running   1          2d4h
kube-proxy-9n774                     1/1     Running   1          2d4h
kube-proxy-dv9r4                     1/1     Running   1          2d4h
kube-scheduler-k8s-master            1/1     Running   1          2d4h
```
④ 查看 deployment
```bash
$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           2d5h
```


# describe
## 查看这个节点(Node)对象的详细信息、状态和事件(Event)
```bash
$ kubectl describe node k8s-master
Name:               k8s-master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
```

## 查看 node、pod 等详细信息
```bash
# 查看 node 详细信息
$ kubectl describe node k8s-node1

# 查看 pod 详细信息
# 查看有哪些 pod
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-pkkrf   1/1     Running   0          9m40s
nginx-deployment-5bf87f5f59-sdvqd   1/1     Running   0          9m40s
# 查看指定 pod 的详细信息
$ kubectl describe pod nginx-deployment-5bf87f5f59-pkkrf
```
在 `kubectl describe` 命令返回的结果中, 你可以清楚地看到这个 Pod 的详细信息, 比如它的 IP 地  `址等等。

其中, 有一个部分值得你特别关注, 它就是 Events（事件）。

在 Kubernetes 执行的过程中, 对 API 对象的所有重要操作, 都会被记录在这个对象的 Events 里, 并且显示在 kubectl describe 指令返回的结果中。

比如, 对于这个 Pod, 我们可以看到它被创建之后, 被调度器调度（Successfully assigned）到了 node-1, 拉取了指定的镜像（pulling image）, 然后启动了 Pod 里定义的容器（Started container）。

所以, 这个部分正是我们将来进行 Debug 的重要依据。如果有异常发生, 你一定要第一时间查看这些 Events, 往往可以看到非常详细的错误信息。


# replace
更新替换资源(升级资源)

例如升级 Nginx(上面 crete 示例使用的是 Nginx1.7.9, 这里升级为1.8)
```yaml
$ vim nginx-deployment.yaml
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

# 更新
$ kubectl replace -f nginx-deployment.yaml
deployment.apps/nginx-deployment replaced
```


# apply
apply 集合了 crete和replace两个指令的功能, 使用 `kubectl apply` kubernetes 会根据 YAML 文件的内容变化, 自动进行具体的处理。

例如上述 创建 Nginx、升级 Nginx的操作在这里可以写成:
```bash
$ kubectl apply -f nginx-deployment.yaml
$ vim nginx-deployment.yaml  # 更新版本
$ kubectl apply -f nginx-deployment.yaml
```

# exec
进入到指定的 pod 中
```bash
# 查看所有的 pod
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9754ccbdf-5spwg   1/1     Running   0          2m12s
nginx-deployment-9754ccbdf-nlbwl   1/1     Running   0          87s

# 进入 pod
$ kubectl exec -it nginx-deployment-9754ccbdf-5spwg -- /bin/bash
root@nginx-deployment-9754ccbdf-5spwg:/# ls /usr/share/nginx/html
```

# delete
① 根据yaml文件删除创建资源
```bash
$ kubectl delete -f nginx-deployment.yaml

# 查看 pod
$ kubectl get pods
NAME                               READY   STATUS        RESTARTS   AGE
nginx-deployment-9754ccbdf-5spwg   1/1     Terminating   0          4m50s
nginx-deployment-9754ccbdf-nlbwl   1/1     Terminating   0          4m5s

# 隔一会在查看 pod 
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE

```
② 删除已经创建的 pod
```bash
$ kubectl delete pod nginx-f89759699-4tz6t
pod "nginx-f89759699-4tz6t" deleted
```
查看 pod 仍然存在
```bash
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx-f89759699-g7smv   1/1     Running   0          53s
```
删除deployment
```bash
# 查看 deployment
$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           2d5h
$ kubectl delete deployment nginx
deployment.apps "nginx" deleted
```
再次查看, pod 消失
```bash
$ kubectl get pods
No resources found in default namespace.
```

# label
## 查看 label
```bash
$ kubectl get nodes --show-label
[root@k8s-master yaml]# kubectl get nodes --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-master   Ready    master   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,label_key=label_value,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux
```

## 修改 label
```bash
# 语法
$ kubectl label nodes <node> <key>=<value> --overwrite

# 例子
$ kubectl label nodes k8s-master label_key=label_value2 --overwrite
node/k8s-master labeled

# 查看
[root@k8s-master yaml]# kubectl get nodes --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-master   Ready    master   7d6h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,label_key=label_value2,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux
```

## 删除 label
```bash
# 语法
$ kubectl label node <node> <key>-
# 例子
$ kubectl get nodes --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-master   Ready    master   7d6h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux
```

# 扩容/缩容
```bash
$ kubectl scale deployment nginx-deployment --replicas 5
```

# 弹性伸缩
```bash
$ kubectl scale deployment web --replicas=5
```

# 导出 Yaml 文件
```bash
$ kubectl create deployment web --image=nginx --dry-run -o yaml > web.yaml
W1117 17:46:18.012487  127855 helpers.go:553] --dry-run is deprecated and can be replaced with --dry-run=client.

# 导出指定端口的 yaml 文件
$ kubectl expose deployment web --port=80 --target-port=80 --dry-run -o yaml > service.yaml
```

# 查看 secret 加密信息
```bash
$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-xd659   kubernetes.io/service-account-token   3      47h
mysecret              Opaque                                2      9m45s

$ kubectl get secret mysecret -o yaml
apiVersion: v1
data:
  password: OXpkYXRhMTIzLg==
  username: emhhbmdzYW4=
kind: Secret
...

$ echo -n "emhhbmdzYW4=" | base64 --decode
zhangsan
```

# 应用升级
```bash
$ kubectl set image deployment web nginx=nginx:1.5
```

# 查看升级版本
```bash
$ kubectl rollout history deployment web
```

# 回滚到上一版本
```bash
$ kubectl rollout undo deployment web
```

# 回滚到指定版本
```bash
$ kubectl rollout undo deployment web --to-revision=2
```

# 查看回滚状态
```bash
$ kubectl rollout status deployment web
```

# 污点相关
## 1.1 添加污点:
```bash
[root@k8s-master ~]# kubectl taint nodes <node-name> <key>=<value>:<effect>
[root@k8s-master ~]# kubectl taint nodes k8s-node1 foo=bar:NoSchedule
kubectl taint node k8s-node1 node-type=bar:NoSchedule
                     
```

## 1.2 查看污点
① 方式一
```bash
[root@k8s-master ~]# kubectl get nodes <nodename> -o go-template={{.spec.taints}}
[root@k8s-master ~]# kubectl get nodes k8s-node1 -o go-template={{.spec.taints}}
[map[effect:NoSchedule key:foo value:bar]]
```
② 方式二
```bash
[root@k8s-master ~]# kubectl describe node k8s-node1
Name:               k8s-node1
Roles:              <none>
...
Taints:             foo=bar:NoSchedule
```

## 1.3 删除污点
① 删除所有污点
```bash
[root@k8s-master ~]# kubectl patch nodes k8s-node1 -p '{"spec":{"taints":[]}}'
```
② 删除指定key的污点
```bash
[root@k8s-master ~]# kubectl taint nodes --all foo-
[root@k8s-master ~]# kubectl taint nodes --all node-type-
```
③ 删除指定key, 指定value的污点
```bash
[root@k8s-master ~]# kubectl taint nodes --all foo:NoSchedule-
[root@k8s-master ~]# kubectl taint nodes --all node-type:NoSchedule-
```

## 1.4 污点容忍度
### ① 等值判断
```yaml
tolerations:
- key: "key1"
  operator: "Equal" #判断条件为 Equal
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 3600
```
### ② 存在性判断
```yaml
tolerations: 
- key: "key1"
  operator: "Exists"#存在性判断，只要污点键存在，就可以匹配
  effect: "NoExecute"
  tolerationSeconds: 3600
apiVersion: v1
kind: Deployment
metadata: 
  name: myapp-deploy
  namespace: default
spec:
  replicas: 3
  selector: 
    matchLabels: 
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
      image: ikubernetes/myapp:v1
      ports:
      - name:http
        containerPort: 80
      tolerations:
      - key:"node-type"
        operator: "Equal"
        value:"production":
        effect: "NoExecute"
        tolerationSeconds: 3600
```

# 存储库相关
## 1. 添加存储库
```bash
$ helm repo add stable http://mirror.azure.cn/kubernetes/charts/
"stable" has been added to your repositories
$ helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
"aliyun" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```
## 2. 查看存储库
```bash
helm repo list
helm search repo stable
```
## 3 移除存储库
```bash
$ helm repo remove aliyun
```