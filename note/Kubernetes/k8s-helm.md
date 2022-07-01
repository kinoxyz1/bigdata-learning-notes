
* [一、 helm 引入](#%E4%B8%80-helm-%E5%BC%95%E5%85%A5)
* [二、概述](#%E4%BA%8C%E6%A6%82%E8%BF%B0)
* [三、helm 下载安装](#%E4%B8%89helm-%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85)
* [四、配置 helm 仓库](#%E5%9B%9B%E9%85%8D%E7%BD%AE-helm-%E4%BB%93%E5%BA%93)
  * [4\.1 添加存储库](#41-%E6%B7%BB%E5%8A%A0%E5%AD%98%E5%82%A8%E5%BA%93)
  * [4\.2 更新存储库](#42-%E6%9B%B4%E6%96%B0%E5%AD%98%E5%82%A8%E5%BA%93)
  * [4\.3 查看存储库](#43-%E6%9F%A5%E7%9C%8B%E5%AD%98%E5%82%A8%E5%BA%93)
  * [4\.4 移除存储库](#44-%E7%A7%BB%E9%99%A4%E5%AD%98%E5%82%A8%E5%BA%93)
* [五、使用 helm 快速部署应用](#%E4%BA%94%E4%BD%BF%E7%94%A8-helm-%E5%BF%AB%E9%80%9F%E9%83%A8%E7%BD%B2%E5%BA%94%E7%94%A8)
  * [5\.1 使用命令搜索应用](#51-%E4%BD%BF%E7%94%A8%E5%91%BD%E4%BB%A4%E6%90%9C%E7%B4%A2%E5%BA%94%E7%94%A8)
  * [5\.2 根据搜索内存选择安装](#52-%E6%A0%B9%E6%8D%AE%E6%90%9C%E7%B4%A2%E5%86%85%E5%AD%98%E9%80%89%E6%8B%A9%E5%AE%89%E8%A3%85)

---
# 一、 helm 引入
k8s 上的应用对象, 都是有特定的资源描述组成, 包括 Deployment, Service等. 它们都保存各自文件中或者集中写到一个配置文件, 然后执行 `kubectl apply -f <yaml文件>` 部署. 

如果应用只由一个或者几个这样的服务组成, 上面部署方式足够了, 而对于一个复杂的应用, 会有很多类似上面的资源描述文件, 例如微服务架构应用 组成应用的服务可能多个几十个. 如果有更新或者回滚应用的需求, 可能就要修改和维护所涉及到的大量资源文件, 而这种组织和管理应用的方式就显得力不从心了. 且由于缺少对发布过的应用版本管理和控制, 是 k8s上的应用维护和更新等面临诸多的挑战, 主要面临的问题如下:
1. 如何将这些服务作为一个整体管理
2. 这些资源文件如何高效复用
3. 不支持应用级别的版本管理


# 二、概述
helm 是一个 k8s 的包管理工具, 就像 Linux 下的包管理器, 如 yum/apt等, 可以很方便的将之前打包好的 yaml 文件部署到 k8s上

helm 有三个组成:
- helm: 一个命令行客户端工具, 主要用于 k8s 应用 chart 的创建、打包、发布、管理
- Chart: 应用描述, 一系列用于描述 k8s 资源相关的文件集合
- Release: 基于 Chart 的部署实体, 一个 Chart 被 helm 运行后将会生成对应的一个 热了; 将在 k8s 中创建出真实运行的资源对象


# 三、helm 下载安装
Helm 客户端下载地址: https://github.com/helm/helm/releases

```bash
$ tar zxvf helm-v3.0.0-linux-amd64.tar.gz 
linux-amd64/
linux-amd64/helm
linux-amd64/README.md
linux-amd64/LICENSE

[root@k8s-master opt]# cd linux-amd64/
[root@k8s-master linux-amd64]# ll
总用量 34768
-rwxr-xr-x 1 3434 3434 35586048 11月 13 2019 helm
-rw-r--r-- 1 3434 3434    11373 11月 13 2019 LICENSE
-rw-r--r-- 1 3434 3434     3248 11月 13 2019 README.md
[root@k8s-master linux-amd64]# mv helm /usr/bin/
```

# 四、配置 helm 仓库
helm 有三个仓库可供选择:
- 微软仓库(http://mirror.azure.cn/kubernetes/charts/)这个仓库推荐, 基本上官网有的 chart 这里都有。
- 阿里云仓库(https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts)
- 官方仓库(https://hub.kubeapps.com/charts/incubator)官方 chart 仓库, 国内有点不好使

## 4.1 添加存储库
```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```
## 4.2 更新存储库
```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```
## 4.3 查看存储库
```bash
$ helm repo list
NAME   	URL
stable 	http://mirror.azure.cn/kubernetes/charts/
bitnami	https://charts.bitnami.com/bitnami
$ helm search repo stable
...
```
## 4.4 移除存储库
```bash
$ helm repo remove stable
"stable" has been removed from your repositories
```

# 五、helm 常用命令
[更多命令查看官方文档](https://helm.sh/zh/docs/helm/helm/)

## 5.1 install
```bash
# --generate-name: 自己生成一个名字, 如果不想要自动生成, 可以自己写名字: helm install kino-mysql bitnami/mysql
$ helm install bitnami/mysql --generate-name
NAME: mysql-1612624192
LAST DEPLOYED: Sat Feb  6 16:09:56 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES: ...
```

Helm按照以下顺序安装资源：
- Namespace
- NetworkPolicy
- ResourceQuota
- LimitRange
- PodSecurityPolicy
- PodDisruptionBudget
- ServiceAccount
- Secret
- SecretList
- ConfigMap
- StorageClass
- PersistentVolume
- PersistentVolumeClaim
- CustomResourceDefinition
- ClusterRole
- ClusterRoleList
- ClusterRoleBinding
- ClusterRoleBindingList
- Role
- RoleList
- RoleBinding
- RoleBindingList
- Service
- DaemonSet
- Pod
- ReplicationController
- ReplicaSet
- Deployment
- HorizontalPodAutoscaler
- StatefulSet
- Job
- CronJob
- Ingress
- APIService



## 5.2 show
> 查看 chart 的基本信息
```bash
$ helm show chart bitnami/mysql
```
> 查看 chart 的全部信息
```bash
$ helm show all bitnami/mysql
```

## 5.3 list
> 查看哪些 chart 发布了
```bash
$ helm list
$ helm ls
```

## 5.4 uninstall
> 卸载
```bash
$ helm uninstall mysql-1612624192
```

## 5.5 status 
> 查看 Release 状态
```bash
$ helm status mysql-1612624192
Status: UNINSTALLED
...
```

## 5.6 search 
> helm search hub 从 Artifact Hub 中查找并列出 helm charts。 Artifact Hub中存放了大量不同的仓库。
```bash
$ helm search hub mysql
URL                                               	CHART VERSION	APP VERSION  	DESCRIPTION
https://artifacthub.io/packages/helm/cloudnativ...	5.0.1        	8.0.16       	Chart to create a Highly available MySQL cluster
https://artifacthub.io/packages/helm/stakater/m...	1.0.6        	             	mysql chart that runs on kubernetes
https://artifacthub.io/packages/helm/saber/mysql  	8.8.21       	8.0.27       	Chart to create a Highly available MySQL cluster
...
```
> helm search repo 从你添加（使用 helm repo add）到本地 helm 客户端中的仓库中进行查找。该命令基于本地数据进行搜索，无需连接互联网。
```bash
$ helm repo list
NAME   	URL
stable 	http://mirror.azure.cn/kubernetes/charts/
bitnami	https://charts.bitnami.com/bitnam

# helm 使用模糊匹配, bitnami 不写全也可以匹配
$ helm search repo bitna
NAME         	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/mysql	9.1.8        	8.0.29     	MySQL is a fast, reliable, scalable, and easy t...
...
```

## 5.7 修改默认配置
```bash
$ helm show values bitnami/wordpress
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
# global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName
#   storageClass: myStorageClass

## Bitnami WordPress image version
## ref: https://hub.docker.com/r/bitnami/wordpress/tags/
##
image:
  registry: docker.io
  repository: bitnami/wordpress
  tag: 5.6.0-debian-10-r35
  [..]
...

$ echo '{mariadb.auth.database: user0db, mariadb.auth.username: user0}' > values.yaml
$ helm install -f values.yaml bitnami/wordpress --generate-name
```
安装过程中有两种方式传递配置数据：
- --values (或 -f)：使用 YAML 文件覆盖配置。可以指定多次，优先使用最右边的文件。
- --set：通过命令行的方式对指定项进行覆盖。

如果同时使用两种方式，则 --set 中的值会被合并到 --values 中，但是 --set 中的值优先级更高。在--set 中覆盖的内容会被被保存在 ConfigMap 中。可以通过 helm get values <release-name> 来查看指定 release 中 --set 设置的值。也可以通过运行 helm upgrade 并指定 --reset-values 字段来清除 --set 中设置的值。

--set 的格式和限制

--set 选项使用0或多个 name/value 对。最简单的用法类似于：--set name=value，等价于如下 YAML 格式：
```yaml
name: value
```

多个值使用逗号分割，因此 --set a=b,c=d 的 YAML 表示是：
```yaml
a: b
c: d
```

支持更复杂的表达式。例如，--set outer.inner=value 被转换成了：
```yaml
outer:
  inner: value
```

列表使用花括号（{}）来表示。例如，--set name={a, b, c} 被转换成了：
```yaml
name:
  - a
  - b
  - c
```

从 2.5.0 版本开始，可以使用数组下标的语法来访问列表中的元素。例如 --set servers[0].port=80 就变成了：
```yaml
servers:
  - port: 80
```

多个值也可以通过这种方式来设置。--set servers[0].port=80,servers[0].host=example 变成了：
```yaml
servers:
  - port: 80
    host: example
```

如果需要在 --set 中使用特殊字符，你可以使用反斜线来进行转义；--set name=value1\,value2 就变成了：
```yaml
name: "value1,value2"
```

类似的，你也可以转义点序列（英文句号）。这可能会在 chart 使用 toYaml 函数来解析 annotations，labels，和 node selectors 时派上用场。--set nodeSelector."kubernetes\.io/role"=master 语法就变成了：
```yaml
nodeSelector:
  kubernetes.io/role: master
```

深层嵌套的数据结构可能会很难用 --set 表达。我们希望 Chart 的设计者们在设计 values.yaml 文件的格式时，考虑到 --set 的使用。（更多内容请查看 Values 文件）

helm install 命令可以从多个来源进行安装：

chart 的仓库（如上所述）
- 本地 chart 压缩包（helm install foo foo-0.1.1.tgz）
- 解压后的 chart 目录（helm install foo path/to/foo）
- 完整的 URL（helm install foo https://example.com/charts/foo-1.2.3.tgz）


# 六、自定义 Charts
[官方文档指南](https://helm.sh/zh/docs/chart_template_guide/getting_started/)

## 6.1 示例
```bash
$ helm create jzdata
$ ls -l
jzdata/
  Chart.yaml
  values.yaml
  charts/
  templates/
  ...
```
`templates/` 目录包括了模板文件。当Helm评估chart时，会通过模板渲染引擎将所有文件发送到templates/目录中。 然后收集模板的结果并发送给Kubernetes。

`values.yaml` 文件也导入到了模板。这个文件包含了chart的 默认值 。这些值会在用户执行helm install 或 helm upgrade时被覆盖。

`Chart.yaml` 文件包含了该chart的描述。你可以从模板中访问它。charts/目录 可以 包含其他的chart(称之为 子chart)。 指南稍后我们会看到当涉及模板渲染时这些是如何工作的。

```yaml
$ vim configmap.yaml
{{- if .Values.etlService.configmap }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "jzdata.fullname" . }}-etl-service
data:
  {{- range $key, $value := .Values.etlService.configmap }}
  {{ $key }}: {{ $value | quote}}
  {{- end}}
{{- end}}

$ vim deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "jzdata.fullname" . }}-web
spec:
  progressDeadlineSeconds: 600
  replicas: {{ .Values.jzWeb.replicas | default .Values.replicas }}
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: {{ include "jzdata.fullname" . }}-web
      {{/* - include "jzdata.selectorLabels" . | nindent 6 */}}
  strategy:
    type: {{ .Values.jzWeb.strategy.type | quote }}
    rollingUpdate:
      maxSurge: {{ .Values.jzWeb.strategy.rollingUpdate.maxSurge | quote }}
      maxUnavailable: {{ .Values.jzWeb.strategy.rollingUpdate.maxUnavailable | quote }}
  template:
    metadata:
      labels:
        app: {{ include "jzdata.fullname" . }}-web
    spec:
      containers:
        - env:
            - name: version
              value: {{ .Values.jzWeb.image.tag }}
            - name: TZ
              value: Asia/Shanghai
          envFrom:
            - configMapRef:
                name: {{ include "jzdata.fullname" . }}-web
          image: "{{ .Values.jzWeb.image.repository }}:{{ .Values.jzWeb.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.jzWeb.image.imagePullPolicy | default .Values.imagePullPolicy }}
          name: {{ include "jzdata.fullname" . }}-web
          ports:
            - containerPort: {{ .Values.jzWeb.containerPort }}
              name: jzdata
              protocol: TCP
          livenessProbe:
            failureThreshold: {{ .Values.jzWeb.livenessProbe.failureThreshold }}
            initialDelaySeconds: {{ .Values.jzWeb.livenessProbe.initialDelaySeconds }}
            periodSeconds: {{ .Values.jzWeb.livenessProbe.periodSeconds }}
            successThreshold: {{ .Values.jzWeb.livenessProbe.successThreshold }}
            timeoutSeconds: {{ .Values.jzWeb.livenessProbe.timeoutSeconds }}
            tcpSocket:
              port: jzdata
          readinessProbe:
            failureThreshold: {{ .Values.jzWeb.readinessProbe.failureThreshold }}
            initialDelaySeconds: {{ .Values.jzWeb.readinessProbe.initialDelaySeconds }}
            periodSeconds: {{ .Values.jzWeb.readinessProbe.periodSeconds }}
            successThreshold: {{ .Values.jzWeb.readinessProbe.successThreshold }}
            timeoutSeconds: {{ .Values.jzWeb.readinessProbe.timeoutSeconds }}
            tcpSocket:
              port: jzdata
          resources:
                  {{- toYaml .Values.jzWeb.resources | nindent 10 }}
          volumeMounts:
            - mountPath: /opt/log/stash
              name: log-volume
            - mountPath: /var/tmp
              name: tmp-volume
            - mountPath: /tmp
              name: tmp-volume
      dnsPolicy: ClusterFirst
      imagePullSecrets:
        - name: jz-registry
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - hostPath:
            path: /opt/log/stash
            type: DirectoryOrCreate
          name: log-volume
        - hostPath:
            path: /opt/tmp
            type: DirectoryOrCreate
          name: tmp-volume
            {{- with .Values.nodeSelector }}
      nodeSelector:
              {{- toYaml . | nindent 8 }}
            {{- end }}
            {{- with .Values.affinity }}
      affinity:
              {{- toYaml . | nindent 8 }}
            {{- end }}
            {{- with .Values.tolerations }}
      tolerations:
              {{- toYaml . | nindent 8 }}
            {{- end }}

$ vim svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "jzdata.fullname" . }}-web
spec:
  ports:
    - name: jzdata
      port: 19006
      protocol: TCP
      targetPort: 18002
  selector:
    app: {{ include "jzdata.fullname" . }}-web
  type: ClusterIP

$ vim ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: datacenter-ds
  labels:
    app.kubernetes.io/name: {{ include "jzdata.fullname" . }}-web
    {{- include "jzdata.common.labels" . | nindent 4 }}
spec:
  ingressClassName: nginx
  rules:
    - host: {{ .Values.ingress.host}}
      http:
        paths:
          - backend:
              service:
                name: {{ include "jzdata.fullname" . }}-dolphinscheduler-api
                port:
                  name: api-port
            path: {{ .Values.ingress.path }}
            pathType: Prefix

```





