




---
# 一、pull 镜像
```bash
$ docker pull gitlab/gitlab-ce:latest
```

# 二、运行容器
```bash
docker run -p 4403:443 -p 12380:80 -p 222:22 --name=gitlab --restart=always -v $GITLAB_HOME/config:/etc/gitlab -v $GITLAB_HOME/logs:/var/log/gitlab -v $GITLAB_HOME/data:/var/opt/gitlab -d gitlab/gitlab-ce
```

# 三、查看容器
```bash
72549aec426c   gitlab/gitlab-ce   "/assets/wrapper"        4 minutes ago    Up 4 minutes (health: starting)   0.0.0.0:222->22/tcp, 0.0.0.0:12380->80/tcp, 0.0.0.0:4403->443/tcp   gitlab
```
status = (health: starting):表示正在启动中, 可以通过: `docker logs -ft --tail 10 72549aec426c` 查看容器运行的日志

当 status = (healthy): 表示启动成功

# 四、查看端口
```bash
$ netstat -tunlp | grep 12380
tcp        0      0 0.0.0.0:12380           0.0.0.0:*               LISTEN      8702/docker-proxy
```

# 五、telnet 端口
在window上进行
```bash
$ telnet 192.168.220.110 12380
```
能通则说明gitlab启动正常

# 六、在浏览器中访问
192.168.220.110:12380 可以看到如下界面
![docker-login](../../img/gitlab/rpm/docker-login.png)