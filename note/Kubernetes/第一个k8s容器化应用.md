





---



# 一、运行 Nginx 容器

## 1.1 前提

kubernetes 运行容器的前提是容器在中央仓库或者本地docker hub 中存在

## 1.2 直接运行（不推荐）

   ```bash
   $ kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...]
   ```

   示例:

   ```bash
   # 运行 Nginx 容器
   [root@k8s-master k8s]# kubectl run nginx --image=nginx:1.7.9
   # 运行 Nginx 容器，并暴露端口
   [root@k8s-master k8s]# kubectl run nginx --image=nginx:1.7.9 --port=80
   # 运行 Nginx 容器，并设置容器中的环境变量
   [root@k8s-master k8s]# kubectl run nginx --image=nginx:1.7.9 --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default" --port=80
   # 启动 Nginx 容器，并指定副本数
   [root@k8s-master k8s]# kubectl run nginx --image=nginx:1.7.9 --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default" --port=80 --replicas=3
   # 运行 Dry 打印相应的API 对象而不创建
   [root@k8s-master k8s]# kubectl run nginx --image=nginx:1.7.9 --dry-run
   ```

   直接运行容器的其他参数可以参考官方文档: [kubectl run](http://docs.kubernetes.org.cn/468.html#kubectl_run)

## 1.3 编写yaml运行（推荐）

   使用 kubernetes 官方推荐的方式是编辑(将容器的定义、参数、配置等)yaml文件，通过`kubectl create -f my_yaml.yaml`运行

   

   这样做的好处是，会有一个文件详细记录 kubernetes 运行了什么

   ```bash
   [root@k8s-master k8s]# vim nginx.yaml
   apiVersion: apps/v1
   kind: Deployment
   # ① 描述当前 Deployment 相关的信息
   metadata:
     # ② 当前 Deployment 的名称是: nginx-deployment
     name: nginx-deployment
     # ③ 当前 Deployment 和管理的pod 都属于 kino 这个namespace
     namespace: kino
   spec: 
     # ⑪ Deployment 控制器可以通过 匹配template 中的 labels 来过滤被控制的对象
     selector:
       # ⑫ 选择匹配 label 是 app: nginx 的 pod 进行管理
       matchLabels:
         app1: nginx
     # ⑬ 管理 label 是 app: nginx 的 pod 必须是一个副本
     replicas: 1
     # ④ 定义 创建的 pod 的细节
     template:
       # ⑤ API 对象的表示, 即: 元数据, 是我们从k8s中找到这个对象的主要依据
       metadata:
         # ⑥ 元数据的名称是 key: value 组成的
         labels:
           app1: nginx
       spec: 
         # ⑦ 镜像相关的信息
         containers:
           # ⑧ 镜像的名字
         - name: nginx
           # ⑨ 镜像是 nginx1.7.9版本的
           image: nginx:1.7.9
           # ⑩ 指定当前镜像的 端口号
           ports:
           - containerPort: 80
   ```

   上面这个yaml 文件例子中，对应到 kubernetes 就是一个 API 对象，yaml 中的 Kind 字段描述了这个 API对象的类型是 Deployment；

   Deployment就是一个多副本应用(多副本pod)的对象，Deployment 会负责在 Pod定义发生变化时，对每个副本进行滚动更新，Deployment会保证该pod的副本数是2(yaml中定义了pod的副本数（spec.replicas）是2)；

   在上面的yaml中，定义了pod的模板(spec.template)，这个pod 只有一个容器，这个容器的镜像(spec.containers.image)就是 nginx:1.7.9，这个容器监听端口号(spec.containers.ports.containerPort)是 80，

