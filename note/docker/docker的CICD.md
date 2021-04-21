




# 一、Jenkins 安装
使用 docker 直接部署
```bash
$ docker run \
-u root \
-d \
-p 8080:8080 \
-p 50000:50000 \
-v jenkins-data:/var/jenkins_home \
-v /etc/localtime:/etc/localtime:ro \
-v /var/run/docker.sock:/var/run/docker.sock \
--restart=always \
jenkinsci/blueocean
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

# 二、Idea 安装 gitee 插件
