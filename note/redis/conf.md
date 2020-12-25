




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

# 六、SNAPSHOTTING 
看后续, 暂不做解释
```bash
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```

```bash
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes
```

```bash
# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes
```

```bash
# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes
```

```bash
# The filename where to dump the DB
dbfilename dump.rdb
```

```bash
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./
```

# 七、REPLICATION
看后续, 暂不做解释


# 八、SECURITY
设置 redis 密码
```bash
# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared

# Command renaming.
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to replicas may cause problems.
```
示例:
```bash
> ping
PONG
# 查看密码
> CONFIG GET requirepass
1) "requirepass"
2) ""

# 设置密码
> CONFIG SET requirepass "123"
OK
> ping 
(error) NOAUTH Authentication required.

# 使用密码登录
> auth "123"
OK
> ping
PONG
```
或者修改 redis.conf 文件:
```bash
requirepass "123"
```
重启 redis 并测试
```bash
$ ps -ef | grep redis | awk '{print $2}' | xargs kill -9
$ bin/redis-server conf/redis.conf
> ping
(error) NOAUTH Authentication required
> auth 123
ok
>ping
PONG
```

# 九、CLIENTS
## 9.1 maxclients
设置 客户端最大连接数
```bash
# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# maxclients 10000
```


# 十、APPEND ONLY MODE
看后续, 暂不做解释
```bash
# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no
```

```bash
# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"
```

```bash
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no
```


```bash
# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no
```

```bash
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

```bash
# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes
```


```bash
# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
aof-use-rdb-preamble yes
```


# 十一、常见配置
```bash
参数说明
redis.conf 配置项说明如下：
1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
  daemonize no
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
  pidfile /var/run/redis.pid
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
  port 6379
4. 绑定的主机地址
  bind 127.0.0.1
5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
  timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
  loglevel verbose
7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
  logfile stdout
8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
  databases 16
9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
  save <seconds> <changes>
  Redis默认配置文件中提供了三个条件：
  save 900 1
  save 300 10
  save 60 10000
  分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
 
10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
  rdbcompression yes
11. 指定本地数据库文件名，默认值为dump.rdb
  dbfilename dump.rdb
12. 指定本地数据库存放目录
  dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
  slaveof <masterip> <masterport>
14. 当master服务设置了密码保护时，slav服务连接master的密码
  masterauth <master-password>
15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
  requirepass foobared
16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
  maxclients 128
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
  maxmemory <bytes>
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
  appendonly no
19. 指定更新日志文件名，默认为appendonly.aof
   appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值： 
  no：表示等操作系统进行数据缓存同步到磁盘（快） 
  always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
  everysec：表示每秒同步一次（折衷，默认值）
  appendfsync everysec
 
21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
   vm-enabled no
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
   vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
   vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
   vm-page-size 32
25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
   vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
   vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
  glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
  hash-max-zipmap-entries 64
  hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
  activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
  include /path/to/local.conf
```