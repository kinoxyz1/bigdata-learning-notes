* [一、 SWAP 是什么。](#%E4%B8%80-swap-%E6%98%AF%E4%BB%80%E4%B9%88)
* [二、扩展](#%E4%BA%8C%E6%89%A9%E5%B1%95)
  * [2\.1 扩展之前查看自己的swap 有多大](#21-%E6%89%A9%E5%B1%95%E4%B9%8B%E5%89%8D%E6%9F%A5%E7%9C%8B%E8%87%AA%E5%B7%B1%E7%9A%84swap-%E6%9C%89%E5%A4%9A%E5%A4%A7)
  * [2\.2 增加交换分区“文件”及大小](#22-%E5%A2%9E%E5%8A%A0%E4%BA%A4%E6%8D%A2%E5%88%86%E5%8C%BA%E6%96%87%E4%BB%B6%E5%8F%8A%E5%A4%A7%E5%B0%8F)
  * [2\.3 设置交换文件](#23-%E8%AE%BE%E7%BD%AE%E4%BA%A4%E6%8D%A2%E6%96%87%E4%BB%B6)
  * [2\.4 立即启用交换分区文件](#24-%E7%AB%8B%E5%8D%B3%E5%90%AF%E7%94%A8%E4%BA%A4%E6%8D%A2%E5%88%86%E5%8C%BA%E6%96%87%E4%BB%B6)
  * [2\.5 再次查看分区，成功扩展2G](#25-%E5%86%8D%E6%AC%A1%E6%9F%A5%E7%9C%8B%E5%88%86%E5%8C%BA%E6%88%90%E5%8A%9F%E6%89%A9%E5%B1%952g)
  * [2\.6 永久挂载](#26-%E6%B0%B8%E4%B9%85%E6%8C%82%E8%BD%BD)
* [三、 删除增加的swap 分区](#%E4%B8%89-%E5%88%A0%E9%99%A4%E5%A2%9E%E5%8A%A0%E7%9A%84swap-%E5%88%86%E5%8C%BA)

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