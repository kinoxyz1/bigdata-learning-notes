







# 一、iowait 过高的情况
如下一段python程序在运行(**直接复制粘贴先不要看写的什么, 避免看问题的时候会先入为主**)

运行 python 之前先运行: `echo 3 > /proc/sys/vm/drop_caches` 把缓存清理了

```bash
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

import logging
import random
import string
import signal
import time

from logging.handlers import RotatingFileHandler

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.INFO)
rHandler = RotatingFileHandler(
    "/tmp/logtest.txt", maxBytes=1024 * 1024 * 1024, backupCount=1)
rHandler.setLevel(logging.INFO)
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s')
rHandler.setFormatter(formatter)
logger.addHandler(rHandler)


def set_logging_info(signal_num, frame):
    '''Set loging level to INFO when receives SIGUSR1'''
    logger.setLevel(logging.INFO)


def set_logging_warning(signal_num, frame):
    '''Set loging level to WARNING when receives SIGUSR2'''
    logger.setLevel(logging.WARNING)


def get_message(N):
    '''Get message for logging'''
    return N * ''.join(
        random.choices(string.ascii_uppercase + string.digits, k=1))


def write_log(size):
    '''Write logs to file'''
    message = get_message(size)
    while True:
        logger.info(message)
        time.sleep(0.1)


signal.signal(signal.SIGUSR1, set_logging_info)
signal.signal(signal.SIGUSR2, set_logging_warning)

if __name__ == '__main__':
    msg_size = 300 * 1024 * 1024
    write_log(msg_size)
```

top 查看系统状态
```bash
$ top
top - 19:13:06 up  1:57,  5 users,  load average: 2.13, 1.72, 1.57
Tasks: 128 total,   4 running, 124 sleeping,   0 stopped,   0 zombie
%Cpu0  :  2.1 us,  3.8 sy,  0.0 ni,  0.0 id, 93.8 wa,  0.0 hi,  0.3 si,  0.0 st
%Cpu1  : 26.7 us, 41.3 sy,  0.0 ni,  7.8 id, 24.2 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   3559.3 total,   1871.0 free,    621.7 used,   1066.6 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   2701.9 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   7894 root      20   0  326308 317748   5172 R  71.8   8.7   0:52.84 python3
   7877 root      20   0       0      0      0 D   1.7   0.0   0:01.71 kworker/u4:4+flush-252:0
   1386 root      20   0   96284   1948      0 S   1.3   0.1   0:32.62 AliYunDun
   1397 root      20   0  135288   9412      0 R   1.3   0.3   0:57.26 AliYunDunMonito
   7896 root      20   0       0      0      0 I   1.0   0.0   0:00.34 kworker/u4:0-events_power_efficient
     92 root      20   0       0      0      0 S   0.3   0.0   0:17.68 kswapd0
```
首先说明这台机器配置是2c4g.

平局负载可以看见系统负载正在增加, 并且其中一个CPU的wa已经到了93.8. 并且 可以看到只有 4G 左右内存, 只有 1G 左右剩余, 600m被使用, buff/cache 使用较多, 并且也在增长, 说明内存主要被缓存占用了. 虽然大部分缓存都可以被回收, 但是还是需要去了解缓存的去处, 确认缓存都是使用合理的.

这里我们先确认到底是 buff 还是 cache 缓存用的比较多
```bash
$ vmstat -Sm 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  1      0    728     10   1401    0    0   160 19041  560  935 10  6 70 15  0
 1  1      0   1469     10   1405    0    0     0 155752 1177 1624 19 28  8 44  0
 0  3      0    580     10   1550    0    0     0 90116 1094 1583 21 23  1 55  0
 1  1      0   1205     10   1728    0    0     0 168008 1143 1801  3  8 19 70  0
 1  1      0    861     10   1728    0    0     0 90112 1174 1579 24 30  6 40  0
 0  2      0    186     10   1942    0    0     0 90112 1148 1776 10 21  0 69  0
 1  1      0    655     10   2008    0    0     0 90168 1197 1764 11 16 31 42  0
```
可以看到缓存全用于 cache 了, 先简单说明 buff 和 cache 的区别:
- buff: 通常指的是用于块设备（如磁盘）的缓存.
- cache: 用于文件系统的缓存.

所以我们推断系统内存主要用于缓存读写文件相关的IO操作.


我们还可以从top中, 进程部分可以看到, 能引起 iowait 的, 可能是 python 程序比较有嫌疑, 那在 pidstat 看看
```bash
$ pidstat -d 1
07:16:10 PM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
07:16:11 PM     0       314      0.00     80.00      0.00       0  jbd2/vda3-8
07:16:11 PM     0      7894      0.00  83196.00      0.00       0  python3
```
这里能看到python程序每秒写磁盘比较多, 达到了80mb/s, 可能导致 iowait 就是这个 python 程序, 并且极大可能是这个python程序在频繁的读写文件.

在使用 iostat 看看具体 io 情况
```bash
$ iostat -xd 1
Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
loop0            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop1            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop2            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop3            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop4            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
vda              0.00      0.00     0.00   0.00    0.00     0.00  126.00 112688.00    16.00  11.27  386.23   894.35    0.00      0.00     0.00   0.00    0.00     0.00    1.00   40.00   48.70 100.00
```
这能观察到, vda 这块盘每秒有126次写入, 每秒写入大约112M的数据量, 写请求的相应时间达到了 0.3秒,请求队列长度也达到了 48.7, 并且该盘基本io接近饱和.

超慢的响应时间和特长的请求队列长度，进一步验证了 I/O 已经饱和的猜想。此时，vda 磁盘已经遇到了严重的性能瓶颈。


使用系统分析工具 strace 看看
```bash
$ strace -p 7894
strace: Process 7894 attached
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc36a0cb000
mmap(NULL, 314576896, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc3574ca000
write(3, "2024-10-24 19:30:28,028 - __main"..., 314572845) = 314572845
munmap(0x7fc3574ca000, 314576896)       = 0
munmap(0x7fc36a0cb000, 314576896)       = 0
munmap(0x7fc37cccc000, 314576896)       = 0
pselect6(0, NULL, NULL, NULL, {tv_sec=0, tv_nsec=100000000}, NULL) = 0 (Timeout)
getpid()                                = 7894
newfstatat(AT_FDCWD, "/tmp/logtest.txt.1", {st_mode=S_IFREG|0644, st_size=629145690, ...}, 0) = 0
newfstatat(AT_FDCWD, "/tmp/logtest.txt.1", {st_mode=S_IFREG|0644, st_size=629145690, ...}, 0) = 0
```
从 write() 系统调用上，我们可以看到，进程向文件描述符编号为 3 的文件中，写入了 300MB 的数据。看来，它应该是我们要找的文件。不过，write() 调用中只能看到文件的描述符编号，文件名和路径还是未知的。

再观察后面的 newfstatat() 调用，你可以看到，它正在获取 /tmp/logtest.txt.1 的状态。 这种“点 + 数字格式”的文件，在日志回滚中非常常见。我们可以猜测，这是第一个日志回滚文件，而正在写的日志文件路径，则是 /tmp/logtest.txt。

当然，这只是我们的猜测，自然还需要验证。这里，我再给你介绍一个新的工具 lsof。它专门用来查看进程打开文件列表，不过，这里的“文件”不只有普通文件，还包括了目录、块设备、动态库、网络套接字等。

接下来，我们在终端中运行下面的 lsof 命令，看看进程 7894 都打开了哪些文件：
```bash
$ lsof -p 7894
COMMAND  PID USER   FD   TYPE DEVICE  SIZE/OFF   NODE NAME
python3 7894 root  cwd    DIR  252,3      4096 784898 /root
python3 7894 root  rtd    DIR  252,3      4096      2 /
python3 7894 root  txt    REG  252,3   5909000 131278 /usr/bin/python3.10
python3 7894 root  mem    REG  252,3     23664 132994 /usr/lib/python3.10/lib-dynload/_queue.cpython-310-x86_64-linux-gnu.so
python3 7894 root  mem    REG  252,3   3048928 130817 /usr/lib/locale/locale-archive
python3 7894 root  mem    REG  252,3   2220400 131975 /usr/lib/x86_64-linux-gnu/libc.so.6
python3 7894 root  mem    REG  252,3    108936 131008 /usr/lib/x86_64-linux-gnu/libz.so.1.2.11
python3 7894 root  mem    REG  252,3    194872 131649 /usr/lib/x86_64-linux-gnu/libexpat.so.1.8.7
python3 7894 root  mem    REG  252,3    940560 131993 /usr/lib/x86_64-linux-gnu/libm.so.6
python3 7894 root  mem    REG  252,3     27002 271363 /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
python3 7894 root  mem    REG  252,3    240936 131415 /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
python3 7894 root    0u   CHR  136,3       0t0      6 /dev/pts/3
python3 7894 root    1u   CHR  136,3       0t0      6 /dev/pts/3
python3 7894 root    2u   CHR  136,3       0t0      6 /dev/pts/3
python3 7894 root    3w   REG  252,3 877273088     42 /tmp/logtest.txt
```
这个输出界面中，有几列我简单介绍一下，FD 表示文件描述符号，TYPE 表示文件类型，NAME 表示文件路径。这也是我们需要关注的重点。

再看最后一行，这说明，这个进程打开了文件 /tmp/logtest.txt，并且它的文件描述符是 3 号，而 3 后面的 w ，表示以写的方式打开。

这跟刚才 strace 完我们猜测的结果一致，看来这就是问题的根源：进程 7894 以每次 300MB 的速度，在“疯狂”写日志，而日志文件的路径是 /tmp/logtest.txt。

既然找出了问题根源，接下来按照惯例，就该查看源代码，然后分析为什么这个进程会狂打日志了。

分析这个源码，我们发现，它的日志路径是 /tmp/logtest.txt，默认记录 INFO 级别以上的所有日志，而且每次写日志的大小是 300MB。这跟我们上面的分析结果是一致的。

一般来说，生产系统的应用程序，应该有动态调整日志级别的功能。继续查看源码，你会发现，这个程序也可以调整日志级别。如果你给它发送 SIGUSR1 信号，就可以把日志调整为 INFO 级；发送 SIGUSR2 信号，则会调整为 WARNING 级：
```python
def set_logging_info(signal_num, frame): 
  '''Set loging level to INFO when receives SIGUSR1''' 
  logger.setLevel(logging.INFO) 

def set_logging_warning(signal_num, frame): 
  '''Set loging level to WARNING when receives SIGUSR2''' 
  logger.setLevel(logging.WARNING) 

signal.signal(signal.SIGUSR1, set_logging_info) 
signal.signal(signal.SIGUSR2, set_logging_warning) 
```

根据源码中的日志调用 logger. info(message) ，我们知道，它的日志是 INFO 级，这也正是它的默认级别。那么，只要把默认级别调高到 WARNING 级，日志问题应该就解决了。

接下来，我们就来检查一下，刚刚的分析对不对。在终端中运行下面的 kill 命令，给进程 7894 发送 SIGUSR2 信号：

```bash
$ kill -SIGUSR2 7894
```

然后，再执行 top 和 iostat 观察一下：
```bash
top - 19:34:45 up  2:19,  5 users,  load average: 1.42, 2.10, 2.09
Tasks: 128 total,   2 running, 126 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.5 us,  0.2 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   3559.3 total,   1517.9 free,    644.1 used,   1397.2 buff/cache
```

```bash
$ iostat -d -x 1
Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
loop0            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop1            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop2            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop3            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
loop4            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
vda              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
```
观察 top 和 iostat 的输出，你会发现，稍等一段时间后，iowait 会变成 0，而 sda 磁盘的 I/O 使用率也会逐渐减少到 0。














