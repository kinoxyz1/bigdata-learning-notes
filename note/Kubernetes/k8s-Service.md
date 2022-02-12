








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





# 二、Service

## 2.1 Service 基础概念

- Service 可以让一组 Pod 可以被别人进行服务发现（Service 选择 一组 Pod）。别人只需要访问这个 Service 就可以访问到被选中的 Pod。
- Service 还会基于 Pod 的探针机制(ReadinessProbe: 就绪探针)完成 Pod 的自动剔除和上线工作。
- Service 即使是无头服务，不能通过 Ip 访问，但是还是可以用 Service 名当域名访问。
- Service 的名字还能当成域名被 Pod 解析。

## 2.2 常用 Service 类型

- `ExternalName`：通过返回 `CNAME` 和对应值，可以将服务映射到 `externalName` 字段的内容；(例如: aaa.bar.example.com.cn)。无需创建任何类型的代理；
- `ClusterIP(默认)`: 通过集群的内部Ip 暴露服务，选择该值时，服务只能在集群内部访问；
- `NodePort`: 通过每个节点上的IP 和 静态端口(`NodePort`)暴露服务，`NodePort` 服务会路由到自动创建的 `ClusterIP` 服务。通过请求 `节点IP:节点端口`可以从集群外部访问一个 `NodePort` 服务；
  - NodePort 端口由 kube-proxy 在机器上开
  - 机器ip+暴露的 NodePort 流量先来到 kube-proxy
- `LoadBalancer`: 使用云提供商的负载均衡器向外部暴露服务，外部负载均衡器可以将流量路由到自动创建的 `NodePort` 服务和 `ClusterIP` 服务上；

## 2.3 简单的 Service 案例

```yaml
$ vi service01.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
  namespace: svc
spec:
  selector:
    matchLabels:
      app: my-deploy
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: my-deploy
    spec:
      containers:
      - name: my-deploy
        image: nginx
        ports:
        - containerPort:  80
          name: my-nginx
      - name: my-redis
        image: redis
        ports:
        - containerPort: 6379
          name: my-redis
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  namespace: svc
spec:
  selector:
    app: my-deploy
  ## type: ClusterIP   # type 很重要, 不写默认是 ClusterIP
  ## type 有四种类型
  ##     ClusterIP & ExternalName & NodePort & LoadBalancer
  type: ClusterIP
  ports:
  # 外部访问 http://serviceIp:80 时, 会访问到这个 service selector 选中的 pod 的 80 端口 
  - name: my-nginx
    protocol: TCP
    port: 80
    targetPort: 80
  # 外部访问 http://serviceIp:81 时, 会访问到这个 service selector 选中的 pod 的 6379 端口 
  - name: my-redis
    protocol: TCP
    port: 81
    targetPort: 6379
  
$ kubectl get pod,svc -n svc
NAME                             READY   STATUS    RESTARTS   AGE     IP             NODE        NOMINATED NODE   READINESS GATES
pod/my-deploy-56d756ddc4-6j972   2/2     Running   0          9m28s   10.233.81.52   k8s-node1   <none>           <none>

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE     SELECTOR
service/my-svc   ClusterIP   10.233.16.140   <none>        80/TCP,81/TCP   9m28s   app=my-deploy

$ kubectl describe service/my-svc -n svc
Name:              my-svc
Namespace:         svc
Labels:            <none>
Annotations:       <none>
Selector:          app=my-deploy
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.233.16.140
IPs:               10.233.16.140
Port:              my-nginx  80/TCP
TargetPort:        80/TCP
Endpoints:         10.233.81.52:80
Port:              my-redis  81/TCP
TargetPort:        6379/TCP
Endpoints:         10.233.81.52:6379
Session Affinity:  None
Events:            <none>
```

- Service 创建后，会生成一组 EndPoint

  ```yaml
  $ kubectl get ep -n svc
  NAME     ENDPOINTS                           AGE
  my-svc   10.233.81.52:6379,10.233.81.52:80   10m
  
  $ kubectl describe ep/my-svc -n svc
  Name:         my-svc
  Namespace:    svc
  Labels:       <none>
  Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2022-01-25T17:16:43+08:00
  Subsets:
    Addresses:          10.233.81.52
    NotReadyAddresses:  <none>
    Ports:
      Name      Port  Protocol
      ----      ----  --------
      my-redis  6379  TCP
      my-nginx  80    TCP
  
  Events:  <none>
  ```

- type 有四种类型，每种对应不同的服务发现机制
- Service 可以利用 Pod 的就绪探针机制，只负载就绪了的 Pod，没有就绪的Pod就删除

## 2.4 创建无 Selector 的 Service(EndPoint)

在创建 Service 的时候，可以不指定 Selector，然后手动创建 EndPoint指定一组 Pod 地址。

此场景用于我们负载均衡其他中间件的场景。

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

## 2.5 ClusterIP

```yaml
# 方式一: 不手动声明 IP 地址, 由 k8s 自动生成
...
type: ClusterIP
...

# 方式二: 手动指定 Cluster 的 IP 地址, 地址必须在合法范围内
...
type: ClusterIP
clusterIP: 10.239.199.88
...

# 方式三: Service 不生成 IP, 通过域名访问该 Service, 或者 Ingress
...
type: ClusterIP
clusterIP: None
...
```

示例: 

```yaml
# 方式一的例子
apiVersion: v1
kind: Service
metadata:
  name: my-service-clusterip-test1
  namespace: svc
spec:
  selector:
    app: my-deploy
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP

$ kubectl get svc -n svc
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE    SELECTOR
service/my-service-clusterip-test    ClusterIP   10.233.2.214    <none>        80/TCP          3m54s   app=my-deploy


---
# 方式二的例子
apiVersion: v1
kind: Service
metadata:
  name: my-service-clusterip-test2
  namespace: svc
spec:
  selector:
    app: my-deploy
  type: ClusterIP
  clusterIP: 10.233.199.140   ## 手动指定 Cluster 的 IP 地址, 地址必须在合法范围内
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
    
$ kubectl get svc -n svc
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE    SELECTOR
service/my-service-clusterip-test2    ClusterIP   10.233.16.88    <none>        80/TCP          3m54s   app=my-deploy

---
# 方式三的例子
apiVersion: v1
kind: Service
metadata:
  name: my-service-clusterip-test3
  namespace: svc
spec:
  selector:
    app: my-deploy
  type: ClusterIP
  clusterIP: None
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
    
$ kubectl get svc -n svc
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE    SELECTOR
service/my-service-clusterip-test3   ClusterIP   None            <none>        80/TCP          6s      app=my-deploy
```

## 2.6 NodePort

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

## 2.7 ExternalName 和 LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-externalname-test
  namespace: day13
spec:
  type: ExternalName
  externalName: www.json.cn  # 只能写 域名    注意跨域问题
  # 如果是无 selector, 得自己写 EP

# ---
# apiVersion: v1
# kind: Service
# metadata:
#   name: service-externalname-test
#   namespace: default
# spec:
#   selector: 
#      app: nginx
#   type: LoadBalancer  ## 负载均衡，开放给云平台实现，阿里云、百度云
#   ports:
#      port: 80
#      target: 888
## k8s自动请求云平台，分配一个负载均衡器
```

# 三、Ingress
## 3.1 Ingress 安装部署
```yaml
$ vim install-ingress.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx

---
# Source: ingress-nginx/templates/controller-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx
  namespace: ingress-nginx
automountServiceAccountToken: true
---
# Source: ingress-nginx/templates/controller-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
data:
---
# Source: ingress-nginx/templates/clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
  name: ingress-nginx
rules:
  - apiGroups:
      - ''
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ''
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingressclasses
    verbs:
      - get
      - list
      - watch
---
# Source: ingress-nginx/templates/clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
  name: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx
subjects:
  - kind: ServiceAccount
    name: ingress-nginx
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/controller-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx
  namespace: ingress-nginx
rules:
  - apiGroups:
      - ''
    resources:
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ''
    resources:
      - configmaps
      - pods
      - secrets
      - endpoints
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - networking.k8s.io   # k8s 1.14+
    resources:
      - ingressclasses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - configmaps
    resourceNames:
      - ingress-controller-leader-nginx
    verbs:
      - get
      - update
  - apiGroups:
      - ''
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ''
    resources:
      - events
    verbs:
      - create
      - patch
---
# Source: ingress-nginx/templates/controller-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx
subjects:
  - kind: ServiceAccount
    name: ingress-nginx
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/controller-service-webhook.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller-admission
  namespace: ingress-nginx
spec:
  type: ClusterIP
  ports:
    - name: https-webhook
      port: 443
      targetPort: webhook
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
---
# Source: ingress-nginx/templates/controller-service.yaml：不要
apiVersion: v1
kind: Service
metadata:
  annotations:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: ClusterIP  ## 改为clusterIP
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
---
# Source: ingress-nginx/templates/controller-deployment.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/component: controller
  revisionHistoryLimit: 10
  minReadySeconds: 0
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    spec:
      dnsPolicy: ClusterFirstWithHostNet   ## dns对应调整为主机网络
      hostNetwork: true  ## 直接让nginx占用本机80端口和443端口，所以使用主机网络
      containers:
        - name: controller
          image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0
          imagePullPolicy: IfNotPresent
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown
          args:
            - /nginx-ingress-controller
            - --election-id=ingress-controller-leader
            - --ingress-class=nginx
            - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
            - --validating-webhook=:8443
            - --validating-webhook-certificate=/usr/local/certificates/cert
            - --validating-webhook-key=/usr/local/certificates/key
          securityContext:
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            runAsUser: 101
            allowPrivilegeEscalation: true
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: LD_PRELOAD
              value: /usr/local/lib/libmimalloc.so
          livenessProbe:
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 1
            successThreshold: 1
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 1
            successThreshold: 1
            failureThreshold: 3
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - name: https
              containerPort: 443
              protocol: TCP
            - name: webhook
              containerPort: 8443
              protocol: TCP
          volumeMounts:
            - name: webhook-cert
              mountPath: /usr/local/certificates/
              readOnly: true
          resources:
            requests:
              cpu: 100m
              memory: 90Mi
      nodeSelector:  ## 节点选择器
        node-role: ingress #以后只需要给某个node打上这个标签就可以部署ingress-nginx到这个节点上了
        #kubernetes.io/os: linux  ## 修改节点选择
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
        - name: webhook-cert
          secret:
            secretName: ingress-nginx-admission
---
# Source: ingress-nginx/templates/admission-webhooks/validating-webhook.yaml
# before changing this value, check the required kubernetes version
# https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#prerequisites
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  name: ingress-nginx-admission
webhooks:
  - name: validate.nginx.ingress.kubernetes.io
    matchPolicy: Equivalent
    rules:
      - apiGroups:
          - networking.k8s.io
        apiVersions:
          - v1beta1
        operations:
          - CREATE
          - UPDATE
        resources:
          - ingresses
    failurePolicy: Fail
    sideEffects: None
    admissionReviewVersions:
      - v1
      - v1beta1
    clientConfig:
      service:
        namespace: ingress-nginx
        name: ingress-nginx-controller-admission
        path: /networking/v1beta1/ingresses
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
rules:
  - apiGroups:
      - admissionregistration.k8s.io
    resources:
      - validatingwebhookconfigurations
    verbs:
      - get
      - update
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx-admission
subjects:
  - kind: ServiceAccount
    name: ingress-nginx-admission
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
rules:
  - apiGroups:
      - ''
    resources:
      - secrets
    verbs:
      - get
      - create
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ingress-nginx-admission
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx-admission
subjects:
  - kind: ServiceAccount
    name: ingress-nginx-admission
    namespace: ingress-nginx
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/job-createSecret.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ingress-nginx-admission-create
  annotations:
    helm.sh/hook: pre-install,pre-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
spec:
  template:
    metadata:
      name: ingress-nginx-admission-create
      labels:
        helm.sh/chart: ingress-nginx-3.30.0
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 0.46.0
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      containers:
        - name: create
          image: docker.io/jettech/kube-webhook-certgen:v1.5.1
          imagePullPolicy: IfNotPresent
          args:
            - create
            - --host=ingress-nginx-controller-admission,ingress-nginx-controller-admission.$(POD_NAMESPACE).svc
            - --namespace=$(POD_NAMESPACE)
            - --secret-name=ingress-nginx-admission
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      restartPolicy: OnFailure
      serviceAccountName: ingress-nginx-admission
      securityContext:
        runAsNonRoot: true
        runAsUser: 2000
---
# Source: ingress-nginx/templates/admission-webhooks/job-patch/job-patchWebhook.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: ingress-nginx-admission-patch
  annotations:
    helm.sh/hook: post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    helm.sh/chart: ingress-nginx-3.30.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.46.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: admission-webhook
  namespace: ingress-nginx
spec:
  template:
    metadata:
      name: ingress-nginx-admission-patch
      labels:
        helm.sh/chart: ingress-nginx-3.30.0
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/version: 0.46.0
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: admission-webhook
    spec:
      containers:
        - name: patch
          image: docker.io/jettech/kube-webhook-certgen:v1.5.1
          imagePullPolicy: IfNotPresent
          args:
            - patch
            - --webhook-name=ingress-nginx-admission
            - --namespace=$(POD_NAMESPACE)
            - --patch-mutating=false
            - --secret-name=ingress-nginx-admission
            - --patch-failure-policy=Fail
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      restartPolicy: OnFailure
      serviceAccountName: ingress-nginx-admission
      securityContext:
        runAsNonRoot: true
        runAsUser: 2000
```














