







---

# 发起一个 HTTP GET 请求

```bash
% curl https://www.baidu.com
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#00v id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btntofocus></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>o123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_eba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登/a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用/a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

在命令行运行以上命令将返回所访问 baidu.com 页面源码。



# 重定向跟中(-L)

当请求一个 URL 返回 301 之类的重定向响应时，可以使用 `-L`参数来自动重定向跟踪响应头里的 `Location`时

```bash
curl kino.com
```

在该网址设置了301重定向到 https 版 `https://www.kino.com`。上面的例子不会自动完成重定向自动追踪。但是可以用以下命令:

```bash
curl -L kino.com
```

# 储存响应体到文件(-O)

使用 `-O`参数指定文件名，可以将响应结果存储到文件中

```bash
% curl -o baidu.com https://www.baidu.com
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2443  100  2443    0     0  12940      0 --:--:-- --:--:-- --:--:-- 13648

% cat baidu.com
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#00v id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btntofocus></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>o123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_eba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登/a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用/a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

还可以使用 `-O`参数直接用服务器上的文件名保存到本地

```bash
% curl -O https://www.baidu.com/index.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2443  100  2443    0     0  17423      0 --:--:-- --:--:-- --:--:-- 17450

% cat index.html
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn" autofocus></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

# 获取 HTTP 响应头(-i)

使用 `-i`参数可以查看请求 URL 的响应头

```bash
% curl -i https://www.baidu.com
HTTP/1.1 200 OK
Accept-Ranges: bytes
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Connection: keep-alive
Content-Length: 2443
Content-Type: text/html
Date: Sun, 09 Oct 2022 03:51:02 GMT
Etag: "58860401-98b"
Last-Modified: Mon, 23 Jan 2017 13:24:17 GMT
Pragma: no-cache
Server: bfe/1.0.8.18
Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/

<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn" autofocus></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

添加了 `-i`参数，URL的响应头将与响应体一起返回打印出来。

如果只想获取响应头，可以使用`-I`参数。

```bash
% curl -I https://www.baidu.com
HTTP/1.1 200 OK
Accept-Ranges: bytes
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Connection: keep-alive
Content-Length: 277
Content-Type: text/html
Date: Sun, 09 Oct 2022 03:52:08 GMT
Etag: "575e1f72-115"
Last-Modified: Mon, 13 Jun 2016 02:50:26 GMT
Pragma: no-cache
Server: bfe/1.0.8.18
```

# 构造 GET 查询参数(-G)

在发起 GET 请求时，可能我们需要在 URL 后面跟上查询参数，如 `https://www.google.com/search?q=apple`

可以通过 `-G` 参数来构造 URL 的查询字符串

```bash
% curl -G -d 'q=apple' https://www.google.com/search
```

上面的示例会将请求参数与请求 URL 拼接然后发出请求，请求地址为`https://www.google.com/search?q=apple`。注意：如果忘记了`-G`参数，curl 会发出 POST 请求。

如果数据需要 URL 编码，可以结合使用`--data--urlencode`参数。

```bash
% curl -G --data-urlencode 'q=中国' https://www.google.com/search
```

# 改变 User Agent  (-A)

User Agent 即用户代理，简称 UA，它使得服务器能够识别客户使用的操作系统及版本、CPU 类型、浏览器及版本、浏览器渲染引擎、浏览器语言等。默认情况下，curl 发送的 User Agent 为 `curl/<version>`，例如：curl/7.64.1。

可以使用`-A`指定 User Agent 为其他值。

```bash
% curl -A 'my-user-agent' https://www.google.com/search
```

如果你尝试了请求前面的【构造 GET 查询参数】的示例，你会发现 Google 拒绝了我们的请求。现在加上一个浏览器的 User agent 请求一次就能得到正常返回结果。

```bash
% curl -G -d 'q=apple' -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36" https://www.google.com/search
```

# 添加 Referrer  (-e)

使用 `-e`参数用来设置 HTTP 请求头的 `Referer`，表示请求的来源。

```bash
curl -e 'https://www.google.com/' https://www.google.com/search?q=apple
```

# 携带 Cookie 请求数据  (-b)

使用 `-b`参数来向服务器发送 Cookie，可以直接接受 Cookie 字符串或者存储了 Cookie 的文件。

```bash
curl -b 'foo=bar' https://www.google.com/search?q=apple
```

或者

```bash
curl -b ./cookies.txt https://www.google.com/search?q=apple
```

该命令会生成一个`Cookie: foo=bar`的 Cookie 请求头发送给目标 URL。

使用`-c`参数可以将目标 URL 携带的 Cookie 写入到一个文件里。

```bash
curl -c cookies.txt https://www.google.com/search?q=apple
```

上面的命令可以将目标 URL `https://www.google.com` 的 Cookie 保存到 cookies.txt 文件中

# 添加 HTTP 请求头  (-H)

curl 可以通过`-H key:value` 的方式添加 HTTP 请求头，要设置多个请求头，可以通过添加多个`-H`参数实现。

```bash
curl -H 'Accept-Language: en-US' https://www.google.com/search?q=apple
```

前面介绍的 User agent 以及 Cookie 也是一个请求头，也可以通过-H 的方式设置在请求头中。

```bash
curl -H 'User-Agent: my-user-agent' https://www.google.com/search?q=apple
curl -H 'Cookie: foo=bar' https://www.google.com/search?q=apple

```

# 发送一个 HTTP POST 请求

默认情况下，curl 发送的是 GET 请求。要使其发送 POST 请求，需要使用`-X POST`命令行参数。

```bash
curl -X POST https://collector.github.com/github/collect
```

# 更改 HTTP 请求方法  (-X)

`-X` 参数可以用来更改 HTTP 请求方法，`-X POST` 将发起 POST 请求，`-X PUT` 将发起 PUT 请求。

# 添加 POST 数据到请求中  (-d)

要将 POST 数据添加到请求中，需要使用`-d`参数。

使用`-d`参数后，HTTP 请求会自动加上标头`Content-Type : application/x-www-form-urlencoded`。并且会自动将请求转为 POST 方法，因此可以省略`-X POST`。

```bash
curl -d'login=kino&password=000000' https://collector.github.com/github/collect
```

# 发送 JSON 数据

现在 JSON 是非常流行的数据格式，当发起请求时，你可能希望发送 JSON 格式的数据。在这种情况下，需要使用`-H`参数来设置`Content-Type`请求头。

```bash
curl -d '{"option": "value", "something": "anothervalue"}' -H "Content-Type: application/json" -X POST https://collector.github.com/github/collect
```

还可以使用`@`直接读取本地 JSON 文件的内容，来发起请求

```bash
curl -d "@my-file.json" -X POST https://collector.github.com/github/collect
```

# HTTP 认证  (-u)

如果目标 URL 需要 HTTP Basic Authentication，可以通过`-u`参数传递`user:password`来鉴权

```bash
curl -u user:pass https://collector.github.com/github/collect
```

# 打印请求的详细日志  (-v)

使用 `-v` 参数可以打印出 curl 请求的所有请求与响应详细日志。它是`--verbose` 的简写。

```bash
curl -v -I https://www.google.com/search?q=apple
```

输出

```bash
* Uses proxy env variable https_proxy == 'http://127.0.0.1:7890'
*   Trying 127.0.0.1:7890...
* Connected to 127.0.0.1 (127.0.0.1) port 7890 (#0)
* allocate connect buffer!
* Establish HTTP proxy tunnel to www.google.com:443
> CONNECT www.google.com:443 HTTP/1.1
> Host: www.google.com:443
> User-Agent: curl/7.79.1
> Proxy-Connection: Keep-Alive
> 
< HTTP/1.1 200 Connection established
HTTP/1.1 200 Connection established
< 

* Proxy replied 200 to CONNECT request
* CONNECT phase completed!
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (OUT), TLS handshake, Client hello (1):
* (304) (IN), TLS handshake, Server hello (2):
* (304) (IN), TLS handshake, Unknown (8):
* (304) (IN), TLS handshake, Certificate (11):
* (304) (IN), TLS handshake, CERT verify (15):
* (304) (IN), TLS handshake, Finished (20):
* (304) (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / AEAD-CHACHA20-POLY1305-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=www.google.com
*  start date: Sep 12 08:19:33 2022 GMT
*  expire date: Dec  5 08:19:32 2022 GMT
*  subjectAltName: host "www.google.com" matched cert's "www.google.com"
*  issuer: C=US; O=Google Trust Services LLC; CN=GTS CA 1C3
*  SSL certificate verify ok.
* Using HTTP2, server supports multiplexing
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x11e811e00)
> HEAD /search?q=apple HTTP/2
> Host: www.google.com
> user-agent: curl/7.79.1
> accept: */*
> 
< HTTP/2 200 
HTTP/2 200 
< content-type: text/html; charset=ISO-8859-1
content-type: text/html; charset=ISO-8859-1
< content-security-policy: object-src 'none';base-uri 'self';script-src 'nonce-wUkC55A701sqS5Qln6pt_g' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/xsrp
content-security-policy: object-src 'none';base-uri 'self';script-src 'nonce-wUkC55A701sqS5Qln6pt_g' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/xsrp
< p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
< date: Sun, 09 Oct 2022 04:06:51 GMT
date: Sun, 09 Oct 2022 04:06:51 GMT
< server: gws
server: gws
< x-xss-protection: 0
x-xss-protection: 0
< x-frame-options: SAMEORIGIN
x-frame-options: SAMEORIGIN
< expires: Sun, 09 Oct 2022 04:06:51 GMT
expires: Sun, 09 Oct 2022 04:06:51 GMT
< cache-control: private
cache-control: private
< set-cookie: 1P_JAR=2022-10-09-04; expires=Tue, 08-Nov-2022 04:06:51 GMT; path=/; domain=.google.com; Secure
set-cookie: 1P_JAR=2022-10-09-04; expires=Tue, 08-Nov-2022 04:06:51 GMT; path=/; domain=.google.com; Secure
< set-cookie: AEC=AakniGPqghCzHz15NJDUX_xon5xRGBXvszny_ykPtKQJN2WXiH4oWe-tAc0; expires=Fri, 07-Apr-2023 04:06:51 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
set-cookie: AEC=AakniGPqghCzHz15NJDUX_xon5xRGBXvszny_ykPtKQJN2WXiH4oWe-tAc0; expires=Fri, 07-Apr-2023 04:06:51 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
< set-cookie: NID=511=SZ36T0r82h6oqmrS8GHleLymMiU-wAMSjME6bv4QEz5KwZva20eIEcVt4UsLox7JtNL_ozV4pWOl4ZMyx8DuOGFbWE55zLdcaTiJtKPEWGfriEk-hCfvKkzUETjuUEClG9vrl3XuAdDbuSFh-JIZgLLmCr5WY7mULs3T2oP2s8k; expires=Mon, 10-Apr-2023 04:06:51 GMT; path=/; domain=.google.com; HttpOnly
set-cookie: NID=511=SZ36T0r82h6oqmrS8GHleLymMiU-wAMSjME6bv4QEz5KwZva20eIEcVt4UsLox7JtNL_ozV4pWOl4ZMyx8DuOGFbWE55zLdcaTiJtKPEWGfriEk-hCfvKkzUETjuUEClG9vrl3XuAdDbuSFh-JIZgLLmCr5WY7mULs3T2oP2s8k; expires=Mon, 10-Apr-2023 04:06:51 GMT; path=/; domain=.google.com; HttpOnly
< alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000,h3-Q050=":443"; ma=2592000,h3-Q046=":443"; ma=2592000,h3-Q043=":443"; ma=2592000,quic=":443"; ma=2592000; v="46,43"
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000,h3-Q050=":443"; ma=2592000,h3-Q046=":443"; ma=2592000,h3-Q043=":443"; ma=2592000,quic=":443"; ma=2592000; v="46,43"

< 
* Connection #0 to host 127.0.0.1 left intact
```

# 限制 HTTP 带宽  (--limit-rate)

默认情况下，curl 使用最大可用带宽，但是通常我们需要放慢速度进行测试。可以使用`--limit-rate` 来限制 curl 的请求和响应的带宽，让请求与响应变慢。

```bash
curl --limit-rate 200k https://github.com
```

上面的命令将 curl 限制在每秒 200K 字节。

**参考链接**

- Everything curl https://everything.curl.de
- Curl Cookbook https://catonmat.net/cookbooks/curl













