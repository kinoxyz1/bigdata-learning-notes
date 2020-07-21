


---
# 一、设置IP地址、网关DNS
说明：CentOS 7.0默认安装好之后是没有自动开启网络连接的！

```bash
#进入网络配置文件目录
cd  /etc/sysconfig/network-scripts/  
#编辑配置文件，添加修改以下内容
vim  ifcfg-ens33 

HWADDR=00:0C:29:8D:24:73
TYPE=Ethernet
BOOTPROTO=static  #启用静态IP地址
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno16777736
UUID=ae0965e7-22b9-45aa-8ec9-3f0a20a85d11
ONBOOT=yes  #开启自动启用网络连接
IPADDR0=192.168.21.128  #设置IP地址
PREFIXO0=24  #设置子网掩码
GATEWAY0=192.168.21.2  #设置网关
DNS1=8.8.8.8  #设置主DNS
DNS2=8.8.4.4  #设置备DNS
:wq!  #保存退出
service network restart   #重启网络
ping www.baidu.com  #测试网络是否正常

#查看IP地址
ip addr  
```

# 二、设置主机名
```bash
#设置主机名为www
hostname  www  

#编辑配置文件
vi /etc/hostname 

#修改localhost.localdomain为www
www   

:wq!  #保存退出

vi /etc/hosts #编辑配置文件

#修改localhost.localdomain为www
127.0.0.1   localhost  www   

:wq!  #保存退出

shutdown -r now  #重启系统
```