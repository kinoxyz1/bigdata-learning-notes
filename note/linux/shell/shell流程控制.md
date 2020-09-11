
* [一、条件判断](#%E4%B8%80%E6%9D%A1%E4%BB%B6%E5%88%A4%E6%96%AD)
  * [1\.1 语法说明](#11-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [1\.2 常用判断条件](#12-%E5%B8%B8%E7%94%A8%E5%88%A4%E6%96%AD%E6%9D%A1%E4%BB%B6)
  * [1\.3 示例](#13-%E7%A4%BA%E4%BE%8B)
* [二、流程控制](#%E4%BA%8C%E6%B5%81%E7%A8%8B%E6%8E%A7%E5%88%B6)
  * [2\.1 if 判断](#21-if-%E5%88%A4%E6%96%AD)
    * [2\.1\.1 语法说明](#211-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
    * [2\.1\.2 示例](#212-%E7%A4%BA%E4%BE%8B)
  * [2\.2 case 语句](#22-case-%E8%AF%AD%E5%8F%A5)
    * [2\.2\.1 语法说明](#221-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
    * [2\.2\.2 示例](#222-%E7%A4%BA%E4%BE%8B)
  * [2\.3 for 循环](#23-for-%E5%BE%AA%E7%8E%AF)
    * [2\.3\.1 语法说明1](#231-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E1)
    * [2\.3\.2 示例](#232-%E7%A4%BA%E4%BE%8B)
    * [2\.3\.3 语法说明2](#233-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E2)
    * [2\.3\.4 示例](#234-%E7%A4%BA%E4%BE%8B)
  * [2\.4 while 循环](#24-while-%E5%BE%AA%E7%8E%AF)
  * [2\.4\.1 语法说明](#241-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
    * [2\.4\.2 示例](#242-%E7%A4%BA%E4%BE%8B)

---
# 一、条件判断
## 1.1 语法说明
`[ condition ]` 

注意: condition 前后要有空格

## 1.2 常用判断条件
① 两个整数之间的比较

|  比较符   | 说明  |
|  ----  | ----  |
| =  | 单元格 |
| -lt  | 小于(less than) |
| -le  | 小于等于(less equal) | 
| -eq  | 等于(equal) |
| -gt  | 大于(greater than) |
| -ge  | 大于等于(greater equal) | 
| -ne  | 不等于(Not equal) |

② 按文件权限进行判断

| 比较符 | 说明 |
| ---   | ---  |
| -r  | 有读的权限(read)|
| -w  | 有写的权限(write) | 
| -x  | 有执行的权限(execute)  |

③ 按照文件类型进行判断

| 比较符 | 说明 |
| --- | --- |
| -f | 文件存在并且是一个常规的文件(file) | 
| -e | 文件存在(existence) | 
| -d | 文件存在并且是一个目录(director) |

## 1.3 示例
① 判断 23 是否大于等于 22
```bash
[root@hadoop1 shell]# [ 23 -ge 22 ]
[root@hadoop1 shell]# echo $?
0
```

② helloworld.sh 是否具有写权限
```bash
[root@hadoop1 shell]# [ -w helloworld.sh ]
[root@hadoop1 shell]# echo $?
0
```

③ /root/shell/ 目录中的文件是否存在
```bash
[root@hadoop1 shell]# [ -e /root/shell/helloworld.sh ]
[root@hadoop1 shell]# echo $?
0
```

④ 多条件判断(`&&` 表示前一条命令执行成功时, 才执行后一条命令, `||` 表示上一条命令执行失败后, 才执行下一条命令)
```bash
[root@hadoop1 shell]# [ condition ] && echo OK || echo notok
OK
[root@hadoop1 shell]# [ condition ] && [  ] || echo notok
notok
```

# 二、流程控制
## 2.1 if 判断
### 2.1.1 语法说明
```bash
if [ condition ]; then
  程序
fi
或者
if [ 条件判断式 ] then
  程序
fi 
```
注意事项:
 1. [ condition ] 中, 括号和 condition 之间必须有空格;
 2. if 后要有空格
 
 ### 2.1.2 示例
 输入一个数字, 如果是 `1`, 则输出 `666`, 如果是 `2`, 则输出 `2333`, 如果是其他, 则输出 `ddd`
 ```bash
[root@hadoop1 shell]# vim if.sh
#!/bin/bash
if [ $1 -eq '1' ]; then
  echo '666'
elif [ $1 -eq '2' ]; then
  echo '2333'
else
  echo 'ddd'
fi

[root@hadoop1 shell]# 
[root@hadoop1 shell]# sh if.sh 1
666
[root@hadoop1 shell]# sh if.sh 2
2333
[root@hadoop1 shell]# sh if.sh 3
ddd
```

## 2.2 case 语句
### 2.2.1 语法说明
```bash
case $变量名 in
"值1")
    程序
;;
"值2")
    程序
;;
....
....
*)
    程序, 如果变量的值都不是以上的值, 则执行此程序
esac
```
注意事项:
  1. case 行尾必须为单词 `in`， 每一个模式匹配必须以有括号 `)` 结束;
  2. 双分号 `;;` 表示命令序列结束, 相当于 java 中的 `break`;
  3. 最后的 `*)` 表示默认模式, 相当于 Java 中的 `default`;
  
### 2.2.2 示例
输入一个数字, 如果是 `1`, 则输出 `北京`, 如果是 `2`, 则输出 `上海`, 如果是其他, 输出 `深圳`
```bash
[root@hadoop1 shell]# vim case.sh
#!/bin/bash
case $1 in
'1')
  echo '北京'
;;
'2')
  echo '上海'
;;
*)
  echo '深圳'
esac

[root@hadoop1 shell]# sh case.sh 1
北京
[root@hadoop1 shell]# sh case.sh 2
上海
[root@hadoop1 shell]# sh case.sh 3
深圳
```

## 2.3 for 循环
### 2.3.1 语法说明1
```bash
for ((初始值;循环控制条件;变量变化 ))
do
  程序
done
```

### 2.3.2 示例
① 输出 1 到 100 的累加值
```bash
#!/bin/bash
sum=0
for ((i=0; i<=100; i++))
do
  sum=$[$sum+$i]
done
echo $sum

[root@hadoop1 shell]# sh for1.sh 
5050
```

### 2.3.3 语法说明2
```bash
for 变量 in 值1 值2 值3...
do 
  程序
done
```

### 2.3.4 示例
① 打印所有输入参数
```bash
[root@hadoop1 shell]# vim for2.sh
#!/bin/bash
for i in $*
do
  echo "for2.sh $i"
done

[root@hadoop1 shell]# sh for2.sh 1 2 3 4
for2.sh 1
for2.sh 2
for2.sh 3
for2.sh 4
```
② 比较 `$*` 和 `$@` 区别
1. `$*` 和 `$@`  都表示传递给函数或脚本的所有参数, 不被双引号 `""`包含时, 都以 $1 $2 $3...$n 的形式输出所有参数.
    ```bash
    [root@hadoop1 shell]# vim for3.sh
    
    #!/bin/bash
    for i in $*
    do
      echo "for3.sh $i"
    done
    echo "--------------------------"
    for j in $@
    do
      echo "for3.sh $j"
    done
    
    [root@hadoop1 shell]# sh for3.sh 1 2 3 4
    for3.sh 1
    for3.sh 2
    for3.sh 3
    for3.sh 4
    --------------------------
    for3.sh 1
    for3.sh 2
    for3.sh 3
    for3.sh 4
    ```
2. 当他们被双引号 `""` 包含时, `$*`会将所有的参数作为一个整体, 以 `$1 $2 $3...$n`的形式输出所有参数, `$@` 会将各个参数分开, 以 `$1`、`$2`、 `$3`... `$n`的形式输出所有参数
```bash
[root@hadoop1 shell]# vim for4.sh
#!/bin/bash
for i in "$*"
# $* 中的所有参数看成是一个整体, 所以这个for循环只会执行一次
do
  echo "for4.sh $i"
done
echo "-------------------------"
for j in "$@"
# $@ 中的每个参数都看成是独立的, 所以 $@ 中有几个参数, 就会循环几次
do
  echo "for4,sh $j"
done

[root@hadoop1 shell]# sh for4.sh 1 2 3 4
for4.sh 1 2 3 4
-------------------------
for4,sh 1
for4,sh 2
for4,sh 3
for4,sh 4
```

## 2.4 while 循环
## 2.4.1 语法说明
```bash
while [ condition ]
do 
  程序
done
```
### 2.4.2 示例
输出从 1 到 100的和
```bash
[root@hadoop1 shell]# vim while.sh
#!/bin/bash
sum=0
i=1
while [ $i -le 100 ]
do
  sum=$[$sum+$i]
  i=$[$i+1]
done
echo $sum

[root@hadoop1 shell]# sh while.sh 
5050
```