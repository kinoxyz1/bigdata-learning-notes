



--- 
# 一、Service 存在的意义
k8s 之所以需要 Service, 有两个方面的原因: 
 - Pod 的 IP 是不固定的
 - 一组 Pod 实例之间总会有负载均衡的需求.


# 二、常用 Service 类型
- ClusterIP(默认): 集群内部使用
- NodePort: 对外访问应用使用
- LoadBalancer: 对外访问应用使用, 公有云


# 三、例子
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
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>
    
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