

---


# 一、 查看安装的 mysql
```
[root@hadoop102 桌面]# rpm -qa|grep -i mysql
mysql-libs-5.1.73-7.el6.x86_64
```

# 二、卸载 
```
[root@hadoop102 桌面]# rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64
```

# 三、删除残留文件
1. 回到 家目录
	```
	cd ~
	```
2. 执行以下命令
	```
	find / -name mysql | xargs rm -rf
	find / -name my.cnf | xargs rm -rf
	```
3. 去到 /var/lib/下 找到 mysql/ 目录， 删除掉
	```
	cd /var/lib/
	
	rm -rf mysql/
	```

# 注: 如果不删除下次再安装 mysql 可能有问题