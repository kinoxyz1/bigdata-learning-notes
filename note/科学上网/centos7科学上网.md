







---


```bash
$ yum install python-setuptools
$ yum -y install python3-pip
$ pip3 install shadowsocks
$ vim /etc/shadowsocks.json
{
        "server": "104.168.215.244",
        "server_port": 29257,
        "local_address": "127.0.0.1",
        "local_port": 1080,
        "password": "233blog.com",
        "method": "aes-256-cfb",
        "fast_open": true,
        "timeout": 300
}
$ vim /usr/local/lib/python3.6/site-packages/shadowsocks/crypto/openssl.py
libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,) ->  libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)
libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx) -> libcrypto.EVP_CIPHER_CTX_reset(self._ctx)

# 启动
nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &
echo " nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &" /etc/rc.local   #设置自启动

# 测试
运行 curl –socks5 127.0.0.1:1080 http://httpbin.org/ip, 如果返回你的 ss 服务器 ip 则测试成功
$ curl http://httpbin.org/ip
{
  "origin": "218.18.163.249"
}

# 安装Privoxy
# Shadowsocks 是一个 socket5 服务，我们需要使用 Privoxy 把流量转到 http／https 上。
$ yum install epel-release
$ yum install privoxy -y
$ rpm -qa|grep privoxy
privoxy-3.0.32-1.el7.x86_64

# 配置Privoxy
$ vim /etc/privoxy/config
listen-address  0.0.0.0:8118    #Privoxy监听地址
forward-socks5t   /               127.0.0.1:1888 .  #本地Shadowsocks地址
forward         192.168.*.*/     .  #不转发的地址
forward            10.*.*.*/     .  #不转发的地址
forward           127.*.*.*/     .  #不转发的地址
forwarded-connect-retries  1    #重试次数
$ systemctl start privoxy
$ systemctl status privoxy
   Loaded: loaded (/usr/lib/systemd/system/privoxy.service; disabled; vendor preset: disabled)
   Active: active (running) since 六 2021-04-10 16:49:57 CST; 3s ago
  Process: 64242 ExecStart=/usr/sbin/privoxy --pidfile /run/privoxy.pid --user privoxy /etc/privoxy/config (code=exited, status=0/SUCCESS)
 Main PID: 64243 (privoxy)
    Tasks: 1
   Memory: 1012.0K
   CGroup: /system.slice/privoxy.service
           └─64243 /usr/sbin/privoxy --pidfile /run/privoxy.pid --user privoxy /etc/privoxy/config

4月 10 16:49:56 k8s-node2 systemd[1]: Starting Privoxy Web Proxy With Advanced Filtering Capabilities...
4月 10 16:49:57 k8s-node2 systemd[1]: Started Privoxy Web Proxy With Advanced Filtering Capabilities.

# 配置http_proxy
```