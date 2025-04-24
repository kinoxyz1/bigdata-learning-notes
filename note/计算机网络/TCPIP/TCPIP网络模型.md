


# TCP/IP 网络模型分层


# 一、应用层
## 1.1 HTTP
### 1.1.1 HTTP 是什么
HTTP 是超文本传输协议，也就是HyperText Transfer Protocol。

- `超文本`: 网页文本、音频、视频、图片、压缩包 的混合体,并且有超链接, 可以从一个超文本跳转到另一个超文本中.
- `传输`: 把 `超文本` 从 A 传输 到 B, 也可以从 B 传输到 A。这是双向的。
- `协议`: `协` 表示至少有2个以上的参与者, `议` 表示对行为的约定和规范。

总结就是: HTTP 是计算机世界中, 在 `两点` 之间 `传输` 文字、图片、音频等 `超文本` 数据的`约定`和`规范`.

### 1.1.2 GET 和 POST 区别
在 RFC 协议中, GET 是从服务器获取资源的. POST 是根据请求报文, 对指定资源做处理的.

GET 传输的数据在URL中(也可以在 body 中), 是明文的, 且有大小限制；

POST 传输的数据可以在URL中, 也可以在 body 中, body 无大小限制.

从 RFC 规范协议的语义上来讲, GET 请求是 安全且幂等 的. 因为 GET 是`只读`操作, 无论操作多少次, 服务器上的数据都是安全的, 且每次结果相同。 所以可以对 GET请求的数据做缓存, 比如缓存可以做到浏览器本身, 也可以做到代理上(nginx), 而且在浏览器中的 GET 请求可以保存为书签。

而 POST 请求是`新增或者提交`数据, 会修改服务器上的资源, 所以是不安全的, 且多次提交数据就会创建多个资源, 所以也不是幂等的.

但是在实际的开发中, 也有人用 GET 请求保存数据, 有人用 POST 请求读取数据.

### 1.1.3 HTTP 缓存技术

#### 强制缓存
强制缓存生效的话, 不会向服务器发起请求.

强制缓存是只要浏览器判断缓存没有过期, 则直接使用浏览器本地缓存, 决定是否使用缓存的主动性在于浏览器.

![强制缓存](../../../img/计算机网络/TCPIP/11.强制缓存.png)

强制缓存利用 HTTP 响应头部字段实现:
1. `Cache-Control` 相对时间. 优先级高于 `Expires`
2. `Expires` 绝对时间.

强制缓存的流程:
1. 第一次请求服务器资源, 服务器会在返回资源的时候, 在 Response header 加上 Cache-Control, Cache-Control 设置了过期时间大小;
2. 再次请求服务器资源, 会先通过请求资源的时间 和 Cache-Control 中设置的过期时间, 来计算出资源是否过期, 如果没有, 就使用缓存, 否则重新请求服务器;
3. 服务器再次收到请求, 会再次更新 Response header 的 Cache-Control.


nginx 配置强制缓存
```bash
# a.html 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Fetch Login</title>
</head>
<body>
  <h2>nginx 强制缓存测试</h2>
</body>
</html>


# nginx config
server {
    listen 80;
    server_name 192.168.1.168;
    location /a.html {
       add_header Cache-Control "max-age=600";  # 设置缓存时间为 600s, max-age 需要加引号
       add_header Last-Modified "";             # 关闭协商缓存
       etag off;                                # 关闭ETag
       root /Users/kino/;
    }
}
```
在浏览器中访问结果:

![强制缓存测试1](../../../img/计算机网络/TCPIP/12.强制缓存测试1.png)

> 注意: 在浏览器中, 要按回车键才能使用到强制缓存, 按刷新按钮或者f5, 会走协商缓存, 原因是浏览器会在 Request Headers 中添加: `Cache-Control: max-age=0`, 告诉服务端帮我确认该资源是否过期, 也就是说浏览器是真的会发出请求的.


#### 协商缓存

### 1.1.4 HTTP 特性

### 1.1.5 HTTP1、HTTP2、HTTP3

### 1.1.6 HTTP 如何优化

## 1.2 HTTPS

### 1.2.1 HTTP 和 HTTPS

### 1.2.2 HTTPS RSA 握手过程

### 1.2.3 HTTPS ECDHE 握手过程

##### 第一次握手
###### client hello
![第一次握手_clienthello](../../../img/计算机网络/TCPIP/1.第一次握手_clienthello.png)
> 客户端发送 TLS 版本、客户端随机字符、支持的秘钥套件列表.


##### 第二次握手
###### 1.server hello
![第二次握手_serverhello](../../../img/计算机网络/TCPIP/2.第二次握手_serverhello.png)

> `收到`: 客户端的 `TLS` 版本、随机字符、支持的秘钥套件列表。
>
> `发送`: 服务端的 `TLS` 版本、随机字符、选择秘钥套件。

###### 2.Certificate
![第二次握手_Certificate](../../../img/计算机网络/TCPIP/3.第二次握手_Certificate.png)
> `发送`: 服务端的 TLS 版本、数字证书。

###### 3.Server Key Exchange
![第二次握手_Server_Key_Exchange](../../../img/计算机网络/TCPIP/4.第二次握手_Server_Key_Exchange.png)
> 发送的内容: 
> 1. TLS 版本在 `Server Hello` 阶段已经协商确定, 该阶段无需再发送.
> 2. 椭圆曲线参数
> 3. 公钥
> 4. 签名算法和签名值
> 
> 过程:
> 1. 生成临时私钥: 服务器生成一个临时的私钥.
> 2. 确定好椭圆曲线(G): 选择用于秘钥交换的椭圆曲线参数.
> 3. 计算临时公钥: 使用临时私钥和椭圆曲线参数(G)计算出公钥(P).
> 4. 签名: 使用服务器的长期私钥(证书中的)对前面协商出来的临时公钥进行签名, 证明这些参数来自服务器.

###### 4.Server Hello Done
![第二次握手_Server_Hello_Done](../../../img/计算机网络/TCPIP/5.第二次握手_Server_Hello_Done.png)
> 服务端提供完信息, 结束.

##### 第三次握手
###### 1.Client Key Exchange
![第三次握手_Client_Key_Exchange](../../../img/计算机网络/TCPIP/6.第三次握手_Client_Key_Exchange.png)
> 收到: 椭圆曲线(G)、公钥(P)、签名算法、签名值。
> 
> 发送: client 端公钥.
> 
> 过程:
> 1. client 生成随机字符，表示私钥。
> 2. 根据服务端发送的 椭圆曲线(G)、公钥(P)，client 私钥, 计算出对应的公钥。

###### 2.Change Cipher Spec
![第三次握手_Change_Cipher_Spec](../../../img/计算机网络/TCPIP/7.第三次握手_Change_Cipher_Spec.png)
> 发送: 通知 server 在后续使用对称加密的会话秘钥。
> 
> 过程:
> 1. 根据client 私钥、server 公钥、根据ECDHE算法，计算出对称加密的会话秘钥。

###### 3.Encrypted Handshake Message
![第三次握手_Encrypted_Handshake_Message](../../../img/计算机网络/TCPIP/8.第三次握手_Encrypted_Handshake_Message.png)
> 发送: 将之前发送的数据做一个摘要, 用对称秘钥加密。


##### 第四次握手
###### 1.Change Cipher Spec
![第三次握手_Encrypted_Handshake_Message](../../../img/计算机网络/TCPIP/8.第三次握手_Encrypted_Handshake_Message.png)
> 收到: client 公钥.
> 
> 发送: 通知client后续使用对称加密会话秘钥。
> 
> 过程:
> 1. 收到 client公钥之后,  server 有了 椭圆曲线、server 公钥、server 私钥、client 私钥。
> 2. 使用 server 私钥、client 公钥、ECDHE算法计算对称加密会话秘钥。

###### 2.Encrypted Handshake Message
![第四次握手_Encrypted_Handshake_Message](../../../img/计算机网络/TCPIP/10.第四次握手_Encrypted_Handshake_Message.png)


### 1.2.4 HTTPS 如何优化

1. 硬件优化: 使用更好的CPU, 如支持 AES_NI 指令集的处理器.
2. 软件优化: 升级内核、openssl.
3. 协议优化: 
   4. 使用 ECDHE 协议, 不使用 RSA 协议.
   4. 秘钥交换算法优化: 使用 x25519 椭圆曲线.
   5. TLS 升级 v1.3.
4. 证书优化
   5. 证书传输优化.
   6. 证书验证优化.
5. 会话复用
   6. Session ID
   7. Session Ticket
   8. Pre-shared Key

## 1.3 HTTP 和 RPC

## 1.4 HTTP 和 WebSocket


# 二、传输层

## 2.1 TCP


## 2.2 UDP



# 三、网络层

## 3.1 IP 












