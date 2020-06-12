* [一、 查看安装的 mysql](#%E4%B8%80-%E6%9F%A5%E7%9C%8B%E5%AE%89%E8%A3%85%E7%9A%84-mysql)
* [二、卸载](#%E4%BA%8C%E5%8D%B8%E8%BD%BD)
* [三、删除残留文件](#%E4%B8%89%E5%88%A0%E9%99%A4%E6%AE%8B%E7%95%99%E6%96%87%E4%BB%B6)
* [注: 如果不删除下次再安装 mysql 可能有问题](#%E6%B3%A8-%E5%A6%82%E6%9E%9C%E4%B8%8D%E5%88%A0%E9%99%A4%E4%B8%8B%E6%AC%A1%E5%86%8D%E5%AE%89%E8%A3%85-mysql-%E5%8F%AF%E8%83%BD%E6%9C%89%E9%97%AE%E9%A2%98)

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