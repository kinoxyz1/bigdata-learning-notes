



* [一、ClickHouse概述](#%E4%B8%80clickhouse%E6%A6%82%E8%BF%B0)
  * [1\.1 什么是ClickHouse](#11-%E4%BB%80%E4%B9%88%E6%98%AFclickhouse)
  * [1\.3 安装前的准备](#13-%E5%AE%89%E8%A3%85%E5%89%8D%E7%9A%84%E5%87%86%E5%A4%87)
    * [1\.3\.1 CentOS取消打开文件数限制](#131-centos%E5%8F%96%E6%B6%88%E6%89%93%E5%BC%80%E6%96%87%E4%BB%B6%E6%95%B0%E9%99%90%E5%88%B6)
    * [1\.3\.2 CentOS取消SELINUX](#132-centos%E5%8F%96%E6%B6%88selinux)
    * [1\.3\.3 关闭防火墙](#133-%E5%85%B3%E9%97%AD%E9%98%B2%E7%81%AB%E5%A2%99)
    * [1\.3\.4 安装依赖](#134-%E5%AE%89%E8%A3%85%E4%BE%9D%E8%B5%96)

---
# 一、ClickHouse概述
## 1.1 什么是ClickHouse
ClickHouse 是俄罗斯的Yandex于2016年开源的列式存储数据库（DBMS），主要用于在线分析处理查询（OLAP），能够使用SQL查询实时生成分析数据报告。

##1.2 什么是列式存储
以下面的表为例：

|  Id   | Name  | Age  |
|  ----  | ----  | ----  |
| 1  | 张三 | 18 |
| 2  | 李四 | 22 |
| 3  | 王五 | 34 |

采用行式存储时，数据在磁盘上的组织结构为：
```bash
1	张三	18	2	李四	22	3	王五	34
```
好处是想查某个人所有的属性时，可以通过一次磁盘查找加顺序读取就可以。但是当想查所有人的年龄时，需要不停的查找，或者全表扫描才行，遍历的很多数据都是不需要的。

而采用列式存储时，数据在磁盘上的组织结构为：
```bash
1	2	3	张三	李四	王五	18	22	34
```
这时想查所有人的年龄只需把年龄那一列拿出来就可以了

## 1.3 安装前的准备
### 1.3.1 CentOS取消打开文件数限制
在 `/etc/security/limits.conf`、`/etc/security/limits.d/90-nproc.conf` 这 2 个文件的末尾加入一下内容：
```bash
[root@kino100 software]# vim /etc/security/limits.conf
```
在文件末尾添加：
```bash
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072
```
```bash
[root@kino100 software]# vim /etc/security/limits.d/90-nproc.conf
```
在文件末尾添加：
```bash
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072
```

重启服务器之后生效，用 `ulimit -n` 或者 `ulimit -a` 查看设置结果
```bash
[root@kino100 ~]# ulimit -n
65536
```
### 1.3.2 CentOS取消SELINUX 
修改 `/etc/selinux/config` 中的 `SELINUX=disabled` 后重启
```bash
[root@kino100 ~]# vim /etc/selinux/config
SELINUX=disabled
```
### 1.3.3 关闭防火墙 
```bash
[root@kino100 ~]# service iptables stop 
[root@kino100 ~]# service ip6tables stop
ip6tables：将 chains 设置为 ACCEPT 策略：filter            [确定]
ip6tables：清除防火墙规则：                                [确定]
：正在卸载模块：                                           [确定]
```
###  1.3.4 安装依赖
```bash
[root@kino100 ~]# yum install -y libtool
[root@kino100 ~]# yum install -y *unixODBC*
```


