




------------
# 一、Docker 容器数据卷是什么
先来看看Docker的理念:
*  将运用与运行的环境打包形成容器运行, 运行可以伴随着容器, 但是我们对数据的要求希望是持久化的
*  容器之间希望有可能共享数据
  
Docker容器产生的数据, 如果不通过docker commit生成新的镜像, 使得数据做为镜像的一部分保存下来, 那么当容器删除后, 数据自然也就没有了.
 
为了能保存数据, 在 docker 中我们使用卷.



# 二、Docker 容器数据卷能干嘛
docker 容器数据卷可以: 
- 容器的持久化
- 容器间的继承+共享数据 

卷就是目录或文件, 存在于一个或多个容器中, 由docker挂载到容器, 但不属于联合文件系统, 因此能够绕过Union File System提供一些用于持续存储或共享数据的特性:
 
    卷的设计目的就是数据的持久化, 完全独立于容器的生存周期, 因此Docker不会在容器删除时删除其挂载的数据卷
 
特点:
1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止


# 三、数据卷
添加数据卷有两种方式:
- 命令添加
- DockerFile 添加

## 3.1 命令添加
### 3.1.1 添加数据卷
语法:
```bash
docker run -it -v 宿主机目录:容器内目录 镜像名
```
案例:
```bash
[root@docker1 mysql]# docker run -it -v /kino:/datakino centos /bin/bash
[root@36a64f54a4d7 /]# ls -l /
total 0
lrwxrwxrwx.   1 root root   7 May 11  2019 bin -> usr/bin
drwxr-xr-x.   3 root root  19 Jul 28 07:56 datakino           <--------------
drwxr-xr-x.   5 root root 360 Jul 29 05:43 dev
drwxr-xr-x.   1 root root  66 Jul 29 05:43 etc
drwxr-xr-x.   2 root root   6 May 11  2019 home
lrwxrwxrwx.   1 root root   7 May 11  2019 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 May 11  2019 lib64 -> usr/lib64
drwx------.   2 root root   6 Jun 11 02:35 lost+found
drwxr-xr-x.   2 root root   6 May 11  2019 media
drwxr-xr-x.   2 root root   6 May 11  2019 mnt
drwxr-xr-x.   2 root root   6 May 11  2019 opt
dr-xr-xr-x. 225 root root   0 Jul 29 05:43 proc
dr-xr-x---.   2 root root 162 Jun 11 02:35 root
drwxr-xr-x.  11 root root 163 Jun 11 02:35 run
lrwxrwxrwx.   1 root root   8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 May 11  2019 srv
dr-xr-xr-x.  13 root root   0 Jul 28 05:06 sys
drwxrwxrwt.   7 root root 145 Jun 11 02:35 tmp
drwxr-xr-x.  12 root root 144 Jun 11 02:35 usr
drwxr-xr-x.  20 root root 262 Jun 11 02:35 var
```

### 3.1.2 查看是否添加成功
```bash
[root@docker1 mysql]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
36a64f54a4d7        centos              "/bin/bash"         2 minutes ago       Up 2 minutes                            stupefied_bassi
[root@docker1 mysql]# docker inspect 36a64f54a4d7

----------------------
"Binds": [
    "/kino:/datakino"
],
----------------------
```

### 3.1.3 容器和宿主机之间数据共享
```bash
在宿主机上
[root@docker1 ~]# cd /kino
[root@docker1 kino]# vim kino.txt
kino test

在容器上
[root@36a64f54a4d7 ~]# cd /datakino/
[root@36a64f54a4d7 datakino]# ls -l
total 4
-rw-r--r--. 1 root root 10 Jul 29 05:50 kino.txt
[root@36a64f54a4d7 datakino]# cat kino.txt
kino test
```

### 3.1.4 带权限命令
只读权限
```bash
docker run -it -v 宿主机目录:容器内目录:ro 镜像名
```


# 四、数据卷容器