






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
## 3.5 指定启动的配置文件
```bash
$ sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```























# 四、反向代理