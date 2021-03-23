



---
# 一、PodPreset 说明
PodPreset 可以实现当开发人员提交一个简单的 Pod Yaml, 就可以自动的给对应的 Pod 对象加上其他必要的信息, 比如: labels, annotations, volumes 等等, 这些相关的信息, 可以是事先定义好的.

# 二、案例
创建 PodPreset 对象
```bash
$ vim podpreset.yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: allow-database
spec:
  selector:
    # 匹配 Pod 的 Label 属性是 role: frontend 的 Pod
    matchLabels:
      role: frontend
  # 匹配到之后为其添加 环境变量
  env:
    - name: DB_PORT
      value: "6379"
  # 匹配到之后为其挂载 Volume
  volumeMounts:
    - mountPath: /cach
      name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
```
创建 Pod
```bash
$ vim podpreset-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
spec:
  containers:
    - name: website
      image: nginx
      ports:
        - containerPort: 80
```
创建 Pod
```bash
$ kubectl apply -f podpreset.yaml
$ kubectl apply -f podpreset-pod.yaml
```
查看 PodPreset
```bash
[root@k8s-master podpreset]# kubectl get podpreset
NAME             CREATED AT
allow-database   2021-03-23T12:30:26Z
```
查看启动的 Pod 的 Yaml 信息
```bash
$ kubectl get pod website -o yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
  annotations:
    podpreset.admission.kubernetes.io/podpreset-allow-database: "resource version"
spec:
  containers:
    - name: website
      image: nginx
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
      ports:
        - containerPort: 80
      env:
        - name: DB_PORT
          value: "6379"
  volumes:
    - name: cache-volume
      emptyDir: {}
```
可以看到, 这个 Pod 里面多了新添加的 labels、env、volumes 和 volumeMount 的声明, 这个配置跟 PodPreset 里面的内容一样, 此外, 这个 Pod 还自动加上了一个 annotation, 表示这个 Pod 对象被 PodPreset 改动过.

需要注意的是, PodPreset 里定义的内容, 只会在  Pod API 对象被创建之前追加在这个对象本身上, 而不会影响任何 Pod 的控制器的定义.

比如现在提交一个 nginx-deployment, 那么这个 Deployment 对象本身是永远不会被 PodPreset 改变的, 被修改的只是这个 Deployment 创建出来的所有的 Pod.

如果一个 Pod 对象有多个 PodPreset 同时作用, Kubernetes 会合并这两个 PodPreset 要做的修改, 如果两个 PodPreset 冲突的话, 冲突字段就不会被修改.