




# 一、Jenkins 安装
使用 docker 直接部署
```bash
$ docker run \
  --name=jenkins \
  --user=root \
  --volume=/root/.ssh:/root/.ssh/ \
  --volume=/var/run/docker.sock:/var/run/docker.sock \
  --volume=/usr/bin/docker:/usr/bin/docker \
  --volume=/usr/bin/kubectl:/usr/bin/kubectl \
  --volume=/root/.docker:/root/.docker \
  --volume=/etc/docker:/etc/docker \
  --volume=/app/jenkins/jenkins_home:/var/jenkins_home \
  --volume=/var/jenkins_home \
  -p 17000:8080 \
  -p 50000:50000 \
  --restart=always \
  --detach=true \
  jenkins/jenkins:lts
```
启动完成后, 在浏览器中直接访问: ip:8080

激活秘钥直接查看 Jenkins 容器的 logs
```bash
$ docker logs 5d22e56c028c
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

e872e6d0f7064308ac5f41b7bf719ee2

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
```

选择 "安装推荐的插件" 即可

> 如果插件安装失败, 可以选择更换镜像源, 这里提供清华的镜像源供选择, https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/
```bash
$ cd /app/jenkins/jenkins_home/
$ vim hudson.model.UpdateCenter.xml
### 将 <url> 标签里面的内容替换掉
```


# 二、Idea 安装 gitee 插件
file  -> setting -> plugins -> 搜索 gitee 直接安装即可

创建 springboot 项目上传 gitee


# 三、Jenkinsfile 和 Dockerfile 配置
Jenkinsfile
```bash
// 写流水线的脚本
pipeline {
    // 全部的 CI/CD 流程都在这里定义

    // 任何一个代理可用就可以执行
    // agent none  // 以后所有的stage 都必须指定自己的 agent
    agent any

    // 定义一些环境信息, key = "value" 格式
    environment {
      hello = "1111"
      world = "2222"
      WORKSPACE = "${WORKSPACE}"
    }

    // 定义流水线的加工流程
    stages {
        // 流水线的全部阶段
        stage('环境检查') {
            steps {
                sh 'printenv'
                echo '正在检测基本信息'
                sh 'java -version'
                sh 'git --version'
                sh 'docker version'
                sh 'pwd && ls -lah'
                sh 'echo ${hello}'
                sh 'echo $world'
            }
        }

        // 1. 编译
        stage('代码编译') {
            // 自定义代理, Jenkins 不配置任何代理的情况下, 仅适用docker 兼容所有场景
            agent {
                docker {
                    // 这种方式也可以实现 maven 的阿里云加速
                    // 但是这种方式是从宿主机拷贝 settings.xml 文件到镜像中, 移植性不好
                    // 推荐在 /var/jenkins_home 放入 settings.xml 文件直接引用
                    // args '-v /a/settings.xml:/app/settings.xml '
                    image 'maven:3-alpine'
                    args '-v /var/jenkins_home/appconfig/maven/.m2:/root/.m2'
                }
            }
            steps {
                // 要做的所有事情
                // git 下载的 代码目录
                sh 'pwd && ls -lah'
                sh 'mvn -v'
                // 打jar包, 默认是从 maven 中央仓库下载
                // 如何使用 阿里云 镜像源
                sh 'cd ${WORKSPACE} && mvn clean package -s "/var/jenkins_home/appconig/maven/settings.xml" -Dmaven.test.skip=true'
                sh 'pwd && ls -lah target/'
                // jar包推送给 maven 仓库或私有仓库
                // docker pull ...
            }
        }

        // 2. 测试
        stage('代码测试') {
            steps {
                // 要做的所有事情
                echo "测试"
            }
        }

        // 3. 打包
        stage('代码打包') {
            steps {
                // 要做的所有事情
                echo "打包"
                // 生成镜像, 检查 docker 命令是否能运行
                sh 'docker version'
                sh 'pwd && ls -lah target/'
                sh 'docker build -t java-devops-demo .'
            }
        }

        // 4. 部署
        stage('代码部署') {
            steps {
                // 要做的所有事情
                echo "部署"
                sh 'docker run -itd --name java_dev_demo -p 8888:8080 java-devops-demo'
            }
        }
    }

}
```

Dockerfile
```bash
FROM openjdk:8-jre-alpine
LABEL maintainer="kino@qq.com"

COPY target/*.jar /app.jar
RUN  apk add -U tzdata; \
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
echo 'Asia/Shanghai' >/etc/timezone; \
touch /app.jar;

ENV JAVA_OPTS=""
ENV PARAMS=""

EXPOSE 8080

ENTRYPOINT [ "sh", "-c", "java -Djava.security.egd=file:/dev/./urandom $JAVA_OPTS -jar /app.jar $PARAMS" ]
```

# 四、gitee 自动触发构建
① 进入 Jenkins 任务中, 点击 "设置" -> "流水线" -> "定义" 选择 "Pipeline script from SCM" -> "SCM" 选择 "git" -> "Repository URL" 填 gitee仓库地址 -> "指定分支" 填 "*/master" -> "源码库浏览器" 选择 "自动"

② 进入 gitee 仓库中, 点击 "管理", 选择 "WebHooks", 选择 "添加webHook"

③ 进入 Jenkins 任务中, 点击 "设置" -> "触发器构建" -> "触发远程构建 (例如,使用脚本)" -> "身份验证令牌" 随便填写

第 ② 步中的 URL 的填写规则: http://<user>:<public_key>@③提示的连接
- user: 登录 Jenkins 的用户名
- public_key: 点击右上角 的 user, 选择 "设置", 选择 "API Token" -> "添加新 Token" -> "生成" -> 拷贝生成的 Token
- jenkins_task_url: Jenkins 任务的 url 地址(公网能访问的) 

例如我这里的url: http://kino:11a42bf2caa09751d9ad748feef8659c07@121.77.237.178:8040/job/java-devops-demo/build?token=kino


此时提交任务即可自动构建, 并且自动打包发布运行


# 五、Jenkins 邮件
进入 Jenkins 主页面, 点击 "系统配置" -> "系统配置" -> 设置 Jenkins Location 中的 "系统管理员邮件地址", 填自己邮箱即可 -> 继续往下找, 找到 "Extended E-mall Notification" -> 设置 "SMTP server": "smtp.qq.com", 设置 "SMTP Port": "465" -> 点击 "高级" -> 设置 "SMTP Username" 为 "自己的邮箱", "SMTP Password" 为 "①得到的授权码", 设置 "Default Content Type" 为 "HTML",  找到 "邮件通知" -> 设置 "SMTP服务器" 为 "smtp.qq.com" -> 点击 "高级" -> 设置使用 "使用SMTP协议" 的用户名为 "自己的邮箱", 密码为 "①得到的授权码" -> 设置使用 "使用SSL协议" -> 设置 "SMTP 端口" 为 "465" -> 点击测试 


①: 打开 qq邮箱, 点击设置,  拉到 "POP3/IM...", 开启 POP3 服务, 发短信后会拿到一个授权码


在 Jenkins 的 工程中, 打开 "流水线语法", 打开 "片段生成器", 示例步骤选择 "setting-input dropdownList", 将要填的信息补齐, 生成流水线脚本, 然后贴入到 Jenkinsfile 文件中去 即可
