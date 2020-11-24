


* [一、概述](#%E4%B8%80%E6%A6%82%E8%BF%B0)
* [二、使用](#%E4%BA%8C%E4%BD%BF%E7%94%A8)
  * [2\.1 创建 label](#21-%E5%88%9B%E5%BB%BA-label)
  * [2\.2 查看 label](#22-%E6%9F%A5%E7%9C%8B-label)
  * [2\.3 修改 label](#23-%E4%BF%AE%E6%94%B9-label)
  * [2\.4 删除 label](#24-%E5%88%A0%E9%99%A4-label)

----
# 一、概述
Label 是 k8s 的一个核心概念

Label 是一个 `key=value` 的键值对, 其中 key 和 value 都是由用户自己制定

Label 可以附加到各种资源对象上, 例如: Node、Pod、Service、RC

一个资源对象可以定义任意数量的Label, 同一个 Label 也可以被添加到任意数量的资源对象上

Label 通常在资源对象定义是确定, 也可以在对象创建后动态 添加或删除

Label 的最常见用法是使用 `metadata.labels` 字段, 来为对象添加 Label, 通过 spec.selector 来引用对象


# 二、使用
## 2.1 创建 label
```bash
# 语法
$ kubectl label node <node> <key>=<value>

# 例子
$ kubectl label node k8s-master label_key=label_value
node/k8s-master labeled
```
## 2.2 查看 label
```bash
$ kubectl get nodes --show-label
[root@k8s-master yaml]# kubectl get nodes --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-master   Ready    master   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,label_key=label_value,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux
```
## 2.3 修改 label
```bash
# 语法
$ kubectl label nodes <node> <key>=<value> --overwrite

# 例子
$ kubectl label nodes k8s-master label_key=label_value2 --overwrite
node/k8s-master labeled

# 查看
[root@k8s-master yaml]# kubectl get nodes --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-master   Ready    master   7d6h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,label_key=label_value2,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux
```

## 2.4 删除 label
```bash
# 语法
$ kubectl label node <node> <key>-
# 例子
$ kubectl get nodes --show-labels
NAME         STATUS   ROLES    AGE    VERSION   LABELS
k8s-master   Ready    master   7d6h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node1    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node1,kubernetes.io/os=linux
k8s-node2    Ready    <none>   7d5h   v1.18.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node2,kubernetes.io/os=linux
```