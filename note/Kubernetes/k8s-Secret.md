

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