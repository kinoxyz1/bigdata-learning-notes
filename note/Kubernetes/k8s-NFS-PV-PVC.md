

* [一、NFS](#%E4%B8%80nfs)
  * [1\.1 NFS 使用案例](#11-nfs-%E4%BD%BF%E7%94%A8%E6%A1%88%E4%BE%8B)
    * [步骤一](#%E6%AD%A5%E9%AA%A4%E4%B8%80)
    * [步骤二](#%E6%AD%A5%E9%AA%A4%E4%BA%8C)
    * [步骤三](#%E6%AD%A5%E9%AA%A4%E4%B8%89)
    * [步骤四](#%E6%AD%A5%E9%AA%A4%E5%9B%9B)
* [二、PV 和 PVC](#%E4%BA%8Cpv-%E5%92%8C-pvc)
* [二、实现流程](#%E4%BA%8C%E5%AE%9E%E7%8E%B0%E6%B5%81%E7%A8%8B)
* [三、案例](#%E4%B8%89%E6%A1%88%E4%BE%8B)

---

# 一、NFS
容器数据卷 emptydir 是本地存储的, pod 重启后, 数据将会不存在, 需要对数据持久化存储

NFS 可以进行网络存储, 当pod 重启后, 数据还存在

## 1.1 NFS 使用案例
步骤: 
- 步骤一: 找一台服务器做 NFS 服务端
- 步骤二: 在 k8s 集群 node 节点安装 nfs
- 步骤三: 在 nfs 服务器启动 nfs 服务
- 步骤四: 在 k8s 集群部署应用使用 nfs 持久化网络存储

### 步骤一
① 安装 nfs

这里新增一个虚拟机<k8s-nfs>, 在这个虚拟机上安装 nfs
```bash
$ yum install -y nfs-utils
```
② 设置挂载路
```bash
$ vim /etc/exports
/data/nfs *(rw,no_root_squash)
```

③ 挂载路径需要手动创建出来
```bash
$ mkdir /data/nfs
$ ll /data/
  总用量 0
  drwxr-xr-x. 2 root root 6 11月 23 17:16 nfs
```

### 步骤二
```bash
$ yum install nfs-utils -y
```


### 步骤三
所有节点上都要执行
```bash
$ systemctl start nfs
$ systemctl enable nfs
```


### 步骤四
```bash
$ vim nfs-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep1
spec:
  replicas: 1
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
        image: nginx
        volumeMounts:
        - name: wwwroot
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
      volumes:
        - name: wwwroot
          nfs:
            server: 192.168.44.134
            path: /data/nfs

$ kubectl apply -f nfs-nginx.yaml 
deployment.apps/nginx-dep1 created

$ kubectl get pod
NAME                          READY   STATUS              RESTARTS   AGE
nginx-dep1-5d49654cbd-n6rk6   0/1     ContainerCreating   0          14s

$ kubectl exec -it nginx-dep1-5d49654cbd-n6rk6 bash

root@nginx-dep1-5d49654cbd-n6rk6:/# ls /usr/share/nginx/html/
root@nginx-dep1-5d49654cbd-n6rk6:/# 
```
在 新增一个虚拟机<k8s-nfs> 的 /data/nfs 目录下新增一个文件
```bash
$ vim index.html
hello k8s
```
再次执行 ls /usr/share/nginx/html/ 命令
```bash
root@nginx-dep1-5d49654cbd-n6rk6:/# ls /usr/share/nginx/html/
index.html
```
可以看见 index.html 在容器中就出现了


# 二、PV 和 PVC
在上面介绍了 nfs 开启网络存储的过程, 对应的 yaml 文件中有一段需要显示的表明 nfs 服务器的IP 和 路径:
```yaml
...
volumes:
- name: wwwroot
  nfs:
    server: 192.168.44.134
    path: /data/nfs
```
如果这个yaml暴露给别人, 是不安全的, 而且使用服务器的IP有很多不方便的地方

使用 nfs 有如上的问题, 为了解决这个问题, 下面继续学习 PV 和 PVC


- PV: 持久化存储, 对存储资源进行抽象, 对外提供可以调用的地方(生产者)
- PVC: 用于调用, 不需要关心内部实现细节(消费者)


# 二、实现流程
![PV和PVC](../../img/k8s/PV和PVC/PV和PVC.png)



# 三、案例
```yaml
$ vim pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    name: pv001
spec:
  nfs:
    path: /data/nfs/v1
    server: 192.168.220.154
  accessModes: ["ReadWriteMany","ReadWriteOnce"]
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv002
  labels:
    name: pv002
spec:
  nfs:
    path: /data/nfs/v1
    server: 192.168.220.154
  accessModes: ["ReadWriteOnce"]
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv003
  labels:
    name: pv003
spec:
  nfs:
    path: /data/nfs/v3
    server: 192.168.220.154
  accessModes: ["ReadWriteMany","ReadWriteOnce"]
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv004
  labels:
    name: pv004
spec:
  nfs:
    path: /data/nfs/v4
    server: 192.168.220.154
  accessModes: ["ReadWriteMany","ReadWriteOnce"]
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv005
  labels:
    name: pv005
spec:
  nfs:
    path: /data/nfs/v5
    server: 192.168.220.154
  accessModes: ["ReadWriteMany","ReadWriteOnce"]
  capacity:
    storage: 1Gi


$ vim pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
  namespace: default
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-pvc
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
    - name: html
      persistentVolumeClaim:
        claimName: mypvc



$ kubectl  apply -f pv.yaml 
persistentvolume/pv001 created
persistentvolume/pv002 created
persistentvolume/pv003 created
persistentvolume/pv004 created
persistentvolume/pv005 created

$ kubectl get pv
NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv001   1Gi        RWO,RWX        Retain           Available                                   5s
pv002   1Gi        RWO            Retain           Available                                   5s
pv003   1Gi        RWO,RWX        Retain           Available                                   5s
pv004   1Gi        RWO,RWX        Retain           Available                                   5s
pv005   1Gi        RWO,RWX        Retain           Available         
                          5s
$ kubectl apply -f pvc.yaml 
persistentvolumeclaim/mypvc created
pod/pod-vol-pvc created

$ kubectl get pod
NAME          READY   STATUS    RESTARTS   AGE
pod-vol-pvc   1/1     Running   0          4s

$ kubectl get pv,pvc
NAME                     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pv001   1Gi        RWO,RWX        Retain           Available                                           33s
persistentvolume/pv002   1Gi        RWO            Retain           Available                                           33s
persistentvolume/pv003   1Gi        RWO,RWX        Retain           Bound       default/mypvc                           33s
persistentvolume/pv004   1Gi        RWO,RWX        Retain           Available                                           33s
persistentvolume/pv005   1Gi        RWO,RWX        Retain           Available                                           33s

NAME                          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/mypvc   Bound    pv003    1Gi        RWO,RWX                       15s
```
在 v3 中创建 index.html 文件
```html
$ vim /data/nfs/v3/index.html
pv, pvc
```
测试
```bash
$ kubectl get pods -o wide | grep pod-vol-pvc
pod-vol-pvc   1/1     Running   0          2m57s   10.244.2.7   k8s-node2   <none>           <none>

$ curl 10.244.2.7
pv, pvc
```