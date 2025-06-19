
---
# 一、Nginx 安装部署
## 1.1 安装编译工具及库文件
```bash
$ yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

## 1.2 安装 pcre
pcre 作用是 Nginx 支持 Rewrite 功能
```bash
$ cd /usr/local/src
$ wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
$ tar zxvf pcre-8.35.tar.gz
$ cd pcre-8.35
$ ./configure
$ make && make install
$ pcre-config --version
8.35
```

## 1.3 编译安装 nginx
[Nginx 下载地址](http://nginx.org/)
```bash
# 查看编译的帮助文档
$ ./configure --help

  # 如果下面几个参数没有设置, 默认都放在 --prefix 指定的路径下
  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --modules-path=PATH                set modules path
  --conf-path=PATH                   set nginx.conf pathname
  --error-log-path=PATH              set error log pathname
  --pid-path=PATH                    set nginx.pid pathname
  --lock-path=PATH                   set nginx.lock pathname

 # --with 和 --without 是确认需要使用什么模块和不使用哪些模块
 # --with: 默认不会被编译进 nginx 中, 编译需要手动指定
 --with-*****
 # --without: 默认会被编译进 nginx 中, 不编译需要手动指定
 --without-*****
 
# 使用默认编译
$ ./configure --prefix=/app/nginx/    # 将nginx 编译到 /app/nginx 目录下
# 编译完成后会输出如下信息: 
...
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/app/nginx/"
  nginx binary file: "/app/nginx//sbin/nginx"
  nginx modules path: "/app/nginx//modules"
  nginx configuration prefix: "/app/nginx//conf"
  nginx configuration file: "/app/nginx//conf/nginx.conf"
  nginx pid file: "/app/nginx//logs/nginx.pid"
  nginx error log file: "/app/nginx//logs/error.log"
  nginx http access log file: "/app/nginx//logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
  
# 编译完成后, 会在 nginx/objs/ 目录(中间文件)下生成 ngx_modules.c 文件, 这个文件包含了编译时会被编译进去的模块
$ cat objs/ngx_modules.c
...

$ make && make install

$ cd /app/nginx && ll
drwxr-xr-x. 2 root root 4096 8月  20 18:56 conf
drwxr-xr-x. 2 root root   40 8月  20 18:56 html
drwxr-xr-x. 2 root root    6 8月  20 18:56 logs
drwxr-xr-x. 2 root root   19 8月  20 18:56 sbin
```

# 二、Nginx 目录结构
```bash
$ cd /usr/local/nginx && ls 
client_body_temp conf fastcgi_temp html logs proxy_temp sbin scgi_temp uwsgi_temp
```

其中这几个文件夹在刚安装后是没有的，主要用来存放运行过程中的临时文件
```bash
client_body_temp fastcgi_temp proxy_temp scgi_temp
```
- conf: 用来存放配置文件相关
- html: 用来存放静态文件的默认目录 html、css等
- sbin: nginx的主程序

# 三、Nginx 常用命令
```bash
$ cd /usr/local/nginx/
```

## 3.1 帮助
```bash
$ sbin/nginx -help
nginx version: nginx/1.19.6
Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
             [-e filename] [-c filename] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -e filename   : set error log file (default: logs/error.log)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
```

## 3.2 启动
```bash
$ ps -ef | grep nginx
root      17986  11127  0 18:49 pts/0    00:00:00 grep --color=auto nginx
$ sbin/nginx 
$ ps -ef | grep nginx
root      17989      1  0 18:49 ?        00:00:00 nginx: master process sbin/nginx
nobody    17990  17989  0 18:49 ?        00:00:00 nginx: worker process
root      17992  11127  0 18:49 pts/0    00:00:00 grep --color=auto nginx
```

## 3.3 暴力停止
```bash
$ sbin/nginx -s stop
```

## 3.4 优雅停止
```bash
$ sbin/nginx -s quit
```

## 3.4 重新加载配置文件
```bash
$ sbin/nginx -s reload
```
reload 流程:
1. 向 master 进程发送 HUP 信号(reload 命令)
2. master 进程校验配置语法是否正确
3. master 进程打开新的监听端口
4. master 进程用新配置启动新的 worker 子进程
5. master 进程向老 worker 子进程发送 QUIT 信号
6. 老 worker 进程关闭监听句柄, 处理完当前连接后, 结束进程

**nginx worker进程长期处于 worker process is shutting down状态的问题**
```bash
root     30261     1  0 3月13 ?       00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    30262 30261  0 3月13 ?       00:49:57 nginx: worker process is shutting down
nginx    30263 30261  0 3月13 ?       00:11:15 nginx: worker process is shutting down
nginx    30266 30261  0 3月13 ?       00:06:33 nginx: worker process is shutting down
nginx    30267 30261  0 3月13 ?       00:05:18 nginx: worker process is shutting down
nginx    30410 30261  0 3月14 ?       02:50:27 nginx: worker process is shutting down
```
Nginx 配置中如果有 WebSocket 或者 upstream 反向代理时, reload 之后, 长连接一直处于未断开状态, 导致每次reload都无法让worker进程退出.

解决办法:
```bash
$ vim nginx.conf
# 增加配置
worker_shutdown_timeout 10s;
```

worker 进程优雅关闭的流程:
1. 设置定时器: worker_shutdown_timeout
2. 关闭监听句柄
3. 关闭空闲连接
4. 在循环中等待全部连接关闭
5. 退出进程


## 3.5 指定启动的配置文件
```bash
$ sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

## 3.6 Nginx 日志切割
```bash
$ ll
drwxr-xr-x  2 nginx nginx         4096 5月  20 10:44 ./
drwxrwxr-x 16 root  syslog        4096 5月  20 00:05 ../
-rw-r--r--  1 root  root   10603437739 5月  20 10:41 access.log
-rw-r--r--  1 nginx root       1628903 5月  20 10:44 error.log
-rw-r--r--  1 nginx root     449966635 5月  20 10:44 tcp-access.log

$ mv access.log access.log.bak
$ ./sbin/nginx -s reopen
$ ll 
drwxr-xr-x  2 nginx nginx         4096 5月  20 10:44 ./
drwxrwxr-x 16 root  syslog        4096 5月  20 00:05 ../
-rw-r--r--  1 root  root   10603437739 5月  20 10:41 access.log.bak
-rw-r--r--  1 nginx root       1628903 5月  20 10:44 error.log
-rw-r--r--  1 nginx root     449966635 5月  20 10:44 tcp-access.log
-rw-r--r--  1 nginx root          1333 5月  20 10:46 access.log
```
mv 到 reopen 之间会不会丢失句? 只要没有在 mv 和 reopen 之间重启 Nginx 或 kill worker 进程, 就不会丢失日志记录.

因为 Nginx 打开日志文件时, 会持有一个指向日志文件的文件描述符, 这个文件描述符绑定到文件的 iNode(而不是路径).

在执行 mv access.log access.log.bak 时, 只是把文件的路径指针改了, 对于Nginx来说, 它任然持有旧的文件描述符, 日志会继续写入原来的文件, 不会修饰日志.

## 3.7 热升级

![nginx 信号](../../img/nginx/nginx信号/NGINX信号.png)


| 信号       | 作用        |
|----------|-----------|
| TERM/INT | 立即关闭整个服务  |
| QUIT     | "优雅"地关闭整个服务 | 
| HUP      | 重读配置文件并使用服务对新配置项生效 |
| USR1 | 重新打开日志文件, 可以用来进行日志切割 | 
| USR2 | 平滑升级到最新版的nginx | 
| WINCH | 所有子进程不在接收处理新链接, 相当于给 worker 进程发送 QUIR 指令 | 



升级Nginx的时候, 可以做到新Nginx进程和旧Nginx进程同时提供服务.

```bash
$ /usr/local/nginx/sbin# ll
total 13272
-rwxr-xr-x 1 nginx nginx 6812410 12月  9  2021 nginx.old   # 当前运行的Nginx
-rwxr-xr-x 1 nginx nginx 6791672 12月  9  2021 nginx       # 新编译出来的Nginx

# 查看当前Nginx进程
$ ps -ef|grep nginx 
root     30261     1  0 3月13 ?       00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx      867 30261  0 5月16 ?       00:23:12 nginx: worker process
nginx      868 30261  0 5月16 ?       00:00:13 nginx: worker process
nginx      869 30261  0 5月16 ?       00:00:01 nginx: worker process
nginx      870 30261  0 5月16 ?       00:00:00 nginx: worker process

# 向当前Nginx master 发送热升级指令, 新master/worker启动之后, 请求将不会再去旧进程中.
$ kill -USR2 30261
$ ps -ef|grep nginx 
root     30261     1  0 3月13 ?       00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx      867 30261  0 5月16 ?       00:23:16 nginx: worker process
nginx      868 30261  0 5月16 ?       00:00:13 nginx: worker process
nginx      869 30261  0 5月16 ?       00:00:01 nginx: worker process
nginx      870 30261  0 5月16 ?       00:00:00 nginx: worker process
root     23021 30261  0 11:06 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    23022 23021  0 11:06 ?        00:00:00 nginx: worker process
nginx    23032 23021  0 11:06 ?        00:00:00 nginx: worker process
nginx    23033 23021  0 11:06 ?        00:00:00 nginx: worker process
nginx    23034 23021  0 11:06 ?        00:00:00 nginx: worker process

# 向旧Nginx master发送退出worker指令, master 进程不会自动退出, 此时如果新进程有问题, 还可以重启旧进程的worker
$ kill -WINCH 30261
$ ps -ef|grep nginx 
root     30261     1  0 3月13 ?       00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx      867 30261  0 5月16 ?       00:23:16 nginx: worker process is shutting down
nginx      868 30261  0 5月16 ?       00:00:13 nginx: worker process is shutting down
nginx      900 30261  0 5月06 ?       01:03:45 nginx: worker process is shutting down
nginx    18185 30261  0 4月24 ?       00:09:19 nginx: worker process is shutting down
```

热升级流程:
1. 将旧Nginx文件换成新Nginx文件(注意备份)
2. 向Master进程发送 USR2 信号
3. Master进程修改pid文件名, 加后缀.oldbin
4. Master进程用新Nginx文件启动新Master进程
5. 向老Master进程发送 QUIR 信号, 关闭老Master进程
6. 回滚: 向老Master发送 HUP, 向新Master发送 QUIT


# 四、Nginx HTTP 模块


## 4.1 处理 HTTP 请求头部的流程

这部分需要补充计算机组成相关的知识. 相关链接如下:

[03 | 基础篇：经常说的 CPU 上下文切换是什么意思？（上）](https://time.geekbang.org/column/article/69859)

[如果这篇文章说不清epoll的本质，那就过来掐死我吧！ （1）](https://zhuanlan.zhihu.com/p/63179839)

[如果这篇文章说不清epoll的本质，那就过来掐死我吧！ （2）](https://zhuanlan.zhihu.com/p/64138532)

[如果这篇文章说不清epoll的本质，那就过来掐死我吧！ （3）](https://zhuanlan.zhihu.com/p/64746509)

[图解 | 深入揭秘 epoll 是如何实现 IO 多路复用的！](https://mp.weixin.qq.com/s/OmRdUgO1guMX76EdZn11UQ)

[I/O 多路复用：select/poll/epoll
](https://www.xiaolincoding.com/os/8_network_system/selete_poll_epoll.html#%E6%9C%80%E5%9F%BA%E6%9C%AC%E7%9A%84-socket-%E6%A8%A1%E5%9E%8B)

首先启动 Nginx, 对应的配置如下:
```bash
$ cat nginx.conf
worker_processes  1;
events {
    use epoll;
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

$ ./sbin/nginx
$ ps -ef|grep nginx
root      4156     1  0 13:30 ?        00:00:00 nginx: master process ./sbin/nginx
nobody   16675  4156  0 16:32 ?        00:00:00 nginx: worker process
$ ll /proc/16675/fd   # 查看当前进程的文件描述符
dr-x------ 2 nobody nogroup  0 5月  22 16:33 ./
dr-xr-xr-x 9 nobody nogroup  0 5月  22 16:32 ../
lrwx------ 1 nobody nogroup 64 5月  22 16:33 0 -> /dev/null
lrwx------ 1 nobody nogroup 64 5月  22 16:33 1 -> /dev/null
lrwx------ 1 nobody nogroup 64 5月  22 16:33 10 -> 'anon_inode:[eventpoll]'            # epoll 本身的 fd 是 10
lrwx------ 1 nobody nogroup 64 5月  22 16:33 11 -> 'anon_inode:[eventfd]'              # eventfd 的 fd 是 11
l-wx------ 1 nobody nogroup 64 5月  22 16:33 2 -> /usr/local/nginx/logs/error.log      # error.log 文件的fd 是 2
l-wx------ 1 nobody nogroup 64 5月  22 16:33 4 -> /usr/local/nginx/logs/access.log     # access.log 文件的fd 是 4
l-wx------ 1 nobody nogroup 64 5月  22 16:33 5 -> /usr/local/nginx/logs/error.log      # error.log 文件的fd 是 5
lrwx------ 1 nobody nogroup 64 5月  22 16:33 6 -> 'socket:[40184994]'                  # 已连接的 socket 的 fd 是 6
lrwx------ 1 nobody nogroup 64 5月  22 16:33 7 -> 'socket:[40188263]'                  # 已连接的 socket 的 fd 是 7
```
整个启动的伪代码如下:
```java
int main(){
    listen(lfd, ...);                              # 监听端口

    cfd1 = accept(...);							   # 接收客户端连接, 阻塞, 生成 fd1
    cfd2 = accept(...);							   # 接收客户端连接, 阻塞, 生成 fd2
    efd = epoll_create(...);					   # 创建 epfd

    epoll_ctl(efd, EPOLL_CTL_ADD, cfd1, ...);	   # epoll_ctl 将需要监听的 socket(fd1) 放到红黑树中
    epoll_ctl(efd, EPOLL_CTL_ADD, cfd2, ...);	   # epoll_ctl 将需要监听的 socket(fd2) 放到红黑树中
    epoll_wait(efd, ...)						   # epoll_wait 是阻塞的, 监听红黑树中的socket是否有事件发生
}
```

Nginx 在启动的时候, 监听了 80端口, 并且创建了两个 socket, 分别是 6/7, 在内核中创建了 epoll 对象本身(fd=10), 还创建了 eventfd(fd=11), eventfd 是用来实现线程间/进程间时间唤醒的机制, 它可以唤醒 epoll_wait 中的 worker, strace 查看该进程:
```bash
$ strace -p 16675
strace: Process 16675 attached
epoll_wait(10,
```
epoll_wait 等待文件描述符10的事件

现在请求Nginx
```bash
$ curl localhost:80
epoll_wait(10, [{EPOLLIN, {u32=2662486032, u64=22843298566160}}], 512, -1) = 1
# 客户端有socket连接上来了, 文件描述符是 3
accept4(6, {sa_family=AF_INET, sin_port=htons(53188), sin_addr=inet_addr("127.0.0.1")}, [112->16], SOCK_NONBLOCK) = 3
# epoll_ctl 将文件描述符 3 添加到 epoll 的红黑树中, 设置监听 EPOLLIN(可读)、EPOLLRDHUP(对端关闭连接)、EPOLLET(边沿触发)
# 文件描述符 3 并不会立即进入就绪链表, 除非内核判断该 socket 当前可读
epoll_ctl(10, EPOLL_CTL_ADD, 3, {EPOLLIN|EPOLLRDHUP|EPOLLET, {u32=2662486480, u64=22843298566608}}) = 0
# 客户端发送了数据, 内核唤醒线程, 将 fd=3 放入就绪列表, epoll_wait 返回后, Nginx 处理该链接数据
epoll_wait(10, [{EPOLLIN, {u32=2662486480, u64=22843298566608}}], 512, 60000) = 1
# 读取客户端请求
recvfrom(3, "GET / HTTP/1.1\r\nHost: localhost\r"..., 1024, 0, NULL, NULL) = 73
stat("/usr/local/nginx/html/index.html", {st_mode=S_IFREG|0644, st_size=21, ...}) = 0
# 打开 index.html 文件
openat(AT_FDCWD, "/usr/local/nginx/html/index.html", O_RDONLY|O_NONBLOCK) = 8
fstat(8, {st_mode=S_IFREG|0644, st_size=21, ...}) = 0
# 写响应数据
writev(3, [{iov_base="HTTP/1.1 200 OK\r\nServer: nginx/1"..., iov_len=236}], 1) = 236
sendfile(3, 8, [0] => [21], 21)         = 21
# 写 access.log 日志文件
write(4, "127.0.0.1 - - [22/May/2025:16:56"..., 85) = 85
# socket 关闭
close(8)                                = 0
setsockopt(3, SOL_TCP, TCP_NODELAY, [1], 4) = 0
epoll_wait(10, [{EPOLLIN|EPOLLRDHUP, {u32=2662486480, u64=22843298566608}}], 512, 65000) = 1
recvfrom(3, "", 1024, 0, NULL, NULL)    = 0
close(3)                                = 0
epoll_wait(10,
```

以下是 操作系统内核 - 事件模块 - HTTP 模块 的流程图

![Nginx处理http请求头的流程](../../img/nginx/http/Nginx处理http请求头的流程.png)

在三次握手中, 只完成了前两次握手, 这个TCP连接仅仅处于就绪队列中, Nginx并不会感知到, 只有当第三次握手完成后, 内核才会触发 Nginx 的 epoll_wait 通知有读事件(返回文件句柄fd), 用 accept 方法, 它会分配 TCP 连接的连接内存池(Nginx 中内存池分为 连接内存池 和 请求内存池), 连接内存池的大小由 `connection_pool_size` 决定, 默认 512 字节, `connection_pool_size` 仅仅维护着 TCP 连接相关的参数。

然后 HTTP 模块接收请求的处理, 当触发 accept 后, ngx_http_init_connection 回调方法就会被执行, 新建立的连接的读事件就会通过 epoll_ctl 函数添加到 epoll 中并且添加定时器, 定时器的作用是: 当指定时间内, 请求头还未完整发完, 就超时(`client_header_timeout`), Nginx 会返回 408 状态码到客户端。

`client_header_timeout` 参数的验证:
```bash 
$ cat nginx.conf   # 确保重启Nginx让配置生效
http {
    ...
    client_header_timeout 10s;
    ...
}

# telnet 127.0.0.1 80
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
# 10s 后, 会输出这行日志, 
Connection closed by foreign host.   
```
以下是tcpdump的结果:
```bash
tcpdump tcp -i lo0 -t -s 0 -c 100 and dst port 80 and src host 127.0.0.1
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on lo0, link-type NULL (BSD loopback), snapshot length 524288 bytes
# telnet 会打印下面两行日志
IP localhost.54727 > localhost.http: Flags [S], seq 2169026392, win 65535, options [mss 16344,nop,wscale 6,nop,nop,TS val 1950842678 ecr 0,sackOK,eol], length 0
IP localhost.54727 > localhost.http: Flags [.], ack 2893272354, win 6380, options [nop,nop,TS val 1950842678 ecr 1099374196], length 0
# 超过10s后会再打印两行日志
IP localhost.54727 > localhost.http: Flags [.], ack 2, win 6380, options [nop,nop,TS val 1950852681 ecr 1099384199], length 0
IP localhost.54727 > localhost.http: Flags [F.], seq 0, ack 2, win 6380, options [nop,nop,TS val 1950852681 ecr 1099384199], length 0
```

`client_header_timeout` 和 `keepalive_timeout` 的区别:
- `client_header_timeout`: Nginx在指定之间内, 未收到完整的 Header, 就会超时。比如 Header 包含一个超大的 Cookie, 就可能被这个参数关闭连接;
- `keepalive_timeout`: Nginx 在三次握手后, http 层面完成了 `请求-响应`, 在等待下次 `请求-响应` 间, 超过了这个参数设置的秒, 会触发超时. 这个参数控制的是 HTTP 超时, 而不是 TCP 超时, 也就是说它关闭的是 HTTP。


当客户端的数据发过来后, 内核会响应 ACK, 同时, 同时, 事件模块的 epoll_wait 会拿到这个请求, 这时会调用设置回调方法 `ngx_http_wait_request_handler`, 下面是它的说明:
1. `ngx_http_wait_request_handler` 将接受到的用户请求读取到用户态中, 读取到用户态就需要操作系统分配对应的内存;
2. 分配的内存大小参数是 `client_header_buffer_size: 1k`, 它会分配读取用户数据的读缓冲区;
3. `client_header_buffer_size` 的内存是从 `connection_pool_size` 中分出来的(`connection_pool_size` 可以扩展), 这个参数也不是越大越好, 如果用户发送的数据量小, 这个内存设置的很大, worker 进程也会分配这么多内存出来; 如果用户的数据超过了设置的 1k, 会根据 `large_client_header_buffers 2 4k` 进行扩容.

下图是Nginx 处理HTTP 请求头部的流程, 取自 [极客时间陶辉老师-Nginx核心知识150讲](https://time.geekbang.org/course/detail/100020301-71458) 的课件截图

![Nginx处理http请求头的流程2](../../img/nginx/http/Nginx处理http请求头的流程2.png)

对于图片中的流程我是表示质疑的, 理由如下:

一个客户端连接到Nginx, 必定经过TCP握手, 前两次握手在操作系统上属于半连接状态, 只有第三次握手之后, 才会触发Nginx的epoll, 这个在上面说过, 这里要明确的第一个参数就是 `connection_pool_size` 维护着TCP连接的参数, 连接建立完成之后, 客户端就开始真的发送数据, 客户端发来的数据首先是放到操作系统内核中的, 这是大小可以由参数 `client_header_buffer_size` 控制, 如果请求头超过了 `client_header_buffer_size` 大小, 就会扩容, 按照 `large_client_header_buffers 2 4k` 进行扩容, 每次扩容是创建一个新的空间, 把之前的数据拷贝过来, 释放之前占用的空间, 假如请求头真的太大太多, 扩容之后还存不下, 就会报错: `Request Header Or Cookie Too Large`. 

> 上述整个过程, 数据是未经过状态机解析的, 这点要明白.

当请求头接收到了之后, 会经过状态机解析, 状态机会解析出URI、Method等等信息, 解析出来的数据存放到 `request_pool_size` 中, `request_pool_size` 如果需要扩容, 并不由 `large_client_header_buffers` 参数决定,  `request_pool_size` 扩容是在本身的pool上进行追加, 每次追加的大小受到操作系统影响.


在这期间, 会涉及一个参数 `client_header_timeout`, 这个参数是说客户端发送的请求中, 因为请求头过大, 导致在 `client_header_timeout` 时间内`没有传输完/未解析完` 请求头, 就会超时, 如果及时传输完了, 就移除超时定时器, 然后开始11个阶段的http请求处理。



### 4.1.1 TCP 连接建立阶段

#### 4.1.1.1 tcp 连接建立流程

```mermaid
sequenceDiagram
    participant Client
    participant OS_Kernel
    participant Nginx
    Client->>OS_Kernel: SYN
    OS_Kernel-->>Client: SYN+ACK
    Client->>OS_Kernel: ACK
    Note over OS_Kernel,Nginx: 三次握手完成
    OS_Kernel->>Nginx: epoll事件触发
    Nginx->>Nginx: 分配connection_pool
```

#### 4.1.1.2 连接池管理

- `connection_pool_size`:  维护 TCP 连接 的核心参数
  - 存储连接状态、SSL上下文等元数据;
  - 声明周期绑定TCP连接(Keep-Alive期间持续存在)
- 连接池特点:
  - 每个TCP连接对应一个连接池;
  - 连接关闭时自动释放资源;
  - 支持连接复用, 减少重复创建开销;

### 4.1.2 请求头接收阶段

#### 4.1.2.1 缓冲区层级结构

```mermaid
graph TD
    A[操作系统内核缓冲区] --> B[client_header_buffer]
    B --> C{是否溢出？}
    C -- 是 --> D[large_client_header_buffers]
    C -- 否 --> E[状态机解析]
    D --> F{缓冲区链空间耗尽？}
    F -- 是 --> G[返回400错误]
    F -- 否 --> E
```

#### 4.1.2.2 缓冲机制详解

1. 初始缓冲层:
   1. `client_header_buffer_size`:  默认1k;
   2. 存储请求行和初始头部数据;
   3. 空间不足时触发扩容;
2. 动态扩容层:
   1. `large_client_header_buffers`: `2 4k`;
   2. 缓冲区耗尽时返回 `400 Bad Request`

3. 错误场景:
   1. 单行超过缓冲区大小: `414 Request-URI Too Large`;
   2. 总头部超过缓冲区大小: `400 Bad Request`;



### 4.1.3 请求头解析阶段

#### 4.1.3.1 解析流程

```mermaid
flowchart TB
    A[检测到\r\n\r\n] --> B[创建request_pool]
    B --> C[状态机解析请求行]
    C --> D[存储method/URI/version]
    D --> E[解析头字段]
    E --> F[存储key-value]
    F --> G{request_pool空间不足？}
    G -- 是 --> H[追加新内存块]
    G -- 否 --> I[继续解析]
```



#### 4.1.3.2 请求内存池特性

1. `request_pool_size`:
   1. 存储解析后的结构化数据(URI、Method、Header 等);
   2. 默认4k初始大小;
   3. 生命周期绑定HTTP请求(非TCP连接);
2. 扩容机制:
   1. 空间不足时自动追加内存块;
   2. 块大小由系统内存分配策略决定;
   3. 典型追加大小: 16kb(Linux glibc);
   4. 与 `large_client_header_buffers` 完全没关系;



### 4.1.4 超时控制机制

#### 4.1.4.1 `client_header_timeout` 作用于

```mermaid
timeline
    title client_header_timeout 控制范围
    section 包含阶段
      接收首字节 ： 0ms
      传输数据 ： 持续计时
      扫描结束符 ： 关键检测点
    section 排除阶段
      结构化解析 ： 定时器已移除
      HTTP处理阶段 ： 不受此限
```

#### 4.1.4.2 超时场景分析

1. 触发条件:
   1. 客户端传输速率低于阈值;
   2. 网络丢包导致传输中断
   3. 客户端恶意不发送结束符
2. 典型表现:
   1. 返回 `408 Request Timeout`;
   2. 连接资源立即被释放;
   3. 错误日志: `client time out ... while reading client request headers`;

3. 验证方法:

   ```bash
   # 模拟慢速攻击
   $ vim test.py
   import socket
   import time
   
   # 配置目标主机和端口（通常是80）
   HOST = '192.168.1.153'  # 可以是IP或域名
   PORT = 80
   
   # 模拟行为：
   # 1. 建立 TCP 连接
   # 2. 发送部分 header
   # 3. 停顿超过 Nginx 设置的 client_header_timeout（比如设为 5 秒，就停 6 秒）
   # 4. 继续发送剩下的 header
   # 5. 尝试读取响应（通常会被 Nginx 强制关闭）
   
   # 建立 socket 连接
   sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
   sock.settimeout(10)  # 设置最大等待时间，防止卡住
   sock.connect((HOST, PORT))
   
   try:
       print("[*] Step 1: Sending partial headers...")
       sock.sendall(b"GET / HTTP/1.1\r\n")
       sock.sendall(f"Host: {HOST}\r\n".encode())
   
       print("[*] Step 2: Sleeping for 6 seconds to trigger client_header_timeout...")
       time.sleep(30)  # > client_header_timeout (e.g., if it's 5s)
   
       print("[*] Step 3: Sending the rest of the headers...")
       sock.sendall(b"Connection: close\r\n\r\n")
   
       print("[*] Step 4: Reading response from server...")
       response = b""
       while True:
           chunk = sock.recv(1024)
           if not chunk:  # 收到空数据表示连接关闭
               break
           response += chunk
   
       if response:
           print(response.decode(errors="ignore"))
       else:
           print("[!] No response (connection closed by server)")
   
   except socket.timeout:
       print("[!] Socket timeout: server didn't respond (possibly closed connection)")
   except socket.error as e:
       print(f"[!] Socket error: {e}")
   finally:
       sock.close()
   
   
   # Nginx配置
   http {
       connection_pool_size 512;
       client_header_timeout 3s;
       keepalive_timeout 5s;
       client_header_buffer_size 1k;
       large_client_header_buffers 1 1k;
       request_pool_size 256;
   
       log_format  main  '$remote_addr - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent" '
                     'request_time=$request_time '
                     'connection=$connection connection_requests=$connection_requests '
                     'host="$host" upstream_addr="$upstream_addr" '
                     'http_version=$server_protocol '
                     'body_bytes_sent=$body_bytes_sent '
                     'request_length=$request_length';
   
       error_log /usr/local/nginx/logs/error.log info;
   }
   
   $ python3 test.py
   python3 test.py
   [*] Step 1: Sending partial headers...
   [*] Step 2: Sleeping for 6 seconds to trigger client_header_timeout...
   [*] Step 3: Sending the rest of the headers...
   [*] Step 4: Reading response from server...
   [!] No response (connection closed by server)
   
   # 日志输出
   ==> logs/error.log <==
   2025/06/03 19:26:19 [info] 29913#0: *41 client timed out (110: Connection timed out) while reading client request headers, client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.1", host: "localhost"
   
   ==> logs/access.log <==
   127.0.0.1 - - [03/Jun/2025:19:26:19 +0800] "GET / HTTP/1.1" 408 0 "-" "-"
   ```
   
   上面看到 python 仍然输出了 Step 3...等日志, 正常来说应该是不会有的, 所以用wireshark抓包, 分析过程如下:

   ![wireshark抓包](../../img/nginx/http/client_header_timeout.png)

   - 序号1-3数据报文: TCP 3次握手.
   - 序号4-7: python 发送数据包, 服务器回应 ACK.
   - 序号8: Nginx `client_header_timeout` 参数生效. Nginx 主动断开连接, 发送 [FIN,ACK] 数据报文, 此时服务端TCP状态是 `FIN_WAIT_1`.
   - 序号9: 客户端回应[FIN,ACK], 发送 [ACK], 此时服务端TCP状态是 `FIN_WAIT_2`. 此时此刻, TCP 处于半连接状态, 也就是服务端不再接收客户端数据, 但是, **客户端仍然可以发送数据给服务端**
   - 序号10: python sleep 结束, 发送数据.
   - 序号11: 客户端发送 [FIN,ACK].
   - 序号12-13: 服务端(Nginx)拒接连接, 发送 [RST].

   对应的TCP连接截图

   python sleep 前:

   ![wireshark抓包](../../img/nginx/http/client_header_timeout2.png)

    python sleep, 并且超过 Nginx `client_header_timeout` 设置时间, TCP连接处于半连接状态

   ![wireshark抓包](../../img/nginx/http/client_header_timeout3.png)

   python sleep 结束, TCP 连接完全关闭

   ![wireshark抓包](../../img/nginx/http/client_header_timeout4.png)



## 4.2 HTTP 请求的 11 个阶段

![HTTP请求的11个阶段](../../img/nginx/http/HTTP请求的11个阶段.png)

![11个阶段的顺序处理](../../img/nginx/http/11个阶段的顺序处理.png)


### 4.2.1 post read 阶段: realip 模块 发现真实IP 

https://nginx.org/en/docs/http/ngx_http_realip_module.html

说明: 
   1. ngx_http_realip_module 模块用于将客户端地址和可选端口更改为指定头字段中发送的地址和可选端口。
   2. 此模块不是默认构建的，应使用 --with-http_realip_module 配置参数。

变量:
   - `realip_remote_addr`: 保留原始客户端地址
   - `realip_remote_port`: 保留原始客户端端口

realip提供的指令:
   ```bash
   # 定义合法的来源地址。如果指定特殊值 unix: 则所有 UNIX 域套接字都将被信任。也可以使用主机名 (1.13.1) 指定可信地址。
   Syntax:  set_real_ip_from address | CIDR | unix:;
   Default: --
   Context: http,server,location
   
   # 来源合法, 取 real_ip_header 设置的记录, 可以是 X-Real-IP 和 X-Forwarded-For、proxy_protocol.
   # 这个值会替换 $remote_addr 的记录.
   Syntax:  real_ip_header field | X-Real-IP | X-Forwarded-For | proxy_protocol;
   Default: real_ip_header X-Real_IP;
   Context: http,server,location
   
   # 如果禁用递归搜索，则原始客户端地址 匹配其中一个受信任的地址被最后一个替换 在请求头字段中发送的地址由 real_ip_header 指令。
   # 如果启用了递归搜索，则与其中一个可信地址匹配的原始客户端地址将被请求标头字段中发送的最后一个非可信地址替换。
   Syntax:  real_ip_recursive on | off;
   Default: real_ip_recursive off;
   Context: http,server,location
   ```

案例, 能够分析出如下几个变量的值, 就算过关:
```bash
$remote_addr
$http_x_real_ip
$proxy_add_x_forwarded_for
$realip_remote_addr
$realip_remote_port
```

案例步骤:
```bash
$ ip addr show lo0
1: lo0: <UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384 status UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8
    inet6 ::1/128
    inet6 fe80::1/64
    inet 192.168.1.201/24
    inet 192.168.1.202/24
    inet 192.168.1.203/24
    inet 192.168.1.204/24
    

$ ip addr show en0
14: en0: <UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500 status UP
    link/ether 12:ce:36:31:f3:bb brd ff:ff:ff:ff:ff:ff
    inet6 fe80::1c0f:4f83:602:31c0/64
    inet 192.168.1.149/23 brd 192.168.1.255    

$ vim nginx.conf
# 增加日志
log_format custom_log_format '$remote_addr - [$time_local] '
                             'XFF="$http_x_forwarded_for" '
                             'X-Real-IP="$http_x_real_ip" '
                             'realip_remote_addr="$realip_remote_addr" '
                             'proxy_add_x_forwarded_for="$proxy_add_x_forwarded_for"';
$ vim realip.conf
server {
    listen 192.168.1.149:80;

    location / {
        proxy_bind 192.168.1.201;

        access_log /opt/homebrew/etc/nginx/logs/149.log custom_log_format;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://192.168.1.202:80;
    }
}

server {
    listen 192.168.1.202:80;

    location / {
        proxy_bind 192.168.1.202;

        access_log /opt/homebrew/etc/nginx/logs/202.log custom_log_format;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        set_real_ip_from 192.168.0.0/22;
        real_ip_header X-Forwarded-For;
        real_ip_recursive on;

        proxy_pass http://192.168.1.203:80;
    }
}

server {
    listen 192.168.1.203:80;

    location / {
        proxy_bind 192.168.1.203;

        access_log /opt/homebrew/etc/nginx/logs/203.log custom_log_format;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        set_real_ip_from 192.168.0.0/22;
        real_ip_header X-Forwarded-For;
        real_ip_recursive on;

        proxy_pass http://192.168.1.204:80;
    }
}

server {
    listen 192.168.1.204:80;

    location / {
        proxy_bind 192.168.1.204;

        access_log /opt/homebrew/etc/nginx/logs/204.log custom_log_format;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        return 200 "----- Final Nginx (204) -----\nremote_addr: $remote_addr\nX-Real-IP: $http_x_real_ip\nX-Forwarded-For: $http_x_forwarded_for\nrealip_remote_addr: $realip_remote_addr\nproxy_add_x_forwarded_for: $proxy_add_x_forwarded_for\n";
    }
}

$ curl http://192.168.1.149:80 -H "X-Forwarded-For: 2.2.2.2,3.3.3.3,4.4.4.4"
Hello Client

# 输出日志
==> logs/204.log <==
192.168.1.203 - [05/Jun/2025:14:31:07 +0800] XFF="2.2.2.2,3.3.3.3,4.4.4.4, 192.168.1.153, 4.4.4.4, 4.4.4.4" X-Real-IP="4.4.4.4" realip_remote_addr="192.168.1.203" proxy_add_x_forwarded_for="2.2.2.2,3.3.3.3,4.4.4.4, 192.168.1.153, 4.4.4.4, 4.4.4.4, 192.168.1.203"

==> logs/203.log <==
4.4.4.4 - [05/Jun/2025:14:31:07 +0800] XFF="2.2.2.2,3.3.3.3,4.4.4.4, 192.168.1.153, 4.4.4.4" X-Real-IP="4.4.4.4" realip_remote_addr="192.168.1.202" proxy_add_x_forwarded_for="2.2.2.2,3.3.3.3,4.4.4.4, 192.168.1.153, 4.4.4.4, 4.4.4.4"

==> logs/202.log <==
4.4.4.4 - [05/Jun/2025:14:31:07 +0800] XFF="2.2.2.2,3.3.3.3,4.4.4.4, 192.168.1.153" X-Real-IP="192.168.1.153" realip_remote_addr="192.168.1.201" proxy_add_x_forwarded_for="2.2.2.2,3.3.3.3,4.4.4.4, 192.168.1.153, 4.4.4.4"

==> logs/149.log <==
192.168.1.153 - [05/Jun/2025:14:31:07 +0800] XFF="2.2.2.2,3.3.3.3,4.4.4.4" X-Real-IP="-" realip_remote_addr="192.168.1.153" proxy_add_x_forwarded_for="2.2.2.2,3.3.3.3,4.4.4.4, 192.168.1.153"
```


### 4.2.2 server_rewrite 阶段 和 rewrite 

https://nginx.org/en/docs/http/ngx_http_rewrite_module.html

- `server_rewrite` 是第二阶段;
- `rewrite` 是第四阶段;

这两个阶段拥有同一个 `rewrite` 模块, 执行顺序是 `server_rewrite` -> `find_config` -> `rewrite`.

#### 4.2.2.1 rewrite 模块 return 指令

https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return

return指令:
   ```bash
   Syntax:  return code [text];
            return code URL;
            return URL;           # 默认code=302
   Default: --
   Context: server,location,if
   
   Syntax:  error_page code ... [=[response]]] uri;
            return code URL;
            return URL;           # 默认code=302
   Default: --
   Context: http,server,location,if in location
   ```
返回状态码:
- Nginx 自定义
  - 444: 关闭连接
- HTTP 1.0 标准
  - 301: HTTP 永久重定向; 浏览器会缓存, 下次访问会直接访问重定向方;
  - 302: 临时重定向, 禁止被缓存;
- HTTP 1.1 标准
  - 303: 临时重定向, 允许改变方法, 禁止被缓存;
  - 307: 临时重定向, 不允许改变方法, 禁止被缓存;
  - 308: 永久重定向, 不允许改变方法;

如何区分 `return` 指令属于哪个阶段:
1. `server_rewrite` 阶段: `return` 指令直接出现在 `server {` 下面;
2. `rewrite` 阶段: `return` 指令出现在 `location {` 下面;

所以, `server` 下面的 `return` 比 `location` 下面的 `return` 先发挥作用.


案例:
```bash
server {
        listen 192.168.1.149:80;

        root /opt/homebrew/etc/nginx/html/;
        error_page 404 =308 404.html;
        return 405;

        location / {
                return 404 "return ";
        }
}
```
这个案例中, 第一个 `return 405` 最先生效, 后续都不会执行.
```bash
$ curl 192.168.1.149:80
<html>
<head><title>405 Not Allowed</title></head>
<body>
<center><h1>405 Not Allowed</h1></center>
<hr><center>nginx/1.27.4</center>
</body>
</html>
```
注释掉 `return 405` 之后, location 下的 `return` 最先生效
```bash
$ curl -v 192.168.1.149:80/1
*   Trying 192.168.1.149:80...
* Connected to 192.168.1.149 (192.168.1.149) port 80
> GET / HTTP/1.1
> Host: 192.168.1.149
> User-Agent: curl/8.9.1
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 404 Not Found
< Server: nginx/1.27.4
< Date: Thu, 05 Jun 2025 08:21:03 GMT
< Content-Type: application/octet-stream
< Content-Length: 7
< Connection: keep-alive
<
* Connection #0 to host 192.168.1.149 left intact
return %
```
把 location 下的 return 也注释掉,
```bash
$ curl -v -L 192.168.1.149:80/1
*   Trying 192.168.1.149:80...
* Connected to 192.168.1.149 (192.168.1.149) port 80
> GET /1 HTTP/1.1
> Host: 192.168.1.149
> User-Agent: curl/8.9.1
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 308 Permanent Redirect
< Server: nginx/1.27.4
< Date: Thu, 05 Jun 2025 08:25:30 GMT
< Content-Type: text/html
< Content-Length: 171
< Connection: keep-alive
< Location: 404.html
* Ignoring the response-body
<
* Connection #0 to host 192.168.1.149 left intact
* Issue another request to this URL: 'http://192.168.1.149:80/404.html'
* Found bundle for host: 0x600003bc89f0 [serially]
* Can not multiplex, even if we wanted to
* Re-using existing connection with host 192.168.1.149
> GET /404.html HTTP/1.1
> Host: 192.168.1.149
> User-Agent: curl/8.9.1
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx/1.27.4
< Date: Thu, 05 Jun 2025 08:25:30 GMT
< Content-Type: text/html
< Content-Length: 4
< Last-Modified: Thu, 05 Jun 2025 06:54:58 GMT
< Connection: keep-alive
< ETag: "68413f42-4"
< Accept-Ranges: bytes
<
404
* Connection #0 to host 192.168.1.149 left intact
```

`error_page` 位于第十个阶段(content), 它的作用是当 server 和 location 都没有return, error_page 就发挥作用


#### 4.2.2.2 rewrite 模块 rewrite 指令 - 重写URL

https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite

rewrite指令:
   ```bash
   Syntax:  rewrite regex replacement [flag];
   Default: --
   Context: server,location,if
   ```
功能:
- 将 regex 指定的 url 替换成 replacement 这个新的 url. 可以使用正则表达式及变量提取.
- 当 replacement 以 http:// 或 https:// 或者 $schema 开头, 则直接返回 302 重定向.
- 替换后的 url 根据 flag 指定的方式进行处理:
  - `last`: 用 replacement 这个 uri 进行新的 location 匹配;
  - `break`: break 指令停止当前脚本指令的执行, 等价于独立的 break 指令;
  - `redirect`: 返回 302 重定向;
  - `permanent`: 返回 301 重定向; 
  - 空着: 继续执行rewrite下面的指令;


案例1:
```bash
$ cat html/three/1.txt
three/1.txt

$ vim rewrite.conf
server {
	listen 192.168.1.149:80;

	types {}
	default_type text/html;

	location /one {
		rewrite /one(.*) /two$1 last;
		return 200 "one";
	}
	location /two {
	    # curl http://192.168.1.149:80/1.txt 的结果
	    ## break: 停止执行, 返回 html/three/1.txt 中的内容
	    ## last: 重新进行 location 匹配, 返回 three
	    ## redirect: 发两次请求(302), 返回 three
	    ## permanent: 发两次请求(301), 返回 three
	    ## 空着: 重新进行 location 匹配, 返回 two
		rewrite /two(.*) /three$1 break;  
		return 200 "two";
	}
	location /three {
		return 200 "three";
	}
}
```
案例2:
```bash
server {
        listen 192.168.1.149:80;

        types {}
        default_type text/html;

        root /opt/homebrew/etc/nginx/html/three;

        location /one {
                rewrite /one(.*) $1 permanent;
        }
        location /two {
                rewrite /two(.*) $1 redirect;
        }
        location /three {
                rewrite /three(.*) http://192.168.1.149:80$1;
        }
        location /four {
                rewrite /four(.*) http://192.168.1.149:80$1 permanent;
        }
        location /five {
                rewrite /five(.*) http://192.168.1.149:80$1 break;
        }
}
```


#### 4.2.2.3 rewrite 模块 if 指令 - 条件判断

https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#if

```bash
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}

if ($request_method = POST) {
    return 405;
}

if ($slow) {
    limit_rate 10k;
}

if ($invalid_referer) {
    return 403;
}
```

### 4.2.5 find_config 阶段: 找到请求处理的 location 指令

https://nginx.org/en/docs/http/ngx_http_core_module.html#location

rewrite指令:
```bash
Syntax:  location [ = | ~ | ~* | ^~ ] uri {...}
         location @name {...}
Default: --
Context: server,location
```
1. `/` 通用匹配, 任何请求都会匹配到.
2. `=` 精确匹配.
3. `~` 正则匹配, 区分大小写.
4. `~*` 正则匹配, 不区分大小写.
5. `^~` 非正则匹配, 匹配以指定模式开头的 location.
6. `*` 代表'任意字符'.
7. `$` 以'什么结尾的匹配'.
8. `^` 以'什么开头'的匹配.
9. `!~*` 不区分大小写匹配失败.
10. `!~` 区分大小写匹配'失败-->!(取反)'.

**备注： `~`开头的都是`正则匹配`,例如`^~`不是正则匹配.**

location 转义
```bash
~^/prefix/.*\.html$ 
 
解释：~ 表示后面跟的是'正则',而且是区分大小写的（ "~ "区分大小写,"~* "不区分大小写）
 
~^/prefix/.*\.html$  就是'正则表达式了'
 
1) ^在正则里表示,以什么'开始'
 
2) /prefix/ 表示符合这个'文件夹路径的'
 
3) ".*" 表示匹配'单个字符多次'
 
4) "\." 表示转义 "."  采用 "." 本身,而非他在'正则里'的意思（非\r\n的单个字符）。
 
5) $ 表示以什么'结尾'
```


location 匹配优先级流程图:
```mermaid
flowchart TD
A[遍历匹配全部前缀字符串 location] --> B{匹配上 = 字符串}
B -- 是 --> C[使用匹配上的 = 精确匹配 location]
B -- 否 --> D{匹配上 ^~ 字符串}
D -- 否 --> F[记住最长匹配的前缀字符串 location]
D -- 是 --> E[使用匹配上的 ^~ 字符串 location]
F --> G[按 nginx.conf 中的顺序依次匹配正则表达式 location]
G --> H{匹配}
H -- 是 --> I[使用匹配上的正则表达式]
H -- 否 --> J[所有正则表达式都不匹配<br>使用最长匹配的前缀字符串 location]
```

location 案例:
```bash
$ vim location.conf
server {
    listen 192.168.1.149:80;
    # 大小写敏感的正则匹配
    location ~ /Test/$ {
        return 200 'first regular expressions match!\n';
    }
    # 忽略大小写的正则匹配
    location ~* /Test1/(\w+)$ {
        return 200 'longest regular expressions match!\n';
    }
    # 匹配上之后不再进行正则匹配
    location ^~ /Test1/ {
        return 200 'stop regular expressions match!\n';
    }
    # 普通的前缀匹配
    location  /Test1/Test2 {
        return 200 'longest prefix string match!\n';
    }
    # 普通的前缀匹配
    location  /Test1 {
        return 200 'prefix string match!\n';
    }
    # 精确匹配
    location  = /Test1 {
        return 200 'exact match! /Test1 \n';
    }
    # 精确匹配
    location  = /Test1/ {
        return 200 'exact match! /Test1/ \n';
    }
}
```
测试:
```bash
$ curl http://192.168.1.149:80/Test1
exact match! /Test1

$ curl http://192.168.1.149:80/Test1/
exact match! /Test1/

$ curl http://192.168.1.149:80/Test11
prefix string match!

$ curl http://192.168.1.149:80/Test1/11
stop regular expressions match!

$ curl http://192.168.1.149:80/Test1/Test2
longest regular expressions match!
```
`curl http://192.168.1.149:80/Test1/Test2` 匹配的详细流程:
1. Nginx 先遍历出所有前缀匹配的 location, 得到如下4个结果:
   ```bash
   location ~* /Test1/(\w+)$
   location ^~ /Test1/
   location /Test1/Test2 
   location /Test1
   ```
2. 是否有精确匹配? **没有**
3. 是否有 ^~ 匹配? **有, 但是路径不完全匹配, 必须要匹配正则的**
4. 是否有正则匹配? **有忽略大小写的正则匹配, 正好合适**
5. 返回 `longest regular expressions match!`

### 4.2.6 preaccess 
#### 4.2.6.1 对连接做限制的 limit_conn 模块

https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html

`ngx_http_limit_conn_module` 模块用于 限制每个 `自定义KYE` 的连接数.

什么是`自定义KEY`, 示例配置:
```bash
# 这里的 自定义KEY 就是 $binary_remote_addr 变量的值
limit_conn_zone $binary_remote_addr zone=one:10;

# 这里的 自定义KEY 就是 $remote_addr 变量的值
limit_conn_zone $remote_addr zone=one:10;

# 这里的 自定义KEY 就是 $http_x_real_ip 变量的值
limit_conn_zone $http_x_real_ip zone=one:10;
```

limit_conn模块的指令:
```bash
# 设置共享内存, 该区域将保存各种KEY的状态. 包括当前连接数. KEY 可以包含文本、变量、组合. KEY 为空的请求不予处理.
# 如果该区域内存空间耗尽, 服务器将返回 `limit_conn_status 503(default: 503)` 状态码
Syntax:  limit_conn_zone key zone=name:size;
Default: --
Context: http,server,location

# 使用示例: 设置一个10m大小, 名为one的共享内存. 
limit_conn_zone $remote_addr zone=one:10;

---
# 设置响应被拒绝的请求的状态码
Syntax:  limit_conn_status code;
Default: limit_conn_status 503;
Context: http,server,location

---
# 设置共享内存区域, 以及该区域自定义KEY允许的最大连接数.
# 只有当服务器正在处理请求, 并且已经读取整个请求的Header时, 该连接才会被计数.
Syntax:  limit_conn zone number;
Default: --
Context: http,server,location
```

案例:
```bash
log_format limit
                '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '$binary_remote_addr '
                '$http_user_agent "$http_x_forwarded_for"'
                'XFF="$http_x_forwarded_for" '
                'X-Real-IP="$http_x_real_ip" '
                'realip_remote_addr="$realip_remote_addr" '
                'proxy_add_x_forwarded_for="$proxy_add_x_forwarded_for"';

limit_conn_zone $binary_remote_addr zone=one:10m;

server {
        listen 192.168.1.149:80;
        access_log /opt/homebrew/etc/nginx/logs/limit.log limit;

        root /opt/homebrew/etc/nginx/html/;
        location / {
                set_real_ip_from 192.168.0.0/22;
                real_ip_header X-Forwarded-For;
                real_ip_recursive on;

                # 连接数限制后, 响应状态码返回 500
                limit_conn_status 500;
                limit_conn_log_level warn;
                # 响应速率限制 50字节/s
                limit_rate 100;
                # 每个IP只允许8次连接
                limit_conn one 8;


                # 使用共享空间 one, 同一个ip调用时, 支持5个突发请求, nodelay 表示立即响应
                # limit_req zone=one burst=5; # nodelay;
        }
}
```
![Nginx_Limit_Conn_压测1](../../img/nginx/http/Nginx_Limit_Conn_压测1.png)

![Nginx_Limit_Conn_压测2](../../img/nginx/http/Nginx_Limit_Conn_压测2.png)

![Nginx_Limit_Conn_压测3](../../img/nginx/http/Nginx_Limit_Conn_压测3.png)

压测日志:
```bash
3.3.3.3 - - [06/Jun/2025:15:03:47 +0800] "GET / HTTP/1.1" 500 177 "-" \x03\x03\x03\x03 Apache-HttpClient/4.5.14 (Java/1.8.0_432) "1.1.1.1,2.2.2.2,3.3.3.3"XFF="1.1.1.1,2.2.2.2,3.3.3.3" X-Real-IP="-" realip_remote_addr="192.168.1.149" proxy_add_x_forwarded_for="1.1.1.1,2.2.2.2,3.3.3.3, 3.3.3.3"
3.3.3.3 - - [06/Jun/2025:15:03:47 +0800] "GET / HTTP/1.1" 500 177 "-" \x03\x03\x03\x03 Apache-HttpClient/4.5.14 (Java/1.8.0_432) "1.1.1.1,2.2.2.2,3.3.3.3"XFF="1.1.1.1,2.2.2.2,3.3.3.3" X-Real-IP="-" realip_remote_addr="192.168.1.149" proxy_add_x_forwarded_for="1.1.1.1,2.2.2.2,3.3.3.3, 3.3.3.3"
3.3.3.3 - - [06/Jun/2025:15:03:47 +0800] "GET / HTTP/1.1" 500 177 "-" \x03\x03\x03\x03 Apache-HttpClient/4.5.14 (Java/1.8.0_432) "1.1.1.1,2.2.2.2,3.3.3.3"XFF="1.1.1.1,2.2.2.2,3.3.3.3" X-Real-IP="-" realip_remote_addr="192.168.1.149" proxy_add_x_forwarded_for="1.1.1.1,2.2.2.2,3.3.3.3, 3.3.3.3"
...
3.3.3.3 - - [06/Jun/2025:15:03:50 +0800] "GET / HTTP/1.1" 200 476 "-" \x03\x03\x03\x03 Apache-HttpClient/4.5.14 (Java/1.8.0_432) "1.1.1.1,2.2.2.2,3.3.3.3"XFF="1.1.1.1,2.2.2.2,3.3.3.3" X-Real-IP="-" realip_remote_addr="192.168.1.149" proxy_add_x_forwarded_for="1.1.1.1,2.2.2.2,3.3.3.3, 3.3.3.3"
...
```

#### 4.2.6.2 preaccess 阶段: 对连接做限制的 limit_req 模块

### 4.2.7 access 阶段
#### 4.2.7.1 对ip做限制的access模块

#### 4.2.7.2 对用户名密码做限制的auth_basic模块

#### 4.2.7.3 使用第三方做权限控制的auth_request模块

#### 4.2.7.4 satisty 指令

### 4.2.8 precontent 阶段
#### 4.2.8.1 按序访问资源的try_files模块

#### 4.2.8.2 mirror 模块

### 4.2.9 content 阶段
#### 4.2.9.1 root 和 alias 指令

#### 4.2.9.2 static 模块


#### 4.2.9.3 index和autoindex 模块

#### 4.2.9.4 concat 模块

### 4.2.10 log阶段
#### 4.2.10.1 access 日志

#### 4.2.10.2 过滤模块


# 五、反向代理
## 5.1 示例1
效果: 在浏览器中输入 `www.kino.com` 跳转到 tomcat 主页面中

### 5.1.1 部署 tomcat

[下载tomcat](https://tomcat.apache.org/download-90.cgi)

###
```bash
$ tar -zxvf apache-tomcat-9.0.41.tar.gz
$ mv apache-tomcat-9.0.41 tomcat-8080
$ cd tomcat-8080
$ bin/startop.sh
$ ps -ef | grep tomcat
root      18406      1 41 19:12 pts/1    00:00:02 /usr/local/jdk1.8.0_131/bin/java -Djava.util.logging.config.file=/opt/software/tomcat-8080/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /opt/software/tomcat-8080/bin/bootstrap.jar:/opt/software/tomcat-8080/bin/tomcat-juli.jar -Dcatalina.base=/opt/software/tomcat-8080 -Dcatalina.home=/opt/software/tomcat-8080 -Djava.io.tmpdir=/opt/software/tomcat-8080/temp org.apache.catalina.startup.Bootstrap start
root      18440  18303  0 19:13 pts/1    00:00:00 grep --color=auto tomcat
```
如果有防火墙, 开启8080 端口
```bash
$ firewall-cmd --add-port=8080/tcp --permanent
success
$ firewall-cmd --reload
```
在浏览器中输入: 虚拟机ip:8080, 即可访问 tomcat 页面

![tomcat-8080](../../img/nginx/tomcat-80.png)

### 5.1.2 配置 Nginx
```config
$ vim conf/nginx.conf
server {
        listen       80;
        server_name  192.168.220.111;
        location / {
            proxy_pass  http://192.168.220.111:8080;
        }
...
```
重新加载配置文件
```bash
$ sbin/nginx -s reload
```
在浏览器中输入: www.kino.com

![反向代理1](../../img/nginx/kino-com.png)


## 5.2 示例2
效果: 根据不同的路径跳转到不同的端口服务中, nginx 监听 9091端口, 访问: `192.168.220.111:9091/edu` 跳转到 8080的tomcat, 访问 `192.168.220.111:9092/vod` 跳转到 8081 端口的tomcat

准备两个tomcat, 修改 tomcat 配置文件改端口
```bash
$ cp -R tomcat-8080 tomcat-8081
$ vim tomcat-8081/conf/server.xml
<Server port="8005" shutdown="SHUTDOWN">
改为
<Server port="8006" shutdown="SHUTDOWN">

<Connector port="8080" protocol="HTTP/1.1"
改为
<Connector port="8081" protocol="HTTP/1.1"
```
在两个tomcat 的 webapps 目录下创建 nginx 目录, 并创建 login.html
```bash
$ mkdir tomcat-8080/webapps/edu
$ mkdir tomcat-8081/webapps/vod
$ vim tomcat-8080/webapps/edu/login.html
<html>
  <head>
    <title>tomcat-8080</title>
  </head>
  <body>
    <h1>tomcat-8080</h1>
  </body>
</html>
$ vim tomcat-8080/webapps/vod/login.html
<html>
  <head>
    <title>tomcat-8081</title>
  </head>
  <body>
    <h1>tomcat-8081</h1>
  </body>
</html>
```
启动两个tomcat
```bash
$ tomcat-8080/bin/startup.sh 
Using CATALINA_BASE:   /opt/software/tomcat-8080
Using CATALINA_HOME:   /opt/software/tomcat-8080
Using CATALINA_TMPDIR: /opt/software/tomcat-8080/temp
Using JRE_HOME:        /usr/local/jdk1.8.0_131
Using CLASSPATH:       /opt/software/tomcat-8080/bin/bootstrap.jar:/opt/software/tomcat-8080/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
$ tomcat-8081/bin/startup.sh 
Using CATALINA_BASE:   /opt/software/tomcat-8081
Using CATALINA_HOME:   /opt/software/tomcat-8081
Using CATALINA_TMPDIR: /opt/software/tomcat-8081/temp
Using JRE_HOME:        /usr/local/jdk1.8.0_131
Using CLASSPATH:       /opt/software/tomcat-8081/bin/bootstrap.jar:/opt/software/tomcat-8081/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
```
有防火墙就开放端口
```bash
$ firewall-cmd --add-port=8081/tcp --permanent
success
$ firewall-cmd --reload
```
在浏览器访问两个tomcat的login.html

![8080](../../img/nginx/8080.png)

![8081](../../img/nginx/8081.png)


配置 Nginx
```bash
server {
        listen       9091;
        server_name  192.168.220.111;

        location ~ /edu/ {
            # alias /opt/nginx/a.html
            proxy_pass  http://192.168.220.111:8080;
        }

        location ~ /vod/ {
            proxy_pass  http://192.168.220.111:8081;
        }
    }
...
```
重新加载 Nginx 配置文件
```bash
$ sbin/nginx -s reload
```
在浏览器中访问: `192.168.220.111:9091/edu/login.html` 和 `192.168.220.111:9092/vod/login.html`

![edu](../../img/nginx/edu.png)

![vod](../../img/nginx/vod.png)

## 5.3 动静分离
如果用户请求为静态资源, 直接通过nginx获取不用将请求转发到后端接口。
```bash
server {
        listen       9091;
        server_name  192.168.220.111;
        
        location / {
            proxy_pass  http://192.168.220.111:8080;
        }
        
        location /css {
            root /usr/local/nginx/static;
            index index.html index.htm;
        }
        location /img {
            root /usr/local/nginx/static;
            index index.html index.htm;
        }
        location /js {
            root /usr/local/nginx/static;
            index index.html index.htm;
        }
}
```
使用一个 location(正则) 转发 `/css、/img、/js`.
```bash
location ~*/(css|img|js) {
  root /usr/local/nginx/static;
  index index.html index.htm;
}
```

# 六、负载均衡
效果: 在浏览器中输入 `192.168.220.111/edu/login.html`, 平均分配到 8080 和 8081 端口上

准备如上两个tomcat, 将 vod 修改成 edu
```bash
$ mv /opt/software/tomcat-8081/webapps/vod/ /opt/software/tomcat-8081/webapps/edu
$ ll /opt/software/tomcat-8081/webapps/
总用量 4
drwxr-x---. 15 root root 4096 1月   9 19:27 docs
drwxr-xr-x.  2 root root   24 1月   9 19:35 edu
drwxr-x---.  7 root root   99 1月   9 19:27 examples
drwxr-x---.  6 root root   79 1月   9 19:27 host-manager
drwxr-x---.  6 root root  114 1月   9 19:27 manager
drwxr-x---.  3 root root  223 1月   9 19:27 ROOT
$ /opt/software/tomcat-8081/bin/shutdown.sh
$ /opt/software/tomcat-8081/bin/startup.sh
```

编辑 Nginx 配置文件
```bash
$ vim conf/nginx.conf
http {
...
    upstream kinoserver{
        server 192.168.220.111:8080;
        server 192.168.220.111:8081;
    }

    server {
        listen       80;
        server_name  192.168.220.111;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
           proxy_pass  http://kinoserver;
           root  html;
           index  index.html  index.htm;
        }
...
```
重新加载 Nginx 配置文件
```bash
$ sbin/nginx -s reload
```
在浏览器中输入: `192.168.220.111/edu/login.html`, nginx 将以**轮询(默认)**的方式进行负载均衡。

## 6.1 负载均衡策略
1. 轮询(默认): 默认情况下使用轮询方式，逐一转发，这种方式适用于无状态请求;
2. weight: 指定轮询几率, weight 和访问率成正比, 用于服务器性能不均的情况;
   ```bash
   upstream kinoserver{
        server 192.168.220.111:8080 weight=5;
        server 192.168.220.111:8081 weight=10;
    }
   ```
3. ip_hash: 按每个请求的ip进行hash结果分配, 这样每个访客固定一个后端服务器, 可以解决 Session 问题;
   ```bash
   upstream kinoserver{
        ip_hash;
        server 192.168.220.111:8080;
        server 192.168.220.111:8081;
    }
   ```
4. fair: 按后台服务器的响应时间来分配请求, 响应时间短的游侠分配, 和 weight 分配策略类似;
   ```bash
   upstream kinoserver{
     server 192.168.220.111:8080;
     server 192.168.220.111:8081;
     fair;
   }
   ```
5. url_hash: 根据用户访问的url定向转发请求
   ```bash
   upstream kinoserver{
        url_hash;
        server 192.168.220.111:8080;
        server 192.168.220.111:8081;
    }
   ```

# 七、root 和 alias 区别
## 示例1
```bash
server {
        listen       9091;
        server_name  192.168.220.111;

        location /nginx/ {
            root /opt/nginx/;
        }
```
- 在 alias 目录配置下, 访问 192.168.220.111:9091/nginx/a.html 实际上访问的是 /opt/nginx/a.html
- 在 root 目录配置下, 访问 192.168.220.111:9091/nginx/a.html 实际上访问的是 /opt/nginx/nginx/a.html

## 示例2
当 location 匹配访问的path 和 alias 设置的目录名不一致时
```bash
    server {
        listen       9091;
        server_name  192.168.220.111;

        location /nginx1/ {
            alias /opt/nginx/;
        }
```
在浏览器中输入: 192.168.220.111:9091/nginx1/a.html 此时可以正常访问

当 location 匹配访问的path 和 root 设置的目录名不一致时
```bash
    server {
        listen       9091;
        server_name  192.168.220.111;

        location /nginx1/ {
            root /opt;
        }
```
在浏览器中输入: 192.168.220.111:9091/nginx1/a.html 此时不能正常访问, 查看 nginx 日志发现实际访问的路径是 `/opt/nginx1/a.html`
```bash
2021/01/09 20:48:19 [error] 22304#0: *148 open() "/opt/nginx1/a.html" failed (2: No such file or directory), client: 192.168.220.1, server: 192.168.220.111, request: "GET /nginx1/a.html HTTP/1.1", host: "192.168.220.111:9091"
```

区别:
- 在 alias 目录配置下, 访问的是 **alias + location<font color='red'>上一级</font>** 组合的地址
- 在 root 目录配置下, 访问的是 **alias + location** 组合的地址
- 当 location 匹配目录 和 alias 目录不一致时, 访问的本地目录还是 alias 配置的目录
- 当 location 匹配目录 和 root 目录不一致时, 需要将 **本地目录名 和 location 匹配访问的path名保持一致**



# 八、Nginx 斜杠(/) 说明
1. location 中的字符有没有 `/` 都没有影响。也就是说 `/user` 和 `/user/` 是一样的。
2. 如果 URL 结构是 `http://domain.com/` 的形式, 尾部有没有 `/` 都不会重定向。因为浏览器在发起请求的时候, 默认加了 `/`。虽然很多浏览器在地址栏里也不会显示 `/`。
3. 如果 URL 的结构是 `http://domain.com/some-dir/`。尾部缺少 `/` 将导致重定向。因为根据约定，URL 尾部的 `/` 表示目录，没有 `/` 表示文件, 当找不到的话会将 some-dir 当成目录, 重定向到 `/some-dir/`,去该目录下找默认文件。

## 8.1 proxy_pass 末尾带/
测试地址: http://localhost/test/hello.html

```bash
测试地址：http://192.168.171.129/test/tes.jsp
 
'场景一'：
location ^~ /test/ {
    proxy_pass http://192.168.171.129:8080/server/;
}
代理后实际访问地址：http://192.168.171.129:8080/server/tes.jsp -->'test由于匹配,所以会去除,然后拼接未匹配的'
 
'场景二'：
location ^~ /test {
    proxy_pass http://192.168.171.129:8080/server/;
}
代理后实际访问地址：http://192.168.171.129:8080/server//tes.jsp
 
'场景三'：
location ^~ /test/ {
    proxy_pass http://192.168.171.129:8080/;
}
代理后实际访问地址：http://192.168.171.129:8080/tes.jsp
 
'场景四':
location ^~ /test {
    proxy_pass http://192.168.171.129:8080/;
}
代理后实际访问地址：http://192.168.171.129:8080//tes.jsp
```

## 8.2 proxy_pass 末尾不带/
测试地址: http://localhost/test/hello.html
```bash
### 末尾不带/

proxy_pass配置中'url末尾不带/时',如url中'不包含path',则直接将'原uri拼接'在proxy_pass中url之后；如url中'包含path',则将原uri'去除location匹配表达式后的内容'拼接在proxy_pass中的url之后

 测试地址：http://192.168.171.129/test/tes.jsp
'场景一'：
 location ^~ /test/{
	proxy_pass http://192.168.171.129:8080/server;
 }
 代理后实际访问地址：http://192.168.171.129:8080/'servertes.jsp' -->'去除"/test/",然后拼接'
'场景二'：
location ^~ /test {
    proxy_pass http://192.168.171.129:8080/server;
}
代理后实际访问地址：http://192.168.171.129:8080/server/tes.jsp   -->'去除"/test",然后拼接"/tes.jsp"-->场景一和场景二的区别'
 
'场景三'：
location ^~ /test/ {
    proxy_pass http://192.168.171.129:8080;
}
代理后实际访问地址：http://192.168.171.129:8080/test/tes.jsp     -->'场景三和场景四常用'
 
'场景四'：
location ^~ /test {
    proxy_pass http://192.168.171.129:8080;
}
代理后实际访问地址：http://192.168.171.129:8080/test/tes.jsp
```
# 九、UrlRewrite

# 十、Nginx+Keepalived
安装 keepalived
```bash
yum install -y keepalived
```
配置nginx1
```bash
vim /etc/nginx/conf.d/web.conf 
server{
        listen 8080;
        root /usr/local/nginx/html;
        index test.html;
}
echo "<h1>This is web1</h1>"  > /usr/local/nginx/html/test.html
```
配置nginx2
```bash
vim /etc/nginx/conf.d/web.conf 
server{
        listen 8080;
        root /usr/local/nginx/html;
        index test.html;
}
echo "<h1>This is web2</h1>"  > /usr/local/nginx/html/test.html
```
启动两个nginx
```bash
cd $NGINX_HOME
./nginx
```
配置 keepalived(nginx1)
```bash
vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
vrrp_script nginx_check {
  script "/tools/nginx_check.sh"
  interval 1
}
vrrp_instance VI_1 {
  state MASTER
  interface ens33
  virtual_router_id 52
  priority 100
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass test
  }
  virtual_ipaddress {
    192.168.149.100
  }
  track_script {
    nginx_check
  }
  notify_master /tools/master.sh
  notify_backup /tools/backup.sh
  notify_fault /tools/fault.sh
  notify_stop /tools/stop.sh
}
```
配置 keepalived(nginx2)
```bash
vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
vrrp_script nginx_check {
  script "/tools/nginx_check.sh"
  interval 1
}
vrrp_instance VI_1 {
  state BACKUP
  interface ens33
  virtual_router_id 52
  priority 99
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass test
  }
  virtual_ipaddress {
    192.168.149.100
  }
  track_script {
    nginx_check
  }
  notify_master /tools/master.sh
  notify_backup /tools/backup.sh
  notify_fault /tools/fault.sh
  notify_stop /tools/stop.sh
}
```
健康检查脚本
```bash
vim /etc/keepalived/nginx_check.sh
#!/bin/bash
if [ -f /usr/local/nginx/logs/nginx.pid ]; then
  echo "success"
  exit 0
else
  echo "failed"
  exit 1
fi

chmod +x /etc/keepalived/nginx_check.sh
```
启动keepalived
```bash
vim /etc/keepalived/keepalived.conf
systemctl stop keepalived.service
systemctl start keepalived.service
systemctl status keepalived.service
```
访问nginx: http://192.168.149.100:8080, 停止启动一个nginx, 再次查看效果



# 十一、解决SEE流缓存
使用了nginx网关,可能会出现nginx缓冲sse流的问题,导致的现象是,客户端调用sse接口时,流数据并不是一条条出现的,而是一口气出现的。

nginx 增加如下配置:
```bash
server {
    listen 80;
    server_name xxxx;
    charset utf-8;
    
    # 增加如下三个配置
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    
    location / {
      # 增加如下三个配置
      proxy_cache off;
      proxy_buffering off;
      add_header X-Accel-Buffering "no";
      
      include /usr/local/nginx/conf/proxy;
      proxy_pass http://xxxxx:xxxx;
    }
}
```
如果是 ingress nginx 增加如下配置:
```bash
    nginx.ingress.kubernetes.io/proxy-buffering: "off"
    nginx.ingress.kubernetes.io/proxy-http-version: "1.1"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_cache off;
      add_header X-Accel-Buffering "no";
```

# 十二、默认转发443端口

nginx 的 https 协议是需要 SSL 模块支持的。

检查 nginx 是否安装了 SSL 模块, 如果没有 SSL 模块, 需要重新编译, 然后替换当前的 nginx。
```bash
$ ./sbin/nginx -V
nginx version: nginx/1.18.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/app/nginx/ --with-http_ssl_module
```

conf 文件, 省略了证书相关步骤
```bash
server {
    listen       80;
    server_name  raomin.com;
    rewrite ^(.*) https://$server_name$1 permanent;
}
server {
    listen 443;
    server_name raomin.com;
    # 配置你的 ssl
    ssl on;
    ssl_certificate      key/server.crt;
    ssl_certificate_key  key/server.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    # ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers  on;


    location / {
        proxy_redirect ~(.*)http://(.*) $1https://$2;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header Host  $host;
        proxy_set_header X-Request-Id $request_id;
        proxy_pass  https://xxxxx; # 你的服务
    }
}
```
重启 nginx 即可
```bash
$ ./sbin/nginx -s reload 
```


# 十三、nginx 缓存
## 强制缓存
```bash
server {
    listen 80;
    server_name 127.0.0.1;
    location ~* \.(jps|png|gif|woff2)$ {
#       add_header Cache-Control "private, max-age=3600";
#       add_header Pragma "no-cache";
       expires 1y;
       etag off;
       add_header Last-Modified "";
       root /Users/kino/Downloads/;
    }
}
```


## 协商缓存
```bash
server {
    listen 80;
    server_name 127.0.0.1;
    location ~* \.(jps|png|gif|woff2)$ {
       etag on;
       root /Users/kino/Downloads/;
    }
}
```