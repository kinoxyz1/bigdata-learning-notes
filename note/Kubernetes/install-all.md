




---
# 一、install Harbor
## 1.1 helm下载charts
```bash
helm repo add harbor https://helm.goharbor.io
helm pull harbor/harbor
```

## 1.2 定制配置
### 1.2.1 TLS证书
```bash
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ${KEY_FILE:tls.key} -out ${CERT_FILE:tls.cert} -subj "/CN=${HOST:itdachang.com}/O=${HOST:itdachang.com}"

kubectl create secret tls ${CERT_NAME:itdachang-tls} --key ${KEY_FILE:tls.key} --cert ${CERT_FILE:tls.cert}


## 示例命令如下
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=*.kino.com/O=*.kino.com"

kubectl create secret tls harbor.kino.com --key tls.key --cert tls.crt -n devops
```

> 原来证书是 itdachang.com 域名
>
> 现在用的是harbor.itdachang.com 域名的。
>
> 单独创建一个


### 1.2.2 values-overrides.yaml 配置
```yaml
expose:  #web浏览器访问用的证书
  type: ingress
  tls:
    certSource: "secret"
    secret:
      secretName: "harbor.kino.com"
      notarySecretName: "harbor.kino.com"
  ingress:
    hosts:
      core: harbor.kino.com
      notary: notary-harbor.kino.com
externalURL: https://harbor.kino.com
internalTLS:  #harbor内部组件用的证书
  enabled: true
  certSource: "auto"
persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:  # 存镜像的
      storageClass: "rook-ceph-block"
      accessMode: ReadWriteOnce
      size: 5Gi
    chartmuseum: #存helm的chart
      storageClass: "rook-ceph-block"
      accessMode: ReadWriteOnce
      size: 5Gi
    jobservice: #
      storageClass: "rook-ceph-block"
      accessMode: ReadWriteOnce
      size: 1Gi
    database: #数据库  pgsql
      storageClass: "rook-ceph-block"
      accessMode: ReadWriteOnce
      size: 1Gi
    redis: #
      storageClass: "rook-ceph-block"
      accessMode: ReadWriteOnce
      size: 1Gi
    trivy: # 漏洞扫描
      storageClass: "rook-ceph-block"
      accessMode: ReadWriteOnce
      size: 5Gi
metrics:
  enabled: true
```

## 1.3 安装
```bash
#注意，由于配置文件用到secret，所以提前在这个名称空间创建好
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.cert -subj "/CN=*.kino.com/O=*.kino.com"
kubectl create secret tls kino.com --key tls.key --cert tls.cert -n devops

helm install itharbor ./ -f values.yaml -f override.yaml  -n devops
```

## 1.4 卸载
```bash
helm uninstall itharbor -n devops
```

## 1.5 harbor使用

https://goharbor.io/docs/2.2.0/working-with-projects/

访问： https://harbor.kino.com:4443/

账号：admin  密码：Harbor12345   修改后：Admin123789

```sh
zSPz26aQuyunPfPPvw7aGuu9JIdkJqLk

docker login <harbor_address<>
Username: <prefix><account_name>
Password: <secret>
```


















