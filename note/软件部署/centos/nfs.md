




---


# 一、server (服务端)
## 1.1 安装
```bash
# install 
yum install nfs-utils rpcbind
```

## 1.2 编辑配置文件
```bash
vim /etc/exports
/app/nfs *(async,insecure,no_root_squash,no_subtree_check,rw)

# 解释说明
<输出目录> 客户端（选项:访问权限,用户映射,其他]
        输出目录是指NFS系统中所定义的共享给客户端使用的文件系统
        客户端是定义网络中可以访问这个NFS共享目录的IP地址或网段或域名等
            客户端常用的指定方式
                指定ip地址的主机：192.168.100.1
                指定一个子网：192.168.100.0/24 也可以写成:192.168.100.0/255.255.255.0
                指定域名的主机：david.bsmart.cn
                指定域中的所有主机：*.bsmart.cn
                所有主机：*
        选项用来设置输出目录的访问权限、用户映射等。
            NFS主要有3类选项：
                设置输出目录只读：ro
                设置输出目录读写：rw
            用户映射选项
                all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
                no_all_squash：与all_squash取反（默认设置）；
                root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；
                no_root_squash：与rootsquash取反；
                anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
                anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；
            其它选项
                secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；
                insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
                sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
                async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
                wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
                no_wdelay：若有写操作则立即执行，应与sync配合使用；
                subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；
                no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
```

## 1.3 授权
```bash
chown -R nfsnobody.nfsnobody /app/nfs
```

## 1.4 设置开机自启
```bash
systemctl enable rpcbind.service
systemctl enable nfs-server.service

systemctl start rpcbind.service
systemctl start nfs-server.service


systemctl stop nfs-server.service
systemctl stop nfs-mountd.service


systemctl status nfs-server.service
systemctl status nfs-mountd.service
```

## 1.5 检查
```bash
showmount -e 192.168.156.60
Export list for 192.168.156.60:
/app/nfs *
```


# 二、clinet (客户端)

## 2.1 安装 nfs
```bash
yum install nfs-utils
```

## 2.2 挂载
```bash
mkdir -p /app/nfs
mount -t nfs 192.168.156.60:/app/nfs /app/nfs
```