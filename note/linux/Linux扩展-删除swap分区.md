


---

# 一、 SWAP 是什么。
SWAP就是 linux 下的<font color='red'>**虚拟内存分区**</font>，它的作用是<font color='red'>**在物理内存使用完之后,将磁盘空间(也就是SWAP分区)虚拟成内存来使用。**</font>

---
# 二、扩展

## 2.1 扩展之前查看自己的swap 有多大
```bash
[kino@hadoop102 ~]$ sudo free -m
             total       used       free     shared    buffers     cached
Mem:          1990       1242        747          0         19        175
-/+ buffers/cache:       1047        943
Swap:          199          0        199
```

## 2.2 增加交换分区“文件”及大小
```bash
[kino@hadoop102 ~]$ sudo dd if=/dev/zero of=/home/swap bs=1G count=2 
```
其中**of=/home/swap**: 交换分区文件， **bs=1G**: 单位， **count=2**: count 等于想要的块大小(这里是 2G)


## 2.3 设置交换文件
```bash
[kino@hadoop102 ~]$ sudo mkswap /home/swap
```

## 2.4 立即启用交换分区文件 
```bash
[kino@hadoop102 ~]$ sudo swapon /home/swap
```

## 2.5 再次查看分区，成功扩展2G
```bash
[kino@hadoop102 ~]$ sudo free -m
             total       used       free     shared    buffers     cached
Mem:          1990        896       1094          0          4        288
-/+ buffers/cache:        604       1386
Swap:         2247          0       22471
```

## 2.6 永久挂载
```bash
[kino@hadoop102 ~]$ sudo vim /etc/fstab

# 增加
/home/swap              swap                    swap    defaults        0 0
```
---
# 三、 删除增加的swap 分区
```bash
[kino@hadoop102 ~]$ sudo swapoff 增加的交换分区文件
```