
* [一、cut](#%E4%B8%80cut)
  * [1\.1 基本用法](#11-%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95)
  * [1\.2 示例](#12-%E7%A4%BA%E4%BE%8B)
* [二、sed](#%E4%BA%8Csed)
  * [2\.1 基本用法](#21-%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95)

---
# 一、cut 
cut 的工作就是 '剪', 具体的说就是在文件中负责裁剪数据用的;

cut 命令从文件的每一行剪切字节、字符、字段 并将这些字节、字段、字符 输出.

## 1.1 基本用法
说明: 默认分隔符是制表符
```bash
cut[选项] filename
```
选项:
 1. -f: 列号, 提取第几列;
 2. -d: 分隔符, 按照指定分隔符分隔列;
 
 ## 1.2 示例
① 准备数据
```bash
[root@hadoop1 shell]# vim cut.txt
zhang san
li si
wang wu
```
② 切割 cut.txt 第1/2/3列
```bash
[root@hadoop1 shell]# cut -f 1 -d " " cut.txt 
zhang
li
wang
[root@hadoop1 shell]# cut -f 2 -d " " cut.txt 
san
si
wu
[root@hadoop1 shell]# cut -f 3 -d " " cut.txt 




```
③ 在 cut.txt 文件中切割出 li
```bash
[root@hadoop1 shell]# cut -f 1 -d " " cut.txt  | grep 'li'
li
或者 
[root@hadoop1 shell]# cat cut.txt | grep "li" | cut -d " " -f 1
li
```

④ 选取系统 PATH 变量值, 取第二个 `:` 后所有的路径
```bash
[root@hadoop1 shell]# echo $PATH |  cut -d ":" -f 2-
/usr/local/bin:/usr/sbin:/usr/bin:/usr/java/jdk1.8.0_144/bin:/usr/bigdata/hadoop/hadoop-2.7.2/bin:/usr/bigdata/hadoop/hadoop-2.7.2/sbin:/usr/bigdata/zookeeper/zookeeper-3.4.10/bin:/usr/bigdata/kafka/kafka_2.11-0.11.0.2/bin:/usr/bigdata/hbase/hbase-1.3.1/bin:/usr/bigdata/scala/scala-2.11.12/bin:/usr/bigdata/spark/spark-2.1.1-bin-hadoop2.7/bin:/usr/bigdata/hive/hive/bin:/root/bin
```

⑤ 切割 ifconfig 后打印的IP地址
```bash
[root@hadoop1 shell]# ifconfig ens33 | grep netmask | cut -f 10 -d " "
192.168.220.15
```

# 二、sed
sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。

## 2.1 基本用法
```bash
sed[选项参数] 'command' filename
```



