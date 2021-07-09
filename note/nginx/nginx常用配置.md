




---

# 一、Nginx 转发带有下划线的Header 异常



部署测试环境进行前后端联调时, 发现后端接口一直报空指针异常, 但是这个参数确实是从浏览器传了的, 大概是nginx做转发的时候出了问题, 导致 后端接口接收参数没有接收到


打开浏览器f12 看详细参数, 发现参数是另一个同事写的名字带有斜划线
![nginx参数下划线](../../img/nginx/nginx参数下划线/1.png)

因为 nginx 默认是过滤带有下划线的变量, 如果需要添加带有下划线的参数, 需要  在 nginx.conf 文件中加入: `underscores_in_headers on` 参数.

设置完参数一定要重启 nginx: `/usr/local/nginx/sbin/nginx -s reload`



# 二、图片超过1M报错

修改如下配置:

```bash
client_max_body_size 20M
```

可以选择在http{ }中设置：client_max_body_size 20m;

也可以选择在server{ }中设置：client_max_body_size 20m;

还可以选择在location{ }中设置：client_max_body_size 20m;

三者有区别

- 设置到http{}内，控制全局nginx所有请求报文大小
- 设置到server{}内，控制该server的所有请求报文大小
- 设置到location{}内，控制满足该路由规则的请求报文大小