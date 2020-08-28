


* [测试 HDFS 写性能](#%E6%B5%8B%E8%AF%95-hdfs-%E5%86%99%E6%80%A7%E8%83%BD)
* [测试 HDFS 读性能](#%E6%B5%8B%E8%AF%95-hdfs-%E8%AF%BB%E6%80%A7%E8%83%BD)
* [删除测试生成数据](#%E5%88%A0%E9%99%A4%E6%B5%8B%E8%AF%95%E7%94%9F%E6%88%90%E6%95%B0%E6%8D%AE)
* [使用 Sort 程序测评 MapReduce](#%E4%BD%BF%E7%94%A8-sort-%E7%A8%8B%E5%BA%8F%E6%B5%8B%E8%AF%84-mapreduce)


---
# 测试 HDFS 写性能
测试内容：向HDFS集群写10个128M的文件
```bash
hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB

20/06/13 22:44:37 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
20/06/13 22:44:37 INFO fs.TestDFSIO:            Date & time: Sat Jun 13 22:44:37 CST 2020
20/06/13 22:44:37 INFO fs.TestDFSIO:        Number of files: 10
20/06/13 22:44:37 INFO fs.TestDFSIO: Total MBytes processed: 10.0
20/06/13 22:44:37 INFO fs.TestDFSIO:      Throughput mb/sec: 0.23219095384043836
20/06/13 22:44:37 INFO fs.TestDFSIO: Average IO rate mb/sec: 0.3201367259025574
20/06/13 22:44:37 INFO fs.TestDFSIO:  IO rate std deviation: 0.13613761220588563
20/06/13 22:44:37 INFO fs.TestDFSIO:     Test exec time sec: 74.528

```


# 测试 HDFS 读性能
测试内容：读取HDFS集群10个128M的文件
```bash
hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -read -nrFiles 10 -fileSize 128MB

20/06/13 22:47:44 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
20/06/13 22:47:44 INFO fs.TestDFSIO:            Date & time: Sat Jun 13 22:47:44 CST 2020
20/06/13 22:47:44 INFO fs.TestDFSIO:        Number of files: 10
20/06/13 22:47:44 INFO fs.TestDFSIO: Total MBytes processed: 9.0
20/06/13 22:47:44 INFO fs.TestDFSIO:      Throughput mb/sec: 1.9214346712211785
20/06/13 22:47:44 INFO fs.TestDFSIO: Average IO rate mb/sec: NaN
20/06/13 22:47:44 INFO fs.TestDFSIO:  IO rate std deviation: NaN
20/06/13 22:47:44 INFO fs.TestDFSIO:     Test exec time sec: 50.619
```


# 删除测试生成数据
```bash
hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -clean
```


# 使用 Sort 程序测评 MapReduce 
- 使用RandomWriter来产生随机数，每个节点运行10个Map任务，每个Map产生大约1G大小的二进制随机数
    ```bash 
    hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar randomwriter random-data
    ```

- 执行Sort程序
    ```bash 
    hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar sort random-data sorted-data
    ```

- 验证数据是否真正排好序了
    ```bash
    hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar testmapredsort -sortInput random-data -sortOutput sorted-data
    ```