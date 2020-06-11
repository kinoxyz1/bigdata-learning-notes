```bash
[root@bigdata-1 ~]$ vim appRestart.sh

#! /bin/bash
status="RUNNING"
programStatus=`yarn application -list |grep YARN-NAME| awk '{print $6}'`
if [ "$status" == "$programStatus" ]
then
        #"程序正在运行..."
else
        echo `date "+%Y-%m-%d %H:%M:%S"` "程序已经结束..." >> error.log
        echo `date "+%Y-%m-%d %H:%M:%S"` "程序重新启动..." >> error.log
        echo -e "\n" >> error.log
        Spark 提交命令
fi

[root@bigdata-1 ~]$ chmod 777 appRestart.sh
```

配置定时任务调用重启脚本

```bash
*/1 * * * * appRestart.sh
```
