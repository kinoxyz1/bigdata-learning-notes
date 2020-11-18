




---
# 一、Secret 作用
加密数据存在 etcd 里面, 让 Pod 容器以挂载 Volume 方式进行访问

使用场景: 凭证


# 二、创建 secret 加密数据
```yaml
$ vim Secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm

$ kubectl apply -f Secret.yaml 
secret/mysecret created
$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-ch88s   kubernetes.io/service-account-token   3      9d
mysecret              Opaque                                2      6s
```

# 三、以变量的形式挂载到 pod 容器中
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
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password

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


# 四、以 volume 形式挂载到 pod 容器中
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

$ kubectl apply -f secret-vol.yaml 
pod/mypod created

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          10s

$ kubectl exec -it mypod bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

root@mypod:/# ls /etc/fo
fonts/ foo/   

root@mypod:/# ls /etc/fo
fonts/ foo/   

root@mypod:/# ls /etc/foo/
password  username

root@mypod:/# cat /etc/foo/password
1f2d1e2e67dfroot@mypod:/# cat /etc/foo/username 
adminroot@mypod:/# 
```