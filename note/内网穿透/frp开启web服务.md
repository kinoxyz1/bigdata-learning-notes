




---
# 一、公网上的操作
[下载 ftp](https://github.com/fatedier/frp/releases)

解压之后进入ftp 目录

作为服务端, 可以将 客户端 的配置文件删除掉
```bash
$ rm -f frpc
$ rm -f frpc.ini
```

编辑 ftps.ini 文件
```bash
$ vim ftps.ini
[common]
bind_port = 7000       #暴露给内网的客户端连接的端口
vhost_http_port = 9099 #暴露给内网的客户端web http连接的端口
```

# 二、内网上的操作
[下载 ftp](https://github.com/fatedier/frp/releases)

解压之后进入ftp 目录

作为客户端, 可以将 服务端 的配置文件删除掉
```bash
$ rm -f frps
$ rm -f frps.ini
```

编辑 frpc.ini 文件
```bash
[common]
server_addr = 61.xxx.xxx.xxx #公网IP
server_port = 7000  # 和 ftps.ini 的 bind_port 保持一致
 
[web]
type = http
local_ip = 127.0.0.1 # 本机 IP
local_port = 5000   # 本机要暴露到公网上的端口
remote_port = 9099  # 在公网上访问的端口, 和服务器vhost_http_port对应
custom_domains = 61.xxx.xxx.xxx # 公网IP 或者 域名(需要有一个已经存在的域名, 否则用 IP)
```

# 三、访问
<custom_domains>:<remote_port> 

# 四、使用systemctl来控制启动
公网:
```bash
$ vim /lib/systemd/system/frps.service
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target
 
[Service]
Type=simple
#启动服务的命令（此处写你的frps的实际安装目录）
ExecStart=/your/path/frps -c /your/path/frps.ini
 
[Install]
WantedBy=multi-user.target
```
内网:
```bash
$ vim /lib/systemd/system/frpc.service
[Unit]
Description=frapc service
After=network.target syslog.target
Wants=network.target
 
[Service]
Type=simple
#启动服务的命令（此处写你的frpc的实际安装目录）
ExecStart=/your/path/frpc -c /your/path/frpc.ini
 
[Install]
WantedBy=multi-user.target
```

启动:
```bash
# 公网
$ systemctl start frps
# 内网
$ systemctl start frpc
```

开机自启:
```bash
# 公网
$ systemctl enable frps
# 内网
$ systemctl enable frpc
```