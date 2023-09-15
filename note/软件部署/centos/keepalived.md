


# install
```bash
yum install -y keepalived
```

# global config
```bash
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
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
```
- `global_defs`: 全局配置标识
- `notification_email`: 用于设置报警的邮件地址，可以设置多个，每行一个。如果要开启邮件报警，需要开启本机的sendmail服务
- `notification_email_from`: 邮件的发送地址
- `smtp_server`: 设置邮件的smtp server地址
- `smtp_connect_timeout`: 设置连接smtp server的超时时间
- `router_id`: 运行keepalived的一个标识，唯一
- `vrrp_skip_check_adv_addr`: 对所有通告报文都检查，会比较消耗性能，启用此配置后，如果收到的通告报文和上一个报文是同一个路由器，则跳过检查，默认值为全检查
- `vrrp_strict`: 严格遵守VRRP协议,启用此项后以下状况将无法启动服务:1.无VIP地址 2.配置了单播邻居 3.在VRRP版本2中有IPv6地址，开启动此项并且没有配置vrrp_iptables时会自动开启iptables防火墙规则，默认导致VIP无法访问,建议不加此项配置
- `vrrp_iptables`: 此项和vrrp_strict同时开启时，则不会添加防火墙规则,如果无配置vrrp_strict项,则无需启用此项配置
- `vrrp_garp_interval`: gratuitous ARP messages 报文发送延迟，0表示不延迟
- `vrrp_gna_interval`: unsolicited NA messages （不请自来）消息发送延迟

# VRRP 实例配置
```bash
vrrp_script nginx_check {
  script"/tools/nginx_check.sh"
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
- `vrrp_instance`: VI_1 #VRRP实例开始的标识 VI_1为实例名称
- `state`: #指定keepalived的角色，MASTER表示此主机是主服务器，BACKUP表示此主机是备服务器
- `interface`: #指定检测网络的网卡接口
- `virtual_router_id`: #虚拟路由标识，数字形式，同一个VRRP实例使用唯一的标识，即在同一个vrrp_instance下，master和backup必须一致
- `priority`: #节点优先级，数字越大表示节点的优先级越高，在一个VRRP实例下，MASTER的优先级必须要比BACKUP高，不然就会切换角色
- `advert_int`: #用于设定MASTER与BACKUP之间同步检查的时间间隔，单位为秒
- `auth_type`: PASS #预共享密钥认证，同一个虚拟路由器的keepalived节点必须一样
- `auth_pass`: #设置密钥
- `virtual_ipaddress`: #设置虚拟IP地址，可以设置多种形式：10.0.0.100不指定网卡，默认为eth0,注意：不指定/prefix,默认为/32；10.0.0.101/24 dev eth1 指定VIP的网卡；10.0.0.102/24 dev eth2 label eth2:1  #指定VIP的网卡label
- `nopreempt`: # 设置为非抢占模式，同一实例下主备设置必须一样
- `preemtp_delay`: # 设置抢占模式的延时时间，单位为秒
- `vrrp_script`: 自定义资源监控脚本，vrrp实例根据脚本返回值进行下一步操作，脚本可被多个实例调用。 track_script:调用vrrp_script定义的脚本去监控资源，定义在实例之内，调用事先定义的vrrp_script。实现主备切换，保证服务高可用。vrrp_script仅仅通过监控脚本返回的状态码来识别集群服务是否正常，如果返回状态码是0，那么就认为服务正常，反之亦然。
- `notify_master`: #当前节点成为主节点时触发的脚本
- `notify_backup`: #当前节点转为备节点时触发的脚本
- `notify_fault`: #当前节点转为“失败”状态时触发的脚本
- `notify_stop`: #当停止VRRP时触发脚本