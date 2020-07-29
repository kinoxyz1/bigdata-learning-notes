



----

# 一、DockerFile 说明
## 1.1 DockerFile 是什么?
DickerFile 是用来构建 Docker 镜像的构建文件, 是由一系列命令和参数构成的脚本.

## 1.2 构建 DockerFile 的步骤
- 编写 DockerFile 文件
- `docker build`
- `docker run`

## 1.3 DockerFile 文件构成
[参考 CentOS 的 DockerFile 文件](https://github.com/CentOS/sig-cloud-instance-images/blob/b521221b5c8ac3ac88698e77941a2414ce6e778d/docker/Dockerfile
)

# 二、DockerFile 构建过程解析
## 2.1 DockerFile 基础知识
1. 每条保留字指定都必须大写, 且后面要跟随至少一个参数;
2. 指令按照从上到下, 顺序执行;
3. `#` 表示注释;
4. 每条指令都会创建一个新的镜像层, 并对镜像进行提交.

## 2.2 Docker 执行 DockerFile 的大致流程
1. docker 从基础镜像运行一个容器
2. 执行一条指令并对容器做出修改
3. 执行类似 `docker commit` 的操作提交一个新的镜像层
4. docker 再基于刚提交的镜像运行一个新容器
5. 执行 DockerFile 中的下一条指令直到所有指令都执行完成

## 2.3 小结
从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，
*  DockerFile 是软件的原材料
*  Docker 镜像是软件的交付品
*  Docker 容器则可以认为是软件的运行态.

Dockerfile面向开发, Docker镜像成为交付标准, Docker容器则涉及部署与运维, 三者缺一不可, 合力充当Docker 体系的基石. 

![dockerfile小结.png](../../img/docker/dockerfile/dockerfile小结.png)

1. Dockerfile: 需要定义一个Dockerfile, Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道, 这时需要考虑如何设计namespace的权限控制)等等;
 
2. Docker镜像: 在用Dockerfile定义一个文件之后, docker build时会产生一个Docker镜像, 当运行 Docker镜像时, 会真正开始提供服务;
 
3. Docker容器: 容器是直接提供服务的

# 三、DockerFile 保留字
- `FROM`: 基础镜像, 当前新镜像是基于哪个镜像.
- `MAINTAINER`: 镜像维护者的姓名和邮箱地址.
- `RUN`: 容器构建时需要运行的命令. 
- `EXPOSE`: 当前容器对外暴露出的端口.
- `WORKDIR`: 指定在创建容器后, 终端默认登录进来的目录, 工作落脚点.
- `ENV`: 用来在构建镜像过程中设置环境变量.
- `ADD`: 将宿主机目录下的文件拷贝进镜像且 ADD 命令会自动处理 URL 和解压 tar 压缩包.
- `COPY`: 类似 ADD, 拷贝文件和目录到镜像中, 将从构建上下文目录中 <源路径>的文件/目录复制到新的一层的镜像内的<目标目录>位置.
  - `COPY src dest`
  - `COPY ["src", "dest"]`
- `VOLUME`: 容器数据卷, 用于数据保存和持久化工作.
- `CMD`: 指定一个容器启动时要运行的命令, DockerFile 中可以有多个 CMD 命令, 但只有最后一个生效, CMD会被 docker run 之后的参数替换.
   - shell 格式: CMD <命令>
   - exec 格式: CMD ["可执行文件", "参数1","参数2",....]
- `ENTRYPOINT`: 指定一个容器启动时要运行的命令, ENTRYPOINT 的目的和 CMD 一样, 都是在指定容器启动程序及参数.
- `ONBUILD`: 当构建一个被继承的 DockerFile 时运行, 父镜像在被子继承后, 父镜像的 ONBUILD 会被触发.

![dockerfile 关键字](../../img/docker/dockerfile/dockerfile关键字.png)


# 四、案例
准备三个案例仅供参考
## 4.1 Base 镜像(scratch)
Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的

```bash
# centos 镜像
# https://github.com/CentOS/sig-cloud-instance-images/blob/b521221b5c8ac3ac88698e77941a2414ce6e778d/docker/Dockerfile
FROM scratch <-----
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
```

## 4.2 自定义 mycentos 镜像
目的: 最终让我们的 mycentos 镜像登录默认在 /tmp 目录下, 具备 vim、ifconfig支持
### 4.2.1 编写

### 4.2.2 构建 

### 4.2.3 运行

### 4.2.4 列出镜像的变更历史




## 4.3 CMD/ENTRYPOINT 镜像案例


## 4.4 自定义 tomcat 镜像


# 五、总结