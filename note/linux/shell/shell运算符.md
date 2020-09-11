
* [一、基本语法](#%E4%B8%80%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95)
* [示例](#%E7%A4%BA%E4%BE%8B)

--- 
# 一、基本语法
1. `$((运算式))`或`$[运算式]`
2. `expr + , - , \* , / , %`  --->  加, 减, 乘 除, 取余

    注意: expr 运算法间要有空格

# 示例
① 计算 3+2 的值
```bash
[root@hadoop1 shell]# expr 3+2
3+2
[root@hadoop1 shell]# expr 3 + 2
5
```

② 计算 3-2 的值
```bash
[root@hadoop1 shell]# expr 3-2
3-2
[root@hadoop1 shell]# expr 3 - 2
1
```

③ 计算 (2+3)*4 的值
```bash
[root@hadoop1 shell]# expr `expr 2 + 3` \* 4
20
```

④ 采用 $[运算式]方式
```bash
[root@hadoop1 shell]# echo $[(2+3)*4]
20
```