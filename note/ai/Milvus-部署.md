





[Milvus 官方文档](https://milvus.io/docs)

[Milvus 管理工具 Attu 官方文档](https://github.com/zilliztech/attu)

# Docker 部署
安装 docker compose 

https://milvus.io/docs/prerequisite-docker.md

在官方的 docker-compose.yaml 文件中增加 attu 部分内容:
```bash
version: '3.5'

services:
  attu:
    container_name: attu
    image: zilliz/attu:latest
    environment:
      - MILVUS_URL=milvus-standalone:19530
    ports:
      - "9081:3000"
  etcd:
    container_name: milvus-etcd
    image: quay.io/coreos/etcd:v3.5.5
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
      - ETCD_SNAPSHOT_COUNT=50000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/etcd:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio:
    container_name: milvus-minio
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    ports:
      - "9001:9001"
      - "9000:9000"
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/minio:/minio_data
    command: minio server /minio_data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  standalone:
    container_name: milvus-standalone
    image: milvusdb/milvus:v2.4.5
    command: ["milvus", "run", "standalone"]
    security_opt:
    - seccomp:unconfined
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9091/healthz"]
      interval: 30s
      start_period: 90s
      timeout: 20s
      retries: 3
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - "etcd"
      - "minio"

networks:
  default:
    name: milvus
```

http://your_ip:9081


# Milvus Operator 部署
Milvus Operator 部署的前提是要在k8s集群提前部署[StorageClass](../Kubernetes/volume.md#741-部署)


## 安装 Milvus Operator
安装 cert-manager
```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```
安装 Milvus Operator
```bash
kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/deploy/manifests/deployment.yaml
```
部署完之后可以看到集群有如下pod
```bash
kubectl get pods -n milvus-operator
NAME                              READY   STATUS    RESTARTS   AGE
milvus-operator-c564dbfd6-22cgp   1/1     Running   0          2d
```
部署Milvus cluster
```bash
kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_cluster_default.yaml
```
部署完可以检查Milvus集群是否正常
```bash
kubectl get milvus my-release -o yaml
status:
  conditions:
  - lastTransitionTime: "2024-07-16T07:41:22Z"
    message: Etcd endpoints is healthy
    reason: EtcdReady
    status: "True"
    type: EtcdReady
  - lastTransitionTime: "2024-07-16T07:41:07Z"
    reason: StorageReady
    status: "True"
    type: StorageReady
  - lastTransitionTime: "2024-07-16T07:43:43Z"
    message: MsgStream is ready
    reason: MsgStreamReady
    status: "True"
    type: MsgStreamReady
  - lastTransitionTime: "2024-07-18T07:43:43Z"
    message: All Milvus components are healthy
    reason: ReasonMilvusHealthy
    status: "True"
    type: MilvusReady
  - lastTransitionTime: "2024-07-18T07:29:43Z"
    message: Milvus components are all updated
    reason: MilvusComponentsUpdated
    status: "True"
    type: MilvusUpdated
  endpoint: milvus-release-milvus.milvus:19530
```
查看启动的容器
```bash
kubectl get pod -n milvus
NAME                                                 READY   STATUS      RESTARTS   AGE
attu-6f447557ff-nxh5v                                1/1     Running     0          47h
milvus-release-etcd-0                                1/1     Running     0          2d
milvus-release-etcd-1                                1/1     Running     0          2d
milvus-release-etcd-2                                1/1     Running     0          2d
milvus-release-milvus-datacoord-5d766b99f6-n9lqf     1/1     Running     0          2d
milvus-release-milvus-datanode-8646766b5c-bxxq4      1/1     Running     0          2d
milvus-release-milvus-indexcoord-745fd4bb78-4sjhz    1/1     Running     0          2d
milvus-release-milvus-indexnode-cfd9688bf-sp9sw      1/1     Running     0          2d
milvus-release-milvus-proxy-74cf6d8445-dwwh2         1/1     Running     0          2d
milvus-release-milvus-proxy-74cf6d8445-qzpn4         1/1     Running     0          2d
milvus-release-milvus-querycoord-6f549b9bd6-xhz5t    1/1     Running     0          2d
milvus-release-milvus-querynode-0-6c54bc8664-v49qd   1/1     Running     0          2d
milvus-release-milvus-rootcoord-77986849b8-mnp49     1/1     Running     0          2d
milvus-release-minio-0                               1/1     Running     0          2d
milvus-release-minio-1                               1/1     Running     0          2d
milvus-release-minio-2                               1/1     Running     0          2d
milvus-release-minio-3                               1/1     Running     0          2d
milvus-release-pulsar-bookie-0                       1/1     Running     0          2d
milvus-release-pulsar-bookie-1                       1/1     Running     0          2d
milvus-release-pulsar-bookie-2                       1/1     Running     0          2d
milvus-release-pulsar-bookie-init-phq2d              0/1     Completed   0          2d
milvus-release-pulsar-broker-0                       1/1     Running     0          2d
milvus-release-pulsar-proxy-0                        1/1     Running     0          2d
milvus-release-pulsar-pulsar-init-sxv5k              0/1     Completed   0          2d
milvus-release-pulsar-recovery-0                     1/1     Running     0          2d
milvus-release-pulsar-zookeeper-0                    1/1     Running     0          2d
milvus-release-pulsar-zookeeper-1                    1/1     Running     0          2d
milvus-release-pulsar-zookeeper-2                    1/1     Running     0          2d
```
修改副本数(直接修改 deployment 里面的副本数是不生效的)
```bash
kubectl edit milvus -n milvus
...
spec:
  components:
    dataCoord:
      paused: false
      replicas: 1
    dataNode:
      paused: false
      replicas: 3
    disableMetric: false
    image: milvusdb/milvus:v2.4.5
    imageUpdateMode: rollingUpgrade
    indexCoord:
      paused: false
      replicas: 1
    indexNode:
      paused: false
      replicas: 3
    metricInterval: ""
    paused: false
    proxy:
      paused: false
      replicas: 2
      serviceType: ClusterIP
    queryCoord:
      paused: false
      replicas: 1
    queryNode:
      paused: false
      replicas: 3
    rootCoord:
      paused: false
      replicas: 1
    standalone:
      paused: false
      replicas: 0
      serviceType: ClusterIP
...
```




















