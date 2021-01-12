




----
# 一、ubuntu 部署 vsftpd
```bash
$ apt-get install vsftpd
```
# 二、配置 vsftpd
## 2.1 备份配置文件
```bash
$ cp /etc/vsftpd.conf /etc/vsftpd.conf.orig
```
## 2.2 编辑vsftpd.config
```bash
vim /etc/vsftpd.config
write_enable=YES
local_umask=022
xferlog_file=/var/log/vsftpd.log
xferlog_std_format=YES
ftpd_banner=Welcome Lincoln Linux FTP Service.
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
pam_service_name=ftp
utf8_filesystem=YES
userlist_enable=YES
userlist_deny=NO
userlist_file=/etc/vsftpd.user_list
allow_writeable_chroot=YES
```

# 三、创建ftp目录
```bash
$ mkdir /data1/ftp/uploadFile
```

# 四、创建登录用户
```bash
#添加用户
$ sudo useradd -d /data1/ftp/ -s /bin/bash ftpuser
#设置用户密码
$ sudo passwd ftpuser
#设置ftp目录用户权限
$ sudo chown ftpuser:ftpuser /data1/ftp/
```

# 五、添加vsftpd 登录用户
```bash
#新建文件/etc/vsftpd.user_list，用于存放允许访问ftp的用户：
$ vim /etc/vsftpd.user_list
ftpuser
```

# 六、添加vsftpd登录用户对目录树的权限
```bash
#新建文件/etc/vsftpd.chroot_list，设置可列出、切换目录的用户：
$ vim /etc/vsftpd.chroot_list
ftpuser
```

# 七、重启 vsftpd 服务
```bash
$ service vsftpd restart
```

# 八、windows 登录 ftp
```bash
C:\Users\raomi>ftp 192.168.1.141
连接到 192.168.1.141。
220 Welcome to blah FTP service.
200 Always in UTF8 mode.
用户(192.168.1.141:(none)): ftpuser
331 Please specify the password.
密码:
230 Login successful.

ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
uploadFile
226 Directory send OK.
ftp: 收到 15 字节，用时 0.00秒 15000.00千字节/秒。
ftp>

ftp> put sanct.log
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
ftp: 发送 486 字节，用时 0.02秒 30.38千字节/秒。

ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
sanct.log
uploadFile
226 Directory send OK.
ftp: 收到 26 字节，用时 0.00秒 13.00千字节/秒。
ftp>
```