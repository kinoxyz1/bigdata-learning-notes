* [一、scp（secure copy）安全拷贝](#%E4%B8%80scpsecure-copy%E5%AE%89%E5%85%A8%E6%8B%B7%E8%B4%9D)
  * [1\.1 scp定义](#11-scp%E5%AE%9A%E4%B9%89)
  * [1\.2 基本语法](#12-%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95)
  * [1\.3 案例实操](#13-%E6%A1%88%E4%BE%8B%E5%AE%9E%E6%93%8D)
* [二、 rsync 远程同步工具](#%E4%BA%8C-rsync-%E8%BF%9C%E7%A8%8B%E5%90%8C%E6%AD%A5%E5%B7%A5%E5%85%B7)
  * [2\.1 基本语法](#21-%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95)
  * [2\.2 案例实操](#22-%E6%A1%88%E4%BE%8B%E5%AE%9E%E6%93%8D)
* [三、 xsync集群分发脚本](#%E4%B8%89-xsync%E9%9B%86%E7%BE%A4%E5%88%86%E5%8F%91%E8%84%9A%E6%9C%AC)
  * [前提: 需要有多个虚拟机, 名字保持相似性，例如: linux01、linux02、linux03\.\.\.\.\.](#%E5%89%8D%E6%8F%90-%E9%9C%80%E8%A6%81%E6%9C%89%E5%A4%9A%E4%B8%AA%E8%99%9A%E6%8B%9F%E6%9C%BA-%E5%90%8D%E5%AD%97%E4%BF%9D%E6%8C%81%E7%9B%B8%E4%BC%BC%E6%80%A7%E4%BE%8B%E5%A6%82-linux01linux02linux03)
  * [3\.1 需求：循环复制文件到所有节点的相同目录下](#31-%E9%9C%80%E6%B1%82%E5%BE%AA%E7%8E%AF%E5%A4%8D%E5%88%B6%E6%96%87%E4%BB%B6%E5%88%B0%E6%89%80%E6%9C%89%E8%8A%82%E7%82%B9%E7%9A%84%E7%9B%B8%E5%90%8C%E7%9B%AE%E5%BD%95%E4%B8%8B)
  * [3\.2 需求分析](#32-%E9%9C%80%E6%B1%82%E5%88%86%E6%9E%90)
* [常见问题](#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)

---

# 一、scp（secure copy）安全拷贝
## 1.1 scp定义
scp可以实现服务器与服务器之间的数据拷贝。（from server1 to server2）


## 1.2 基本语法

```
scp    -r          $pdir/$fname               $user@linux01$host:$pdir/$fname
命令   递归       要拷贝的文件路径/名称            目的用户@主机:目的路径/名称
```
## 1.3 案例实操
- 在linux01上，将linux01中/opt/module目录下的软件拷贝到linux02
`[kino@linux01 /]$ scp -r /opt/module  root@linux02:/opt/module`

- 在linux03上，将linux01服务器上的/opt/module目录下的软件拷贝到linux03上
`[kino@linux03 opt]$sudo scp -r kino@linux01:/opt/module root@linux03:/opt/module`
- 在linux03上操作将linux01中/opt/module目录下的软件拷贝到linux04上。
`[kino@linux03 opt]$ scp -r kino@linux01:/opt/module root@linux04:/opt/module`

---
# 二、 rsync 远程同步工具
rsync主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

<font color='red'>rsync和scp区别：用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。</font>


## 2.1 基本语法

```
rsync    -av       $pdir/$fname              $user@linux01$host:$pdir/$fname
命令   选项参数   要拷贝的文件路径/名称        目的用户@主机:目的路径/名称

```
选项参数说明
|选项	|功能|
| -- | -- |
-a	|归档拷贝
-v	|显示复制过程


## 2.2 案例实操
把 `linux01` 机器上的`/opt/software`目录同步到`linux02`服务器的`root用户`下的`/opt/`目录

`[kino@linux01opt]$ rsync -av /opt/software/ linux02:/opt/software`

---

# 三、 xsync集群分发脚本
注意权限, 如果当前登录用户没有被操作文件夹权限, 将什么都不执行

## <font color='red'>前提: 需要有多个虚拟机, 名字保持相似性，例如: linux01、linux02、linux03.....

## 3.1 需求：循环复制文件到所有节点的相同目录下
## 3.2 需求分析
- rsync命令原始拷贝：
	`rsync  -av     /opt/module  		 root@linux01:/opt/`
- 期望脚本：
	xsync要同步的文件名称
- 说明：在`/home/kino/bin`这个目录下存放的脚本，kino用户可以在系统任何地方直接执行。
	因为  `/home/kino/bin`  被配置在了环境变量中, 可以用 `echo $PATH` 查看当前环境变量
- 脚本实现
	- 在`/home/kino` 目录下创建bin目录，并在bin目录下xsync创建文件，文件
		

		```
		[kino@linux01 ~]$ mkdir bin
		[kino@linux01 ~]$ cd bin/
		[kino@linux01 bin]$ touch xsync
		[kino@linux01 bin]$ vi xsync
		```
		在该文件中编写如下代码
		```
		#!/bin/bash
		#1 获取输入参数个数，如果没有参数，直接退出
		pcount=$#
		if ((pcount==0)); then
		echo no args;
		exit;
		fi
		
		#2 获取文件名称
		p1=$1
		fname=`basename $p1`
		echo fname=$fname
		
		#3 获取上级目录到绝对路径
		pdir=`cd -P $(dirname $p1); pwd`
		echo pdir=$pdir
		
		#4 获取当前用户名称
		user=`whoami`
		
		###############################根据自己的主机名更改$user@xxxx$host###############################
		#5 循环
		for((host=1; host<3; host++)); do
		        echo ------------------- linux0$host --------------
		        rsync -av $pdir/$fname $user@linux0$host:$pdir
		done
		```
	- 修改脚本 xsync 具有执行权限
	`[kino@linux01 bin]$ chmod 777 xsync`
	- 调用脚本形式：xsync 文件名称
	`[kino@linux01  bin]$ xsync /home/kino/bin`


# 常见问题
如果有如下报错
```
mkdir "/opt/software" failed: Permission denied (13)
```
是因为权限不够, 要么换成 `root` 用户执行命令, 要么用`sudo 命令`