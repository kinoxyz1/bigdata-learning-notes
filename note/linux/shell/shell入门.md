
* [一、Shell 解析器](#%E4%B8%80shell-%E8%A7%A3%E6%9E%90%E5%99%A8)
  * [1\.1 Linux 提供的 Shell 解析器有如下几种:](#11-linux-%E6%8F%90%E4%BE%9B%E7%9A%84-shell-%E8%A7%A3%E6%9E%90%E5%99%A8%E6%9C%89%E5%A6%82%E4%B8%8B%E5%87%A0%E7%A7%8D)
  * [2\.2 bash 和 sh 的关系](#22-bash-%E5%92%8C-sh-%E7%9A%84%E5%85%B3%E7%B3%BB)
  * [2\.3 CentOS 默认解析器](#23-centos-%E9%BB%98%E8%AE%A4%E8%A7%A3%E6%9E%90%E5%99%A8)
* [二、Shell 入门](#%E4%BA%8Cshell-%E5%85%A5%E9%97%A8)
  * [2\.1 Shell 脚本格式](#21-shell-%E8%84%9A%E6%9C%AC%E6%A0%BC%E5%BC%8F)
  * [2\.2 Shell 脚本常用的执行方式](#22-shell-%E8%84%9A%E6%9C%AC%E5%B8%B8%E7%94%A8%E7%9A%84%E6%89%A7%E8%A1%8C%E6%96%B9%E5%BC%8F)
    * [2\.2\.1 第一种](#221-%E7%AC%AC%E4%B8%80%E7%A7%8D)
    * [2\.2\.2 第二种](#222-%E7%AC%AC%E4%BA%8C%E7%A7%8D)
  * [2\.2 HelloWorld](#22-helloworld)
  * [2\.3 多命令处理](#23-%E5%A4%9A%E5%91%BD%E4%BB%A4%E5%A4%84%E7%90%86)

---
# 一、Shell 解析器
## 1.1 Linux 提供的 Shell 解析器有如下几种:
```bash
[root@hadoop1 ~]# cat /etc/shells 
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```

## 2.2 bash 和 sh 的关系
```bash
[root@hadoop1 ~]# cd /bin
[root@hadoop1 bin]# ll | grep bash
-rwxr-xr-x. 1 root root    964536 4月   1 10:17 bash
lrwxrwxrwx. 1 root root         4 8月  11 16:13 sh -> bash
```

## 2.3 CentOS 默认解析器
```bash
[root@hadoop1 bin]# echo $SHELL
/bin/bash
```

# 二、Shell 入门
## 2.1 Shell 脚本格式
脚本以 `#!/bin/bash` 开头, 表示指定解析器为 `/bin/bash`

## 2.2 Shell 脚本常用的执行方式
### 2.2.1 第一种
采用 `bash` 或 `sh` + 脚本的相对路径或绝对路径(不为脚本赋予+x权限)

① sh+脚本相对路径
```bash
[root@hadoop1 shell]# sh helloworld.sh 
helloworld
```
② sh+脚本绝对路径
```bash
[root@hadoop1 shell]# sh /root/shell/helloworld.sh 
helloworld
```
③ bash+脚本相对路径
```bash
[root@hadoop1 shell]# bash helloworld.sh 
helloworld
```
④ bash+脚本绝对路径
```bash
[root@hadoop1 shell]# bash /root/shell/helloworld.sh 
helloworld
```

### 2.2.2 第二种
采用 `bash` 或 `sh` + 脚本的相对路径或绝对路径(<font color='red'>不为脚本赋予+x权限</font>)

① 为脚本赋予 `+x` 权限
```bash
[root@hadoop1 shell]# chmod 777 helloworld.sh 
[root@hadoop1 shell]# ll
总用量 4
-rwxrwxrwx. 1 root root 30 8月  13 20:50 helloworld.sh
```

① 脚本相对路径
```bash
[root@hadoop1 shell]# ./helloworld.sh 
helloworld
```
② 脚本绝对路径
```bash
[root@hadoop1 shell]# /root/shell/helloworld.sh 
helloworld
```

## 2.2 HelloWorld
创建一个 Shell 脚本, 输出 'helloworld'
```bash
[root@hadoop1 shell]# vim helloworld.sh
#!/bin/bash
echo "helloworld"

[root@hadoop1 shell]# sh helloworld.sh 
helloworld
```

## 2.3 多命令处理
用 shell 命令在 `/root/shell` 目录下创建一个 a.txt 并在文件中增加 "shell six six six"
```bash
[root@hadoop1 shell]# vim batch.sh

#!/bin/bash

cd /root/shell
touch a.txt
echo "shell six six six" >> a.txt

[root@hadoop1 shell]# ll
总用量 8
-rw-r--r--. 1 root root 78 8月  13 21:02 batch.sh
-rwxrwxrwx. 1 root root 30 8月  13 20:50 helloworld.sh

[root@hadoop1 shell]# sh batch.sh 

[root@hadoop1 shell]# ll
总用量 12
-rw-r--r--. 1 root root 18 8月  13 21:03 a.txt
-rw-r--r--. 1 root root 74 8月  13 21:02 batch.sh
-rwxrwxrwx. 1 root root 30 8月  13 20:50 helloworld.sh
```