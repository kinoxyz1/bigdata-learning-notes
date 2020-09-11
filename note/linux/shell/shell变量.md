
* [一、系统变量](#%E4%B8%80%E7%B3%BB%E7%BB%9F%E5%8F%98%E9%87%8F)
  * [1\.1 常用系统变量](#11-%E5%B8%B8%E7%94%A8%E7%B3%BB%E7%BB%9F%E5%8F%98%E9%87%8F)
* [二、自定义变量](#%E4%BA%8C%E8%87%AA%E5%AE%9A%E4%B9%89%E5%8F%98%E9%87%8F)
  * [2\.1 基本语法](#21-%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95)
  * [2\.2 变量定义规则](#22-%E5%8F%98%E9%87%8F%E5%AE%9A%E4%B9%89%E8%A7%84%E5%88%99)
  * [2\.3 示例](#23-%E7%A4%BA%E4%BE%8B)
* [三、特殊变量: $n](#%E4%B8%89%E7%89%B9%E6%AE%8A%E5%8F%98%E9%87%8F-n)
  * [3\.1 语法说明](#31-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [3\.2 示例](#32-%E7%A4%BA%E4%BE%8B)
* [四、特殊变量: $\#](#%E5%9B%9B%E7%89%B9%E6%AE%8A%E5%8F%98%E9%87%8F-)
  * [4\.1 语法说明](#41-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [4\.2 示例](#42-%E7%A4%BA%E4%BE%8B)
* [五、特殊变量: $\* 和 $@](#%E4%BA%94%E7%89%B9%E6%AE%8A%E5%8F%98%E9%87%8F--%E5%92%8C-)
  * [5\.1 语法说明](#51-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [5\.2 示例](#52-%E7%A4%BA%E4%BE%8B)
* [六、特殊变量: $?](#%E5%85%AD%E7%89%B9%E6%AE%8A%E5%8F%98%E9%87%8F-)
  * [6\.1 语法说明](#61-%E8%AF%AD%E6%B3%95%E8%AF%B4%E6%98%8E)
  * [6\.2 示例](#62-%E7%A4%BA%E4%BE%8B)

--- 

# 一、系统变量
## 1.1 常用系统变量
`$HOME`、`$PAD`、`$SHELL` 等

查看系统变量的值
```bash
[root@hadoop1 shell]# echo $HOME
/root
```

显示当前Shell中所有变量
```bash
[root@hadoop1 shell]# set
BASH=/bin/bash
BASHOPTS=checkwinsize:cmdhist:expand_aliases:extquote:force_fignore:histappend:hostcomplete:interactive_comments:login_shell:progcomp:promptvars:sourcepath
BASH_ALIASES=()
BASH_ARGC=()
BASH_ARGV=()
BASH_CMDS=()
BASH_LINENO=()
BASH_SOURCE=()
BASH_VERSINFO=([0]="4" [1]="2" [2]="46" [3]="2" [4]="release" [5]="x86_64-redhat-linux-gnu")
BASH_VERSION='4.2.46(2)-release'
COLUMNS=178
DIRSTACK=()
EUID=0
GROUPS=()
HADOOP_HOME=/usr/bigdata/hadoop/hadoop-2.7.2
HBASE_HOME=/usr/bigdata/hbase/hbase-1.3.1
HISTCONTROL=ignoredups
HISTFILE=/root/.bash_history
HISTFILESIZE=1000
HISTSIZE=1000
HIVE_HOME=/usr/bigdata/hive/hive
HOME=/root
HOSTNAME=hadoop1
HOSTTYPE=x86_64
ID=0
IFS=$' \t\n'
JAVA_HOME=/usr/java/jdk1.8.0_144
KAFKA_HOME=/usr/bigdata/kafka/kafka_2.11-0.11.0.2
LANG=zh_CN.UTF-8
LESSOPEN='||/usr/bin/lesspipe.sh %s'
LINES=30
LOGNAME=root
LS_COLORS='rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:'
MACHTYPE=x86_64-redhat-linux-gnu
MAIL=/var/spool/mail/root
MAILCHECK=60
OLDPWD=/root
OPTERR=1
OPTIND=1
OSTYPE=linux-gnu
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/java/jdk1.8.0_144/bin:/usr/bigdata/hadoop/hadoop-2.7.2/bin:/usr/bigdata/hadoop/hadoop-2.7.2/sbin:/usr/bigdata/zookeeper/zookeeper-3.4.10/bin:/usr/bigdata/kafka/kafka_2.11-0.11.0.2/bin:/usr/bigdata/hbase/hbase-1.3.1/bin:/usr/bigdata/scala/scala-2.11.12/bin:/usr/bigdata/spark/spark-2.1.1-bin-hadoop2.7/bin:/usr/bigdata/hive/hive/bin:/root/bin
PIPESTATUS=([0]="0")
PPID=12626
PROMPT_COMMAND='printf "\033]0;%s@%s:%s\007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
PS1='[\u@\h \W]\$ '
PS2='> '
PS4='+ '
PWD=/root/shell
SCALA_HOME=/usr/bigdata/scala/scala-2.11.12
SELINUX_LEVEL_REQUESTED=
SELINUX_ROLE_REQUESTED=
SELINUX_USE_CURRENT_RANGE=
SHELL=/bin/bash
SHELLOPTS=braceexpand:emacs:hashall:histexpand:history:interactive-comments:monitor
SHLVL=1
SPARK_HOME=/usr/bigdata/spark/spark-2.1.1-bin-hadoop2.7
SSH_CLIENT='192.168.220.1 14112 22'
SSH_CONNECTION='192.168.220.1 14112 192.168.220.15 22'
SSH_TTY=/dev/pts/1
TERM=xterm
UID=0
USER=root
XDG_RUNTIME_DIR=/run/user/0
XDG_SESSION_ID=50
ZOOKEEPER_HOME=/usr/bigdata/zookeeper/zookeeper-3.4.10
_=/root
colors=/root/.dircolors
```

# 二、自定义变量
## 2.1 基本语法
1. 定义变量: 变量=值
2. 撤销变量: unset 变量
3. 声明静态变量: readonly 变量, 注意: 静态变量不能 unset

## 2.2 变量定义规则
1. 变量名称可以由字母、数字和下划线组成, 但是不能以数字开头, 环境变量建议大写.
2. 等号两侧不能有空格
3. 在 bash 中, 变量默认类型都是字符串类型, 无法直接进行数值运算.
4. 变量的值如果有空格, 需要使用双引号或单引号括起来.

## 2.3 示例
1. 定义变量 A
```bash
[root@hadoop1 shell]# A=5
[root@hadoop1 shell]# echo $A
5
```
2. 给变量 A 重新赋值
```bash
[root@hadoop1 shell]# A=10
[root@hadoop1 shell]# echo $A
10
```
3. 撤销变量
```bash
[root@hadoop1 shell]# unset A
[root@hadoop1 shell]# echo $A

```
4. 声明静态的变量`B=2`, 该变量不能 unset
```bash
[root@hadoop1 shell]# readonly B=2
[root@hadoop1 shell]# echo $B
2
[root@hadoop1 shell]# B=9
-bash: B: 只读变量
```
5. 在 bash 中, 变量默认类型都是字符串类型, 无法直接进行数值运算
```bash
[root@hadoop1 shell]# C=1+2
[root@hadoop1 shell]# echo $C
1+2
```
6. 如果变量的值有空格, 需要使用双引号或单引号括起来
```bash
[root@hadoop1 shell]# D=I am a Kino
-bash: am: 未找到命令

[root@hadoop1 shell]# D="I am a Kino"
[root@hadoop1 shell]# echo $D
I am a Kino
```
7. 可以把变量提升为全局环境变量, 提供其他shell程序使用: `export 变量名`
```bash
在helloworld.sh文件中增加echo $B
#!/bin/bash
echo "helloworld"
echo $B

[root@hadoop1 shell]# sh helloworld.sh 
helloworld

```
并没有打印输出变量 B 的值
```bash
[root@hadoop1 shell]# export B
[root@hadoop1 shell]# sh helloworld.sh 
helloworld
2
```


# 三、特殊变量: $n
## 3.1 语法说明
`$n`: n 为数字, $0 代表该脚本的名称, $1-$9代表第1到底9个参数, 10以上的参数需要用大括号包含:${10}

## 3.2 示例
① 输出该脚本文件的名称、输入参数1和输入参数2的值
```bash
[root@hadoop1 shell]# vim parameter.sh
#!/bin/bash
echo "$0        $1       $2"

[root@hadoop1 shell]# sh parameter.sh kino shell
parameter.sh 	kino	 shell
```

# 四、特殊变量: $#
## 4.1 语法说明
`$#`: 获取所有输入参数的个数, 常用语循环中

## 4.2 示例
① 获取输入参数的个数
```bash
[root@hadoop1 shell]# vim parameter.sh 
#!/bin/bash
echo "$0        $1       $2"
echo "参数个数为: $#"

[root@hadoop1 shell]# sh parameter.sh kino shell
parameter.sh 	kino	 shell
参数个数为: 2
```

# 五、特殊变量: $* 和 $@
## 5.1 语法说明
`$*`: 代表命令行中所有的参数, `$*`把所有的参数看成一个整体
`$@`: 也代表命令行中所有的参数, 不过 `$@`把每个参数区分对待

## 5.2 示例
① 打印输入的所有参数
```bash
[root@hadoop1 shell]# sh parameter.sh kino shell
parameter.sh 	kino	 shell
参数个数为: 2
kino shell
kino shell
```

# 六、特殊变量: $?
## 6.1 语法说明
`$?`: 最后一次执行的命令的返回状态; 如果这个变量的值为0, 证明行一个命令正确执行; 否则反之;

## 6.2 示例
① 判断 helloworld.sh 脚本是否正确执行
```bash
[root@hadoop1 shell]# sh helloworld.sh 
helloworld
2
[root@hadoop1 shell]# echo $?
0
```



