
* [一、Flink 执行环境](#%E4%B8%80flink-%E6%89%A7%E8%A1%8C%E7%8E%AF%E5%A2%83)
  * [1\.1 Environment](#11-environment)
    * [1\.1\.1 getExecutionEnvironment](#111-getexecutionenvironment)
    * [1\.1\.2 createLocalEnvironment](#112-createlocalenvironment)
    * [1\.1\.3 createRemoteEnvironment](#113-createremoteenvironment)

---

Flink 流处理过程

![Flink运行流程图](../../img/flink/Source(1)/Flink运行流程图.png)


# 一、Flink 执行环境
## 1.1 `Environment`
### 1.1.1 `getExecutionEnvironment`
  创建一个执行环境, 表示当前执行程序的上下文. 如果程序是独立调用的, 则此方法返回本地执行环境; 如果从命令行客户端调用程序提交到集群, 则此方法返回此集群的执行环境, 也就是说, `getExecutionEnvironment` 会根据查询运行的方式决定返回什么样的运行环境, 是最常用的一种创建执行环境的方式.
  
```scala
val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
```

如果没有设置并行度, 会以 flink-conf.yaml 中的配置为准, 默认是 1.

### 1.1.2 `createLocalEnvironment`
  返回本地执行环境, 需要在调用时指定默认的并行度
```scala
val env = ExecutionEnvironment.createLocalEnvironment(1)
```

### 1.1.3 `createRemoteEnvironment`
  返回集群执行环境, 将 jar 提交到远程服务器. 需要在调用时指定 JobManager 的 IP 和 端口号, 并且要指定集群中运行的 jar 包.
```scala
val env = ExecutionEnvironment.createRemoteEnvironment("jobmanage-hostname", 6123,"YOURPATH//wordcount.jar")
```