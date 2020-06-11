@[TOC]


----
# 一、组件版本
| 组件 |版本  |
|--|--|
|Hadoop	 | 3.0.0+cdh6.1.1  |
|Hive	|2.1.1+cdh6.1.1|
|spark	|2.4.0+cdh6.1.1|

---
#  二、问题描述
在 Spark 向 Hive分区表 写入数据时，抛出异常如下：
```java
org.apache.spark.SparkException: Requested partitioning does not match the test_table_name table:
Requested partitions: 
Table partitions: city, year, month, day
	at org.apache.spark.sql.hive.execution.InsertIntoHiveTable.processInsert(InsertIntoHiveTable.scala:141)
	at org.apache.spark.sql.hive.execution.InsertIntoHiveTable.run(InsertIntoHiveTable.scala:99)
	at org.apache.spark.sql.hive.execution.CreateHiveTableAsSelectCommand.run(CreateHiveTableAsSelectCommand.scala:66)
	at org.apache.spark.sql.execution.command.DataWritingCommandExec.sideEffectResult$lzycompute(commands.scala:104)
	at org.apache.spark.sql.execution.command.DataWritingCommandExec.sideEffectResult(commands.scala:102)
	at org.apache.spark.sql.execution.command.DataWritingCommandExec.doExecute(commands.scala:122)
	at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:131)
	at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:127)
	at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeQuery$1.apply(SparkPlan.scala:155)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
	at org.apache.spark.sql.execution.SparkPlan.executeQuery(SparkPlan.scala:152)
	at org.apache.spark.sql.execution.SparkPlan.execute(SparkPlan.scala:127)
	at org.apache.spark.sql.execution.QueryExecution.toRdd$lzycompute(QueryExecution.scala:80)
	at org.apache.spark.sql.execution.QueryExecution.toRdd(QueryExecution.scala:80)
	at org.apache.spark.sql.DataFrameWriter$$anonfun$runCommand$1.apply(DataFrameWriter.scala:654)
	at org.apache.spark.sql.DataFrameWriter$$anonfun$runCommand$1.apply(DataFrameWriter.scala:654)
	at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:77)
	at org.apache.spark.sql.DataFrameWriter.runCommand(DataFrameWriter.scala:654)
	at org.apache.spark.sql.DataFrameWriter.createTable(DataFrameWriter.scala:458)
	at org.apache.spark.sql.DataFrameWriter.saveAsTable(DataFrameWriter.scala:437)
	at org.apache.spark.sql.DataFrameWriter.saveAsTable(DataFrameWriter.scala:393)
```
出错的代码如下:

```java
val rdd = hdfs2RDD(dir: String, spark: SparkSession, task: EtlTask)
    import spark.implicits._
    val res = spark.createDataset(rdd).as[String]
      .map(x => {
        val obj = JSON.parseObject(x)
        val event_refid = obj.getString("event_refid")  //事件全局唯一id
        val event_cate = obj.getString("event_cate") //事件大类
        val event_type = obj.getString("event_type") //事件小类
        val server_fid = obj.getString("server_id")  //分析主机的id
        val event_date = obj.getString("event_date") //事件上传时间
        val camera_fid = obj.getInteger("camera_id")  //关联的摄像头id
        val pole_fid = "-99" //杆件ID  通过 camera_fid 关联  s_common_base_camera 表 获取
        val vai_source = obj.getInteger("source_id")  //视频算法供应商编码
        val version = obj.getString("version")  //协议编码
        val city = 440300 //通过 camera_fid 关联  s_common_base_camera 表 获取

        val tmpTime = obj.getString("event_date").trim //时间
        val dataTime = DateTime.parse(tmpTime, DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss"))
        val tmpTime2 = dataTime.toString("yyyyMMddHH")
        val year = tmpTime2.substring(0, 4)
        val month = tmpTime2.substring(4, 6)
        val day = tmpTime2.substring(6, 8)

        s_vai_primary(UUID.randomUUID().toString, event_refid, event_cate, event_type, server_fid, event_date,
          camera_fid, pole_fid, vai_source, version, city, year, month, day)
      })
    res.show(10, false)
TaskOutput.hive[s_vai_primary](spark, task, Array("city", "year", "month", "day"), res, isPersist = false, dupliCols = Seq(""), isCopy2Tag = false)
```

---
# 三、问题分析
从上面的报错信息，可以看到是分区不匹配造成的
```java
org.apache.spark.SparkException: Requested partitioning does not match the test_table_name table:
Requested partitions: 
Table partitions: city,day,month,day
```
Spark在向Hive分区表写入数据时候，找不到这个数据按照哪个分区写入

Requested partitions后面的值为空，但是去hive元数据里面查发现要插入的表是按照city,day,month,day字段分区的，Table partitions: city,day,month,day。这样造成了不一致匹配问题，从而抛出异常。

问题代码如下，我们走的是if分支，因为表存在，我们也没有删除操作，那么创建InsertIntoHiveTable对象时把分区字段置成了一个空的map

CreateHiveTableAsSelectCommand类 的 run方法
```java
  override def run(sparkSession: SparkSession, child: SparkPlan): Seq[Row] = {
    val catalog = sparkSession.sessionState.catalog
    //查看表是否存在，如果存在走if分支，不存在走else分支
    if (catalog.tableExists(tableIdentifier)) {
      //如果是Overwrite写入模式，那么需要drop掉表
      assert(mode != SaveMode.Overwrite,
        s"Expect the table $tableIdentifier has been dropped when the save mode is Overwrite")

      if (mode == SaveMode.ErrorIfExists) {
        throw new AnalysisException(s"$tableIdentifier already exists.")
      }
      if (mode == SaveMode.Ignore) {
        // Since the table already exists and the save mode is Ignore, we will just return.
        return Seq.empty
      }

      //我们使用的append模式写入hive,并且表已经创建，那么创建如下的InsertIntoHiveTable对象并调用其run方法
      InsertIntoHiveTable(
        tableDesc,
        Map.empty, //分区字段对应了一个空map
        query,
        overwrite = false, //写入模式不是overwriter
        ifPartitionNotExists = false,
        outputColumns = outputColumns).run(sparkSession, child)
     } else {
     //表不存在情况
     //如果schema不为空那么将要抛出异常
      assert(tableDesc.schema.isEmpty)
      //创建表
      catalog.createTable(tableDesc.copy(schema = query.schema), ignoreIfExists = false)

      try {
        // Read back the metadata of the table which was created just now.
        val createdTableMeta = catalog.getTableMetadata(tableDesc.identifier)
        // For CTAS, there is no static partition values to insert.
        //分区字段对应的是partitionBy指定的字段
        val partition = createdTableMeta.partitionColumnNames.map(_ -> None).toMap
        InsertIntoHiveTable(
          createdTableMeta,
          partition,
          query,
          overwrite = true,   //写入模式是overwrite
          ifPartitionNotExists = false,
          outputColumns = outputColumns).run(sparkSession, child)
      } catch {
        case NonFatal(e) =>
          // drop the created table.
          catalog.dropTable(tableIdentifier, ignoreIfNotExists = true, purge = false)
          throw e
      }
    }

    Seq.empty[Row]
  }
```
我们看看InsertIntoHiveTable.run方法如何使用这个空map的

```java
override def run(sparkSession: SparkSession, child: SparkPlan): Seq[Row] = {
    val externalCatalog = sparkSession.sharedState.externalCatalog
    val hadoopConf = sparkSession.sessionState.newHadoopConf()

    val hiveQlTable = HiveClientImpl.toHiveTable(table)
    // Have to pass the TableDesc object to RDD.mapPartitions and then instantiate new serializer
    // instances within the closure, since Serializer is not serializable while TableDesc is.
    val tableDesc = new TableDesc(
      hiveQlTable.getInputFormatClass,
      // The class of table should be org.apache.hadoop.hive.ql.metadata.Table because
      // getOutputFormatClass will use HiveFileFormatUtils.getOutputFormatSubstitute to
      // substitute some output formats, e.g. substituting SequenceFileOutputFormat to
      // HiveSequenceFileOutputFormat.
      hiveQlTable.getOutputFormatClass,
      hiveQlTable.getMetadata
    )
    val tableLocation = hiveQlTable.getDataLocation
    val tmpLocation = getExternalTmpPath(sparkSession, hadoopConf, tableLocation)

    try {
      processInsert(sparkSession, externalCatalog, hadoopConf, tableDesc, tmpLocation, child)
    } finally {
      // Attempt to delete the staging directory and the inclusive files. If failed, the files are
      // expected to be dropped at the normal termination of VM since deleteOnExit is used.
      deleteExternalTmpPath(hadoopConf)
    }

    // un-cache this table.
    sparkSession.catalog.uncacheTable(table.identifier.quotedString)
    sparkSession.sessionState.catalog.refreshTable(table.identifier)

    CommandUtils.updateTableStats(sparkSession, table)

    // It would be nice to just return the childRdd unchanged so insert operations could be chained,
    // however for now we return an empty list to simplify compatibility checks with hive, which
    // does not return anything for insert operations.
    // TODO: implement hive compatibility as rules.
    Seq.empty[Row]
  }

private def processInsert(
      sparkSession: SparkSession,
      externalCatalog: ExternalCatalog,
      hadoopConf: Configuration,
      tableDesc: TableDesc,
      tmpLocation: Path,
      child: SparkPlan): Unit = {
    val fileSinkConf = new FileSinkDesc(tmpLocation.toString, tableDesc, false)

    val numDynamicPartitions = partition.values.count(_.isEmpty)
    val numStaticPartitions = partition.values.count(_.nonEmpty)
    val partitionSpec = partition.map {
      case (key, Some(value)) => key -> value
      case (key, None) => key -> ""
    }

    // All partition column names in the format of "<column name 1>/<column name 2>/..."
    val partitionColumns = fileSinkConf.getTableInfo.getProperties.getProperty("partition_columns")
    val partitionColumnNames = Option(partitionColumns).map(_.split("/")).getOrElse(Array.empty)

    // By this time, the partition map must match the table's partition columns
    // 此处就是报错的详细地方了
    if (partitionColumnNames.toSet != partition.keySet) {
      throw new SparkException(
        s"""Requested partitioning does not match the ${table.identifier.table} table:
           |Requested partitions: ${partition.keys.mkString(",")}
           |Table partitions: ${table.partitionColumnNames.mkString(",")}""".stripMargin)
    }

    // Validate partition spec if there exist any dynamic partitions
    if (numDynamicPartitions > 0) {
      // Report error if dynamic partitioning is not enabled
      if (!hadoopConf.get("hive.exec.dynamic.partition", "true").toBoolean) {
        throw new SparkException(ErrorMsg.DYNAMIC_PARTITION_DISABLED.getMsg)
      }

      // Report error if dynamic partition strict mode is on but no static partition is found
      if (numStaticPartitions == 0 &&
        hadoopConf.get("hive.exec.dynamic.partition.mode", "strict").equalsIgnoreCase("strict")) {
        throw new SparkException(ErrorMsg.DYNAMIC_PARTITION_STRICT_MODE.getMsg)
      }

      // Report error if any static partition appears after a dynamic partition
      val isDynamic = partitionColumnNames.map(partitionSpec(_).isEmpty)
      if (isDynamic.init.zip(isDynamic.tail).contains((true, false))) {
        throw new AnalysisException(ErrorMsg.PARTITION_DYN_STA_ORDER.getMsg)
      }
    }

    table.bucketSpec match {
      case Some(bucketSpec) =>
        // Writes to bucketed hive tables are allowed only if user does not care about maintaining
        // table's bucketing ie. both "hive.enforce.bucketing" and "hive.enforce.sorting" are
        // set to false
        val enforceBucketingConfig = "hive.enforce.bucketing"
        val enforceSortingConfig = "hive.enforce.sorting"

        val message = s"Output Hive table ${table.identifier} is bucketed but Spark " +
          "currently does NOT populate bucketed output which is compatible with Hive."

        if (hadoopConf.get(enforceBucketingConfig, "true").toBoolean ||
          hadoopConf.get(enforceSortingConfig, "true").toBoolean) {
          throw new AnalysisException(message)
        } else {
          logWarning(message + s" Inserting data anyways since both $enforceBucketingConfig and " +
            s"$enforceSortingConfig are set to false.")
        }
      case _ => // do nothing since table has no bucketing
    }

    val partitionAttributes = partitionColumnNames.takeRight(numDynamicPartitions).map { name =>
      query.resolve(name :: Nil, sparkSession.sessionState.analyzer.resolver).getOrElse {
        throw new AnalysisException(
          s"Unable to resolve $name given [${query.output.map(_.name).mkString(", ")}]")
      }.asInstanceOf[Attribute]
    }

    saveAsHiveFile(
      sparkSession = sparkSession,
      plan = child,
      hadoopConf = hadoopConf,
      fileSinkConf = fileSinkConf,
      outputLocation = tmpLocation.toString,
      partitionAttributes = partitionAttributes)

    if (partition.nonEmpty) {
      if (numDynamicPartitions > 0) {
        externalCatalog.loadDynamicPartitions(
          db = table.database,
          table = table.identifier.table,
          tmpLocation.toString,
          partitionSpec,
          overwrite,
          numDynamicPartitions)
      } else {
        // scalastyle:off
        // ifNotExists is only valid with static partition, refer to
        // https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertingdataintoHiveTablesfromqueries
        // scalastyle:on
        val oldPart =
          externalCatalog.getPartitionOption(
            table.database,
            table.identifier.table,
            partitionSpec)

        var doHiveOverwrite = overwrite

        if (oldPart.isEmpty || !ifPartitionNotExists) {
          // SPARK-18107: Insert overwrite runs much slower than hive-client.
          // Newer Hive largely improves insert overwrite performance. As Spark uses older Hive
          // version and we may not want to catch up new Hive version every time. We delete the
          // Hive partition first and then load data file into the Hive partition.
          if (oldPart.nonEmpty && overwrite) {
            oldPart.get.storage.locationUri.foreach { uri =>
              val partitionPath = new Path(uri)
              val fs = partitionPath.getFileSystem(hadoopConf)
              if (fs.exists(partitionPath)) {
                if (!fs.delete(partitionPath, true)) {
                  throw new RuntimeException(
                    "Cannot remove partition directory '" + partitionPath.toString)
                }
                // Don't let Hive do overwrite operation since it is slower.
                doHiveOverwrite = false
              }
            }
          }

          // inheritTableSpecs is set to true. It should be set to false for an IMPORT query
          // which is currently considered as a Hive native command.
          val inheritTableSpecs = true
          externalCatalog.loadPartition(
            table.database,
            table.identifier.table,
            tmpLocation.toString,
            partitionSpec,
            isOverwrite = doHiveOverwrite,
            inheritTableSpecs = inheritTableSpecs,
            isSrcLocal = false)
        }
      }
    } else {
      externalCatalog.loadTable(
        table.database,
        table.identifier.table,
        tmpLocation.toString, // TODO: URI
        overwrite,
        isSrcLocal = false)
    }
  }
```


----
# 四、解决办法
从github下载spark源码，找到 `CreateHiveTableAsSelectCommand` 类 的 `run`方法
org\apache\spark\sql\hive\execution\CreateHiveTableAsSelectCommand.scala
修改如下代码:

将
```java
InsertIntoHiveTable(
        tableDesc,
        Map.empty,
        query,
        overwrite = false,
        ifPartitionNotExists = false,
        outputColumnNames = outputColumnNames).run(sparkSession, child)
```
修改为
```java
val partition = tableDesc.partitionColumnNames.map(_ -> None).toMap
InsertIntoHiveTable(
        tableDesc,
        partition,
        query,
        overwrite = false,
        ifPartitionNotExists = false,
        outputColumnNames = outputColumnNames).run(sparkSession, child)
```
编译代码，替换CDH中 spark-hive jar包的`CreateHiveTableAsSelectCommand` class文件
![编译代码](../../img/spark/20200601102530865.png)
具体相关操作我已经改好了，只需下载替换即可，下载链接
链接：https://pan.baidu.com/s/10B-1sdIN3NGJtNrcxqxFHg 
提取码：lic7 
复制这段内容后打开百度网盘手机App，操作更方便哦

替换完记得重启