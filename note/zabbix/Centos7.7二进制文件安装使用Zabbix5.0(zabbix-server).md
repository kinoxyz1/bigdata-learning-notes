* [一、系统环境](#%E4%B8%80%E7%B3%BB%E7%BB%9F%E7%8E%AF%E5%A2%83)
* [二、配置阿里云yum源](#%E4%BA%8C%E9%85%8D%E7%BD%AE%E9%98%BF%E9%87%8C%E4%BA%91yum%E6%BA%90)
* [三、Zabbix5\.0 YUM源](#%E4%B8%89zabbix50-yum%E6%BA%90)
* [四、Zabbix5\.0 YUM 源开启web前端安装](#%E5%9B%9Bzabbix50-yum-%E6%BA%90%E5%BC%80%E5%90%AFweb%E5%89%8D%E7%AB%AF%E5%AE%89%E8%A3%85)
* [五、创建 Zabbix 用户](#%E4%BA%94%E5%88%9B%E5%BB%BA-zabbix-%E7%94%A8%E6%88%B7)
* [六、导入初始数据](#%E5%85%AD%E5%AF%BC%E5%85%A5%E5%88%9D%E5%A7%8B%E6%95%B0%E6%8D%AE)
* [七、Zabbix server 配置修改](#%E4%B8%83zabbix-server-%E9%85%8D%E7%BD%AE%E4%BF%AE%E6%94%B9)
* [八、更改fpm配置](#%E5%85%AB%E6%9B%B4%E6%94%B9fpm%E9%85%8D%E7%BD%AE)
* [九、启动Zabbix server和agent进程](#%E4%B9%9D%E5%90%AF%E5%8A%A8zabbix-server%E5%92%8Cagent%E8%BF%9B%E7%A8%8B)

---
# 一、系统环境
	Centos7.7、
	关闭防火墙、
	关闭selinux

# 二、配置阿里云yum源
```bash
curl -o /etc/yum.repos.d/epel.repos http://mirrors.aliyun.com/repo/epel-7.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

# 三、Zabbix5.0 YUM源
```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
```
Zabbix5.0 YUM源切换到阿里 
```bash
vim /etc/yum.repos.d/zabbix.repo
%s#repo.zabbix.com#mirrors.aliyun.com/zabbix#g
yum clean all
yum makecache
```

# 四、Zabbix5.0 YUM 源开启web前端安装

安装 Zabbix 服务端和客户端
```bash
yum install zabbix-server-mysql zabbix-agent -y
```
安装 centos scl
```bash
yum install centos-release-scl -y
```
安装 Zabbix Web
```bash
yum install zabbix-web-mysql-scl zabbix-nginx-conf-scl -y
```
安装Zabbix前端软件包

```bash
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl
```

安装 Mariadb 服务器
```bash
yum install mariadb-server mariadb -y
systemctl restart mariadb
mysql_secure_installation 设置密码: 
```

# 五、创建 Zabbix 用户
```bash
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@localhost identified by '';
grant all privileges on zabbix.* to zabbix@localhost;
flush privileges;
```

# 六、导入初始数据
```bash
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uroot -pKino123. zabbix
```

# 七、Zabbix server 配置修改
```bash
vim /etc/zabbix/zabbix_server.conf
DBPassword=密码
```

# 八、更改fpm配置
```bash
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
listen.acl users = apache,nginx
php_value[max_execution_time] = 300
php_value[memory_limit] = 128M
php_value[post_max_size] = 16M
php_value[upload_max_filesize] = 2M
php_value[max_input_time] = 300
php_value[max_input_vars] = 10000
php_value[data.timezone] = Asia/Shanghai
```


​	
# 九、启动Zabbix server和agent进程
启动Zabbix server和agent进程，并为它们设置开机自启：
```bash
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```

访问网页体验zabbix5.0