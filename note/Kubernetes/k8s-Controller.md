



---
# 一、什么是 controller
*

# 二、 Pod 和 Controller 关系
*

# 三、 Deployment 控制器应用场景
1. 部署无状态应用
2. 管理Pod和ReplicaSet
3. 部署, 滚动升级等功能
4. web 服务, 微服务

# 四、Yaml 文件字段说明
* [K8s YAML文件配置详解](note/Kubernetes/k8s-YAML文件配置详解.md)


# 五、Deployment 控制器部署应用
## 5.1 导出 Yaml
# 导出 Yaml 文件
```bash
$ kubectl create deployment web --image=nginx --dry-run -o yaml > web.yaml
W1117 17:46:18.012487  127855 helpers.go:553] --dry-run is deprecated and can be replaced with --dry-run=client.
```

## 5.2 使用 Yaml 部署应用
```bash
$ kubectl apply -f web.yaml
deployment.apps/web created

$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
web-5dcb957ccc-ccc64   1/1     Running   0          93s   10.244.2.13   k8s-node2   <none>           <none>
```

## 5.3 对外发布(暴露端口)
```bash
$ kubectl expose deployment web --port=80 --type=NodePort --target-port=80 --name=web1 -o yaml > web1.yaml
```

## 5.4 再次使用 Yaml 部署应用
```bash
$ kubectl apply -f web1.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
service/web1 configure

[root@k8s-master k8s-work]# kubectl get svc
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        8d
service/web1         NodePort    10.99.248.199   <none>        80:31180/TCP   4m27s
```


# 六、升级回滚
## 6.1 应用升级
```bash
$ kubectl set image deployment web nginx=nginx:1.5
deployment.apps/web image updated
```
## 6.2 查看升级版本
```bash
$ kubectl rollout history deployment web
deployment.apps/web 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```
## 6.3 回滚到上一个版本
```bash
# 回滚
$ kubectl rollout undo deployment web
# 查看回滚状态
$ kubectl rollout status deployment web
```

## 6.4 回滚到指定的版本
```bash
# 回滚
$ kubectl rollout undo deployment web --to-revision=2
# 查看回滚状态
$ kubectl rollout status deployment web
```




# 七、弹性伸缩
```bash
$ kubectl scale deployment web --replicas=5
```