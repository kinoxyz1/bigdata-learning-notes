
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
$ helm repo add stable http://mirror.azure.cn/kubernetes/charts/
"stable" has been added to your repositories
$ helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
"aliyun" has been added to your repositories
```
## 4.2 更新存储库
```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```
## 4.3 查看存储库
```bash
$ helm repo list
$ helm search repo stable
```
## 4.4 移除存储库
```bash
$ helm repo remove aliyun
"aliyun" has been removed from your repositories
```

# 五、使用 helm 快速部署应用
## 5.1 使用命令搜索应用
```bash
$ helm search repo weave
NAME              	CHART VERSION	APP VERSION	DESCRIPTION                                       
stable/weave-cloud	0.3.9        	1.4.0      	DEPRECATED - Weave Cloud is a add-on to Kuberne...
stable/weave-scope	1.1.12       	1.12.0     	DEPRECATED - A Helm chart for the Weave Scope c...
```
## 5.2 根据搜索内存选择安装
```bash

```