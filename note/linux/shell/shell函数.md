
* [一、系统函数](#%E4%B8%80%E7%B3%BB%E7%BB%9F%E5%87%BD%E6%95%B0)
  * [1\.1 basename 语法说明](#11-basename-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [1\.2 示例](#12-%E7%A4%BA%E4%BE%8B)
  * [1\.3 dirname 语法说明](#13-dirname-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [1\.4 示例](#14-%E7%A4%BA%E4%BE%8B)
* [二、自定义函数](#%E4%BA%8C%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0)
  * [2\.1 语法说明](#21-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [2\.2 小技巧](#22-%E5%B0%8F%E6%8A%80%E5%B7%A7)
  * [2\.3 示例](#23-%E7%A4%BA%E4%BE%8B)

---
# 一、系统函数
## 1.1 basename 语法说明
```bash
basename [string/pathname] [suffix]
```
说明: basename 命令会删掉所有的前缀包括最后一个(`/`)字符, 然后将字符串显示出来

选项: suffix 为后缀, 如果 suffix 被指定了, basename 会将 pathname 或 string 中的 suffix 去掉.

## 1.2 示例
截取该 /root/shell/for1.sh 路径下的文件名
```bash
[root@hadoop1 shell]# basename /root/shell/case.sh case.sh
case.sh

[root@hadoop1 shell]# basename /root/shell/case.sh sh
case.
```

## 1.3 dirname 语法说明
```bash
dirname 文件绝对路径
```
说明: 从给定的包含绝对路径的文件名中去除文件名(非目录部分),然后返回剩下的路径(目录的部分)

## 1.4 示例
获取 for1.sh 文件的路径
```bash
[root@hadoop1 shell]# dirname /root/shell/for1.sh 
/root/shell
```


# 二、自定义函数
## 2.1 语法说明
```bash
[ function ] functionname[()]
{
  action;
  [return int;]
}
functionname
```
## 2.2 小技巧
1. 必须在调用函数地方之前, 先声明函数,shell 脚本是逐行运行的. 不会像其他语言一样先编译;
2. 函数返回值, 只能通过 `$?` 系统变量获取, 可以显示加: `return` 返回, 如果不加, 将以最后一条命令运行结果, 作为返回值. return 后跟随数组 n(0-255)

## 2.3 示例
计算两个输入参数的和
```bash
[root@hadoop1 shell]# vim myfun.sh
#!/bin/bash
function sum()
{
  sum=0
  sum=$[ $1 + $2 ]
  echo $sum
}
read -p "请输入参数1(数值型):" n1;
read -p "请输入参数2(数值型):" n2;
sum $n1 $n2;

[root@hadoop1 shell]# sh myfun.sh 
请输入参数1(数值型):2
请输入参数2(数值型):3
5
```