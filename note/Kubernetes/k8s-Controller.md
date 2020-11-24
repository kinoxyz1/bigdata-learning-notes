

* [一、什么是 controller](#%E4%B8%80%E4%BB%80%E4%B9%88%E6%98%AF-controller)
* [二、 Pod 和 Controller 关系](#%E4%BA%8C-pod-%E5%92%8C-controller-%E5%85%B3%E7%B3%BB)
* [三、 Deployment 控制器应用场景](#%E4%B8%89-deployment-%E6%8E%A7%E5%88%B6%E5%99%A8%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)
* [四、Yaml 文件字段说明](#%E5%9B%9Byaml-%E6%96%87%E4%BB%B6%E5%AD%97%E6%AE%B5%E8%AF%B4%E6%98%8E)
* [五、Deployment 控制器部署应用](#%E4%BA%94deployment-%E6%8E%A7%E5%88%B6%E5%99%A8%E9%83%A8%E7%BD%B2%E5%BA%94%E7%94%A8)
  * [5\.1 导出 Yaml](#51-%E5%AF%BC%E5%87%BA-yaml)
* [导出 Yaml 文件](#%E5%AF%BC%E5%87%BA-yaml-%E6%96%87%E4%BB%B6)
  * [5\.2 使用 Yaml 部署应用](#52-%E4%BD%BF%E7%94%A8-yaml-%E9%83%A8%E7%BD%B2%E5%BA%94%E7%94%A8)
  * [5\.3 对外发布(暴露端口)](#53-%E5%AF%B9%E5%A4%96%E5%8F%91%E5%B8%83%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3)
  * [5\.4 再次使用 Yaml 部署应用](#54-%E5%86%8D%E6%AC%A1%E4%BD%BF%E7%94%A8-yaml-%E9%83%A8%E7%BD%B2%E5%BA%94%E7%94%A8)
* [六、升级回滚](#%E5%85%AD%E5%8D%87%E7%BA%A7%E5%9B%9E%E6%BB%9A)
  * [6\.1 应用升级](#61-%E5%BA%94%E7%94%A8%E5%8D%87%E7%BA%A7)
  * [6\.2 查看升级版本](#62-%E6%9F%A5%E7%9C%8B%E5%8D%87%E7%BA%A7%E7%89%88%E6%9C%AC)
  * [6\.3 回滚到上一个版本](#63-%E5%9B%9E%E6%BB%9A%E5%88%B0%E4%B8%8A%E4%B8%80%E4%B8%AA%E7%89%88%E6%9C%AC)
  * [6\.4 回滚到指定的版本](#64-%E5%9B%9E%E6%BB%9A%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9A%84%E7%89%88%E6%9C%AC)
* [七、弹性伸缩](#%E4%B8%83%E5%BC%B9%E6%80%A7%E4%BC%B8%E7%BC%A9)

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