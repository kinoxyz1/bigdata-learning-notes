



----

# 一、volume 类型
k8s 中有很多种类型的 volume: 
- awsElasticBlockStore
- azureDisk
- azureFile
- cephfs
- cinder
- configMap
- downwardAPI
- emptyDir
- fc (光纤通道)
- flocker （已弃用）
- gcePersistentDisk
- gitRepo (已弃用)
- glusterfs
- hostPath
- iscsi
- local
- nfs
- persistentVolumeClaim
- portworxVolume
- projected
- quobyte
- rbd
- scaleIO （已弃用）
- secret
- storageOS
- vsphereVolume

详细可以查看官方文档
[volume types](https://kubernetes.io/zh/docs/concepts/storage/volumes/#volume-types)


# 二、emptyDir
emptyDir 不会保存数据到磁盘上, pod 被移除掉的时候 emptyDir 也会被永久删掉
```yaml
$ vi emptydir.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx
  namespace: aaa
  labels:
    app:  nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app:  nginx
    spec:
      containers:
      - name:  nginx
        image:  nginx
        ports:
        - containerPort:  80
          name:  nginx
        volumeMounts:
        - name: my-empty-dir
          mountPath: /opt/
      volumes:
        - name: my-empty-dir
          emptyDir: {}
      restartPolicy: Always

$ kubectl apply -f emptydir.yaml
$ kubectl exec -it nginx-59f8b57fc8-vqntf -n aaa bash
> echo 111 > /opt/1.txt
> cat /opt/1.txt
111
```
删除容器
```yaml
$ kubectl delete pod/nginx-59f8b57fc8-vqntf -n aaa
# 等待容器重启, 再查看容器中的 /opt 目录下是否有 1.txt 文件
$ kubectl exec -it nginx-59f8b57fc8-7mz9b -n aaa ls /opt

```

# 三、hostPath
```yaml
$ vim hostpath.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: aaa
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeName: jz-desktop-02
      containers:
        - name: nginx
          image: nginx
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
          ports:
            - containerPort: 80
              name: nginx
          volumeMounts:
            - name: localtime
              mountPath: /opt
      volumes:
        - name: localtime
          hostPath:
            path: /home/kino/hostpath
      restartPolicy: Always

$ kubectl apply -f hostpath.yaml

$ kubectl exec -it nginx-86bcd5dd95-4bx5x -n aaa ls /opt
a1.yaml
```
删掉pod重新创建后再看 /opt 目录下是否有文件
```yaml
$ kubectl delete pod/nginx-86bcd5dd95-4bx5x -n aaa
$ kubectl exec -it nginx-86bcd5dd95-hqxvt -n aaa ls /opt
a1.yaml
```

> 需要注意的是: </br>
> - hostPath 仅仅会将 pod 所在机器的指定 path 和 容器mountPath 进行绑定, 如果每次 pod 运行在不同服务器上, 那容器内部将会是空目录


# 四、configMap
ConfigMap 保存的是键值对数据(非秘密性), Pod 可以将 ConfigMap 的数据使用在 环境变量、命令行参数、volume中。

## 4.1 当 volume 用
```yaml
$ vim configmap.yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configmap
  namespace: aaa
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Hello World</em></p>
    </body>
    </html>
  50x.html: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Hello World 500</em></p>
    </body>
    </html>

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx
  namespace: aaa
  labels:
    app:  nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app:  nginx
    spec:
      containers:
      - name:  nginx
        image:  nginx
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort:  80
          name:  nginx
        volumeMounts:
        - name: vol-conf
          mountPath: /usr/share/nginx/html/
      volumes:
        - name: vol-conf
          configMap:    # volume 类型是 configMap
            name: nginx-configmap  # 写 configMap 的名字
            items: 
              - key: index.html
              path: index.html
      restartPolicy: Always

$ kubectl apply -f vim configmap.yaml

# 查看 nginx 首页
$ kubectl exec -it nginx-5c595bdbf5-p5n8l -n aaa curl localhost:80
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, the nginx web server is successfully installed and
  working. Further configuration is required.</p>

  <p>For online documentation and support please refer to
  <a href="http://nginx.org/">nginx.org</a>.<br/>
  Commercial support is available at
  <a href="http://nginx.com/">nginx.com</a>.</p>

  # 是在 configmap 中自己配置的
  <p><em>Hello World</em></p>
  </body>
  </html>

```



