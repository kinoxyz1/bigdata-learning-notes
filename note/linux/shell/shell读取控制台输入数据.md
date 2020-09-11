
* [一、语法说明](#%E4%B8%80%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
* [二、示例](#%E4%BA%8C%E7%A4%BA%E4%BE%8B)

---
# 一、语法说明
```bash
read(选项)(参数)
```
- 选项:
  1. -p: 指定读取值时的提示符;
  2. -t: 指定读取值时等待的时间(秒);
- 参数:
  1. 变量: 指定读取值得变量;
  
# 二、示例
提示 7 秒内, 读取控制台输入的名称
```bash
[root@hadoop1 shell]# vim read.sh
#!/bin/bash
read -t 7 -p "Enter your name in 7 seconds" NAME
echo $NAME
```