

```bash
## openssh version request must pass 4.8p1
ssh -V  
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017

## create a user and set up a user homepage
mkdir -p /app/sftp/sftpuser
useradd -d /app/sftp/sftpuser -M -s /sbin/nologin sftpuser
passwd sftpuser  ## set up sftpuser password
chmod 755 /app/sftp/sftpuser
chown root.sftpuser /app/sftp/sftpuser
# mkdir /data/sftpuser/upload
# chmod 755 /data/sftpuser/upload/
# chown -R sftpuser.sftpuser /data/sftpuser/upload/

vim /etc/ssh/sshd_config
### comment out this
#X11Forwarding yes
#Subsystem      sftp    /usr/libexec/openssh/sftp-server
Subsystem sftp internal-sftp
Match Group sftpusers
ChrootDirectory /app/sftp/sftpuser/%u
X11Forwarding no
AllowTcpForwarding no

## close selinux 
sed -i s#enforcing#disabled#g /etc/sysconfig/selinux 
setenforce 0 && getenforce
getenforce

## restart sshd
systemctl restart sshd 

## login 
sftp -P 22 sftpuser@127.0.0.1
```