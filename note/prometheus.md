



---


# k8s 部署 prometheus 和 grafana

[官方 github 地址](https://github.com/prometheus-operator/kube-prometheus)

[kubeshpere 官方文档步骤2](https://kubesphere.io/zh/docs/faq/observability/byop/#%E6%AD%A5%E9%AA%A4-2%E5%AE%89%E8%A3%85%E6%82%A8%E8%87%AA%E5%B7%B1%E7%9A%84-prometheus-%E5%A0%86%E6%A0%88)

按照官方文档的 Quickstart 依次执行即可部署，部署时间由机器网络环境决定(需要网络不受限)。

部署时可以适当去除不需要的服务
```bash
$ rm -rf manifests/prometheus-adapter-*.yaml
```

部署完成之后，需要暴露 grafana 端口用以访问其 webui
```bash
$ vim manifests/grafana-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 8.1.1
  name: grafana
  namespace: monitoring
spec:
  ports:
  - name: http
    port: 3000
    protocol: TCP
    targetPort: 3000
    nodePort: 31300
  type: NodePort
  selector:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus

$ kubectl apply -f manifests/grafana-service.yaml

# 查看状态
$ kubectl get pod -n monitoring
NAME                                   READY   STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2     Running   0          20m
alertmanager-main-1                    2/2     Running   0          20m
alertmanager-main-2                    2/2     Running   0          20m
blackbox-exporter-6798fb5bb4-kkpgc     3/3     Running   0          20m
grafana-7476b4c65b-wrstc               1/1     Running   0          20m
kube-state-metrics-74964b6cd4-k8fh6    3/3     Running   0          20m
node-exporter-6kzrh                    2/2     Running   0          20m
node-exporter-b2t6c                    2/2     Running   0          20m
node-exporter-shr5q                    2/2     Running   0          20m
prometheus-k8s-0                       2/2     Running   0          19m
prometheus-k8s-1                       2/2     Running   0          19m
prometheus-operator-75d9b475d9-hdg7l   2/2     Running   0          30m
```

# 监控 k8s 和 微服务资源明细
[开源项目模板](https://github.com/starsliao/Prometheus/tree/master/kubernetes)

按照开源项目描述操作即可(需要 grafana 版本为 v7.5.x 或者 v8.x, 否则报错)。

