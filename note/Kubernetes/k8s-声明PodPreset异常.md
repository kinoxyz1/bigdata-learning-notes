





# 一、异常信息
```bash
[root@k8s-master podpreset]#  kubectl apply -f podpreset.yaml
error: unable to recognize "podpreset.yaml": no matches for kind "PodPreset" in version "settings.k8s.io/v1alpha1"
```

# 二、解决
在 `/etc/kubernetes/manifests/kube-apiserver.yaml` 添加如下内容
```bash
- --enable-admission-plugins=NodeRestriction,PodPreset
- --runtime-config=settings.k8s.io/v1alpha1=true
```
然后重启 kubelet
```bash
$ systemctl restart kubelet
```