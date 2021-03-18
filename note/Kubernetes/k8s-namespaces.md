





---

# 一、Namespace 概述

namespace 是对一组资源和对象的抽象集合，比如可以将系统内部的对象划分为不同的项目组合用户组。常用的pods，services，replication controllers 和 deployments 等都是属于某一个namespace(默认是 default)，而 node，persistentVolumes 等则不属于任何一个 namespace。



namespace 常用来隔离不同的用户，比如 kubernetes 自带的服务一般运行在 kube-system namespace中。



kubernetes 中的namespace与docker 中的namespace 不同，kubernetes的namespace只是做了一个逻辑上的隔离。



# 二、对namespace的操作

## 2.1 查询所有namespace

```bash
root@jz-desktop-01:~# kubectl get namespaces
NAME              STATUS   AGE
default           Active   89d
kube-node-lease   Active   89d
kube-public       Active   89d
kube-system       Active   89d
```

## 2.2 查询某个namespace的详细情况

```bash
root@jz-desktop-01:~# kubectl describe namespaces default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```

























































