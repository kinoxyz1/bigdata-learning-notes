
* [一、Docker 容器数据卷是什么](#%E4%B8%80docker-%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%E6%98%AF%E4%BB%80%E4%B9%88)
* [二、Docker 容器数据卷能干嘛](#%E4%BA%8Cdocker-%E5%AE%B9%E5%99%A8%E6%95%B0%E6%8D%AE%E5%8D%B7%E8%83%BD%E5%B9%B2%E5%98%9B)
* [三、数据卷](#%E4%B8%89%E6%95%B0%E6%8D%AE%E5%8D%B7)
  * [3\.1 命令添加](#31-%E5%91%BD%E4%BB%A4%E6%B7%BB%E5%8A%A0)
    * [3\.1\.1 添加数据卷](#311-%E6%B7%BB%E5%8A%A0%E6%95%B0%E6%8D%AE%E5%8D%B7)
    * [3\.1\.2 查看是否添加成功](#312-%E6%9F%A5%E7%9C%8B%E6%98%AF%E5%90%A6%E6%B7%BB%E5%8A%A0%E6%88%90%E5%8A%9F)
    * [3\.1\.3 容器和宿主机之间数据共享](#313-%E5%AE%B9%E5%99%A8%E5%92%8C%E5%AE%BF%E4%B8%BB%E6%9C%BA%E4%B9%8B%E9%97%B4%E6%95%B0%E6%8D%AE%E5%85%B1%E4%BA%AB)
    * [3\.1\.4 带权限命令](#314-%E5%B8%A6%E6%9D%83%E9%99%90%E5%91%BD%E4%BB%A4)
  * [3\.2 DockerFile 添加](#32-dockerfile-%E6%B7%BB%E5%8A%A0)
    * [3\.2\.1 在 根目录 下创建 mydocker 文件夹进入](#321-%E5%9C%A8-%E6%A0%B9%E7%9B%AE%E5%BD%95-%E4%B8%8B%E5%88%9B%E5%BB%BA-mydocker-%E6%96%87%E4%BB%B6%E5%A4%B9%E8%BF%9B%E5%85%A5)
    * [3\.2\.2 添加 dockerfile1 文件, 拷贝如下内容](#322-%E6%B7%BB%E5%8A%A0-dockerfile1-%E6%96%87%E4%BB%B6-%E6%8B%B7%E8%B4%9D%E5%A6%82%E4%B8%8B%E5%86%85%E5%AE%B9)
    * [3\.2\.3 build 后生成镜像](#323-build-%E5%90%8E%E7%94%9F%E6%88%90%E9%95%9C%E5%83%8F)
    * [3\.2\.5 run 容器](#325-run-%E5%AE%B9%E5%99%A8)
    * [3\.2\.6 容器内数据卷对应主机目录地址](#326-%E5%AE%B9%E5%99%A8%E5%86%85%E6%95%B0%E6%8D%AE%E5%8D%B7%E5%AF%B9%E5%BA%94%E4%B8%BB%E6%9C%BA%E7%9B%AE%E5%BD%95%E5%9C%B0%E5%9D%80)
  * [4\. 备注](#4-%E5%A4%87%E6%B3%A8)
* [四、数据卷容器](#%E5%9B%9B%E6%95%B0%E6%8D%AE%E5%8D%B7%E5%AE%B9%E5%99%A8)

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
# 容器内
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

# 宿主机上
[root@docker1 ~]# ll /
总用量 32
lrwxrwxrwx.   1 root root     7 7月  22 01:49 bin -> usr/bin
dr-xr-xr-x.   6 root root  4096 7月  22 02:01 boot
drwxr-xr-x.  20 root root  3320 7月  28 13:06 dev
drwxr-xr-x. 150 root root 12288 7月  28 16:09 etc
drwxr-xr-x.   3 root root    18 7月  22 02:21 home
drwxr-xr-x.   3 root root    35 7月  29 13:50 kino                <--------------------
lrwxrwxrwx.   1 root root     7 7月  22 01:49 lib -> usr/lib
lrwxrwxrwx.   1 root root     9 7月  22 01:49 lib64 -> usr/lib64
drwxr-xr-x.   2 root root     6 4月  11 2018 media
drwxr-xr-x.   2 root root     6 4月  11 2018 mnt
drwxr-xr-x.   4 root root    34 7月  21 18:43 opt
dr-xr-xr-x. 227 root root     0 7月  28 13:06 proc
dr-xr-x---.  15 root root  4096 7月  29 13:50 root
drwxr-xr-x.  47 root root  1400 7月  29 06:10 run
lrwxrwxrwx.   1 root root     8 7月  22 01:49 sbin -> usr/sbin
drwxr-xr-x.   2 root root     6 4月  11 2018 srv
dr-xr-xr-x.  13 root root     0 7月  28 13:06 sys
drwxrwxrwt.  36 root root  4096 7月  29 16:30 tmp
drwxr-xr-x.  13 root root   155 7月  22 01:49 usr
drwxr-xr-x.  22 root root  4096 7月  22 02:18 var
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

## 3.2 DockerFile 添加
### 3.2.1 在 根目录 下创建 mydocker 文件夹进入
```bash
[root@docker1 ~]# mkdir mydocker
[root@docker1 ~]# cd mydocker/
```

### 3.2.2 添加 dockerfile1 文件, 拷贝如下内容
```bash
[root@docker1 ~]# vim /dockerfile1
# volume test
FROM centos
# 使用 VOLUME 指令来给镜像添加一个或多个数据卷
# 出于可移植和分享的考虑，用-v 主机目录:容器目录这种方法不能够直接在Dockerfile中实现。
# 由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished,--------success1"
CMD /bin/bash
```

### 3.2.3 build 后生成镜像
```bash
[root@docker1 mydocker]# docker build -f /mydocker/dockerfile1 -t kino/centos .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 831691599b88
Step 2/4 : VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
 ---> Running in 1f88f305676f
Removing intermediate container 1f88f305676f
 ---> 360172a0f2de
Step 3/4 : CMD echo "finished,--------success1"
 ---> Running in 2d5ffb9f23f5
Removing intermediate container 2d5ffb9f23f5
 ---> c761e9fd967f
Step 4/4 : CMD /bin/bash
 ---> Running in 0d9c38ee53ae
Removing intermediate container 0d9c38ee53ae
 ---> c051364b610b
Successfully built c051364b610b
Successfully tagged kino/centos:latest
[root@docker1 mydocker]# 
```

### 3.2.5 run 容器
```bash
[root@docker1 mydocker]# docker run -it kino/centos /bin/bash
[root@827ca3c2b83c /]# ls -l
total 0
lrwxrwxrwx.   1 root root   7 May 11  2019 bin -> usr/bin
drwxr-xr-x.   2 root root   6 Jul 29 08:58 dataVolumeContainer1          <-----------
drwxr-xr-x.   2 root root   6 Jul 29 08:58 dataVolumeContainer2          <-----------
drwxr-xr-x.   5 root root 360 Jul 29 08:58 dev
drwxr-xr-x.   1 root root  66 Jul 29 08:58 etc
drwxr-xr-x.   2 root root   6 May 11  2019 home
lrwxrwxrwx.   1 root root   7 May 11  2019 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 May 11  2019 lib64 -> usr/lib64
drwx------.   2 root root   6 Jun 11 02:35 lost+found
drwxr-xr-x.   2 root root   6 May 11  2019 media
drwxr-xr-x.   2 root root   6 May 11  2019 mnt
drwxr-xr-x.   2 root root   6 May 11  2019 opt
dr-xr-xr-x. 227 root root   0 Jul 29 08:58 proc
dr-xr-x---.   2 root root 162 Jun 11 02:35 root
drwxr-xr-x.  11 root root 163 Jun 11 02:35 run
lrwxrwxrwx.   1 root root   8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 May 11  2019 srv
dr-xr-xr-x.  13 root root   0 Jul 28 05:06 sys
drwxrwxrwt.   7 root root 145 Jun 11 02:35 tmp
drwxr-xr-x.  12 root root 144 Jun 11 02:35 usr
drwxr-xr-x.  20 root root 262 Jun 11 02:35 var

[root@827ca3c2b83c /]# vim dataVolumeContainer1/dataVolumeContainer1.txt
dataVolumeContainer1 text
```

### 3.2.6 容器内数据卷对应主机目录地址
```bash
"Mounts": [
            {
                "Type": "volume",
                "Name": "fa6b53a5f9a649c66945885503ef4fcdeb8c597fa863f1f65fcef806d9a70cb0",
                "Source": "/var/lib/docker/volumes/fa6b53a5f9a649c66945885503ef4fcdeb8c597fa863f1f65fcef806d9a70cb0/_data",
                "Destination": "/dataVolumeContainer1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "c55b78d4bcfca8855aa0af175e669068e2ff2ab78a83bee5937ca9ab84e0747a",
                "Source": "/var/lib/docker/volumes/c55b78d4bcfca8855aa0af175e669068e2ff2ab78a83bee5937ca9ab84e0747a/_data",
                "Destination": "/dataVolumeContainer2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],


[root@docker1 mysql]# cd /var/lib/docker/volumes/fa6b53a5f9a649c66945885503ef4fcdeb8c597fa863f1f65fcef806d9a70cb0/_data
[root@docker1 _data]# ll
总用量 4
-rw-r--r--. 1 root root 26 7月  29 17:04 dataVolumeContainer1.txt
[root@docker1 _data]# cat dataVolumeContainer1.txt
dataVolumeContainer1 text
```

## 4. 备注
Docker挂载主机目录Docker访问出现 `cannot open directory .: Permission denied`

解决办法：在挂载目录后多加一个--privileged=true参数即可


# 四、数据卷容器