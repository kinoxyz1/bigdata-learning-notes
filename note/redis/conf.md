




---

# 一、units
声明基本的度量单位, 只支持 bytes 不支持 bit, 且对大小写不明感
```shrinker_config
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```

# 二、INCLUDES
可以通过 INCLUDES 包含其他 conf 的配置文件
```bash
################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
```

# 三、MODULES
启动 redis 时, 需要加载的模块, 如果不存在, 将会被终止, 可以使用多个 loadmodule 指令
```bash
################################## MODULES #####################################

# Load modules at startup. If the server is not able to load modules
# it will abort. It is possible to use multiple loadmodule directives.
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so
```

# 四、NETWORK
## 4.1 bind
绑定本机的IP地址(本机的网卡对应的IP地址, 每个网卡都有一个IP地址), 并不是 redis 允许来自其他计算机的 IP 地址;

如果指定了 bind, 则说明只允许来自指定网卡的 Redis 请求, 如果没有指定, 则可以允许任意一个网卡的 Redis 请求

举个例子, 如果 安装redis 的服务器上有两个网卡, 每个网卡对应一个IP地址, 例如 IP1 和 IP2, 我们配置 bind=IP1, 此时, 只有我们通过IP1来访问 redis 服务器, 才允许连接到 redis, 如果通过IP2来访问 redis 服务器, 就会连接不上 redis

查看本机网络信息: `ifconfig`
```bash
[root@hadoop1 redis-5.0.8]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.220.51  netmask 255.255.255.0  broadcast 192.168.220.255
        inet6 fe80::d9a9:52aa:a97c:731b  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:77:c1:d7  txqueuelen 1000  (Ethernet)
        RX packets 76298  bytes 71073987 (67.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 15537  bytes 2531839 (2.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8132  bytes 471053 (460.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8132  bytes 471053 (460.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
从上面看此机器上有两个网卡, 也就是说, 我们只能通过设置 `192.168.220.51` 或 `127.0.0.1` 为 bind 地址, 否则 redis 起不来

例如我们将 bind 设置为 10.0.0.1, 并指定 redis log文件
```bash
$ vim redis.conf
# bind=127.0.0.1
bind=10.0.0.1

# loglevel notice
loglevel debug
# logfile ""
logfile "/usr/bigdata/redis-5.0.8/logs/redis.log"
```
杀死redis并重启
```bash
$ ps -ef | grep redis
root      10629      1  0 10:04 ?        00:00:05 ./redis-server 127.0.0.1:6379
root      10683   1458  0 11:27 pts/0    00:00:00 grep --color=auto redis

$ kill -9 10629

$ mkdir -p /usr/bigdata/redis-5.0.8/logs
$ touch /usr/bigdata/redis-5.0.8/logs/redis.log

$ bin/redis-server conf/redis.conf
```
查看redis是否启动
```bash
[root@hadoop1 redis-5.0.8]# ps -ef | grep redis
root      10691   1458  0 11:30 pts/0    00:00:00 grep --color=auto redis
```
可以看到redis并未启动, 查看 redis 的 log 文件
```bash
10711:C 25 Dec 2020 11:34:58.593 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
10711:C 25 Dec 2020 11:34:58.593 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=10711, just started
10711:C 25 Dec 2020 11:34:58.593 # Configuration loaded
10712:M 25 Dec 2020 11:34:58.595 * Increased maximum number of open files to 10032 (it was originally set to 1024).
10712:M 25 Dec 2020 11:34:58.595 # Could not create server TCP listening socket 10.0.0.1:6379: bind: Cannot assign requested address
10712:M 25 Dec 2020 11:34:58.595 # Configured to not listen anywhere, exiting.
```
日志文件中可以看见: `Could not create server TCP listening socket 10.0.0.1:6379: bind: Cannot assign requested address`

此时再将 bind 修改为: 192.168.220.51
```bash
$ vim redis.conf
# bind=127.0.0.1
bind=192.168.220.51

# loglevel notice
loglevel debug
# logfile ""
logfile "/usr/bigdata/redis-5.0.8/logs/redis.log"
```
再次启动
```bash
$ bin/redis-server conf/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.8 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 10719
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

10719:M 25 Dec 2020 11:38:00.138 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
10719:M 25 Dec 2020 11:38:00.138 # Server initialized
10719:M 25 Dec 2020 11:38:00.138 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
10719:M 25 Dec 2020 11:38:00.138 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
10719:M 25 Dec 2020 11:38:00.138 * Ready to accept connections
10719:M 25 Dec 2020 11:38:00.140 - 0 clients connected (0 replicas), 791472 bytes in use
```
发现 redis 正常启动

bind 相关的默认配置如下:
```bash
# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only into
# the IPv4 loopback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1
```
redis 的 `bind` 值默认是 `127.0.0.1`, 此时就只有本机能访问, 其他机器并不可以访问该redis, 因为从上面 `ifconfig` 可以看到: `127.0.0.1` 是 `lo网卡`, 这是一个回环地址, 仅只有本机才能访问该回环地址, 其他机器也只能访问他们自己的回环地址, 所以配置 `127.0.0.1` 就只有本机可以访问

当 `1`bind=0.0.0.0` 时, 表示所有主机都可以连接到该redis

## 4.2 protected-mode
保护模式是否开启
```bahs
# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode yes
```
接上面 bind 部分继续来说, redis 本身并不能限制 **只有指定主机** 能连接到redis, 只能通过 redis 的 bind 参数来设置, 此时会有两种设置情况:
1. `bind=127.0.0.0`: 非常的安全, 此时只有本机才能连接到 redis, 就算不设置密码, 也是很安全的, 除非有人能登录你的redis服务器
2. `bind=0.0.0.0`: 所有主机均可连接(开启防火墙时需要开放端口), 此时的安全仅能通过设置密码来保护, 也就是说知道密码的所有主机都可以访问redis

而 `protected-mode` 就是做这个安全保护的, 当 `protected-mode=yes` 时, 就表示只有本机可以访问redis, 其他任何机器都不可以访问redis, 但是这个参数生效有三个前提条件:
1. `protected-mode=yes`
2. 没有 bind 指令
2. 没有设置密码

当满足这三个条件之后, redis 的保护机制就会被打开, 如果三个条件有一个不满足, 则保护机制不生效