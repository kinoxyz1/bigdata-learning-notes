






环境: macOS


# 一、下载安装
[官方下载地址](https://gradle.org/releases/)


```bash
# 解压
$ unzip gradle-7.4.2-all.zip
# 配置环境变量
$ sudo vim /etc/profile
export GRADLE_HOME=/Users/kino/software/gradle-7.4.2
# GRADLE_USER_HOME 配置成自己的 maven 目录(也可以是别的插件的本地仓库目录)
# GRADLE_USER_HOME 相当于 Gradle 本地仓库位置和 Gradle Wrapper 缓存目录
export GRADLE_USER_HOME=/Users/kino/software/apache-maven-3.8.3/repository
export PATH=$PATH:$GRADLE_HOME/bin:$GRADLE_USER_HOME
# 校验安装情况
$ source /etc/profile
$ gradle --version
Welcome to Gradle 7.4.2!

Here are the highlights of this release:
 - Aggregated test and JaCoCo reports
 - Marking additional test source directories as tests in IntelliJ
 - Support for Adoptium JDKs in Java toolchains

For more details see https://docs.gradle.org/7.4.2/release-notes.html


------------------------------------------------------------
Gradle 7.4.2
------------------------------------------------------------

Build time:   2022-03-31 15:25:29 UTC
Revision:     540473b8118064efcc264694cbcaa4b677f61041

Kotlin:       1.5.31
Groovy:       3.0.9
Ant:          Apache Ant(TM) version 1.10.11 compiled on July 10 2021
JVM:          1.8.0_211 (Oracle Corporation 25.211-b12)
OS:           Mac OS X 10.16 x86_64
```

# 二、Idea 配置 Gradle
关闭 Idea 所有窗口之后, 再设置 Gradle, 把环境变量 GRADLE_USER_HOME 的值配置到 Gradle User Home 中.

# 三、Gradle 修改镜像源
Gradle 自带的 Maven 镜像源是国外, 下载镜像很慢.

在 gradle 安装目录的 init.d 文件夹中创建: init.gradle 文件
```bash
$ vim init.d/init.gradle
# 添加如下内容
allprojects {
    repositories {
        mavenLocal()
        maven { name "Alibaba" ; url "https://maven.aliyun.com/repository/public" }
        maven { name "Bstek" ; url "https://nexus.bsdn.org/content/groups/public/"}
        mavenCentral()
    }

    buildscript { 
        repositories { 
            maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public' }
            maven { name "Bstek" ; url 'http://nexus.bsdn.org/content/groups/public/' }
            maven { name "M2" ; url 'https://plugins.gradle.org/m2/' }
        }
    }
}
```

note:
> 关于 init.gradle 文件如何启用, 有如下办法
> 
> 1.在命令行指定文件,例如：gradle --init-script yourdir/init.gradle -q taskName。你可以多次输入此命令来指定多个init文件
> 
> 2.把init.gradle文件放到 USER_HOME/.gradle/ 目录下
>
> 3.把以.gradle结尾的文件放到 USER_HOME/.gradle/init.d/ 目录下
>
> 4.把以.gradle结尾的文件放到 GRADLE_HOME/init.d/ 目录下
>
> 如果存在上面的4种方式的2种以上，gradle会按上面的1-4序号依次执行这些文件，如果给定目录下存在多个init脚本，会按拼音a-z顺序执行这些脚本， 每个init脚本都存在一个对应的gradle实例,你在这个文件中调用的所有方法和属性， 都会委托给这个gradle实例， 每个init脚本都实现了Script接口。

note:
> 关于仓库地址说明:
> mavenLocal(): 指定使用maven本地仓库，而本地仓库在配置maven时settings文件指定的仓库位置。如 /repository， gradle 查找jar包顺序如下：USER_HOME/.m2/settings.xml >> M2_HOME/conf/settings.xml >> USER_HOME/.m2/repository
> 
> maven { url 地址}，指定maven仓库，一般用私有仓库地址或其它的第三方库【比如阿里镜像仓库地址】。
> 
> mavenCentral()：这是Maven的中央仓库，无需配置，直接声明就可以使用。
> 
> jcenter():JCenter中央仓库，实际也是是用的maven搭建的，但相比Maven仓库更友好，通过CDN分发，并且支持https访问,在新版本中已经废弃了，替换为了mavenCentral()。
> 
> 总之, gradle可以通过指定仓库地址为本地maven仓库地址和远程仓库地址相结合的方式，避免每次都会去远程仓库下载依赖库。这种方式也有一定的问题，如果本地maven仓库有这个依赖，就会从直接加载本地依赖，如果本地仓库没有该依赖，那么还是会从远程下载。但是下载的jar不是存储在本地maven仓库中，而是放在自己的缓存目录中， 默认在 USER_HOME/.gradle/caches目录,当然如果我们配置过GRADLE_USER_HOME环境变量， 则会放在GRADLE_USER_HOME/caches目录,那么可不可以将gradle caches指向maven repository。 我们说这是不行的， caches下载文件不是按照maven仓库中存放的方式。

note:
> 阿里云镜像仓库指南 > gradle 配置指南: https://developer.aliyun.com/mvn/guide
















