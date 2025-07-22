







# 基本语法

| | `table` | `command` | `chain` | `parameter` | `target` |
| -- |-------|---------|-------|-----------| --- |
| iptables | -t filter(默认) | -A<br />-D<br />-I<br />-R<br />-L<br />-F<br />-Z<br />-N<br />-X<br />-P | INPUT<br />OUTPUT<br />FORWARD<br />PREROUTING<br />POSTROUTING | -p<br />-s<br />-d<br />-i<br />-o<br />--sport<br />--dport | -j ACCEPT<br />-j DROP<br />-j REDIRECT<br />-j SNAT<br />-j DNAT<br />-j MASQUERADE<br />-j LOG<br />-j SEMARK |

iptables 主要就是对 `table` 和 `chain` 的操作. 以下是详细解释:



- `table`: table 有 4 中, 分别是:
  - `filter`: 不加 `-t` 默认是 `filter`, `filter` 只有三个 `chain`:
    - `INPUT`
    - `OUTPUT`
    - `FORWARD`
  - `nat`: 不常用, 可以查看 `man iptables` 学习.
  - `mangle`: 不常用, 可以查看 `man iptables` 学习.
  - `row`: 不常用, 可以查看 `man iptables` 学习.

- `command`: 对 `chain` 的操作, 通常只能指定一个:
  - `-A`: 将一个或多个规则添加到所选 `chain` 的末尾;
  - `-D`: 从所选 `chain` 中删除一个或多个规则;
  - `-I`: 将一个或多个规则添加到所选`chain` 的开头(index=0);
  - `-R`: 替换规则链;
  - `-L`: 列出 `chain` 下所有的规则. 如果未指定 `chain`, 默认列出 `filter` 下所有规则;
  - `-F`: 清空 `chain`下所有的规则(默认规则除外);
  - `-Z`: 将所有 `table` 的所有 `chain` 的字节和数据包计数器清零;
  - `-n`: 使用数字形式显示输出结果;
  - `-v`: 查看规则表详细信息
  - `-V`: 查看版本;
  - `-P`: 设置指定默认`chain` 的默认策略;

- `chain`: 规则链名, 有五种
  - `INPUT`: 处理输入数据包。
  - `OUTPUT`: 处理输出数据包。
  - `FORWARD`: 处理转发数据包。
  - `PREROUTING`: 用于目标地址转换(DNAT)。
  - `POSTOUTING`: 用于源地址转换(SNAT)。

- `parameter`: 匹配规则规范:
  - `-p`: 指定规则或数据包的协议, 可以是 `tcp`、`udp`、`udplite`、`icmp`、`icmpv6`、`esp`、`ah`、`sctp`、`mh` 或者 `all`;
  - `-s`: 源地址, 地址可以是 `网络名称`、`主机名`、`IP地址(带/mask)`;
  - `-d`: 目标地址;
  - `-i`: 接收数据包的网卡名;
  - `-o`: 输出数据包的网卡名;
  - `--sport`: 数据包来源端口;
  - `--dport`: 数据包出去端口;

- `target`: 对数据包的操作:
  - `ACCEPT`: 接收;
  - `DROP`: 删除;
  - `REDIRECT`: 重定向、映射、透明代理。
  - `SNAT`: 源地址转换。
  - `DNAT`: 目标地址转换。
  - `MASQUERADE`: IP伪装（NAT），用于ADSL。
  - `LOG`: 日志记录。
  - `SEMARK`: 添加SEMARK标记以供网域内强制访问控制（MAC）



# 示例

## 允许ssh端口连接

```bash
# 允许 192.168.1.0 所有网段能访问 22 端口
$ iptables -t filter -I INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT
```

## 设置默认的规则

```shell
$ iptables -t filter -P INPUT -j DROP # 配置默认的不让进
$ iptables -P FORWARD DROP # 默认的不允许转发
$ iptables -P OUTPUT ACCEPT # 默认的可以出去
```

## 配置白名单

```shell
# 允许 192.168.1.100 可以发送 TCP 请求
$ iptables -t filter -I INPUT -s 192.168.1.100 -p tcp -j ACCEPT
# 允许 192.168.1.0 网段下所有机器访问
$ iptables -t filter -I INPUT -s 192.168.1.0.24 -p all -j ACCEPT
# 允许 192.168.1.100 机器访问 3306(mysql) 端口
$ iptables -t filter -I INPUT -p tcp -s 192.168.1.100 --dport 3306 -j ACCEPT
```

## 开启指定服务的端口可以被访问

```bash
# 开启 80 端口可以被访问
$ iptables -t filter -I INPUT -p tcp --dport 80 -j ACCEPT
# 允许被 ping 
$ iptables -t filter -I INPUT -p icmp -j ACCEPT
```

## 过滤指定IP&PORT 的TCP(SYN,ACK)报文

```bash
iptables -t filter -I INPUT -p tcp --sport 9999 --src 192.168.1.100 --tcp-flags SYN,ACK SYN,ACK -j DROP
```

## 删除已添加的规则

```bash
# 显示规则的序号
$ iptables -t filter -nL --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80

# 删除指定序号的规则
$ iptables -t filter -D INPUT 1
```









