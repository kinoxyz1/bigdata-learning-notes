* [一、新建Zabbix用户](#%E4%B8%80%E6%96%B0%E5%BB%BAzabbix%E7%94%A8%E6%88%B7)
* [二、编译环境准备](#%E4%BA%8C%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
* [三、去官网下载编译安装的Zabbix](#%E4%B8%89%E5%8E%BB%E5%AE%98%E7%BD%91%E4%B8%8B%E8%BD%BD%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85%E7%9A%84zabbix)
* [四、修改zabbix\-agent配置文件](#%E5%9B%9B%E4%BF%AE%E6%94%B9zabbix-agent%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
* [五、启动Zabbix\-Agent](#%E4%BA%94%E5%90%AF%E5%8A%A8zabbix-agent)

---
# 一、新建Zabbix用户

```bash
[root@zabbix ~]# groupadd zabbix
[root@zabbix ~]# useradd zabbix -g zabbix -s /sbin/nologin
```

# 二、编译环境准备

```bash
[root@zabbix ~]# yum install -y gcc  libxml2-devel libevent-devel net-snmp net-snmp-devel  curl  curl-devel php  php-bcmath  php-mbstring mariadb mariadb-devel java-1.6.0-openjdk-devel --skip-broken
```

# 三、去官网下载编译安装的Zabbix
https://www.zabbix.com/download_sources
```bash
[root@zabbix ~]# wget https://www.xxshell.com/download/sh/zabbix/zabbix4.4/zabbix-4.4.1.tar.gz
[root@zabbix ~]# tar -xzvf zabbix-4.4.1.tar.gz
[root@zabbix ~]# cd zabbix-4.4.1
[root@zabbix ~]# ./configure --enable-agent
[root@zabbix ~]# make -j 2 && make install
```
---
# 四、修改zabbix-agent配置文件

```bash
sudo vim /etc/init.d/zabbix_agentd

Server=zabbix_server_hostname
#ServerActive=127.0.0.1
#Hostname=Zabbix server
```

# 五、启动Zabbix-Agent

```bash
/etc/init.d/zabbix_agentd restart
systemctl enable zabbix_agentd
```