


# TCP/IP 网络模型分层


# 一、应用层
## 1.1 HTTP
### 1.1.1 HTTP 是什么
HTTP 是超文本传输协议，也就是HyperText Transfer Protocol。

- `超文本`: 网页文本、音频、视频、图片、压缩包 的混合体,并且有超链接, 可以从一个超文本跳转到另一个超文本中.
- `传输`: 把 `超文本` 从 A 传输 到 B, 也可以从 B 传输到 A。这是双向的。
- `协议`: `协` 表示至少有2个以上的参与者, `议` 表示对行为的约定和规范。

总结就是: HTTP 是计算机世界中, 在 `两点` 之间 `传输` 文字、图片、音频等 `超文本` 数据的约定和规范.

### 1.1.2 GET 和 POST 区别

### 1.1.3 HTTP 缓存技术

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












