







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


# 三、不可中断&僵尸进程


# 四、软中断


# 五、案例模拟




































