


# 一、官方文档
`flinkx` 已改名为 **[chunjun](https://github.com/DTStack/chunjun/tree/1.12_release)**

# 二、编译
```bash
# clone 项目
$ git clone https://github.com/DTStack/chunjun
# 切换分支
$ git checkout 1.12_release
# install 额外的驱动
$ cd bin && sh install_jars.sh && cd ..
# 编译
$ mvn clean package -DskipTests # 或者  sh build/build.sh
```

# 三、构建镜像
## 3.1 修改 flink 版本
```bash
$ cd flinkx-docker/docker
$ vim Dockerfile
将
wget -nv -O flink.tgz https://archive.apache.org/dist/flink/flink-1.12.2/flink-1.12.2-bin-scala_2.12.tgz; \
替换为
wget -nv -O flink.tgz https://archive.apache.org/dist/flink/flink-1.12.7/flink-1.12.7-bin-scala_2.12.tgz; \
```

## 3.2 cp flinkx-dist 
```bash
$ cp -r flink-dist ./flinkx-docker/docker
```

## 3.3 build image
```bash
$ cd ./flinkx-docker/docker
$ docker build -t ${image_name} .  ## 填自己的镜像名, 例如 <harbor私库>/<项目名>/<镜像名>:<版本号>
$ docker push <harbor私库>/<项目名>/<镜像名>:<版本号>
```

# 四、Application Mode Run
[官方文档](https://github.com/DTStack/chunjun/blob/1.12_release/docs/quickstart.md#kubernetes-application%E6%A8%A1%E5%BC%8F%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1)

官方文档中的: `-remotePluginPath` 需要替换成: `-remoteFlinkxDistDir`

完整的run命令:
```bash
bin/flinkx \
    -mode kubernetes-application \
    -jobType sync \
    -job flinkx-examples/json/stream/stream.json \
    -jobName kubernetes-job \
    -jobType sync \
    -flinkxDistDir flinkx-dist \
    -remoteFlinkxDistDir /opt/flinkx-dist \
    -pluginLoadMode classpath \
    -flinkLibDir $FLINK_HOME/lib \
    -flinkConfDir $FLINK_HOME/conf \
    -confProp "{\"kubernetes.config.file\":\"/root/.kube/config\",\"kubernetes.container.image\":\"flinkx:v1.0.0\",\"kubernetes.namespace\":\"flink112\",\"kubernetes.jobmanager.service-account\":\"flink\",\"kubernetes.cluster-id\":\"my-first-application-cluster\"}"
```

几个参数的解释:
- `-flinkxDistDir`: 在宿主机上编译出来的 flinkx-dist 目录, 可以是绝对路径。
- `-remoteFlinkxDistDir`: Dockerfile 中, 环境变量 `FLINKX_HOME` 的路径。
- `-confProp`: 可以在这指定 apache flink 指定的参数。






















