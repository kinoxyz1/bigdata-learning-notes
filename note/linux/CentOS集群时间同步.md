
* [一、检查 ntp 是否安装](#%E4%B8%80%E6%A3%80%E6%9F%A5-ntp-%E6%98%AF%E5%90%A6%E5%AE%89%E8%A3%85)
* [二、修改ntp配置文件](#%E4%BA%8C%E4%BF%AE%E6%94%B9ntp%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
* [三、启动 ntpd 服务](#%E4%B8%89%E5%90%AF%E5%8A%A8-ntpd-%E6%9C%8D%E5%8A%A1)
* [四、设置开机自启](#%E5%9B%9B%E8%AE%BE%E7%BD%AE%E5%BC%80%E6%9C%BA%E8%87%AA%E5%90%AF)
* [五、配置时间同步](#%E4%BA%94%E9%85%8D%E7%BD%AE%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5)

---
时间同步的方式: 找一个机器, 作为时间服务器, 所有的机器与这台集群时间进行定时的同步, 比如: 每隔十分钟, 同步一次时间

# 一、检查 ntp 是否安装
集群中所有机器都要执行
```bash
rpm -qa|grep ntp

# 如果没有安装ntp, 则执行此步骤
yum -y install ntp
```

# 二、修改ntp配置文件
仅在时间服务器上执行
```bash
[root@hadoop1 ~]# vim /etc/ntp.conf
# 修改
# restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap 
# 为
restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

# 修改
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
# 为
# server 0.centos.pool.ntp.org iburst
# server 1.centos.pool.ntp.org iburst
# server 2.centos.pool.ntp.org iburst
# server 3.centos.pool.ntp.org iburst

# 添加
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

修改 `/etc/sysconfig/ntpd` 文件
```bash
[root@hadoop1 ~]# vim /etc/sysconfig/ntpd
SYNC_HWCLOCK=yes
```

# 三、启动 ntpd 服务
所有机器都要执行
```bash
[root@hadoop1 ~]# systemctl start ntpd
```

# 四、设置开机自启
所有机器都要执行
```bash
[root@hadoop1 ~]# systemctl enable ntpd.service
```

# 五、配置时间同步
需要同步时间的机器执行
```bash
crontab -e
*/10 * * * * /usr/sbin/ntpdate hadoop1
```

修改任意机器时间
```bash
date -s "2017-9-11 11:11:11"
```

执行 `/usr/sbin/ntpdate hadoop1` , 再查看时间.
