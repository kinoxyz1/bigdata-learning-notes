





---

# 一、NameSpace 强制删除
集群中存在一直处于 Terminating 的 ns
```bash
[root@jz-desktop-02 flink112]# kubectl get ns
NAME                          STATUS        AGE
datacenter                    Terminating   22d
jzcost                        Terminating   11d
kube-node-lease               Active        116d
kube-public                   Active        116d
kube-system                   Active        116d
monitoring                    Terminating   88d
```

删除步骤
```bash
kubectl get namespace <terminating-namespace> -o json >tmp.json

# 删除 finalizers 的 kubernetes 属性
"spec": {
        "finalizers": [
          ## 删除这里
        ]
    },

kubectl proxy

curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/<terminating-namespace>/finalize
```







