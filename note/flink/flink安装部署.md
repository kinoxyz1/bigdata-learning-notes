




---
# 一、单机模式
* [Flink 安装部署](./flink部署和运行.md)

# 二、standalone 模式
[参考 flink 官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.14/docs/deployment/resource-providers/standalone/overview/#standalone)

# 三、Flink on Kubernetes
## 3.1 Standalone Kubernetes
```bash
## apply 如下几个 yaml
$ ls -l
-rw-r--r-- 1 root root      946 3月  28 18:44 flink-configuration-configmap.yaml
-rw-r--r-- 1 root root     1461 3月  28 18:55 jobmanager-deployment.yaml
-rw-r--r-- 1 root root      224 3月  28 18:41 jobmanager-rest-service.yaml
-rw-r--r-- 1 root root      256 3月  28 18:41 jobmanager-service.yaml
-rw-r--r-- 1 root root     1370 3月  28 18:56 taskmanager-deployment.yaml

$ kubectl apply -f ./
```

flink-configuration-configmap.yaml
```shell
vim flink-configuration-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flink-config
  namespace: flink
  labels:
    app: flink
data:
  flink-conf.yaml: |+
    jobmanager.rpc.address: flink-jobmanager
    taskmanager.numberOfTaskSlots: 10
    blob.server.port: 6124
    jobmanager.rpc.port: 6123
    taskmanager.rpc.port: 6122
    jobmanager.heap.size: 10240m
    taskmanager.memory.process.size: 10240m
  log4j.properties: |+
    log4j.rootLogger=INFO, file
    log4j.logger.akka=INFO
    log4j.logger.org.apache.kafka=INFO
    log4j.logger.org.apache.hadoop=INFO
    log4j.logger.org.apache.zookeeper=INFO
    log4j.appender.file=org.apache.log4j.FileAppender
    log4j.appender.file.file=${log.file}
    log4j.appender.file.layout=org.apache.log4j.PatternLayout
    log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p %-60c %x - %m%n
    log4j.logger.org.apache.flink.shaded.akka.org.jboss.netty.channel.DefaultChannelPipeline=ERROR, file
```

jobmanager-deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flink-jobmanager
  namespace: flink
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flink
      component: jobmanager
  template:
    metadata:
      labels:
        app: flink
        component: jobmanager
    spec:
      containers:
      - name: jobmanager
        # image: flink:latest
        image: flink:1.10.1-scala_2.11
        workingDir: /opt/flink
        command: ["/bin/bash", "-c", "$FLINK_HOME/bin/jobmanager.sh start;\
          while :;
          do
            if [[ -f $(find log -name '*jobmanager*.log' -print -quit) ]];
              then tail -f -n +1 log/*jobmanager*.log;
            fi;
          done"]
        ports:
        - containerPort: 6123
          name: rpc
        - containerPort: 6124
          name: blob
        - containerPort: 8081
          name: ui
        livenessProbe:
          tcpSocket:
            port: 6123
          initialDelaySeconds: 30
          periodSeconds: 60
        volumeMounts:
        - name: flink-config-volume
          mountPath: /opt/flink/conf
        securityContext:
          runAsUser: 9999  # refers to user _flink_ from official flink image, change if necessary
      volumes:
      - name: flink-config-volume
        configMap:
          name: flink-config
          items:
          - key: flink-conf.yaml
            path: flink-conf.yaml
          - key: log4j.properties
            path: log4j.properties
```
jobmanager-rest-service.yaml
```bash
apiVersion: v1
kind: Service
metadata:
  name: flink-jobmanager-rest
  namespace: flink
spec:
  type: NodePort
  ports:
  - name: rest
    port: 8081
    targetPort: 8081
  selector:
    app: flink
    component: jobmanager
```
jobmanager-service.yaml
```bash
apiVersion: v1
kind: Service
metadata:
  name: flink-jobmanager
  namespace: flink
spec:
  type: ClusterIP
  ports:
  - name: rpc
    port: 6123
  - name: blob
    port: 6124
  - name: ui
    port: 8081
  selector:
    app: flink
    component: jobmanager
```
taskmanager-deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flink-taskmanager
  namespace: flink
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flink
      component: taskmanager
  template:
    metadata:
      labels:
        app: flink
        component: taskmanager
    spec:
      containers:
      - name: taskmanager
        # image: flink:latest
        image: flink:1.10.1-scala_2.11
        workingDir: /opt/flink
        command: ["/bin/bash", "-c", "$FLINK_HOME/bin/taskmanager.sh start; \
          while :;
          do
            if [[ -f $(find log -name '*taskmanager*.log' -print -quit) ]];
              then tail -f -n +1 log/*taskmanager*.log;
            fi;
          done"]
        ports:
        - containerPort: 6122
          name: rpc
        livenessProbe:
          tcpSocket:
            port: 6122
          initialDelaySeconds: 30
          periodSeconds: 60
        volumeMounts:
        - name: flink-config-volume
          mountPath: /opt/flink/conf/
        securityContext:
          runAsUser: 9999  # refers to user _flink_ from official flink image, change if necessary
      volumes:
      - name: flink-config-volume
        configMap:
          name: flink-config
          items:
          - key: flink-conf.yaml
            path: flink-conf.yaml
          - key: log4j.properties
            path: log4j.properties
```



## 3.2 Native Kubernetes 
### 前置知识 RBAC
[K8S RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

### 3.2.1 Application Mode
制作镜像
```bash
FROM flink:1.12.7-scala_2.11-java8
RUN mkdir -p $FLINK_HOME/usrlib
# COPY core-site.xml $FLINK_HOME/hadoopconf
# COPY hdfs-site.xml $FLINK_HOME/hadoopconf
COPY WordCount.jar $FLINK_HOME/usrlib/WordCount.jar
COPY TopSpeedWindowing.jar $FLINK_HOME/usrlib/TopSpeedWindowing.jar
COPY StreamingWordCount.jar $FLINK_HOME/usrlib/StreamingWordCount.jar
COPY flink-shaded-hadoop-2-uber-3.0.0-10.0.jar $FLINK_HOME/lib
```
运行
```shell
kubectl create serviceaccount flink -n flink112
kubectl create clusterrolebinding flink-role-binding-default \
  --clusterrole=edit \
  --serviceaccount=flink112:flink \
  -n flink112
  
# run 
./bin/flink run-application \
    --target kubernetes-application \
    -Dkubernetes.namespace=flink112 \
    -Dkubernetes.jobmanager.service-account=flink \
    -Dkubernetes.cluster-id=my-first-application-cluster \
    -Dkubernetes.container.image=flink-kino:v112 \
    -Dkubernetes.rest-service.exposed.type=NodePort \
    -Dstate.backend=jobmanager \
    -Dstate.backend.incremental=true \
    local:///opt/flink/usrlib/TopSpeedWindowing.jar 

# stop 
kubectl delete deployment/my-first-flink-cluster 
```



### 3.2.2 Session Mode
```shell

## create RBAC namespacel=default 
kubectl create serviceaccount flink -n flink112
kubectl create clusterrolebinding flink-role-binding-default \
  --clusterrole=edit \
  --serviceaccount=flink112:flink \
  -n flink112
  
## run
./bin/kubernetes-session.sh \
  -Dkubernetes.namespace=flink112 \
  -Dkubernetes.jobmanager.service-account=flink \
  -Dkubernetes.cluster-id=my-first-flink-cluster \
  -Dtaskmanager.memory.process.size=4096m \
  -Dkubernetes.taskmanager.cpu=1 \
  -Dtaskmanager.numberOfTaskSlots=4 \
  -Dresourcemanager.taskmanager-timeout=3600000
  
./bin/flink run \
    --target kubernetes-session \
    -Dkubernetes.namespace=flink112 \
    -Dkubernetes.jobmanager.service-account=flink \
    -Dkubernetes.cluster-id=my-first-flink-cluster \
    ./examples/streaming/TopSpeedWindowing.jar
  
## get run job
./bin/flink list --target kubernetes-application -Dkubernetes.cluster-id=my-first-flink-cluster -Dkubernetes.namespace=flink112

# stop 
kubectl delete deployment/my-first-flink-cluster 
```

### 3.2.3 Pre-Job Mode
暂不支持



# 四、Flink on Yarn(CDH)
[CDH6.3.2 集成 FLink1.12](https://blog.csdn.net/qq_39945938/article/details/121073819?spm=1001.2101.3001.6650.18&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-18.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-18.pc_relevant_default&utm_relevant_index=24)
























