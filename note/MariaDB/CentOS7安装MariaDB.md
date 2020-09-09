



---
```bash
[root@centos ~]# yum install -y mariadb-server mariadb
[root@centos ~]# systemctl start mariadb
[root@centos ~]# systemctl enable mariadb 
[root@centos ~]# mysql_secure_installation

首先是设置密码，会提示先输入密码

Enter current password for root (enter for none):<–初次运行直接回车

设置密码

Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
New password: <– 设置root用户的密码
Re-enter new password: <– 再输入一次你设置的密码

其他配置

Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车

Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,

Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车

Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车

[root@centos ~]# mysql -uroot -p
输入密码.
```