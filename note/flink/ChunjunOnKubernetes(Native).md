

# Chunjun On Kubernetes(Native)

> 该文档记录 chunjun master branch 在 Kubernetes Native Mode 的安装、部署、运行相关操作.

## 1. 版本说明

> 你需要有一个 kubernetes 集群

| Chunjun version | Flink version | Kubernetes version |
| --------------- | ------------- | ------------------ |
| master(1.16)    | 1.16.1        | >= v1.20           |

## 2. flink 源码编译

clone flink sourcecode

```bash
$ git clone https://github.com/apache/flink.git
$ cd flink
$ git checkout -b release-1.16.1 release-1.16.1  # 切换分支
# 修改配置
$ vim flink/flink-dist/src/main/flink-bin/bin/flink-console.sh
将以下配置
(kubernetes-session)
	  CLASS_TO_RUN=org.apache.flink.kubernetes.entrypoint.KubernetesSessionClusterEntrypoint
;;
改成
(kubernetes-session)
    CLASS_TO_RUN=org.apache.flink.kubernetes.entrypoint.KubernetesSessionClusterEntrypoint
    # if CHUNJUN_HOME exists, added to classpath
    if [ -d $CHUNJUN_HOME ]; then
      plugin_jars=`find $CHUNJUN_HOME -type f -maxdepth 4 | grep 'chunjun-' | grep '.jar' | sort -n`
      CHUNJUN_CLASSPATH=${plugin_jars//[[:space:]]/\:}
      FLINK_TM_CLASSPATH=$FLINK_TM_CLASSPATH:$CHUNJUN_CLASSPATH
    fi
;;
$ mvn -T2C clean install -DskipTests -Dfast      # 编译源码
$ cd flink/flink-dist/target/flink-1.16.1-bin
$ tar -zcvf flink-1.16.1-bin-scala_2.12.tgz flink-1.16.1  # 打 tar 包
```

## 3. Chunjun 源码编译

clone chunjun sourcecode

```bash
$ git clone https://github.com/DTStack/chunjun.git
$ cd chunjun
$ mvn clean package -Dmaven.test.skip=true     # 编译源码
$ tar -zcvf chunjun-dist.tgz chunjun-dist      # 打 tar 包
```

## 4. flink build image

### prepare

因为我们需要改一个flink的配置，所以我们会自己打镜像.

> docker pull apache/flink:1.12.7

手动构建镜像, 可以参考[官方提供的示例](https://github.com/apache/flink-docker/tree/master/1.16/scala_2.12-java8-ubuntu), 这里展示我本地打镜像的目录结构(可以直接贴官方的Dockerfile、docker-entrypoint.sh)

```bash
$ tree .
├── build.sh
├── chunjun-dist.tgz
├── conf
│   ├── log4j-cli.properties
│   ├── log4j-console.properties
│   ├── log4j.properties
│   ├── log4j-session.properties
│   ├── logback-console.xml
│   ├── logback-session.xml
│   └── logback.xml
├── docker-entrypoint.sh
├── Dockerfile
├── flink-1.16.1-bin-scala_2.12.tgz          # flink 编译出来的安装包
├── gosu-amd64
├── README.md
└── usrlib     # 根据自己的需要自行拷贝
    ├── assertj-core-3.23.1.jar
    ├── commons-cli-1.5.0.jar
    ├── flink-metrics-prometheus-1.16.1.jar
    ├── flink-shaded-guava-30.1.1-jre-16.1.jar
    ├── guava-29.0-jre.jar
    ├── httpclient-4.5.13.jar
    ├── jackson-annotations-2.12.3.jar
    ├── jackson-core-2.12.3.jar
    ├── jackson-databind-2.12.3.jar
    ├── jackson-datatype-jsr310-2.12.3.jar
    ├── jackson-module-jaxb-annotations-2.12.2.jar
    ├── log4j-1.2-api-2.17.1.jar
    ├── log4j-api-2.17.1.jar
    ├── log4j-core-2.17.1.jar
    ├── log4j-slf4j-impl-2.17.1.jar
    ├── logback-classic-1.2.10.jar
    ├── logback-core-1.2.10.jar
    └── logstash-logback-encoder-6.6.jar
```

> 上面目录中的文件会在下方列出,并且会标识哪些可以直接使用官方的

build.sh(复制)

```bash
#!/bin/bash
VERSION=$1
docker rmi flink:$VERSION
docker build --no-cache -t flink:$VERSION .
docker push flink:$VERSION
```

docker-entrypoint.sh(可以使用官方的)

```bash
#!/usr/bin/env bash
###############################################################################
#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance
#  with the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
# limitations under the License.
###############################################################################

COMMAND_STANDALONE="standalone-job"
# Deprecated, should be remove in Flink release 1.13
COMMAND_NATIVE_KUBERNETES="native-k8s"
COMMAND_HISTORY_SERVER="history-server"

# If unspecified, the hostname of the container is taken as the JobManager address
JOB_MANAGER_RPC_ADDRESS=${JOB_MANAGER_RPC_ADDRESS:-$(hostname -f)}
CONF_FILE="${FLINK_HOME}/conf/flink-conf.yaml"

drop_privs_cmd() {
    if [ $(id -u) != 0 ]; then
        # Don't need to drop privs if EUID != 0
        return
    elif [ -x /sbin/su-exec ]; then
        # Alpine
        echo su-exec flink
    else
        # Others
        echo gosu flink
    fi
}

copy_plugins_if_required() {
  if [ -z "$ENABLE_BUILT_IN_PLUGINS" ]; then
    return 0
  fi

  echo "Enabling required built-in plugins"
  for target_plugin in $(echo "$ENABLE_BUILT_IN_PLUGINS" | tr ';' ' '); do
    echo "Linking ${target_plugin} to plugin directory"
    plugin_name=${target_plugin%.jar}

    mkdir -p "${FLINK_HOME}/plugins/${plugin_name}"
    if [ ! -e "${FLINK_HOME}/opt/${target_plugin}" ]; then
      echo "Plugin ${target_plugin} does not exist. Exiting."
      exit 1
    else
      ln -fs "${FLINK_HOME}/opt/${target_plugin}" "${FLINK_HOME}/plugins/${plugin_name}"
      echo "Successfully enabled ${target_plugin}"
    fi
  done
}

set_config_option() {
  local option=$1
  local value=$2

  # escape periods for usage in regular expressions
  local escaped_option=$(echo ${option} | sed -e "s/\./\\\./g")

  # either override an existing entry, or append a new one
  if grep -E "^${escaped_option}:.*" "${CONF_FILE}" > /dev/null; then
        sed -i -e "s/${escaped_option}:.*/$option: $value/g" "${CONF_FILE}"
  else
        echo "${option}: ${value}" >> "${CONF_FILE}"
  fi
}

prepare_configuration() {
    set_config_option jobmanager.rpc.address ${JOB_MANAGER_RPC_ADDRESS}
    set_config_option blob.server.port 6124
    set_config_option query.server.port 6125

    TASK_MANAGER_NUMBER_OF_TASK_SLOTS=${TASK_MANAGER_NUMBER_OF_TASK_SLOTS:-1}
    set_config_option taskmanager.numberOfTaskSlots ${TASK_MANAGER_NUMBER_OF_TASK_SLOTS}
    set_config_option kubernetes.flink.log.dir /opt/flink/log
    set_config_option env.java.opts.jobmanager "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005"
    set_config_option env.java.opts.taskmanager "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5006"
    set_config_option env.java.opts.client "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5008"

    if [ -n "${FLINK_PROPERTIES}" ]; then
        echo "${FLINK_PROPERTIES}" >> "${CONF_FILE}"
    fi
    envsubst < "${CONF_FILE}" > "${CONF_FILE}.tmp" && mv "${CONF_FILE}.tmp" "${CONF_FILE}"
}

maybe_enable_jemalloc() {
    if [ "${DISABLE_JEMALLOC:-false}" == "false" ]; then
        export LD_PRELOAD=$LD_PRELOAD:/usr/lib/x86_64-linux-gnu/libjemalloc.so
    fi
}

set_host_aliases() {
    if [[ $KUBERNETES_HOST_ALIASES != "" ]]; then
        host_msg="\n----------set host-----------\n $KUBERNETES_HOST_ALIASES \n"
        echo -e $host_msg
        KUBERNETES_HOST_ALIASES=${KUBERNETES_HOST_ALIASES//;/\\n}
        echo -e "$KUBERNETES_HOST_ALIASES" >> /etc/hosts
        cat /etc/hosts
    fi
}


cp $FLINK_HOME/conf/* $FLINK_HOME/cust-conf/
export FLINK_CONF_DIR=$FLINK_HOME/cust-conf

maybe_enable_jemalloc

copy_plugins_if_required

prepare_configuration

set_host_aliases

#hadoop_jars=`find $HADOOP_LIB -type f -maxdepth 4 | grep '.jar'`
#HADOOP_CLASSPATH=${hadoop_jars//[[:space:]]/\:}

args=("$@")
if [ "$1" = "help" ]; then
    printf "Usage: $(basename "$0") (jobmanager|${COMMAND_STANDALONE}|taskmanager|${COMMAND_HISTORY_SERVER})\n"
    printf "    Or $(basename "$0") help\n\n"
    printf "By default, Flink image adopts jemalloc as default memory allocator. This behavior can be disabled by setting the 'DISABLE_JEMALLOC' environment variable to 'true'.\n"
    exit 0
elif [ "$1" = "jobmanager" ]; then
    args=("${args[@]:1}")

    echo "Starting Job Manager"

    exec $(drop_privs_cmd) "$FLINK_HOME/bin/jobmanager.sh" start-foreground "${args[@]}"
elif [ "$1" = ${COMMAND_STANDALONE} ]; then
    args=("${args[@]:1}")

    echo "Starting Job Manager"

    exec $(drop_privs_cmd) "$FLINK_HOME/bin/standalone-job.sh" start-foreground "${args[@]}"
elif [ "$1" = ${COMMAND_HISTORY_SERVER} ]; then
    args=("${args[@]:1}")

    echo "Starting History Server"

    exec $(drop_privs_cmd) "$FLINK_HOME/bin/historyserver.sh" start-foreground "${args[@]}"
elif [ "$1" = "taskmanager" ]; then
    args=("${args[@]:1}")

    echo "Starting Task Manager"

    exec $(drop_privs_cmd) "$FLINK_HOME/bin/taskmanager.sh" start-foreground "${args[@]}"
elif [ "$1" = "$COMMAND_NATIVE_KUBERNETES" ]; then
    args=("${args[@]:1}")

    export _FLINK_HOME_DETERMINED=true
    . $FLINK_HOME/bin/config.sh
    export FLINK_CLASSPATH="/opt/flink/lib/log4j-over-slf4j-1.8.0-beta2.jar:/opt/flink/lib/logback-classic-1.2.10.jar:/opt/flink/lib/logback-core-1.2.10.jar:/opt/flink/lib/logstash-logback-encoder-6.6.jar:`constructFlinkClassPath`"
    echo "###Set FlINK_CONF_DIR:$FLINK_CONF_DIR"

    # Start commands for jobmanager and taskmanager are generated by Flink internally.
    echo "###FLINK_CLASSPATH:$FLINK_CLASSPATH"
    start_command=`echo "${args[@]}" | sed -e 's/org.apache.flink.kubernetes.entrypoint.Kubernetes.*ClusterEntrypoint/-Dlog.file=\/opt\/flink\/log\/${HOSTNAME}.log -Dlogback.configurationFile=file:\/opt\/flink\/cust-conf\/logback-console.xml -Dlogback.configurationFile=file:\/opt\/flink\/cust-conf\/logback.xml -Dlogback.configurationFile=file:\/opt\/flink\/cust-conf\/logback-session.xml & /g'`

    if [[ ${args[@]} == *KubernetesTaskExecutorRunner* ]]
    then
        start_command=`echo "${args[@]}" | sed -e 's/org.apache.flink.kubernetes.taskmanager.KubernetesTaskExecutorRunner/-Dlog.file=\/opt\/flink\/log\/${HOSTNAME}.log -Dlogback.configurationFile=file:\/opt\/flink\/cust-conf\/logback-console.xml -Dlogback.configurationFile=file:\/opt\/flink\/cust-conf\/logback.xml -Dlogback.configurationFile=file:\/opt\/flink\/cust-conf\/logback-session.xml & /g'`
    fi

    echo "###start td-agent"
    nohup td-agent  2>&1 &

    echo "### run command:${start_command}"
    exec $(drop_privs_cmd) bash -c "${start_command}"
fi

args=("${args[@]}")

# Set the Flink related environments
export _FLINK_HOME_DETERMINED=true
. $FLINK_HOME/bin/config.sh
export FLINK_CLASSPATH="/opt/flink/lib/log4j-over-slf4j-1.8.0-beta2.jar:/opt/flink/lib/logback-classic-1.2.10.jar:/opt/flink/lib/logback-core-1.2.10.jar:/opt/flink/lib/logstash-logback-encoder-6.6.jar:`constructFlinkClassPath`:$INTERNAL_HADOOP_CLASSPATHS"

# Running command in pass-through mode
exec $(drop_privs_cmd) "${args[@]}"
```

Dockerfile(复制)

```bash
## 第一阶段
FROM centos:7 as prepare

WORKDIR /opt

COPY flink-1.16.1-bin-scala_2.12.tgz .
COPY chunjun-dist.tgz .
COPY usrlib ./usrlib
COPY conf ./conf
COPY gosu-amd64 .

RUN groupadd --system --gid=9999 flink && \
    useradd --system --home-dir /app/flink --uid=9999 --gid=flink flink

RUN set -ex; \
  mkdir /app; \
  tar -zxf flink-1.16.1-bin-scala_2.12.tgz -C /app; \
  cd /app && mv flink-1.16.1 flink; \
  mkdir -p flink/cust-conf; \
  mkdir -p flink/plugins/s3-fs-hadoop; \
  cp -r flink/opt/flink-s3-fs-hadoop-1.16.1.jar flink/plugins/s3-fs-hadoop; \
  mv /opt/usrlib/* flink/lib/; \
  mv /opt/conf/log* flink/cust-conf; \
  tar -zxf /opt/chunjun-dist.tgz -C /app; \
  chmod +x /opt/gosu-amd64; \
  chown -R flink: /app/*

## 第二阶段
FROM openjdk:8-jdk

COPY --from=prepare /app /opt
COPY --from=prepare /opt/gosu-amd64 /usr/local/bin/gosu

RUN { \
        echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free"; \
        echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free"; \
        echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free"; \
        echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free"; \
} > /etc/apt/sources.list

RUN set -ex; \
  curl https://packages.treasuredata.com/GPG-KEY-td-agent | apt-key add - && \
  echo "deb http://packages.treasuredata.com/3/ubuntu/bionic/ bionic contrib" > /etc/apt/sources.list.d/treasure-data.list && \
  apt-get update && \
  apt-get -y install gpg libsnappy1v5 gettext-base libjemalloc-dev; \
  rm -rf /var/lib/apt/lists/*

# Grab gosu for easy step-down from root
ENV FLINK_HOME=/opt/flink
ENV FLINK_CONF_DIR=/opt/flink/cust-conf
ENV CHUNJUN_HOME=/opt/chunjun-dist
ENV PATH=$FLINK_HOME/bin:$PATH

WORKDIR $FLINK_HOME

RUN groupadd --system --gid=9999 flink && \
    useradd --system --home-dir $FLINK_HOME --uid=9999 --gid=flink flink

# Install Flink
RUN set -ex; \
  gosu nobody true

# Configure container
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 6123 8081
CMD ["help"]
```

### build

```bash
sh build.sh 1.16.1
```

> 镜像需要推送到自己的私服，或者将镜像拷贝到k8s的每一台节点(此方法需要设置镜像拉取策略为: IfNotPresent)

## 5. 运行任务

### application

```bash
./bin/chunjun-kubernetes-application.sh -job /Users/kino/works/opensource/chunjun/chunjun-examples/json/stream/stream.json \
-jobName kubernetes-job \
-chunjunDistDir /opt/chunjun-dist \
-flinkConfDir /opt/flink/conf \
-flinkLibDir  /opt/flink/lib \
-confProp "{\"kubernetes.config.file\":\"/root/.kube/conf\",\"kubernetes.container.image\":\"flink:1.16.1\",\"kubernetes.namespace\":\"flink\",\"kubernetes.container.image.pull-secrets\":\"kino-registry\",\"kubernetes.cluster-id\":\"flink-kubernetes-app-job\",\"kubernetes.container.image.pull-policy\":\"Always\",\"metrics.reporter.promgateway.factory.class\":\"org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporterFactory\",\"metrics.reporter.promgateway.hostUrl\":\"http://10.10.10.10:9200\",\"metrics.reporter.promgateway.jobName\":\"myJob\",\"metrics.reporter.promgateway.randomJobNameSuffix\":\"true\",\"metrics.reporter.promgateway.deleteOnShutdown\":\"true\",\"metrics.reporter.promgateway.interval\":\"15000\"}"
```

参数说明

> kubernetes.config.file: k8s 集群的配置文件, 安装好k8s文件后, 在k8s机器的 /root/.kube/conf 
>
> kubernetes.container.image: flink 镜像名
>
> kubernetes.namespace: 运行在k8s的哪个名称空间下
>
> kubernetes.container.image.pull-secrets: 私服拉取的 secrets
>
> kubernetes.cluster-id: 启动的任务名称
>
> kubernetes.container.image.pull-policy: 镜像的拉取策略
>
> metrics.reporter.promgateway.factory.class: prometheus 类路径
>
> metrics.reporter.promgateway.hostUrl: prometheus 地址

关于 Kubernetes 更多参数选项, 可以查看[官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.16/deployment/config.html#kubernetes-namespace)

### session

```bash
$ cd $FLINK_HOME
$ ./bin/kubernetes-session.sh \
  -Dkubernetes.cluster-id=flink-kubernetes-session-job \
  -Dkubernetes.namespace=random-insert-00 \
  -Dkubernetes.container.image=flink:1.16.1 \
  -Dkubernetes.jobmanager.cpu=1 \
  -Dkubernetes.taskmanager.cpu=1 \
  -Dkubernetes.taskmanager.slots=1 \
  -Dkubernetes.rest-service.exposed.type=NodePort \
  -Dkubernetes.taskmanager.memory.process.size=1024m \
  -Dkubernetes.jobmanager.memory.process.size=1024m \
  -Dkubernetes.container.image.pull-policy=Always \
  -Dkubernetes.container.image.pull-secrets=kino-registry
```

```bash
$ cd $CHUNJUN_HOME
$ ./bin/chunjun-kubernetes-session.sh -job /Users/kino/works/opensource/chunjun/chunjun-examples/json/stream/stream.json \
-jobName kubernetes-job \
-chunjunDistDir /opt/chunjun-dist \
-flinkConfDir /opt/flink/conf \
-flinkLibDir  /opt/flink/lib \
-confProp "{\"kubernetes.config.file\":\"/root/.kube/conf\",\"kubernetes.container.image\":\"flink:1.16.1\",\"kubernetes.namespace\":\"flink\",\"kubernetes.container.image.pull-secrets\":\"kino-registry\",\"kubernetes.cluster-id\":\"flink-kubernetes-session-job\",\"kubernetes.container.image.pull-policy\":\"Always\",\"metrics.reporter.promgateway.factory.class\":\"org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporterFactory\",\"metrics.reporter.promgateway.hostUrl\":\"http://10.10.10.10:9200\",\"metrics.reporter.promgateway.jobName\":\"myJob\",\"metrics.reporter.promgateway.randomJobNameSuffix\":\"true\",\"metrics.reporter.promgateway.deleteOnShutdown\":\"true\",\"metrics.reporter.promgateway.interval\":\"15000\"}"
```

