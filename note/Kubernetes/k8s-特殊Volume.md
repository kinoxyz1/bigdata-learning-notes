

* [一、Secret 作用](#%E4%B8%80secret-%E4%BD%9C%E7%94%A8)
* [二、创建 secret 加密数据](#%E4%BA%8C%E5%88%9B%E5%BB%BA-secret-%E5%8A%A0%E5%AF%86%E6%95%B0%E6%8D%AE)
* [三、以变量的形式挂载到 pod 容器中](#%E4%B8%89%E4%BB%A5%E5%8F%98%E9%87%8F%E7%9A%84%E5%BD%A2%E5%BC%8F%E6%8C%82%E8%BD%BD%E5%88%B0-pod-%E5%AE%B9%E5%99%A8%E4%B8%AD)
* [四、以 volume 形式挂载到 pod 容器中](#%E5%9B%9B%E4%BB%A5-volume-%E5%BD%A2%E5%BC%8F%E6%8C%82%E8%BD%BD%E5%88%B0-pod-%E5%AE%B9%E5%99%A8%E4%B8%AD)


---
在 kubernetes v1.11 之后有一种特殊的 Volume, 称之为 "投射数据卷"(Projected Volume);

kubernetes 中一共支持四种 projected volume:
- Secret
- ConfigMap
- Downward Api
- ServiceAccountToken

在 kubernetes 中, 这几种特殊的 Volume 存在的意义不是为了存放容器里面的数据, 也不是用来进行容器和宿主机之间的数据交换, 而是为容器提供预先定义好的数据. 所以从容器的角度来看, 这些 Volume 里面的信息就是仿佛被 kubernetes "投射" 进容器当中的.


# 一、特殊 Volume 之 Secret
Secret 常见的使用场景就是存放数据库的相关信息(用户名、密码等), 

## 1.1 创建 Secret - 方式一
创建两个名为 name 和 pass 的 secret: 
```bash
$ vim ./user.txt
admin
$ vim ./pass.txt
c1oudc0w!

$ kubectl create secret generic user --from-file=./user.txt
$ kubectl create secret generic pass --from-file=./pass.txt
```
其中, user.txt 和 pass.txt 两个文件里面, 存放的就是 用户名 和 密码, 而 user 和 pass 则是为 Secret 对象指定的名字

查看 Secret 对象: 
```bash
$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-lmxpl   kubernetes.io/service-account-token   3      9d
pass                  Opaque                                1      81m
user                  Opaque                                1      81m
```

## 1.2 创建 Secret - 方式二(yaml)
创建 Secret 除了使用 `kubectl create secret` 命令之外, 还可以通过编写 YAML 文件的方式来创建 Secret 对象, 例如:
```bash
$ vim secret-yaml.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  user: YWRtaW4=
  pass: MWYyZDFlMmU2N2Rm
  
$ kubectl apply -f secret-yaml.yaml
secret/mysecret created

$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-lmxpl   kubernetes.io/service-account-token   3      13d
mysecret              Opaque                                2      6s
```
通过编写 YAML 文件创建出来的 Secret 对象只有一个, 但是它的 data 字段是以 Key-Value 键值对格式保存了两份 Secret 数据, 其中 "user" 就是第一份数据的 Key, "pass" 就是第二份数据的 Key.

需要注意的是, Secret 对象要求这些数据必须是经过 Base64 转码的, 以免出现明文密码的安全隐患

明文转 Base64 可以使用如下命令
```bash
$ echo -n 'admin' | base64
YWRtaW4=
$ echo -n '123456' | base64
```

像这样创建 Secret 对象, 它里面的内容仅仅是经过了转码, 并没有被加密, 在生产环境中, 需要在 kubernetes 中开启 Secret 的加密插件, 增强数据的安全性.

## 1.3、将 kubectl create secret 创建的 secret 挂载到 Pod 中
```bash
$ test-projected-volume-load-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-projected-volume 
spec:
  containers:
  - name: test-secret-volume
    image: busybox
    args:
    - sleep
    - "86400"
    volumeMounts:
    - name: mysql-cred
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: mysql-cred
    projected:
      sources:
      - secret:
          name: user
      - secret:
          name: pass
          
$ kubectl apply -f test-projected-volume-load-secret.yaml
pod/test-projected-volume created

$ kubectl get pod 
NAME                    READY   STATUS    RESTARTS   AGE
test-projected-volume   1/1     Running   0          12s
```
当 Pod 变成 Running 之后, 验证 Secret 是否已经在容器中:
```bash
$ kubectl exec -it test-projected-volume -- /bin/sh
/ # ls /projected-volume/
password.txt  username.txt
/ # cat /projected-volume/pass.txt 
c1oudc0w!
/ # cat /projected-volume/user.txt 
admin
```

## 1.4 将 YAML 方式创建的 secret 以 volume 的方式挂载到 pod 中
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
```
查看挂载信息:
```bash
$ kubectl get pod 
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          2m46s

$ kubectl exec -it mypod bash
root@mypod:/# ls -l /etc/foo/
total 0
lrwxrwxrwx 1 root root 11 Mar 22 04:00 pass -> ..data/pass
lrwxrwxrwx 1 root root 11 Mar 22 04:00 user -> ..data/user

root@mypod:/# cat /etc/foo/user 
adminroot@mypod:/# 
root@mypod:/# cat /etc/foo/pass 
1f2d1e2e67dfroot@mypod:/#
```

## 1.5 将 YAML 方式创建的 secret 以 变量 的方式挂载到 pod 中
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
      # 指定变量的名称是 SECRET_USERNAME
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            # name 是创建的 secret 的名称
            name: mysecret
            # key 是创建的 secret 里面执行的 Key
            key: user
      # 指定变量的名称是 SECRET_PASSWORD
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            # name 是创建的 secret 的名称
            name: mysecret
            # key 是创建的 secret 里面执行的 Key
            key: pass

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

## 1.6 删除 secret
```bash
$ kubectl get secret
NAME                         TYPE                                  DATA   AGE
secret/default-token-lmxpl   kubernetes.io/service-account-token   3      13d
secret/mysecret              Opaque                                2      4d20h
secret/pass                  Opaque                                1      4d20h
secret/user                  Opaque                                1      4d20h

$ kubectl delete secret user
secret "user" deleted
$ kubectl delete secret pass
secret "pass" deleted
$ kubectl delete secret mysecret
secret "mysecret" deleted
```

# 二、特殊 Volume 之 ConfigMap
ConfigMap 和 Secret 类似, ConfigMap 是将**不加密**数据存储到 etcd 中, 让 Pod 以变量的形式或者 Volume 挂载到容器中.

ConfigMap 的用于与 Secret 几乎一致, 可以通过 `kubectl create configmap` 从文件或者目录创建 ConfigMap, 也可以直接编写 ConfigMap 对象的 Yaml 文件.

## 2.1 从文件创建 ConfigMap
### 2.1.1 准备文件
```bash
$ vim redis.properties
redis.host=127.0.0.1
redis.port=6379
redis.password=123456
```
### 2.1.2 创建
```bash
$ kubectl create configmap redis-config --from-file=redis.properties 
configmap/redis-config created
```
### 2.1.3 查看
```bash
[root@k8s-master ConfigMap]# kubectl get configmap
NAME           DATA   AGE
redis-config   1      9s
[root@k8s-master ConfigMap]# kubectl get cm
NAME           DATA   AGE
redis-config   1      12s
```
### 2.1.4 查看 ConfigMap 详情
```bash
$ kubectl describe configmap redis-config
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

## 2.2 从 yaml 创建 ConfigMap 
```bash
$ vim configMap-var.yaml
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
```

## 2.2 将 ConfigMap(文件创建) 以 Volume 的形式挂载到 Pod
```bash
$ vim configMap-vol.yaml
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

## 2.3 将 ConfigMap(yaml) 以 变量 的形式挂载到 Pod
```bash
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

# 三、Downward API
Downward API 的作用是: 让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息.

Downward API 提供了两种方式用于将 Pod 里的信息注入到容器内部:
- 环境变量: 用于单个变量, 可以将 Pod 信息和容器信息直接注入容器内部;
- Volume 挂载: 将 Pod 信息生成为文件, 直接挂在到容器内部中去.

## 3.1 环境变量的方式挂载
```bash
$ vim downward-api-var.yaml
apiVersion: v1
kind: Pod
metadata:
    name: test-env-pod
spec:
    containers:
    - name: test-env-pod
      image: nginx:1.8
      command: ["/bin/sh", "-c", "env"]
      env:
      - name: POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: POD_NAMESPACE
        valueFrom:
          fieldRef:
            fieldPath: metadata.namespace
      - name: POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
```
在上面这个例子中, 将 podIp、namespace、metadata.name 挂载到环境变量中去了
```bash
$ kubectl apply -f downward-api-var.yaml
```
然后可以进入容器查看相应的环境变量

## 3.2 Volume的方式挂载
```bash
$ vim downward-api-vol.yaml
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
      image: k8s.gcr.io/busybox
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
在这个例子中, 声明了Volume 的名称是 podinfo, 类型是 Downward API, 挂载的是当前pod 的 labels信息到容器的 /etc/podinfo/labels 目录下, 当容器启动后, 每隔 5s 就会打印 /etc/podinfo/labels 下的文件, 所以可以通过 `kubectl logs ` 命令查看挂载的信息
```bash
$ kubectl logs test-downwardapi-volume


cluster="test-cluster1"
rack="rack-22"
zone="us-est-coast"
```

到目前为止, downwardAPI 支持的字段如下:
```bash

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

需要注意的是, Downward API 能够获取到的信息一定是 Pod 里的容器进程启动前就能够确定下来的信息, 如果要获取 Pod 容器运行后才出现的信息, 比如容器进程的 PID, 那就肯定不能使用 Downward API 了.


# 四、注意
Secret、ConfigMap、Downward API 这三种特殊的 Volume, 可以通过 Volume 的形式挂载, 也可以通过 环境变量 的形式挂载, 但是通过 环境变量 的形式挂载的话, 不具备自动更新的能力, 一般情况下, 建议使用 Volume 文件的方式获取这些信息。

# 五、Service Account
Service Account 对象的作用, 就是 kubernetes 系统内置的一种 "服务账户", 它是 Kubernetes 进行权限分配的对象, 比如: Service Account A 只允许对 Kubernetes API 进行 GET 操作, 而 Service Account B 则可以有 Kubernetes API 的所有操作权限.

像这样的 Service Account 的授权信息和文件, 实际上保存在它所绑定的一个特殊的 Secret 对象里, 这个特殊的 Secret 对象, 叫做 **ServiceAccountToken**. 任何运行在Kubernetes 集群上的应用, 都必须使用这个 ServiceAccountToken 里保存的授权信息, 也就是 Token, 才可以合法的访问 API Server.

所以说, Kubernetes 里的特殊Volume其实只有三种, 因为第四种 ServiceAccountToken 知识一种特殊的 Secret。

另外, 为了方便使用, Kubernetes 已经提供了一个默认的 "服务账户"(default Service Account), 并且, 任何一个运行在 Kubernetes 里的 Pod, 都可以直接使用这个默认的 Service Account, 无需显示的声明挂载

每一个已经运行的 Pod, 都自动声明了一个类型是 Secret、名为 default-token-xxxx 的 Volume, 然后自动挂载在每个容器的一个固定目录上, 例如:
```bash
$ kubectl describe pod test-downwardapi-volume
...
Volumes:
  podinfo:
    Type:         Projected (a volume that contains injected data from multiple sources)
    DownwardAPI:  true
  default-token-lmxpl:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-lmxpl
    Optional:    false
...
```