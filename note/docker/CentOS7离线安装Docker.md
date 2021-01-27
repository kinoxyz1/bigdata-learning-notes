




--- 
# 一、Centos版本
```bash
$ cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)

$ uname -a 
Linux centos7 3.10.0-1127.18.2.el7.x86_64 #1 SMP Sun Jul 26 15:27:06 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

# 二、下载安装包
https://download.docker.com/linux/static/stable/x86_64/

# 三、解压、启动
```bash
$ tar -zxvf docker-19.03.12.tgz && mv docker/* /usr/bin/ && rm -rf docker*.tgz
```
增加配置文件
```bash
$ vim /etc/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
  
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
  
[Install]
WantedBy=multi-user.target
```
赋权
```bash
$ chmod +x /etc/systemd/system/docker.service
```
设置开机自启
```bash
//重载systemd下 xxx.service文件
$ systemctl daemon-reload   
//启动Docker
$ systemctl start docker       
//设置开机自启
$ systemctl enable docker.service   
```
查看docker运行状态及版本
```bash
$ systemctl status docker
$ docker -v
Docker version 19.03.12, build 48a66213fe
```