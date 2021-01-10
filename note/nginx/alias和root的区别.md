




---

# 示例1
```bash
server {
        listen       9091;
        server_name  192.168.220.111;

        location /nginx/ {
            root /opt/nginx/;
        }
```
- 在 alias 目录配置下, 访问 192.168.220.111:9091/nginx/a.html 实际上访问的是 /opt/nginx/a.html 
- 在 root 目录配置下, 访问 192.168.220.111:9091/nginx/a.html 实际上访问的是 /opt/nginx/nginx/a.html

# 示例2
当 location 匹配访问的path 和 alias 设置的目录名不一致时
```bash
    server {
        listen       9091;
        server_name  192.168.220.111;

        location /nginx1/ {
            alias /opt/nginx/;
        }
```
在浏览器中输入: 192.168.220.111:9091/nginx1/a.html 此时可以正常访问

当 location 匹配访问的path 和 root 设置的目录名不一致时
```bash
    server {
        listen       9091;
        server_name  192.168.220.111;

        location /nginx1/ {
            root /opt;
        }
```
在浏览器中输入: 192.168.220.111:9091/nginx1/a.html 此时不能正常访问, 查看 nginx 日志发现实际访问的路径是 `/opt/nginx1/a.html`
```bash
2021/01/09 20:48:19 [error] 22304#0: *148 open() "/opt/nginx1/a.html" failed (2: No such file or directory), client: 192.168.220.1, server: 192.168.220.111, request: "GET /nginx1/a.html HTTP/1.1", host: "192.168.220.111:9091"
```

区别: 
- 在 alias 目录配置下, 访问的是 **alias + location<font color='red'>上一级</font>** 组合的地址
- 在 root 目录配置下, 访问的是 **alias + location** 组合的地址
- 当 location 匹配目录 和 alias 目录不一致时, 访问的本地目录还是 alias 配置的目录 
- 当 location 匹配目录 和 root 目录不一致时, 需要将 **本地目录名 和 location 匹配访问的path名保持一致**





