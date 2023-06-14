




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






# 部署单机kafka
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zookeeper-use-password
  namespace: bigdata
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: zookeeper-use-password
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: zookeeper-use-password
    spec:
      containers:
        - image: wurstmeister/zookeeper
          imagePullPolicy: IfNotPresent
          name: zookeeper-use-password
          ports:
            - containerPort: 2181
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30


---
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-use-password
  namespace: bigdata
spec:
  ports:
    - name: zookeeper-port
      port: 2181
      protocol: TCP
      targetPort: 2181
  selector:
    app: zookeeper-use-password
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-use-password-np
  namespace: bigdata
spec:
  externalTrafficPolicy: Cluster
  ports:
    - name: zookeeper-port
      nodePort: 32099
      port: 2181
      protocol: TCP
      targetPort: 2181
  selector:
    app: zookeeper-use-password
  sessionAffinity: None
  type: NodePort


---

# cat << EOF >> kafka-config.properties
# security.protocol=SASL_PLAINTEXT
# sasl.mechanism=PLAIN
# sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="kafka@2023*&^";
# EOF
# kafka-console-producer.sh --topic aa --bootstrap-server 127.0.0.1:9092 --producer.config kafka-config.properties
# kafka-console-consumer.sh --topic aa --bootstrap-server 127.0.0.1:9092 --consumer.config kafka-config.properties --from-beginning
---

apiVersion: v1
data:
  kafka.properties: |
    listeners=SASL_PLAINTEXT://0.0.0.0:9092
    advertised.listeners=SASL_PLAINTEXT://192.168.1.248:9092
    security.inter.broker.protocol=SASL_PLAINTEXT
    sasl.enabled.mechanisms=PLAIN
    sasl.mechanism.inter.broker.protocol=PLAIN
    authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
    super.users=User:admin
    allow.everyone.if.no.acl.found=false
  kafka-credentials.properties: |
    username=admin
    password=kafka@2023*&^
  kafka_server_jaas.conf: |
    KafkaServer {
      org.apache.kafka.common.security.plain.PlainLoginModule required
      username="admin"
      password="kafka@2023*&^"
      user_admin="kafka@2023*&^";
    };
kind: ConfigMap
metadata:
  name: kafka-use-password
  namespace: bigdata


---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-use-password
  namespace: bigdata
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: kafka-use-password
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: kafka-use-password
    spec:
      containers:
      - env:
        - name: KAFKA_ADVERTISED_PORT
          value: "30098"
        - name: KAFKA_PORT
          value: "9092"
        - name: KAFKA_ADVERTISED_HOST_NAME
          value: 192.168.1.248
        - name: KAFKA_ZOOKEEPER_CONNECT
          value: zookeeper-use-password:2181
        - name: KAFKA_MESSAGE_MAX_BYTES
          value: "10000000"
        - name: KAFKA_BROKER_ID
          value: "10"
        - name: KAFKA_OPTS
          value: "-Djava.security.auth.login.config=/opt/kafka/kafka_server_jaas.conf"
        - name: KAFKA_LISTENERS
          value: SASL_PLAINTEXT://0.0.0.0:9092
        - name: KAFKA_ADVERTISED_LISTENERS
          value: SASL_PLAINTEXT://192.168.1.248:30098
        - name: KAFKA_SECURITY_INTER_BROKER_PROTOCOL
          value: SASL_PLAINTEXT
        - name: KAFKA_SASL_ENABLED_MECHANISMS
          value: PLAIN
        - name: KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL
          value: PLAIN
        - name: KAFKA_AUTHORIZER_CLASS_NAME
          value: kafka.security.auth.SimpleAclAuthorizer
        - name: KAFKA_SUPER_USERS
          value: User:admin
        - name: KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND
          value: "false"
        image: wurstmeister/kafka
        imagePullPolicy: IfNotPresent
        name: kafka-use-password
        ports:
        - containerPort: 9092
          protocol: TCP
        volumeMounts:
        - mountPath: /opt/kafka/config/kafka.properties
          name: kafka-properties
          subPath: kafka.properties
        - mountPath: /opt/kafka/config/kafka-credentials.properties
          name: kafka-properties
          subPath: kafka-credentials.properties
        - mountPath: /opt/kafka/kafka_server_jaas.conf
          name: kafka-properties
          subPath: kafka_server_jaas.conf
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: kafka-properties
        configMap:
          name: kafka-use-password
          items:
          - key: "kafka.properties"
            path: "kafka.properties"
          items:
          - key: "kafka-credentials.properties"
            path: "kafka-credentials.properties"
          items:
          - key: "kafka_server_jaas.conf"
            path: "kafka_server_jaas.conf"

---
apiVersion: v1
kind: Service
metadata:
  name: kafka-use-password
  namespace: bigdata
spec:
  selector:
    name: kafka-use-password
  ports:
  - protocol: TCP
    port: 9092
    targetPort: 9092

---
apiVersion: v1
kind: Service
metadata:
  name: kafka-use-password-np
  namespace: bigdata
spec:
  externalTrafficPolicy: Cluster
  ports:
  - name: kafka-port
    nodePort: 30098
    port: 9092
    protocol: TCP
    targetPort: 9092
  selector:
    name: kafka-use-password
  sessionAffinity: None
  type: NodePort
```










