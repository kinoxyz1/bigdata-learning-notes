




---
# 一、异常信息
```java
client connection to 192.168.1.222:7051: BlockingRecv error: recv error: Connection reset by peer (e
```

# 二、解决方式
1. 打开cdh kudu, 配置 gflagfile 的 Master 高级配置代码段（安全阀）
```bash
--trusted_subnets=0.0.0.0/0
```
2. 每台服务器添加依赖
```bash
yum install gcc python-devel
yum install cyrus-sasl*
```