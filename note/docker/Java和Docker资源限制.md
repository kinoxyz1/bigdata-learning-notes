


# 一、问题描述
在k8s生产环境中发现，Java服务设置jvm内存并不生效。经排查发现，与jdk版本有关.


# 二、验证
| 服务          | 版本       | 说明        | 
|-------------|----------|-----------|
| docker | 24.0.2   |           | 
| 宿主机 | centos7  | 20c88G    |
| jdk | jdk8u131 | jvm 参数不生效 | 
| jdk | jdk8u222 | jvm 参数生效  |

 ## 2.1 jdk8u131
给容器限制300m内存
```bash 
docker run --name jdk --rm -m 300m -it openjdk:8u131 /bin/bash
```
由于 top 和 free 都是从 proc 文件系统读取的指标, 如 `/proc/meminfo` 、 `/proc/vmstat` 、 `/proc/PID/smaps` 等，而容器资源隔离是由 cgroup 完成的，上述说的 /proc 等不支持cgroup，所以这里不在展示容器中显示的内容，因为与服务器一致。

如何验证容器限制的300m内存不生效? 运行以下Java程序:
```bash
cat > MemEat.java << EOF
import java.util.ArrayList;
import java.util.List;
import java.lang.management.ManagementFactory;

public class MemEat {
    public static void main(String[] args) {
        // 获取当前 JVM 的名称，格式通常为 "pid@hostname"
        String jvmName = ManagementFactory.getRuntimeMXBean().getName();
        System.out.println("JVM Name: " + jvmName);

        // 提取 PID（进程号）
        String pid = jvmName.split("@")[0];
        System.out.println("Process ID (PID): " + pid);
    
        // 等待键盘回车
        System.out.println("Press Enter to continue...");
        try {
            System.in.read(); // 等待用户按下回车
        } catch (Exception e) {
            System.err.println("Error reading input: " + e.getMessage());
        }
    
        List l = new ArrayList<>();
        while (true) {
            byte b[] = new byte[1048576];
            l.add(b);
            Runtime rt = Runtime.getRuntime();
            System.out.println( "free memory: " + rt.freeMemory() );
        }
    }
}
EOF
javac MemEat.java
java MemEat
# 等待键盘事件, 暂时不要回车
```
另启动一个窗口(在宿主机上, 不要进容器中), 查询系统日志:
```bash
ps -ef|grep MemEat
root     11433 12916  0 21:44 pts/0    00:00:00 java MemEat
```
可以看见进程号是: 11433

然后再这个窗口上:
```bash
tail -f /var/log/syslog
```
回到之前执行java命令的窗口按下回车，可以看到如下结果:
```bash
JVM Name: 226@8580b0e84bfe
Process ID (PID): 226
Press Enter to continue...
free memory: 1247218200
...
free memory: 1166183088
free memory: 1165134496
Killed
```
再观察第二个窗口输出:
```bash
Dec  9 21:47:11 jz-desktop-08 kernel: [3217913.648993] [  12916]     0 12916     4994      951    77824        0             0 bash
Dec  9 21:47:11 jz-desktop-08 kernel: [3217913.648994] [  11433]     0 11433  6251105    26852   733184        0             0 java
Dec  9 21:47:11 jz-desktop-08 kernel: [3217913.648995] oom-kill:constraint=CONSTRAINT_MEMCG,nodemask=(null),cpuset=8580b0e84bfe78b99724b9556fe787044fbdb4721c806f1c0361fe576842e899,mems_allowed=0,oom_memcg=/docker/8580b0e84bfe78b99724b9556fe787044fbdb4721c806f1c0361fe576842e899,task_memcg=/docker/8580b0e84bfe78b99724b9556fe787044fbdb4721c806f1c0361fe576842e899,task=java,pid=11433,uid=0
Dec  9 21:47:11 jz-desktop-08 kernel: [3217913.649049] Memory cgroup out of memory: Killed process 11433 (java) total-vm:25004420kB, anon-rss:91796kB, file-rss:15612kB, shmem-rss:0kB, UID:0 pgtables:716kB oom_score_adj:0
```
能够看到: `Memory cgroup out of memory: Killed process 11433 (java)`, 通过两个窗口的输入总结:
1. 第一个窗口显示, 容器剩余内存还剩 `1165134496`, 但是直接被killed 了，
2. 第二个窗口显示, cgroup 内存不足, 主动kill了java进程。

这里需要明确的是: 
1. 容器在最开始只设置了300m内存。
2. Java 进程由容器主动kill，此时Java程序明显认为还有内存没使用，所以并没有引发oom。

其实这里可以借助 arthas 工具查看堆内存大小, 然后在第二个窗口下载arthas查看堆内存大小:
```bash
# 第一个窗口操作在上面, 第二个窗口操作如下:
curl -# -o arthas-bin.zip https://jzdata-gz.tos-cn-guangzhou.volces.com/kino/arthas-bin.zip
unzip arthas-bin.zip
java -jar arthas-boot.jar
# 输入对应的进程号,
[arthas@119]$ memory
Memory                                                                            used                       total                      max                         usage
heap                                                                              53M                        702M                       17813M                      0.30%
ps_eden_space                                                                     30M                        314M                       6576M                       0.47%
ps_survivor_space                                                                 0K                         53248K                     53248K                      0.00%
ps_old_gen                                                                        22M                        336M                       13360M                      0.17%
nonheap                                                                           31M                        32M                        -1                          97.59%
code_cache                                                                        7M                         7M                         240M                        2.93%
metaspace                                                                         21M                        22M                        -1                          97.48%
compressed_class_space                                                            2M                         2M                         1024M                       0.25%
direct                                                                            0K                         0K                         -                           0.00%
mapped                                                                            0K                         0K                         -                           0.00%
```
这里就能看到, jvm 认为堆内存有 17813M(大约17G). 并且，Java8 如果不设置Xmx参数，默认为系统的1/4左右, 可以通过 `java -XX:+PrintFlagsFinal -version | grep MaxHeapSize` 查看。
```bash
java -XX:+PrintFlagsFinal -version | grep MaxHeapSize
    uintx MaxHeapSize                              := 21013463040                         {product}
```
所以容器启动时的300m内存在jvm中并没有生效, jvm还是认为有宿主机那么大的内存可以使用。



## 2.1 jdk8u222
重新执行上面验证步骤
```bash
docker run --name jdk --rm -m 300m -it openjdk:8u222 /bin/bash
```

如何验证容器限制的300m内存不生效? 运行以下Java程序:
```bash
cat > MemEat.java << EOF
import java.util.ArrayList;
import java.util.List;
import java.lang.management.ManagementFactory;

public class MemEat {
    public static void main(String[] args) {
        // 获取当前 JVM 的名称，格式通常为 "pid@hostname"
        String jvmName = ManagementFactory.getRuntimeMXBean().getName();
        System.out.println("JVM Name: " + jvmName);

        // 提取 PID（进程号）
        String pid = jvmName.split("@")[0];
        System.out.println("Process ID (PID): " + pid);
    
        // 等待键盘回车
        System.out.println("Press Enter to continue...");
        try {
            System.in.read(); // 等待用户按下回车
        } catch (Exception e) {
            System.err.println("Error reading input: " + e.getMessage());
        }
    
        List l = new ArrayList<>();
        while (true) {
            byte b[] = new byte[1048576];
            l.add(b);
            Runtime rt = Runtime.getRuntime();
            System.out.println( "free memory: " + rt.freeMemory() );
        }
    }
}
EOF
javac MemEat.java
java MemEat
# 等待键盘事件, 暂时不要回车
```
另启动一个窗口(在宿主机上, 不要进容器中), 查询系统日志:
```bash
ps -ef|grep MemEat
root     12954  9021 0 21:44 pts/0    00:00:00 java MemEat
```
可以看见进程号是: 12954

然后再这个窗口上:
```bash
tail -f /var/log/syslog
```
回到之前执行java命令的窗口按下回车，可以看到如下结果:
```bash
JVM Name: 28@287bc803f295
Process ID (PID): 28
Press Enter to continue...

free memory: 6574224
free memory: 1634744
free memory: 1089296
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at MemEat.main(MemEat.java:25)
```
可以看到，这里变成了Java抛出OOM，再观察第二个窗口输出搜 12954, 发现系统日志中没有相关输出。

由此可以得出结论:
1. 容器在最开始只设置了300m内存被jvm已经识别到了。
2. 这一次是由Java主动抛出OOM，而不是容器主动killed。

这里可以借助 arthas 工具查看堆内存大小, 然后在第二个窗口下载arthas查看堆内存大小:
```bash
# 第一个窗口操作在上面, 第二个窗口操作如下:
curl -# -o arthas-bin.zip https://jzdata-gz.tos-cn-guangzhou.volces.com/kino/arthas-bin.zip
unzip arthas-bin.zip
java -jar arthas-boot.jar
# 输入对应的进程号,
[arthas@119]$ memory
Memory                                                                            used                       total                      max                         usage
heap                                                                              20M                        27M                        121M                        17.12%
eden_space                                                                        2M                         7M                         33M                         8.29%
survivor_space                                                                    959K                       960K                       4288K                       22.39%
tenured_gen                                                                       17M                        18M                        84M                         20.39%
nonheap                                                                           31M                        32M                        -1                          97.51%
code_cache                                                                        6M                         6M                         240M                        2.86%
metaspace                                                                         21M                        22M                        -1                          97.35%
compressed_class_space                                                            2M                         2M                         1024M                       0.25%
direct                                                                            0K                         0K                         -                           0.00%
mapped                                                                            0K                         0K                         -                           0.00%
```
这里就能看到, jvm 认为堆内存有 121M. 并且可以通过 `java -XX:+PrintFlagsFinal -version | grep MaxHeapSize` 查看
```bash
java -XX:+PrintFlagsFinal -version | grep MaxHeapSize
    uintx MaxHeapSize                              := 132120576                         {product}
```
对比上一次，可以看到本地jvm堆内存已经被容器内存限制住了。

# 三、说明
经过查找验证, 在Java8和Java9中，对于容器内存、CPU的支持并不理想, java8u131+到java9这些版本中, 可以通过设置如下参数来强制JVM查看Linux cgroup配置:
```bash
-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap
```
只有这样，Java在内存达到容器限制内存后, 会尝试gc, 如果内存还是不足, 则会抛出OutOfMemoryException。

在jdk10中增加了新的参数: `-XX:+UseContainerSupport`, 该参数默认开启, 如果不设置该参数也能让JVM查看Linux cgroup配置。



对于cpu限制, 测试过上面两个版本的jdk，都能限制住。
```bash
docker run --cpus=2 --name jdk --rm -m 300m -it openjdk:8u131 /bin/bash

cat > CpuEat.java << EOF
public class CpuEat {
    public static void main(String[] args) {
        for (int i = 0; i < 40; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    while (true) {
                        // 执行更频繁的空循环
                        for (int j = 0; j < Integer.MAX_VALUE; j++) {
                            // 空循环
                        }
                    }
                }
            }).start();
        }
    }
}
EOF
javac CpuEat.java
java CpuEat
```
另一个创建top
```bash
top - 14:22:26 up 37 days,  6:26,  0 users,  load average: 10.72, 7.75, 3.81
Tasks:   4 total,   1 running,   3 sleeping,   0 stopped,   0 zombie
%Cpu(s): 13.2 us,  0.6 sy,  0.0 ni, 85.4 id,  0.4 wa,  0.0 hi,  0.4 si,  0.0 st
KiB Mem : 82081744 total,   906016 free, 17883808 used, 63291916 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 63211820 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
   43 root      20   0 26.334g  28292  15556 S 202.7  0.0   3:04.45 java
    1 root      20   0   19960   3788   3228 S   0.0  0.0   0:00.02 bash
  118 root      20   0   19956   3548   3092 S   0.0  0.0   0:00.01 bash
```



# 四、结论
1. 对于jdk8u131及以下版本的jdk，如果不能升级jdk，则一定要设置上: `-Xmx`, 否则容器会kill掉你的Java进程而不会触发gc;
2. 对于jdk8u131以上和jdk9, 可以设置: `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`, 不过经过测试发现jdk8u222 没设置这两个参数也生效了。这里需要说明: 虽然jvm已经识别到了cgroup配置, 但是不设置 `-Xmx` 参数, [堆内存默认为容器内存的1/4](https://stackoverflow.com/questions/28272923/what-is-the-default-max-heap-size-xmx-in-java-8)。
3. 对于jdk10及以上, 需要明确: `UseContainerSupport` 默认已经开启。




















