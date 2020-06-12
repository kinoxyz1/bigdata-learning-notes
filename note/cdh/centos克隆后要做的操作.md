

---
# 一、修改主机名

```bash
hostnamectl set-hostname 主机名
```

---
# 二、修改 ip

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33 

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=192.168.161.160 # 修改ip
NETMASK=255.255.255.0 
GATEWAY=192.168.161.2
DNS1=192.168.161.2
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=0a20f443-9c23-4274-a20f-c3f26d04f559
DEVICE=ens33
ONBOOT=yes
```
