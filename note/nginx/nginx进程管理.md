





 # 一、信号
![nginx 信号](../../img/nginx/nginx信号/NGINX信号.png)


| 信号       | 作用        |
|----------|-----------|
| TERM/INT | 立即关闭整个服务  |
| QUIT     | "优雅"地关闭整个服务 | 
| HUP      | 重读配置文件并使用服务对新配置项生效 |
| USR1 | 重新打开日志文件, 可以用来进行日志切割 | 
| USR2 | 平滑升级到最新版的nginx | 
| WINCH | 所有子进程不在接收处理新链接, 相当于给 worker 进程发送 QUIR 指令 | 


# 二、reload重载配置流程


# 三、热升级流程


# 四、优雅关闭worker




