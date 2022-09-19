








---
# 一、官网
https://github.com/kylemanna/docker-openvpn

# 二、docker 部署
```bash
OVPN_DATA="/opt/openvpn/openvpn-data"
## 指定 TCP 协议和端口号来初始化数据容器
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u tcp://公网IP:公网端口
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
docker run -v $OVPN_DATA:/etc/openvpn --rm -p 本地端口:1194/tcp --cap-add=NET_ADMIN kylemanna/openvpn ovpn_run --proto tcp

# 生成客户端.ovpn test为自定义名称
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full kino nopass
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient kino > kino.ovpn
#使用scp把test.ovpn导到本地，使用openvpn客户端打开

-- 撤销客户端证书
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa revoke kino
```

# 三、frp 内网穿透
[frp 安装](../../note/内网穿透/暴露内网端口.md)

```bash
## frps config
[common]
# 设置bind_port为连接到frp客户端
bind_port = 公网服务器端口
# 通过自定义域访问LAN中的Web服务 vhost HTTP端口设置为8040
#vhost_http_port = 8040
token = token


## frpc config
[common]
server_addr = 公网服务器ip
server_port = 公网服务器端口
token = token

[openvpn]
type = tcp
local_ip = 127.0.0.1
local_port = 本地端口
remote_port = 公网端口
```
示例
```bash
## frps config
$ vim frps.ini
[common]
# 设置bind_port为连接到frp客户端
bind_port = 7000
# 通过自定义域访问LAN中的Web服务 vhost HTTP端口设置为8040
#vhost_http_port = 8040
token = 8898

## frpc config
[common]
server_addr = 110.110.110.110
server_port = 7000
token = 8898

[openvpn]
type = tcp
local_ip = 127.0.0.1
local_port = 14430
remote_port = 14430
```
frps 和 frpc 的启动看 [frp 安装](../../note/内网穿透/暴露内网端口.md)

# 四、download openvpn
[openvpn下载地址](https://github.com/fatedier/frp/releases)

将 第二步获取的 kino.ovpn 添加到安装包的 openvpn 即可。





















