



---
# 一、ConfigMap 作用
和 Secret 类似, ConfigMap 的作用是将不加密数据存储到 etcd 中, 让 Pod 以变量的形式或者 Volume 挂载到容器中.

场景: 配置文件


# 二、创建 ConfigMap
```bash
# 1. 准备 redis.properties 配置文件, 如下:
redis.host=127.0.0.1
redis.port=6379
redis.password=123456

# 2. 创建
$ kubectl create configmap redis-config --from-file=redis.properties 
configmap/redis-config created

$ kubectl get cm
NAME           DATA   AGE
redis-config   1      5s

$ kubectl describe cm redis-config
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

# 三、以 Volume 的形式挂载到 Pod 中去
```bash
$ vim cm.yaml
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

# 四、以 变量 的形式挂载到 Pod 中去
```bash
$ vim myconfig.yaml
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