







# 一、平均负载
查看系统负载
```bash
$ uptime
 11:45:19 up 46 days, 20:09,  4 users,  load average: 3.92, 7.41, 8.94
```
- `11:45:19`: 当前时间.
- `up 46 days`: 系统运行时间.
- `4 users`: 正在登录用户数.
- `3.92, 7.41, 8.94`: 过去 1分钟、5分钟、15分钟系统的平均负载.

平均负载的官方说明
```bash
$ man uptime 
    DESCRIPTION
       uptime  gives  a  one line display of the following information.  The current time, how long the system has been running, how many users are currently logged on, and the
       system load averages for the past 1, 5, and 15 minutes.

       This is the same information contained in the header line displayed by w(1).

       System load averages is the average number of processes that are either in a runnable or uninterruptable state.  A process in a runnable state is either using the CPU or
       waiting  to  use the CPU.  A process in uninterruptable state is waiting for some I/O access, eg waiting for disk.  The averages are taken over the three time intervals.
       Load averages are not normalized for the number of CPUs in a system, so a load average of 1 means a single CPU system is loaded all the time while on a 4 CPU  system  it
       means it was idle 75% of the time.
```

# 二、上下文切换
上下文切换是对任务当前运行状态的暂存和恢复.

当多个进程竞争CPU的时候，CPU为了保证每个进程能公平被调度运行，采取了处理任务时间分片的机制，轮流处理多个进程，由于CPU处理速度非常快，在人类的感官上认为是并行处理，实际是"伪"并行，同一时间只有一个任务在运行处理。

根据 Tsuna 的测试报告，每次上下文切换都需要几十纳秒到到微秒的CPU时间，这些时间对CPU来说，就好比人类对1分钟或10分钟的感觉概念。在分秒必争的计算机处理环境下，浪费太多时间在切换上，只能会降低真正处理任务的时间，表象上导致延时、排队、卡顿现象发生。

进程上下文切换、线程上下文切换、中断上下文切换

系统调用、进程状态转换(运行、就绪、阻塞)、时间片耗尽、系统资源不足、sleep、优先级调度、硬件中断等

线程是调度的基本单位，进程是资源拥有的基本单位，同属一个进程的线程，发生上下文切换，只切换线程的私有数据，共享数据不变，因此速度非常快。

举例:
1. 银行分配各个窗口给来办理业务的人
2. 如果只有1个窗口开放（系统资源不足），大部分都得等
3. 如果正在办理业务的突然说自己不办了（sleep）,那他就去旁边再想想（等）
4. 如果突然来了个VIP客户，可以强行插队
5. 如果突然断电了（中断），都得等。。

```bash
$ vmstat -Sm
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0    737   4045  60876    0    0     2   142    0    0  1  1 96  2  0
```
- cs: 每秒上下文切换的次数
- in: 每秒中断的次数
- r: 就绪队列的长度
- b: 处于不可中断睡眠状态的进程数


```bash
$ pidstat -w 1
14时04分21秒   UID       PID   cswch/s nvcswch/s  Command
14时04分22秒     0         1      3.92      0.00  systemd
14时04分22秒     0         9      1.96      0.00  ksoftirqd/0
14时04分22秒     0        10    125.49      0.00  rcu_sched
14时04分22秒     0        18      4.90      0.00  ksoftirqd/1
```
- cswch/s: 自愿上下文切换的次数, 是指进程无法获取所需资源，导致的上下文切换。比如说， I/O、内存等系统资源不足时，就会发生自愿上下文切换
- nvcswch/s: 非自愿上下文切换的次数, 是指进程由于时间片已到等原因，被系统强制调度，进而发生的上下文切换。比如说，大量进程都在争抢 CPU 时，就容易发生非自愿上下文切换

## 2.1 上下文切换案例
机器配置：2 CPU，8GB 内存, 预先安装 sysbench 和 sysstat 包，如 `apt install sysbench sysstat`

先观察空间状态的上下文切换次数
```bash
# 间隔1秒后输出1组数据
$ vmstat 1 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 6984064  92668 830896    0    0     2    19   19   35  1  0 99  0  0
```
在第一个终端里运行 sysbench ，模拟系统多线程调度的瓶颈：
```bash
# 以10个线程运行5分钟的基准测试，模拟多线程切换的问题
$ sysbench --threads=10 --max-time=300 threads run
```
在第二个终端运行 vmstat ，观察上下文切换情况：
```bash
# 每隔1秒输出1组数据（需要Ctrl+C才结束）
$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 6  0      0 6487428 118240 1292772    0    0     0     0 9019 1398830 16 84  0  0  0
 8  0      0 6487428 118240 1292772    0    0     0     0 10191 1392312 16 84  0  0  0
```
可以发现，cs 列的上下文切换次数从之前的 35 骤然上升到了 139 万。同时，注意观察其他几个指标：
- r 列：就绪队列的长度已经到了 8，远远超过了系统 CPU 的个数 2，所以肯定会有大量的 CPU 竞争。
- us（user）和 sy（system）列：这两列的 CPU 使用率加起来上升到了 100%，其中系统 CPU 使用率，也就是 sy 列高达 84%，说明 CPU 主要是被内核占用了。
- in 列：中断次数也上升到了 1 万左右，说明中断处理也是个潜在的问题。

综合这几个指标，我们可以知道，系统的就绪队列过长，也就是正在运行和等待 CPU 的进程数过多，导致了大量的上下文切换，而上下文切换又导致了系统 CPU 的占用率升高。

那么到底是什么进程导致了这些问题呢？

我们继续分析，在第三个终端再用 pidstat 来看一下， CPU 和进程上下文切换的情况：
```bash
# 每隔1秒输出1组数据（需要 Ctrl+C 才结束）
# -w参数表示输出进程切换指标，而-u参数则表示输出CPU使用指标
$ pidstat -w -u 1
08:06:33      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
08:06:34        0     10488   30.00  100.00    0.00    0.00  100.00     0  sysbench
08:06:34        0     26326    0.00    1.00    0.00    0.00    1.00     0  kworker/u4:2

08:06:33      UID       PID   cswch/s nvcswch/s  Command
08:06:34        0         8     11.00      0.00  rcu_sched
08:06:34        0        16      1.00      0.00  ksoftirqd/1
08:06:34        0       471      1.00      0.00  hv_balloon
08:06:34        0      1230      1.00      0.00  iscsid
08:06:34        0      4089      1.00      0.00  kworker/1:5
08:06:34        0      4333      1.00      0.00  kworker/0:3
08:06:34        0     10499      1.00    224.00  pidstat
08:06:34        0     26326    236.00      0.00  kworker/u4:2
08:06:34     1000     26784    223.00      0.00  sshd
```

从 pidstat 的输出你可以发现，CPU 使用率的升高果然是 sysbench 导致的，它的 CPU 使用率已经达到了 100%。但上下文切换则是来自其他进程，包括非自愿上下文切换频率最高的 pidstat ，以及自愿上下文切换频率最高的内核线程 kworker 和 sshd。

> pidstat 输出的上下文切换次数，加起来也就几百，比 vmstat 的 139 万明显小了太多。这是怎么回事呢？难道是工具本身出了错吗？

pidstat 默认显示进程的指标数据，加上 -t 参数后，才会输出线程的指标。

在第三个终端里， Ctrl+C 停止刚才的 pidstat 命令，再加上 -t 参数，重试一下看看：
```bash
# 每隔1秒输出一组数据（需要 Ctrl+C 才结束）
# -wt 参数表示输出线程的上下文切换指标
$ pidstat -wt 1
08:14:05      UID      TGID       TID   cswch/s nvcswch/s  Command
...
08:14:05        0     10551         -      6.00      0.00  sysbench
08:14:05        0         -     10551      6.00      0.00  |__sysbench
08:14:05        0         -     10552  18911.00 103740.00  |__sysbench
08:14:05        0         -     10553  18915.00 100955.00  |__sysbench
08:14:05        0         -     10554  18827.00 103954.00  |__sysbench
...
```

小结:
- 自愿上下文切换变多了，说明进程都在等待资源，有可能发生了 I/O 等其他问题；
- 非自愿上下文切换变多了，说明进程都在被强制调度，也就是都在争抢 CPU，说明 CPU 的确成了瓶颈；
- 中断次数变多了，说明 CPU 被中断处理程序占用，还需要通过查看 `/proc/interrupts` 文件来分析具体的中断类型。


# 三、不可中断
- 中断: 由硬件或软件触发信号, 通知CPU需要处理某个任务. 中断可以打断当前正在执行的程序, 以便处理更加紧急的任务, 例如设备输入、系统调用等。处理完中断之后, CPU会恢复到之前的任务.
- 不可中断: 不可中断可以保证任务处理到关键时候不允许被打断, 通常都是涉及到了资源访问, 比如磁盘IO. 当进程处于不可中断状态时候, 即使有中断请求, CPU也不会打断这个进程. 这种状态通常用于确保数据一致性和系统稳定性.

看个案例: centos服务器(8c32g)部署了mysql服务, 集群负载高, wa 高
```bash
$ top
top - 12:05:23 up 45 days, 18:32,  1 user,  load average: 20.67, 20.02, 18.36
Tasks: 156 total,   1 running, 154 sleeping,   1 stopped,   0 zombie
%Cpu(s): 12.7 us,  0.2 sy,  0.0 ni,  5.1 id, 82.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 32779912 total,   221988 free,  8047468 used, 24510456 buff/cache
KiB Swap:  5242876 total,  4694396 free,   548480 used. 24257928 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
29166 mysql     20   0   10.4g   7.1g  12328 S 102.0 22.9  20497:57 mysqld
 7548 root      20   0  714688  59780   2408 T   0.7  0.2 327:46.54 titanagent
 9172 root      20   0       0      0      0 S   0.3  0.0   0:03.70 kworker/4:1
    1 root      20   0  128172   5288   3348 S   0.0  0.0  15:10.56 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:01.53 kthreadd
```
查看 mysqld 的子线程状态
```bash
$ top -Hp 29166
# 按 O 之后输入 S=D 过滤不可中断的进程
top - 12:06:46 up 45 days, 18:33,  1 user,  load average: 22.28, 20.70, 18.73
Threads: 147 total,   1 running, 146 sleeping,   0 stopped,   0 zombie
%Cpu(s): 12.9 us,  0.2 sy,  0.0 ni,  0.0 id, 86.9 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 32779912 total,   222992 free,  8052244 used, 24504676 buff/cache
KiB Swap:  5242876 total,  4694396 free,   548480 used. 24252904 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
29856 mysql     20   0   10.4g   7.1g  12476 D  0.7 22.9   8:01.31 connection
31580 mysql     20   0   10.4g   7.1g  12476 D  0.4 22.9   4:25.51 connection
 2255 mysql     20   0   10.4g   7.1g  12476 D  0.4 22.9   0:27.42 connection
29186 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9  13:53.34 ib_io_wr-4
29204 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9  45:58.13 ib_log_flush
29766 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9   2:06.32 ib_src_main
29807 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9  10:25.67 connection
29814 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9  16:47.92 connection
29826 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9   6:11.39 connection
29853 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9  14:27.05 connection
29859 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9  49:34.41 connection
29882 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9  25:17.89 connection
31579 mysql     20   0   10.4g   7.1g  12476 D  0.0 22.9 689:47.32 connection
```
可以看到大量不可中断的子线程, 其中更是有一些连接长时间处于不可中断状态, 并且可以看到 wa 高达 86.9, 于是查看系统IO
```bash
$ iostat -x 1 10
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.00    0.00    0.00     0.00     0.00     0.00   217.00    0.00    0.00    0.00   0.00 100.00
sdb               0.00     0.00    1.00   12.00    16.00   208.00    34.46    21.52 1970.15 2686.00 1910.50  76.92 100.00
dm-0              0.00     0.00    0.00    0.00     0.00     0.00     0.00   220.00    0.00    0.00    0.00   0.00 100.00
dm-1              0.00     0.00    0.00    0.00     0.00     0.00     0.00     4.00    0.00    0.00    0.00   0.00 100.00

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.13    0.00    0.13   95.74    0.00    4.01
```
可以看到 sda 和 sdb 两块挂载盘的 %util 满了, 说明 磁盘IO 有问题

# 四、僵尸进程
正常情况下，当一个进程创建了子进程后，它应该通过系统调用 wait() 或者 waitpid() 等待子进程结束，回收子进程的资源；而子进程在结束时，会向它的父进程发送 SIGCHLD 信号，所以，父进程还可以注册 SIGCHLD 信号的处理函数，异步回收资源。

如果父进程没这么做，或是子进程执行太快，父进程还没来得及处理子进程状态，子进程就已经提前退出，那这时的子进程就会变成僵尸进程。换句话说，父亲应该一直对儿子负责，善始善终，如果不作为或者跟不上，都会导致“问题少年”的出现。

通常，僵尸进程持续的时间都比较短，在父进程回收它的资源后就会消亡；或者在父进程退出后，由 init 进程回收后也会消亡。

一旦父进程没有处理子进程的终止，还一直保持运行状态，那么子进程就会一直处于僵尸状态。大量的僵尸进程会用尽 PID 进程号，导致新进程不能创建，所以这种情况一定要避免。


# 五、不可中断进程和僵尸进程的案例
提示: 可能会卡死服务器,建议用虚拟机或者临时云服务器.

环境: ubuntu22 1c1g
预安装: `apt install docker.io dstat sysstat`

运行
```bash
docker rm -f app
docker run --privileged --name=app -itd feisky/app:iowait /app -d /dev/vda3
```
观察集群负载
```bash
$ top
top - 13:53:31 up 18 min,  1 user,  load average: 2.68, 0.63, 0.23
Tasks: 121 total,   2 running, 119 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  3.4 sy,  0.0 ni,  0.0 id, 95.3 wa,  0.0 hi,  0.7 si,  0.0 st
MiB Mem :    955.6 total,     72.9 free,    719.0 used,    163.7 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.     84.2 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   4362 root      20   0   70056  65832    324 D   0.0   6.7   0:00.03 app
   4363 root      20   0   70056  65832    324 D   0.3   6.7   0:00.04 app
   4360 root      20   0   70056  65516      8 D   0.0   6.7   0:00.04 app
   4361 root      20   0   70056  65516      8 D   0.0   6.7   0:00.04 app
   4356 root      20   0   70056  65508      0 D   0.0   6.7   0:00.05 app
   4357 root      20   0   70056  65508      0 R   0.0   6.7   0:00.05 app
   1068 root      20   0 1912656  52496  25928 S   0.0   5.4   0:01.17 dockerd
   4365 root      20   0   70056  51312    324 D   0.0   5.2   0:00.02 app
   4364 root      20   0   70056  45768    324 D   0.0   4.7   0:00.02 app
    734 root      20   0 1801044  27876  13016 S   0.3   2.8   0:00.96 containerd
    408 root      rt   0  289312  27096   9072 S   0.0   2.8   0:00.10 multipathd
```
最近1分钟、5分钟、15分钟的负载依次降低, 代表这集群负载正在升高, 过滤僵死进程
```bash
$ top  ### 按 o 之后输入 S=Z
top - 13:59:57 up 2 min,  1 user,  load average: 42.99, 12.97, 4.52
Tasks: 144 total,   1 running, 126 sleeping,   0 stopped,  17 zombie
%Cpu(s):  0.3 us, 44.2 sy,  0.0 ni,  0.0 id, 95.2 wa,  0.0 hi,  0.2 si,  0.0 st
MiB Mem :    955.6 total,     49.3 free,    866.0 used,     40.3 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.      9.0 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   1577 root      20   0   70056  65516      0 D   0.1   6.7   0:00.07 app
   1587 root      20   0   70056  65516      0 D   0.4   6.7   0:00.09 app
   1588 root      20   0   70056  65516      0 D   0.1   6.7   0:00.05 app
   1602 root      20   0   70056  65516      0 D   1.6   6.7   0:00.24 app
   1603 root      20   0   70056  65516      0 D   2.2   6.7   0:00.32 app
   1604 root      20   0   70056  65516      0 D   4.4   6.7   0:00.62 app
   1607 root      20   0   70056  65516      0 D   3.7   6.7   0:00.51 app
   1608 root      20   0   70056  45972      0 D   3.4   4.7   0:00.48 app
   1610 root      20   0   70056  44920      0 D   2.1   4.6   0:00.29 app
   1612 root      20   0   70056  32496      0 D   0.9   3.3   0:00.13 app
   1098 root      20   0 1912144  27152   2552 S   0.1   2.8   0:00.32 dockerd
   ...
```
可以看到最近1分钟、5分钟、15分钟的平均负载依次降低, 代表着服务器负载正在上升, 且 iowait 已经达到了 55.2%, 并且存在 17 个僵尸进程。

这能得出以下结论:
1. iowait 太高, 导致了系统负载升高.
2. 僵尸进程在不断增多, 说明程序没有正常清理子进程的资源.


# 六、软中断

中断是一种异步的事件处理机制，用来提高系统的并发处理能力。中断事件发生，会触发执行中断处理程序，而中断处理程序被分为上半部和下半部这两个部分。

- 上半部对应硬中断，用来快速处理中断；
- 下半部对应软中断，用来异步处理上半部未完成的工作。

Linux 中的软中断包括网络收发、定时、调度、RCU 锁等各种类型，我们可以查看 proc 文件系统中的 `/proc/softirqs` ，观察软中断的运行情况。

软中断案例
1. 机器配置: 1c1g
2. 预先安装 docker、sysstat、sar 、hping3、tcpdump 等工具，比如 `apt-get install -y docker.io sysstat hping3 tcpdump`。

> sar 是一个系统活动报告工具，既可以实时查看系统的当前活动，又可以配置保存和报告历史统计数据。
> 
> hping3 是一个可以构造 TCP/IP 协议数据包的工具，可以对系统进行安全审计、防火墙测试等。
> 
> tcpdump 是一个常用的网络抓包工具，常用来分析各种网络问题。

```bash
$ docker run -itd --name=nginx -p 80:80 nginx
```
尝试访问这个nginx确保正常启动
```bash
$ curl http://172.29.254.69
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
开启一个新的终端, 运行 hping3 命令, 来模拟Nginx客户端请求:
```bash
# -S参数表示设置TCP协议的SYN（同步序列号），-p表示目的端口为80# 
# -i u100表示每隔100微秒发送一个网络帧
# 注：如果你在实践过程中现象不明显，可以尝试把100调小，比如调成10甚至1$ 
$ hping3 -S -p 80 -i u100 172.29.254.69
```
此时回到上一个终端, 应该能感受出响应变慢了, 尝试 top 看系统状况
```bash
$ top
top - 22:23:57 up  8:03,  2 users,  load average: 0.66, 0.76, 0.54
Tasks: 114 total,   2 running, 112 sleeping,   0 stopped,   0 zombie
%Cpu(s): 21.8 us, 37.1 sy,  0.0 ni, 10.2 id,  0.0 wa,  0.0 hi, 31.0 si,  0.0 st
MiB Mem :    955.6 total,     84.4 free,    255.1 used,    616.1 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.    532.3 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   4083 root      20   0   11832   5976   5528 R  77.1   0.6  12:11.22 hping3
   1444 root      20   0  130504  20660  15832 S   1.0   2.1   4:45.70 AliYunDunMonito
     13 root      20   0       0      0      0 S   0.7   0.0   0:23.03 ksoftirqd/0
     ...
```
可以看到系统平均负载、wa等都相对比较正常, 唯独 si 稍微有点高达到了 31%, 系统可能存在比较频繁的软中断, 软中断是CPU开始正式处理高优先级的任务了, 不太可能像硬中断频繁那样是因为等待系统资源, 软中断的情况也比较多, 再看看软中断的相关信息
```bash
$ watch -d cat /proc/softirqs
Every 2.0s: cat /proc/softirqs              iZbp1643sf4zgucdf3bh7bZ: Tue Oct 22 22:27:48 2024
                    CPU0       CPU1
          HI:          2          0
       TIMER:    1265933          0
      NET_TX:          4          0
      NET_RX:   80778417          0
       BLOCK:      38377          0
    IRQ_POLL:          0          0
     TASKLET:        449          0
       SCHED:          0          0
     HRTIMER:       2845          0
         RCU:   17240947          0
```
多观察会发现 TIMER、NET_RX、RCU 都有很大的变化, 但是 NET_RX 相对来说变化更大一下, 所以开始怀疑是网络发送引起的软中断, 再看看网卡情况
```bash
$ sar -n DEV 1
Linux 5.15.0-122-generic (iZbp1643sf4zgucdf3bh7bZ) 	10/22/2024 	_x86_64_	(1 CPU)

10:29:21 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
10:29:22 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
10:29:22 PM      eth0      3.00      1.00      0.17      0.04      0.00      0.00      0.00      0.00
10:29:22 PM   docker0  20764.00  41528.00    892.20   2189.95      0.00      0.00      0.00      0.00
10:29:22 PM veth324d733  20764.00  41528.00   1176.09   2189.95      0.00      0.00      0.00      0.18

10:29:22 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
10:29:23 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
10:29:23 PM      eth0      2.00     16.00      0.13      2.52      0.00      0.00      0.00      0.00
10:29:23 PM   docker0  21129.00  42258.00    907.89   2228.45      0.00      0.00      0.00      0.00
10:29:23 PM veth324d733  21129.00  42258.00   1196.76   2228.45      0.00      0.00      0.00      0.18
```
先解释参数:
- 第一列 `10:29:21 PM`: 时间
- 第二列 `IFACE`: 网卡
- 第三四列 `rxpck/s`、`txpck/s`: 每秒网卡 收到、发送 的帧数, 也就是 PPS
- 第五六列 `rxkB/s`、`txkB/s`: 每秒网卡 收到、发送 的字节数, 也就是 BPS

这里能看到 docker0 这个网卡的 收发 帧数 比 收发 字节数多很多, 稍微计算一下 892.20 * 1024 / 20764.00 = 43.99, 说明平均每个网络帧只有 44字节, 很显然是很小的网络帧, 属于小包问题. 

再用 tcpdump 抓取这个网卡的信息
```bash
$ tcpdump -i docker0 -n tcp
22:14:11.562935 IP 172.29.254.69.774 > 172.17.0.2.80: Flags [S], seq 1779350409, win 512, length 0
22:14:11.562938 IP 172.17.0.2.80 > 172.29.254.69.774: Flags [S.], seq 3488339637, ack 1779350410, win 64240, options [mss 1460], length 0
22:14:11.562941 IP 172.29.254.69.774 > 172.17.0.2.80: Flags [R], seq 1779350410, win 0, length 0
22:14:11.563002 IP 172.29.254.69.775 > 172.17.0.2.80: Flags [S], seq 1917762204, win 512, length 0
22:14:11.563006 IP 172.17.0.2.80 > 172.29.254.69.775: Flags [S.], seq 648867213, ack 1917762205, win 64240, options [mss 1460], length 0
22:14:11.563015 IP 172.29.254.69.775 > 172.17.0.2.80: Flags [R], seq 1917762205, win 0, length 0
22:14:11.563034 IP 172.29.254.69.776 > 172.17.0.2.80: Flags [S], seq 2062694594, win 512, length 0
22:14:11.563036 IP 172.17.0.2.80 > 172.29.254.69.776: Flags [S.], seq 1428040479, ack 2062694595, win 64240, options [mss 1460], length 0
22:14:11.563040 IP 172.29.254.69.776 > 172.17.0.2.80: Flags [R], seq 2062694595, win 0, length 0
22:14:11.563101 IP 172.29.254.69.777 > 172.17.0.2.80: Flags [S], seq 1327119535, win 512, length 0
22:14:11.563105 IP 172.17.0.2.80 > 172.29.254.69.777: Flags [S.], seq 2952525786, ack 1327119536, win 64240, options [mss 1460], length 0
22:14:11.563115 IP 172.29.254.69.777 > 172.17.0.2.80: Flags [R], seq 1327119536, win 0, length 0
22:14:11.563131 IP 172.29.254.69.778 > 172.17.0.2.80: Flags [S], seq 1469159056, win 512, length 0
22:14:11.563135 IP 172.17.0.2.80 > 172.29.254.69.778: Flags [S.], seq 1226500663, ack 1469159057, win 64240, options [mss 1460], length 0
22:14:11.563138 IP 172.29.254.69.778 > 172.17.0.2.80: Flags [R], seq 1469159057, win 0, length 0
22:14:11.563199 IP 172.29.254.69.779 > 172.17.0.2.80: Flags [S], seq 1305617053, win 512, length 0
22:14:11.563202 IP 172.17.0.2.80 > 172.29.254.69.779: Flags [S.], seq 3218348663, ack 1305617054, win 64240, options [mss 1460], length 0
22:14:11.563211 IP 172.29.254.69.779 > 172.17.0.2.80: Flags [R], seq 1305617054, win 0, length 0
22:14:11.563230 IP 172.29.254.69.780 > 172.17.0.2.80: Flags [S], seq 1485853587, win 512, length 0
22:14:11.563232 IP 172.17.0.2.80 > 172.29.254.69.780: Flags [S.], seq 479197061, ack 1485853588, win 64240, options [mss 1460], length 0
22:14:11.563236 IP 172.29.254.69.780 > 172.17.0.2.80: Flags [R], seq 1485853588, win 0, length 0
22:14:11.563295 IP 172.29.254.69.781 > 172.17.0.2.80: Flags [S], seq 1140929301, win 512, length 0
....
```
能看到非常多的如上信息, 摘取其中一条先解释:
```bash
22:14:11.562935 IP 172.29.254.69.774 > 172.17.0.2.80: Flags [S], seq 1779350409, win 512, length 0
```
- 第一列 `22:14:11.562935`: 时间
- 第二列 `IP 172.29.254.69.774 > 172.17.0.2.80`: 表示这个网络帧是从 172.29.254.69 的 774 端口发送到 172.17.0.2 的 80端口的
- 第三列 `Flags [S]`: 表示这是一个 SYN 包(三次握手), 后面还有 [S.] 表示 SYN-ACK 包, 是服务器表示同意建立连接了

这么多类似的信息基本表名 172.17.0.2.80 收到了 SYN Flood 攻击, 最简单的办法就是封掉172.29.254.69这个ip



# 总结
![CPU性能指标总结](../../../img/linux/CPU性能指标/CPU性能指标总结.png)

![CPU性能指标总结2](../../../img/linux/CPU性能指标/CPU性能指标总结2.png)

![CPU性能指标总结3](../../../img/linux/CPU性能指标/CPU性能指标总结3.png)































