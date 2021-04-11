







---
# 一、Portainer 是什么
[Portainer 官方文档](https://documentation.portainer.io/)

Portainer 社区 2.0 拥有超过 50万的普通用户, 是功能强大的开源工具集, 可以轻松地在 Docker、Swarm、Kubernetes 和 Azure ACI 中构建和管理容器.

# 二、安装
## 2.1 服务端部署
```bash
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```
在浏览器中访问 9000 端口即可


## 2.2 agent 端部署
```bash
docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent
```


第一次登陆需要设置admin的密码