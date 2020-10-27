


# 一、下载JDK
[下载地址](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

# 二、解压
```bash
[root@hadoop1 ~]# tar -zxvf /opt/software/jdk-8u202-linux-x64.tar.gz -C /usr/java/
```

# 三、配置环境变量
```bash
[root@hadoop1 ~]# source /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_202
export PATH=$PATH:$JAVA_HOME/bin
```

# 四、查看安装版本
```bash
[root@hadoop1 ~]# java -version
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```