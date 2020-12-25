




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

当 `bind=0.0.0.0` 时, 表示所有主机都可以连接到该redis

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

## 4.3 port
redis 的 端口号, 默认是 6379, 如果修改之后, 连接 `redis-cli` 的命令应该变成: `redis-cli -p your-port`
```bash
# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379
```

## 4.4 tcp-backlog
说明如下:
```bash
# TCP listen() backlog.
#
# In high requests-per-second environments you need an high backlog in order
# to avoid slow clients connections issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
tcp-backlog 511
```
此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须**不大于**Linux系统定义的 `/proc/sys/net/core/somaxconn` 值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。

建议修改为 2048

修改somaxconn, 该内核参数默认值一般是128，对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。
```bash
echo 2048 > /proc/sys/net/core/somaxconn 但是这样系统重启后保存不了
```
在/etc/sysctl.conf中添加如下
```bash
net.core.somaxconn = 2048
```
然后在终端中执行
```bash
sysctl -p
```

## 4.5 timeout
设置 客户端空闲超时时间, 当`timeout` 为 `0` 时, 表示禁用
```bash
# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0
```

## 4.6 tcp-keepalive
客户端TCP连接的健康性检查，如果不设置为0就表示Redis服务端会定时发送SO_KEEPALIVE心跳机制检测客户端的反馈情况。该配置的默认值为300秒，既是300秒检测一次。健康性检查的好处是，在客户端异常关闭的情况下，Redis服务端可以发现这个问题，并主动关闭对端通道。这个参数建议开启。
```bash
# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
tcp-keepalive 300
```

# 五、GENERAL
## 5.1 daemonize
是否以守护进程的模式运行
```bash
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes
```

## 5.2 supervised
管理 redis 守护进程

选项：
- `supervised no`: 没有监督互动
- `supervised upstart`: 通过将 `Redis` 置于 `SIGSTOP` 模式来启动信号
- `supervised systemd`: `signal systemd` 将 `READY = 1` 写入`$ NOTIFY_SOCKET`
- `supervised auto`: 检测 `upstart` 或 `systemd` 方法基于 `UPSTART_JOB` 或 `NOTIFY_SOCKET` 环境变量
```bash
# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no
```

## 5.3 pidfile
守护进程的 pid 文件
```bash
# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid
```

## 5.4 loglevel
log 日志的级别
```bash
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
# loglevel notice
loglevel debug
```
选项: 
- debug: 输出信息最多, 适用于 **开发、测试** 环境
- verbose: 输出信息不如 debug 多
- notice: 输出信息 不如 verbose 多, 适用于 **生产** 环境
- warning: 仅输出 **非常重要、关键** 的信息

## 5.5 logfile
log 文件目录
```bash
# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""
```

## 5.6 syslog-enabled
是否将日志输出到系统log中
```bash
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no
```

## 5.7 syslog-ident
系统日志的前缀
```bash
# Specify the syslog identity.
# syslog-ident redis
```

## 5.8 syslog-facility
指定 syslog 设备, 可以使 LOCAL0-LOCAL7.
```bash
# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0
```

## 5.9 databases
设置数据库数
```bash
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```

## 5.10 always-show-logo
是否显示 redis 的logo
```bash
# By default Redis shows an ASCII art logo only when started to log to the
# standard output and if the standard output is a TTY. Basically this means
# that normally a logo is displayed only in interactive sessions.
#
# However it is possible to force the pre-4.0 behavior and always show a
# ASCII art logo in startup logs by setting the following option to yes.
always-show-logo yes
```