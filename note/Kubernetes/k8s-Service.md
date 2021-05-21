








---
# 一、Kubernetes 网络

Kubernetes 网络解决了四类问题:

1. 一个 Pod 中的容器之间通过 **本地回路(loopback) 通信**。
2. 集群网络在不同 Pod 之间提供通信, Pod 和 Pod 之间互通
3. Service 资源允许对外暴露 Pod 中运行的应用程序，以支持来自于集群外部的访问(Service 和 Pod互通)。
4. 可以使用 Services 来发布仅供集群内部使用的服务。

## 1.1 Kubernetes 网络架构图

![Kubernetes网络架构图](../../img/k8s/service/Kubernetes架构图.png)

## 1.2 访问流程

![访问流程](../../img/k8s/service/访问流程.png)

## 1.3 网络连通原理

## 1.3.1 容器 和 容器 互通

![容器与容器互通](../../img/k8s/service/容器与容器互通.png)

### 1.3.2 Pod 和 Pod 互通

1. 同节点

   ![pod&pod同节点](../../img/k8s/service/pod&pod同节点.png)

2. 跨节点

   ![pod&pod跨节点](../../img/k8s/service/pod&pod跨节点.png)

### 1.3.3 Pod 和 Service 通信

![pod和service通信](../../img/k8s/service/pod和service通信.png)

### 1.3.4 service 和 Pod 通信

![service和pod通信](../../img/k8s/service/service和pod通信.png)

### 1.3.5 Pod 和 Internet

![pod和internet](../../img/k8s/service/pod和internet.png)

### 1.3.6 Internet 和 Pod（LoadBalancer--四层网络协议）

![Internet和pod](../../img/k8s/service/Internet和pod.png)

### 1.3.7 Internet 和 Pod（LoadBalancer--七层网络协议）

![Internet和pod(七层网络协议)](../../img/k8s/service/Internet和pod(七层网络协议).png)



# 二、Service 基础概念

- Service 可以让一组 Pod 可以被别人进行服务发现（Service 选择 一组 Pod）。别人只需要访问这个 Service 就可以访问到被选中的 Pod。
- Service 还会基于 Pod 的探针机制(ReadinessProbe: 就绪探针)完成 Pod 的自动剔除和上线工作。
- Service 即使是无头服务，不能通过 Ip 访问，但是还是可以用 Service 名当域名访问。
- Service 的名字还能当成域名被 Pod 解析。




# 二、常用 Service 类型
- ExternalName：通过返回 `CNAME` 和对应值，可以将服务映射到 `externalName` 字段的内容；(例如: aaa.bar.example.com.cn)。无需创建任何类型的代理；
- ClusterIP(默认): 通过集群的内部Ip 暴露服务，选择该值时，服务只能在集群内部访问；
- NodePort: 通过每个节点上的IP 和 静态端口(`NodePort`)暴露服务，`NodePort` 服务会路由到自动创建的 `ClusterIP` 服务。通过请求 `节点IP:节点端口`可以从集群外部访问一个 `NodePort` 服务；
  - NodePort 端口由 kube-proxy 在机器上开
  - 机器ip+暴露的 NodePort 流量先来到 kube-proxy
- LoadBalancer: 使用云提供商的负载均衡器向外部暴露服务，外部负载均衡器可以将流量路由到自动创建的 `NodePort` 服务和 `ClusterIP` 服务上；

# 三、简单的 Service 案例

```yaml
$ vi service01.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: day13
spec:
  selector:
    app: nginx-test
  # type: ClusterIP   # type 很重要, 不写默认是 ClusterIP
  ## type 有四种类型
  ##     ClusterIP & ExternalName & NodePort & LoadBalancer
  ports:
  - name: my-service
    port: 80           # service 的端口
    targetPort: 80     # Pod 的端口
    protocol: TCP
  #- name: my-service2
  #  port: 3306           # service 的端口
  #  targetPort: 3306     # Pod 的端口
  #  protocol: TCP  

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: day13
  labels:
    app: nginx-test
spec:
  containers:
  - name: nginx-pod
    image: nginx
    ports:
    - name: nginx
      containerPort: 80
  restartPolicy: Always
  
$ kubectl get pod,svc -n day13
NAME            READY   STATUS    RESTARTS   AGE
pod/nginx-pod   1/1     Running   0          98s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/my-service   ClusterIP   10.103.122.157   <none>        80/TCP    98s

$ curl 10.103.122.157
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
...
```

- Service 创建后，会生成一组 EndPoint

  ```yaml
  $ kubectl get ep -n day13
  NAME         ENDPOINTS        AGE
  my-service   10.244.2.94:80   3m35s
  ```

- type 有四种类型，每种对应不同的服务发现机制
- Service 可以利用 Pod 的就绪探针机制，只负载就绪了的 Pod，没有就绪的Pod就删除



# 四、创建无 Selector 的 Service(EndPoint)

在创建 Service 的时候，可以不指定 Selector，然后手动创建 EndPoint指定一组 Pod 地址；



此场景用于我们负载均衡其他中间件的场景

```yaml
# 先创建一个 3个副本的 deployment 
$ vim deployment01.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: day13
spec:
  selector:
    matchLabels:
      app: nginx-deployment
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx-deployment
        image: nginx
        ports:
        - containerPort: 80
      restartPolicy: Always
      
## 通过 kubectl get pod -n day13 查看创建的 三个 Pod 的IP
## 通过 kubectl exec -it 修改 每个 pod 的nginx 页面:
## echo "nginx11111" > /usr/share/nginx/html/index.html
## echo "nginx22222" > /usr/share/nginx/html/index.html
## echo "nginx33333" > /usr/share/nginx/html/index.html
## echo "nginx44444" > /usr/share/nginx/html/index.html
```

再创建 service 和 EndPoint

```yaml
$ vim service-no-seletor.yaml 
# 创建 selector 的 service
apiVersion: v1
kind: Service
metadata:
  name: my-service-no-selector
  namespace: day13
spec:
  # 在这里不写 selector 选中 Pod, 而是通过下面的 EndPoint 手动指定要访问的 Pod IP
  ports:
  - name: http      # 可以写，也可以不写
    port: 80        # service 的端口
    targetPort: 80  # 目标 Pod 的端口
    protocol: TCP

---
# 创建一个 EndPoint
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service-no-selector   # EndPoint 和 service 绑定规则是: 和 svc 同名且同名称空间, port 同名或通端口
  namespace: day13
subsets:
- addresses:
  - ip: 10.244.2.99
  - ip: 10.244.1.139
  - ip: 10.244.2.97
  ports:   
  - name: http    # service 有 name 的话, 这里一定要有, 并且要和 service 的 ports.name 保持一致
    port: 80
    protocol: TCP
```

# 五、ClusterIP

```yaml
...
type: ClusterIP
clusterIP:  ## 手动指定 Cluster 的 IP 地址, 地址必须在合法范围内
...

# 例子
apiVersion: v1
kind: Service
metadata:
  name: my-service-clusterip-test
  namespace: day13
spec:
  selector:
    app: nginx-deployment
  type: ClusterIP
  clusterIP: "10.102.1.87"   ## 手动指定 Cluster 的 IP 地址, 地址必须在合法范围内
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP

# 这里的 CLUSTER-IP 就是手动指定的
$ kubectl get svc -n day13
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
my-service-clusterip-test   ClusterIP   10.102.1.87      <none>        80/TCP    6s
```



# 六、NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-nodeport-test01
  namespace: day13
spec:
  selector:
    app: nginx-deployment
  type: NodePort      ## 在每一台机器都为这个 service 分配一个指定的端口
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 31180   ## 如果不指定，系统会在 30000-32765 之间随机分配
```

# 七、ExternalName 和 LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-externalname-test
  namespace: day13
spec:
  type: ExternalName
  externalName: www.json.cn  # 只能写 域名    注意跨域问题
  # 无 selector, 自己写 EP

# ---
# apiVersion: v1
# kind: Service
# metadata:
#   name: service-externalname-test
#   namespace: default
# spec:
    # selector: 
    #    app: nginx
#   type: LoadBalancer  ## 负载均衡，开放给云平台实现，阿里云、百度云
    # ports:
    #    port: 80
    #    target: 888
  ## k8s自动请求云平台，分配一个负载均衡器
```



# 八、externalIP(IP 白名单)

```yaml

```











## Service 为 ClusterIP
1. 导出 Nginx 的 Yaml 文件
    ```bash
    $ kubectl create deployment web --image=nginx --dry-run -o yaml > web.yaml
    ```
2. 根据导出的 Yaml 文件部署 Nginx
    ```bash
    $ kubectl apply -f web.yaml
      
    $ kubectl get pod,svc
    NAME                       READY   STATUS              RESTARTS   AGE
    pod/web-5dcb957ccc-c4fjh   0/1     ContainerCreating   0          14s
    
    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8d
    ```
3. 导出 Service 的 Yaml 文件
    ```bash
    $ kubectl expose deployment web --port=80 --target-port=80 --dry-run -o yaml > service.yaml
    ```
4. 根据导出的 Service 的 Yaml 文件部署
    ```bash
    $ kubectl apply -f service.yaml
    NAME                       READY   STATUS    RESTARTS   AGE
    pod/web-5dcb957ccc-c4fjh   1/1     Running   0          2m42s
    
    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   8d
    service/web          ClusterIP   10.101.97.233   <none>        80/TCP    12s
    ```
5. 访问

      可以看到 `service/web` 此时是对集群内使用的, 输入如下命令: 
    ```html
    $ curl 10.101.97.233`
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>
    
    <p>For online documentation and support please refer to
    <a href="http:/nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http:/nginx.com/">nginx.com</a>.</p>
    
    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

## Service 为 NodePort
1. 修改 service.yaml 文件
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: web
      name: web
    spec:
      type: NodePort
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        app: web
    status:
      loadBalancer: {}
    ```
2. 重新部署
    ```bash
    $ kubectl apply -f service.yaml
      
    $ kubectl get pod,svc
    NAME                       READY   STATUS    RESTARTS   AGE
    pod/web-5dcb957ccc-c4fjh   1/1     Running   0          6m26s
    
    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        8d
    service/web          NodePort    10.101.97.233   <none>        80:32283/TCP   3m56s
    ```

3. 访问

    此时, 可以看见 `service/web` 是对外暴露了 32283 端口的, 在浏览器中输入: <Master IP>:<PORT>, 可以看见 Nginx 页面.
    
## Service 为 LoadBalancer
1. 修改 service.yaml 文件
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: web
      name: web
    spec:
      type: LoadBalancer
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        app: web
    status:
      loadBalancer: {}
    ```
   
2. 重新部署
    ```bash
    $ kubectl apply -f service.yaml
      
    $ kubectl get pod,svc
    NAME                       READY   STATUS    RESTARTS   AGE
    pod/web-5dcb957ccc-c4fjh   1/1     Running   0          6m26s
    
    NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        8d
    service/web          LoadBalancer   10.101.97.233   <pending>     80:32283/TCP   7m4s
    ```