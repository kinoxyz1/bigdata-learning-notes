

* [一、无状态部署和有状态部署](#%E4%B8%80%E6%97%A0%E7%8A%B6%E6%80%81%E9%83%A8%E7%BD%B2%E5%92%8C%E6%9C%89%E7%8A%B6%E6%80%81%E9%83%A8%E7%BD%B2)
* [二、部署有状态应用](#%E4%BA%8C%E9%83%A8%E7%BD%B2%E6%9C%89%E7%8A%B6%E6%80%81%E5%BA%94%E7%94%A8)

---
# 一、无状态部署和有状态部署
- 无状态部署(Deployment):
  - 认为 Pod 都是一样的
  - 没有顺序要求
  - 不用考虑在哪个 Node 运行
  - 随意进行伸缩和扩展
- 有状态部署(StatefulSet):
  - 上面的因素都要考虑到
  - 让每个 Pod 独立, 保持 Pod 启动顺序和唯一性
  - 唯一的网络标识符, 持久存储
  - 有序, 比如 MySQL 主从
  
  
# 二、部署有状态应用
```bash
$ vim statefulSet.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
  namespace: default
spec:
  serviceName: nginx
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
        image: nginx:latest
        ports:
        - containerPort: 80
```
查看 pod, 可以看到有 3 个pod, 并且每个 pod 都是逐个创建的, 每个 pod 都是唯一的
```bash
$ kubectl get pod
NAME                  READY   STATUS    RESTARTS   AGE
nginx-statefulset-0   1/1     Running   0          3m20s
nginx-statefulset-1   1/1     Running   0          3m9s
nginx-statefulset-2   1/1     Running   0          2m57s
```
查看 service
```bash
[root@k8s-master k8s-work]# kubectl get service
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   9d
nginx        ClusterIP   None         <none>        80/TCP    4m18s
```

当扩展 StatefulSet 的时候, 会看见逐步增加了两个 pod
```bash
$ kubectl scale StatefulSet nginx-statefulset --replicas=5
```

# 三、statefulset 部署 mysql