



---


```bash
# 查看 docker 版本
yum list docker-ce --showduplicates | sort -r

# 安装指定版本
yum downgrade --setopt=obsoletes=0 -y docker-ce-${version} docker-ce-selinux-${version} docker-ce-cli-${version}
```


