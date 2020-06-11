* [一、安装环境](#%E4%B8%80%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83)
* [二、关闭SElinux](#%E4%BA%8C%E5%85%B3%E9%97%ADselinux)
* [三、关闭防火墙](#%E4%B8%89%E5%85%B3%E9%97%AD%E9%98%B2%E7%81%AB%E5%A2%99)
* [四、安装JDK](#%E5%9B%9B%E5%AE%89%E8%A3%85jdk)
* [五、在线安装 mariadb](#%E4%BA%94%E5%9C%A8%E7%BA%BF%E5%AE%89%E8%A3%85-mariadb)
* [六、修改Centos镜像](#%E5%85%AD%E4%BF%AE%E6%94%B9centos%E9%95%9C%E5%83%8F)
* [七、安装PHP环境](#%E4%B8%83%E5%AE%89%E8%A3%85php%E7%8E%AF%E5%A2%83)
* [八、安装配置httpd](#%E5%85%AB%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AEhttpd)
* [九、建Zabbix用户](#%E4%B9%9D%E5%BB%BAzabbix%E7%94%A8%E6%88%B7)
* [十、编译Zabbix软件](#%E5%8D%81%E7%BC%96%E8%AF%91zabbix%E8%BD%AF%E4%BB%B6)
  * [10\.1 安装Zabbix编译的软件包](#101-%E5%AE%89%E8%A3%85zabbix%E7%BC%96%E8%AF%91%E7%9A%84%E8%BD%AF%E4%BB%B6%E5%8C%85)
  * [10\.2 去官网下载编译安装的Zabbix：](#102-%E5%8E%BB%E5%AE%98%E7%BD%91%E4%B8%8B%E8%BD%BD%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85%E7%9A%84zabbix)
  * [10\.3 编译安装Zabbix](#103-%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85zabbix)
* [十一、导入Zabbix数据库文件](#%E5%8D%81%E4%B8%80%E5%AF%BC%E5%85%A5zabbix%E6%95%B0%E6%8D%AE%E5%BA%93%E6%96%87%E4%BB%B6)
* [十二、配置root用户远程访问权限](#%E5%8D%81%E4%BA%8C%E9%85%8D%E7%BD%AEroot%E7%94%A8%E6%88%B7%E8%BF%9C%E7%A8%8B%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90)
* [十三、修改Zabbix配置文件](#%E5%8D%81%E4%B8%89%E4%BF%AE%E6%94%B9zabbix%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
* [十四、修改PHP配置文件](#%E5%8D%81%E5%9B%9B%E4%BF%AE%E6%94%B9php%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
* [十五、部署Zabbix前台文件](#%E5%8D%81%E4%BA%94%E9%83%A8%E7%BD%B2zabbix%E5%89%8D%E5%8F%B0%E6%96%87%E4%BB%B6)
* [十六、重启所有服务使配置生效](#%E5%8D%81%E5%85%AD%E9%87%8D%E5%90%AF%E6%89%80%E6%9C%89%E6%9C%8D%E5%8A%A1%E4%BD%BF%E9%85%8D%E7%BD%AE%E7%94%9F%E6%95%88)
* [十七、通过网页安装Zabbix](#%E5%8D%81%E4%B8%83%E9%80%9A%E8%BF%87%E7%BD%91%E9%A1%B5%E5%AE%89%E8%A3%85zabbix)



---
# 一、安装环境
操作系统：CentOS7.7
WEB：Apache/2.4.6
PHP：7.0.33
数据库：MySQL

---
# 二、关闭SElinux

```bash
[root@zabbix ~]# setenforce 0
[root@zabbix ~]# vim /etc/selinux/config  
SELINUX=enforcing     #将enforcing替换为disabled
SELINUX=disabled
```
---
# 三、关闭防火墙

```bash
#如果开启了iptables防火墙可以关闭
[root@zabbix ~]# systemctl stop firewalld
[root@zabbix ~]# systemctl disable firewalld.service
```
---
# 四、安装JDK

```bash
[root@zabbix ~]# mkdir /usr/java
```
将放在服务器上的 jdk-8u181-linux-x64.tar.gz 解压到 /usr/java 目录下

```bash
[root@zabbix ~]# tar -zxvf /opt/software/jdk-8u181-linux-x64.tar.gz -C /usr/java/
```
配置 JAVA_HOME 环境变量

```bash
[root@zabbix ~]# cat >> /etc/profile << EOF
> #JAVA_HOME
> export JAVA_HOME=/usr/java/jdk1.8.0_181
> export PATH=$PATH:$JAVA_HOME/bin
> EOF
[root@zabbix ~]# source /etc/profile
[root@zabbix ~]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

---
# 五、在线安装 mariadb
[离线安装跳转](https://blog.csdn.net/Java_Road_Far/article/details/98041498)

```bash
[root@zabbix ~]# yum install -y mariadb-server mariadb
[root@zabbix ~]# systemctl start mariadb
[root@zabbix ~]# systemctl enable mariadb 
```
---
# 六、修改Centos镜像
[跳转修改Centos镜像](https://blog.csdn.net/Java_Road_Far/article/details/106413786)

---
# 七、安装PHP环境
```bash
[root@zabbix ~]# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
[root@zabbix ~]# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
[root@zabbix ~]# yum install -y  php70w* --skip-broken
[root@zabbix ~]# systemctl start php-fpm
[root@zabbix ~]# systemctl enable php-fpm
```

---
# 八、安装配置httpd
```bash
[root@zabbix ~]# systemctl start httpd
[root@zabbix ~]# systemctl enable httpd
```
---
# 九、建Zabbix用户

```bash
[root@zabbix ~]# groupadd zabbix
[root@zabbix ~]# useradd zabbix -g zabbix -s /sbin/nologin
```

---
# 十、编译Zabbix软件
## 10.1 安装Zabbix编译的软件包
```bash
### [root@zabbix ~]# yum install -y gcc  libxml2-devel libevent-devel net-snmp net-snmp-devel  curl  curl-devel php  php-bcmath  php-mbstring mariadb mariadb-devel java-1.6.0-openjdk-devel --skip-broken

[root@zabbix ~]# yum install -y libcurl libcurl-devel libxml2 libxml2-devel net-snmp-devel libevent-devel pcre-devel gcc-c++
```
## 10.2 去官网下载编译安装的Zabbix：
https://www.zabbix.com/download_sources

```bash
[root@zabbix ~]# wget https://www.xxshell.com/download/sh/zabbix/zabbix4.4/zabbix-4.4.1.tar.gz
[root@zabbix ~]# tar -xzvf zabbix-4.4.1.tar.gz
[root@zabbix ~]# cd zabbix-4.4.1
 
[root@zabbix ~]# ./configure  \
--prefix=/usr/local/zabbix  \
--enable-server  \
--enable-agent  \
--with-mysql=/usr/bin/mysql_config   \
--with-net-snmp  \
--with-libcurl  \
--with-libxml2  \
--enable-java  
```
## 10.3 编译安装Zabbix
```bash
[root@zabbix ~]# make -j 2 && make install 
```
---
# 十一、导入Zabbix数据库文件

```bash
1、配置数据库密码(离线安装跳过此步骤)
[root@zabbix ~]# mysqladmin -uroot -p  password [新密码]
#修改数据库密码
 
2、连接数据库
[root@zabbix ~]# mysql -uroot -p
3、建立zabbix空数据库
[root@zabbix ~]# CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_general_ci;
[root@zabbix ~]# SHOW DATABASES;
[root@zabbix ~]# use zabbix;
#选择数据库
4、导入数据（注意sql文件的路径、并按照顺序导入）
[root@zabbix ~]# source database/mysql/schema.sql;
[root@zabbix ~]# source database/mysql/images.sql;
[root@zabbix ~]# source database/mysql/data.sql;
quit
```
---

# 十二、配置root用户远程访问权限

```bash
mysql> grant all privileges on *.* to 'root' @'%' identified by '上面设置的密码';
mysql> flush privileges;
```

---
# 十三、修改Zabbix配置文件

```bash
1、修改启动文件
cp misc/init.d/fedora/core/* /etc/init.d/
#拷贝启动文件到/etc/init.d/下
 
sed -i "s#BASEDIR=/usr/local#BASEDIR=/usr/local/zabbix#" /etc/init.d/zabbix_server
sed -i "s#BASEDIR=/usr/local#BASEDIR=/usr/local/zabbix#" /etc/init.d/zabbix_agentd
#快速替换，如果手动修改可以直接编辑下面的文件
vim /etc/init.d/zabbix_agentd
vim /etc/init.d/zabbix_server
分别将”BASEDIR=/usr/local“替换为”BASEDIR=/usr/local/zabbix“
 
2、修改Zabbix配置文件
vim /usr/local/zabbix/etc/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=root
DBPassword=[数据库密码]
 
#启动zabbix服务
/etc/init.d/zabbix_server restart
/etc/init.d/zabbix_agentd restart
 
systemctl restart zabbix_server 
systemctl restart zabbix_agentd
#重启验证服务
#通过”netstat -an | grep LIS“查看10050、10051端口能否正常监听，如果不能正常监听可能数据库或配置文件有问题。
 
systemctl enable zabbix_server
systemctl enable zabbix_agentd
#设置开机启动
```
---
# 十四、修改PHP配置文件
```bash
sed -i "s/post_max_size = 8M/post_max_size = 32M/" /etc/php.ini
sed -i "s/max_execution_time = 30/max_execution_time = 600/" /etc/php.ini
sed -i "s/max_input_time = 60/max_input_time = 600/" /etc/php.ini
sed -i "s#;date.timezone =#date.timezone = Asia/Shanghai#" /etc/php.ini
 
#或手动修改文件/etc/php.ini
post_max_size = 8M      替换为 post_max_size = 32M
max_execution_time = 30 替换为 max_execution_time = 600
max_input_time = 60     替换为 max_input_time = 600
;date.timezone =        替换为 date.timezone = Asia/Shanghai
```
# 十五、部署Zabbix前台文件

```bash
rm -rf /var/www/html/*
#清空网站根目录
cp -r frontends/php/* /var/www/html/
#复制PHP文件到网站根目录
chown -R apache:apache  /var/www/html/
chmod -R 777 /var/www/html/conf/
#给网站目录添加属主
```
---
# 十六、重启所有服务使配置生效
```bash
systemctl restart php-fpm httpd mariadb zabbix_server zabbix_agentd
```

---
# 十七、通过网页安装Zabbix
待补全