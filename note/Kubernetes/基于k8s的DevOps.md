





# 一、k8s 集群部署
[k8s 部署](../../note/Kubernetes/k8s部署.md)

# 二、NFS 部署和创建 storageClass
[文档查看 <五、NFS> 和 <6.4 动态存储类（Storage Class）>](../../note/Kubernetes/volume.md)

# 三、ceph 部署和创建 storageClass
[volume # 六、ceph ](volume.md)



# 四、Jenkins 部署
docker 方式
```bash
docker run \
--name=jenkins \
--user=root \
--env=LANG=C.UTF-8 \
--volume=/app/jenkins/jenkins_home:/var/jenkins_home \
--volume=/root/.ssh:/root/.ssh/ \
--volume=/var/run/docker.sock:/var/run/docker.sock \
--volume=/usr/bin/docker:/usr/bin/docker \
--volume=/usr/bin/kubectl:/usr/bin/kubectl \
--volume=/root/.docker:/root/.docker \
--volume=/etc/docker:/etc/docker \
--volume=/app/maven3.8:/app/maven3.8 \
--volume=/var/jenkins_home \
-p 17000:8080 \
-p 50000:50000 \
--restart=always \
--detach=true \
jenkins/jenkins:2.346.3-centos7-jdk8
```
helm 方式
```bash
# 添加 repo
helm repo add jenkinsci https://charts.jenkins.io/
helm pull jenkinsci/jenkins --version 3.3.18

# value.yaml
controller:
  componentName: "jenkins-controller"
  image: "jenkinsci/blueocean"
  tag: "1.24.7"
  imagePullPolicy: "Always"
  adminSecret: true


  adminUser: "admin"
  adminPassword: "admin"


  servicePort: 8080
  targetPort: 8080
  serviceType: ClusterIP

  healthProbes: true
  probes:
    startupProbe:
      httpGet:
        path: '{{ default "" .Values.controller.jenkinsUriPrefix }}/login'
        port: http
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 12
    livenessProbe:
      failureThreshold: 5
      httpGet:
        path: '{{ default "" .Values.controller.jenkinsUriPrefix }}/login'
        port: http
      periodSeconds: 10
      timeoutSeconds: 5
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: '{{ default "" .Values.controller.jenkinsUriPrefix }}/login'
        port: http
      periodSeconds: 10
      timeoutSeconds: 5
  agentListenerServiceType: "ClusterIP"

  installPlugins:
    - kubernetes:1.29.4
    - workflow-aggregator:2.6
    - git:4.7.1
    - configuration-as-code:1.51

  initializeOnce: true

  ingress:
    enabled: true
    paths: 
    - backend:
        service:
            name: jenkins
            port:
              number: 8080
      path: "/"
      pathType: Prefix
    apiVersion: "networking.k8s.io/v1s"
    kubernetes.io/ingress.class: nginx

    hostName: jenkins.itdachang.com
    tls:
    - secretName: itdachang.com
      hosts:
        - jenkins.itdachang.com

  prometheus:
    enabled: true
    scrapeInterval: 60s
    scrapeEndpoint: /prometheus

agent:
  enabled: true
  workspaceVolume: 
     type: PVC
     claimName: jenkins-workspace-pvc
     readOnly: false

additionalAgents:
  maven:
    podName: maven
    customJenkinsLabels: maven
    image: jenkins/jnlp-agent-maven

persistence:
  enabled: true
  storageClass: "rook-ceph-block"
  accessMode: "ReadWriteOnce"
  size: "8Gi"
  
# 证书
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.cert -subj "/CN=*.kino.com/O=*.kino.com"
kubectl create ns devops
kubectl create secret tls kino.com --key tls.key --cert tls.cert -n devops

# 安装
helm install -f values.yaml -f override.yaml jenkins ./ -n devops
```
k8s yaml 安装
```bash
#创建证书，或者使用以前的证书
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=*.kino.com/O=*.kino.com"

# 1、编写Jenkins配置文件
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: jenkins
  namespace: devops
spec:
  selector:
    matchLabels:
      app: jenkins # has to match .spec.template.metadata.labels
  serviceName: "jenkins"
  replicas: 1
  template:
    metadata:
      labels:
        app: jenkins # has to match .spec.selector.matchLabels
    spec:
      serviceAccountName: "jenkins"
      terminationGracePeriodSeconds: 10
      containers:
      - name: jenkins
        image: jenkinsci/blueocean:1.24.7
        securityContext:                     
          runAsUser: 0                      #设置以ROOT用户运行容器
          privileged: true                  #拥有特权
        ports:
        - containerPort: 8080
          name: web
        - name: jnlp                        #jenkins slave与集群的通信口
          containerPort: 50000
        resources:
          limits:
            memory: 2Gi
            cpu: "2000m"
          requests:
            memory: 700Mi
            cpu: "500m"
        env:
        - name: LIMITS_MEMORY
          valueFrom:
            resourceFieldRef:
              resource: limits.memory
              divisor: 1Mi
        - name: "JAVA_OPTS" #设置变量，指定时区和 jenkins slave 执行者设置
          value: " 
                   -Xmx$(LIMITS_MEMORY)m 
                   -XshowSettings:vm 
                   -Dhudson.slaves.NodeProvisioner.initialDelay=0
                   -Dhudson.slaves.NodeProvisioner.MARGIN=50
                   -Dhudson.slaves.NodeProvisioner.MARGIN0=0.75
                   -Duser.timezone=Asia/Shanghai
                 "  
        volumeMounts:
        - name: home
          mountPath: /var/jenkins_home
  volumeClaimTemplates:
  - metadata:
      name: home
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "rook-ceph-block"
      resources:
        requests:
          storage: 5Gi

---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: devops
spec:
  selector:
    app: jenkins
  type: ClusterIP
  ports:
  - name: web
    port: 8080
    targetPort: 8080
    protocol: TCP
  - name: jnlp
    port: 50000
    targetPort: 50000
    protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jenkins
  namespace: devops
spec:
  tls:
  - hosts:
      - jenkins.kino.com
    secretName: kino.com
  rules:
  - host: jenkins.kino.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: jenkins
            port:
              number: 8080
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: devops

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins
rules:
  - apiGroups: ["extensions", "apps"]
    resources: ["deployments"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["create", "delete", "get", "list", "watch", "patch", "update"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create","delete","get","list","patch","update","watch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create","delete","get","list","patch","update","watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get","list","watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins
roleRef:
  kind: ClusterRole
  name: jenkins
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: devops
```

# 四、gitlab 部署(docker)
```bash
# 安装 pgsql
docker run --name postgresql \
  -d \
  --detach=true \
  --restart=always \
  --privileged=true \
  -e 'DB_NAME=gitlabhq_production' \
  -e 'DB_USER=gitlab' \
  -e 'DB_PASS=gitlab123#EDc' \
  -e 'DB_EXTENSION=pg_trgm' \
  -v /app/docker/postgresql/data:/var/lib/postgresql \
  sameersbn/postgresql
  
# 安装 redis
docker run --name redis \
  -d \
  --detach=true \
  --restart=always \
  --privileged=true \
  -v /app/docker/redis/data:/var/lib/redis \
  sameersbn/redis
  
# 安装 gitlab
docker run --name gitlab \
  -d \
  --restart=always \
  --link postgresql:postgresql \
  --link redis:redis \
  --hostname 172.20.135.8 \
  -p 10022:22 \
  -p 8899:80 \
  -e 'GITLAB_PORT=8899' \
  -e 'GITLAB_SSH_PORT=10022' \
  -e 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
  -e 'GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alpha-numeric-string' \
  -e 'GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alpha-numeric-string' \
  -e 'GITLAB_HOST=gitlab.hyunteng.com' \
  -e 'SMTP_AUTHENTICATION=login' \
  -e 'GITLAB_NOTIFY_ON_BROKEN_BUILDS="true"' \
  -e 'GITLAB_NOTIFY_PUSHER="false"' \
  -e 'GITLAB_ROOT_EMAIL=kinoxyz1@gmail.com' \
  -e 'GITLAB_BACKUP_TIME=01:20' \
  -e '-e GITLAB_LOG_LEVEL=error' \
  -v /app/docker/gitlab/opt:/var/opt/gitlab \
  -v /app/docker/gitlab/data:/home/git/data \
  gitlab/gitlab-ce
```
查看默认密码以及改忘记密码
```bash
# 获取超级用户密码
$ docker logs gitlab | grep password
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.
$ docker exec -it gitalb bash
$ cat /etc/gitlab/initial_root_password

# 忘记密码
sudo gitlab-rails console
user = User.where(id: 1).first
user.password = 'your_new_password'
user.password_confirmation = 'your_new_password'
user.save!
exit
```

# 五、Harbor 安装部署(Helm)
[Kubernetes helm](../../note/Kubernetes/k8s-helm.md)

```bash
helm repo add harbor https://helm.goharbor.io
helm pull harbor/harbor
tar -zxvf harbor.zip

kubectl create ns devops
# 创建secret(阿里云证书)
kubectl create secret tls harbor.kino.com --key harbor.kino.com.key --cert harbor.kino.com_public.crt -n devops

# 创建secret(本地自建)
# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ${KEY_FILE:tls.key} -out ${CERT_FILE:tls.cert} -subj "/CN=${HOST:itdachang.com}/O=${HOST:itdachang.com}"
# kubectl create secret tls ${CERT_NAME:itdachang-tls} --key ${KEY_FILE:tls.key} --cert ${CERT_FILE:tls.cert}
## 示例命令如下
# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=*.kino.com/O=*.kino.com"
# kubectl create secret tls harbor.kino.com --key tls.key --cert tls.crt -n devops

cd harbor
# 关于value.yaml 的说明, 根据自身情况修改即可
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
externalURL: https://harbor.kino.com  ## 外部能访问的域名
internalTLS:  #harbor内部组件用的证书
  enabled: true
  certSource: "auto"
persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:  # 步骤二中有讲怎么部署 storageClass
      storageClass: "managed-nfs-storage"
      accessMode: ReadWriteOnce
      size: 5Gi
    chartmuseum: #存helm的chart
      storageClass: "managed-nfs-storage"
      accessMode: ReadWriteOnce
      size: 5Gi
    jobservice: #
      storageClass: "managed-nfs-storage"
      accessMode: ReadWriteOnce
      size: 1Gi
    database: #数据库  pgsql
      storageClass: "managed-nfs-storage"
      accessMode: ReadWriteOnce
      size: 1Gi
    redis: #
      storageClass: "managed-nfs-storage"
      accessMode: ReadWriteOnce
      size: 1Gi
    trivy: # 漏洞扫描
      storageClass: "managed-nfs-storage"
      accessMode: ReadWriteOnce
      size: 5Gi
metrics:
  enabled: true
  

## 安装 
helm install harbor ./ -f values.yaml -n devops
```
docker 从 harbor 拉取镜像
```bash
# 自建证书需要在 docker 所有机器添加 hosts
# 登录
docker login --username=<YOUR_HARBOR_NAME> harbor.kino.com
```

# 六、Rancher 部署
```bash
docker run \
--detach=true \
--name=rancher \
--restart=always \
--privileged \
-p 15080:80 \
-p 15443:443 \
-p 46760:46760 \
--volume=/app/rancher/cni:/var/lib/cni \
--volume=/app/rancher/kubelet:/var/lib/kubelet \
--volume=/app/rancher/data:/var/lib/rancher \
--volume=/app/rancher/log:/var/log \
rancher/rancher:v2.6.8

# nginx
server {
    listen 443 ssl;

    ssl_certificate /app/nginx/conf.d/cert/rancher.kino.com.pem;
    ssl_certificate_key /app/nginx/conf.d/cert/rancher.kino.com.key;
    ssl_session_timeout  5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    server_name rancher.kino.com;

    location / {
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      #proxy_set_header Origin "";
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      proxy_connect_timeout 60;
      proxy_read_timeout 600;
      proxy_send_timeout 600;
      proxy_pass https://172.16.0.13:15443;
    }
}

server {
    listen 80;
    server_name rancher.kino.com;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```
# 七、prometheus & grafana
[官方文档](https://github.com/prometheus-operator/kube-prometheus/tree/v0.9.0)

```bash
# 删除无用服务
rm -rf manifests/prometheus-adapter-*.yaml
```
按官方文档安装即可。

设置模版: https://github.com/starsliao/Prometheus/tree/master/kubernetes








