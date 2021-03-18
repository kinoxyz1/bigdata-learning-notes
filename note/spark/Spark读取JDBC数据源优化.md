





---



# 一、单并发(单 Partition)
```scala
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName(this.getClass.getName)

val spark = SparkSession.builder().config(conf).getOrCreate()

val prop = new Properties()
prop.put("user", "root")
prop.put("password", "123456")
prop.put("driver", "com.mysql.jdbc.Driver")

val readDf = spark.read.jdbc("jdbc:mysql://***.***.***.**:3306/data_base?useUnicode=true&characterEncoding=utf-8&useSSL=false",
  "salej", prop)

println("读取数据库的并行度是: "+readDf.rdd.partitions.size)
```
此时 Spark 读取 MySQL 就只有一个并行度.

# 二、多并发(多 Partition)
## 2.1 根据 Int 类型 分区
```scala
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName(this.getClass.getName)

val spark = SparkSession.builder().config(conf).getOrCreate()

val prop = new Properties()
prop.put("user", "root")
prop.put("password", "123456")
prop.put("driver", "com.mysql.jdbc.Driver")
prop.put("partitionColumn", "seqid")
prop.put("numPartitions", "300")
prop.put("lowerBound", "0")
prop.put("upperBound", "200000")

val readDf = spark.read.jdbc("jdbc:mysql://***.***.***.**:3306/data_base?useUnicode=true&characterEncoding=utf-8&useSSL=false",
  "salej", prop)

println("读取数据库的并行度是: "+readDf.rdd.partitions.size)
```
参数说明:
- partitionColumn: 根据该字段进行分区, 需要传入的数据类型必须为整形;
- numPartitions: 分区的个数;
- lowerBound: 分区的下界;
- upperBound: 分区的上界;

此时, Spark 的并行度就是: 300 了;


## 2.2 根据 任意字段 分区
```scala
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName(this.getClass.getName)

val spark = SparkSession.builder().config(conf).getOrCreate()

val prop = new Properties()
prop.put("user", "root")
prop.put("password", "123456")
prop.put("driver", "com.mysql.jdbc.Driver")

val predicates =
    Array(
      "2015-09-16" -> "2015-09-30",
      "2015-10-01" -> "2015-10-15",
      "2015-10-16" -> "2015-10-31",
      "2015-11-01" -> "2015-11-14",
      "2015-11-15" -> "2015-11-30",
      "2015-12-01" -> "2015-12-15"
    ).map {
      case (start, end) =>
        s"cast(time as date) >= date '$start' " + s"AND cast(time as date) <= date '$end'"
    }

val readDf = spark.read.jdbc("jdbc:mysql://***.***.***.**:3306/data_base?useUnicode=true&characterEncoding=utf-8&useSSL=false",
  "salej", predicates)

println("读取数据库的并行度是: "+readDf.rdd.partitions.size)
```
此时, Spark 的并行度就是: 6;

# 三、多并发读取指定量数据
```scala
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName(this.getClass.getName)

val spark = SparkSession.builder().config(conf).getOrCreate()

val prop = new Properties()
prop.put("user", "root")
prop.put("password", "123456")
prop.put("driver", "com.mysql.jdbc.Driver")
prop.put("partitionColumn", "seqid")
prop.put("numPartitions", "300")
prop.put("lowerBound", "0")
prop.put("upperBound", "200000")

val readDf = spark.read.jdbc("jdbc:mysql://***.***.***.**:3306/data_base?useUnicode=true&characterEncoding=utf-8&useSSL=false",
  "(select * from salej where id > 1)", prop)

println("读取数据库的并行度是: "+readDf.rdd.partitions.size)
```
此时, Spark 的并行度就是: 300, 并且每个分区的数据都是已经过滤了的;

此方法**可能**造成性能问题, 原因是因为 Spark 生成最终执行 sql 时, 产生了子查询;

# 四、源码解析
首先查看 Spark read jdbc 的方法
```scala
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName(this.getClass.getName)

val spark = SparkSession.builder().config(conf).getOrCreate()

val prop = new Properties()
prop.put("user", "root")
prop.put("password", "123456")
prop.put("driver", "com.mysql.jdbc.Driver")
prop.put("partitionColumn", "seqid")
prop.put("numPartitions", "300")
prop.put("lowerBound", "0")
prop.put("upperBound", "200000")

val readDf = spark.read.jdbc("jdbc:mysql://***.***.***.**:3306/data_base?useUnicode=true&characterEncoding=utf-8&useSSL=false",
  "(select * from salej where id > 1)", prop)
```
点进 read.jdbc 方法中:
```scala
def jdbc(url: String, table: String, properties: Properties): DataFrame = {
  assertNoSpecifiedSchema("jdbc")
  // properties should override settings in extraOptions.
  this.extraOptions ++= properties.asScala
  // explicit url and dbtable should override all
  this.extraOptions += (JDBCOptions.JDBC_URL -> url, JDBCOptions.JDBC_TABLE_NAME -> table)
  // 直接 load 加载
  format("jdbc").load()
}
```
进入 .load() 方法中(最终会进入重载的 `def load(paths: String*)` 方法中):
```scala
@scala.annotation.varargs
  def load(paths: String*): DataFrame = {
    if (source.toLowerCase(Locale.ROOT) == DDLUtils.HIVE_PROVIDER) {
      throw new AnalysisException("Hive data source can only be used with tables, you can not " +
        "read files of Hive data source directly.")
    }

    val cls = DataSource.lookupDataSource(source, sparkSession.sessionState.conf)
    if (classOf[DataSourceV2].isAssignableFrom(cls)) {
      val ds = cls.newInstance().asInstanceOf[DataSourceV2]
      if (ds.isInstanceOf[ReadSupport]) {
        val sessionOptions = DataSourceV2Utils.extractSessionConfigs(
          ds = ds, conf = sparkSession.sessionState.conf)
        val pathsOption = {
          val objectMapper = new ObjectMapper()
          DataSourceOptions.PATHS_KEY -> objectMapper.writeValueAsString(paths.toArray)
        }
        Dataset.ofRows(sparkSession, DataSourceV2Relation.create(
          ds, sessionOptions ++ extraOptions.toMap + pathsOption,
          userSpecifiedSchema = userSpecifiedSchema))
      } else {
        loadV1Source(paths: _*)
      }
    } else {
      // if 判断的结果是 false, 进入这里
      loadV1Source(paths: _*)
    }
  }
```
进入 loadV1Source 方法:
```scala
private def loadV1Source(paths: String*) = {
    // Code path for data source v1.
    sparkSession.baseRelationToDataFrame(
      DataSource.apply(
        sparkSession,
        paths = paths,
        userSpecifiedSchema = userSpecifiedSchema,
        className = source,
        options = extraOptions.toMap).resolveRelation())
  }
```
这里会拿到所有的信息, 执行伴生对象 DataSource 的 resolveRelation 方法:
```scala
def resolveRelation(checkFilesExist: Boolean = true): BaseRelation = {
    val relation = (providingClass.newInstance(), userSpecifiedSchema) match {
      // TODO: Throw when too much is given.
      case (dataSource: SchemaRelationProvider, Some(schema)) =>
        dataSource.createRelation(sparkSession.sqlContext, caseInsensitiveOptions, schema)
      case (dataSource: RelationProvider, None) =>
        dataSource.createRelation(sparkSession.sqlContext, caseInsensitiveOptions)
      case (_: SchemaRelationProvider, None) =>
        throw new AnalysisException(s"A schema needs to be specified when using $className.")
      case (dataSource: RelationProvider, Some(schema)) =>
        val baseRelation =
          dataSource.createRelation(sparkSession.sqlContext, caseInsensitiveOptions)
        if (baseRelation.schema != schema) {
          throw new AnalysisException(s"$className does not allow user-specified schemas.")
        }
        baseRelation

      // We are reading from the results of a streaming query. Load files from the metadata log
      // instead of listing them using HDFS APIs.
      case (format: FileFormat, _)
          if FileStreamSink.hasMetadata(
            caseInsensitiveOptions.get("path").toSeq ++ paths,
            sparkSession.sessionState.newHadoopConf()) =>
        val basePath = new Path((caseInsensitiveOptions.get("path").toSeq ++ paths).head)
        val fileCatalog = new MetadataLogFileIndex(sparkSession, basePath, userSpecifiedSchema)
        val dataSchema = userSpecifiedSchema.orElse {
          format.inferSchema(
            sparkSession,
            caseInsensitiveOptions,
            fileCatalog.allFiles())
        }.getOrElse {
          throw new AnalysisException(
            s"Unable to infer schema for $format at ${fileCatalog.allFiles().mkString(",")}. " +
                "It must be specified manually")
        }

        HadoopFsRelation(
          fileCatalog,
          partitionSchema = fileCatalog.partitionSchema,
          dataSchema = dataSchema,
          bucketSpec = None,
          format,
          caseInsensitiveOptions)(sparkSession)

      // This is a non-streaming file based datasource.
      case (format: FileFormat, _) =>
        val globbedPaths =
          checkAndGlobPathIfNecessary(checkEmptyGlobPath = true, checkFilesExist = checkFilesExist)
        val useCatalogFileIndex = sparkSession.sqlContext.conf.manageFilesourcePartitions &&
          catalogTable.isDefined && catalogTable.get.tracksPartitionsInCatalog &&
          catalogTable.get.partitionColumnNames.nonEmpty
        val (fileCatalog, dataSchema, partitionSchema) = if (useCatalogFileIndex) {
          val defaultTableSize = sparkSession.sessionState.conf.defaultSizeInBytes
          val index = new CatalogFileIndex(
            sparkSession,
            catalogTable.get,
            catalogTable.get.stats.map(_.sizeInBytes.toLong).getOrElse(defaultTableSize))
          (index, catalogTable.get.dataSchema, catalogTable.get.partitionSchema)
        } else {
          val index = createInMemoryFileIndex(globbedPaths)
          val (resultDataSchema, resultPartitionSchema) =
            getOrInferFileFormatSchema(format, Some(index))
          (index, resultDataSchema, resultPartitionSchema)
        }

        HadoopFsRelation(
          fileCatalog,
          partitionSchema = partitionSchema,
          dataSchema = dataSchema.asNullable,
          bucketSpec = bucketSpec,
          format,
          caseInsensitiveOptions)(sparkSession)

      case _ =>
        throw new AnalysisException(
          s"$className is not a valid Spark SQL Data Source.")
    }

    relation match {
      case hs: HadoopFsRelation =>
        SchemaUtils.checkColumnNameDuplication(
          hs.dataSchema.map(_.name),
          "in the data schema",
          equality)
        SchemaUtils.checkColumnNameDuplication(
          hs.partitionSchema.map(_.name),
          "in the partition schema",
          equality)
        DataSourceUtils.verifyReadSchema(hs.fileFormat, hs.dataSchema)
      case _ =>
        SchemaUtils.checkColumnNameDuplication(
          relation.schema.map(_.name),
          "in the data schema",
          equality)
    }

    relation
  }
```
这个方法分为两部分, 第一部分会首先进入 第二个 case 的方法, 注意, 这里的 dataSource 的实现类是: JdbcRelationProvider
```scala
override def createRelation(
      sqlContext: SQLContext,
      parameters: Map[String, String]): BaseRelation = {
    val jdbcOptions = new JDBCOptions(parameters)
    val resolver = sqlContext.conf.resolver
    val timeZoneId = sqlContext.conf.sessionLocalTimeZone
    val schema = JDBCRelation.getSchema(resolver, jdbcOptions)
    val parts = JDBCRelation.columnPartition(schema, resolver, timeZoneId, jdbcOptions)
    JDBCRelation(schema, parts, jdbcOptions)(sqlContext.sparkSession)
  }
```
spark 会在这里获取到源表的 schema 信息: `JDBCRelation.getSchema`, 最终被执行的sql是: 
```scala
def getSchema(resolver: Resolver, jdbcOptions: JDBCOptions): StructType = {
  val tableSchema = JDBCRDD.resolveTable(jdbcOptions)
  jdbcOptions.customSchema match {
    case Some(customSchema) => JdbcUtils.getCustomSchema(
      tableSchema, customSchema, resolver)
    case None => tableSchema
  }
}
```
进入 JDBCRDD.resolveTable, 最终返回 tableSchema 信息
```scala
def resolveTable(options: JDBCOptions): StructType = {
    val url = options.url
    val table = options.tableOrQuery
    val dialect = JdbcDialects.get(url)
    val conn: Connection = JdbcUtils.createConnectionFactory(options)()
    try {
      val statement = conn.prepareStatement(dialect.getSchemaQuery(table))
      try {
        statement.setQueryTimeout(options.queryTimeout)
        val rs = statement.executeQuery()
        try {
          JdbcUtils.getSchema(rs, dialect, alwaysNullable = true)
        } finally {
          rs.close()
        }
...
```
在这个方法中, 首先根据传入的 url 信息, 获取到MySQL 方言(dialect=JdbcDialect), 然后会生成一段sql进行查询, 然后再获取schema: `dialect.getSchemaQuery`
```scala
@Since("2.1.0")
def getSchemaQuery(table: String): String = {
  s"SELECT * FROM $table WHERE 1=0"
}
```
在拿到返回的结果集(rs) 后, 执行 `JdbcUtils.getSchema` 获取源表的 schema 信息
```bash
def getSchema(
      resultSet: ResultSet,
      dialect: JdbcDialect,
      alwaysNullable: Boolean = false): StructType = {
    val rsmd = resultSet.getMetaData
    val ncols = rsmd.getColumnCount
    val fields = new Array[StructField](ncols)
    var i = 0
    while (i < ncols) {
      // 挨个获取 column 的信息、类型等
      val columnName = rsmd.getColumnLabel(i + 1)
      val dataType = rsmd.getColumnType(i + 1)
      val typeName = rsmd.getColumnTypeName(i + 1)
      val fieldSize = rsmd.getPrecision(i + 1)
      val fieldScale = rsmd.getScale(i + 1)
      val isSigned = {
        try {
          rsmd.isSigned(i + 1)
        } catch {
          // Workaround for HIVE-14684:
          case e: SQLException if
          e.getMessage == "Method not supported" &&
            rsmd.getClass.getName == "org.apache.hive.jdbc.HiveResultSetMetaData" => true
        }
      }
      val nullable = if (alwaysNullable) {
        true
      } else {
        rsmd.isNullable(i + 1) != ResultSetMetaData.columnNoNulls
      }
      val metadata = new MetadataBuilder().putLong("scale", fieldScale)
      // 根据不同方言约定好的映射关系做映射, 如未找到则使用默认的规则映射
      // 现支持:
      //    AggregateDialect
      //    DB2Dialect
      //    DerbyDialect
      //    JdbcDialect
      //    MsSqlServerDialect
      //    MySQLDialect
      //    OracleDialect
      //    PostgresDialect
      val columnType =
        dialect.getCatalystType(dataType, typeName, fieldSize, metadata).getOrElse(
          getCatalystType(dataType, fieldSize, fieldScale, isSigned))
      fields(i) = StructField(columnName, columnType, nullable)
      i = i + 1
    }
    // 将中的结果封装到 StructType 中返回 
    new StructType(fields)
  }
```
看一下 MySQL 规则是什么样的: 
```scala
override def getCatalystType(
    sqlType: Int, typeName: String, size: Int, md: MetadataBuilder): Option[DataType] = {
  // MySQL 仅对 bit 和 VARBINARY 做了类型约束, 其他类型依旧使用 Spark 默认配置
  // 在读取数据时, 需要考虑 Spark 中已经存在的方言映射 是否会对 load 进来的数据造成影响, 比如精度丢失等
  if (sqlType == Types.VARBINARY && typeName.equals("BIT") && size != 1) {
    // This could instead be a BinaryType if we'd rather return bit-vectors of up to 64 bits as
    // byte arrays instead of longs.
    md.putLong("binarylong", 1)
    Option(LongType)
  } else if (sqlType == Types.BIT && typeName.equals("TINYINT")) {
    Option(BooleanType)
  } else None
}
```
Spark 中默认的映射规则如下: 
```scala
private def getCatalystType(
      sqlType: Int,
      precision: Int,
      scale: Int,
      signed: Boolean): DataType = {
    val answer = sqlType match {
      // scalastyle:off
      case java.sql.Types.ARRAY         => null
      case java.sql.Types.BIGINT        => if (signed) { LongType } else { DecimalType(20,0) }
      case java.sql.Types.BINARY        => BinaryType
      case java.sql.Types.BIT           => BooleanType // @see JdbcDialect for quirks
      case java.sql.Types.BLOB          => BinaryType
      case java.sql.Types.BOOLEAN       => BooleanType
      case java.sql.Types.CHAR          => StringType
      case java.sql.Types.CLOB          => StringType
      case java.sql.Types.DATALINK      => null
      case java.sql.Types.DATE          => DateType
      case java.sql.Types.DECIMAL
        if precision != 0 || scale != 0 => DecimalType.bounded(precision, scale)
      case java.sql.Types.DECIMAL       => DecimalType.SYSTEM_DEFAULT
      case java.sql.Types.DISTINCT      => null
      case java.sql.Types.DOUBLE        => DoubleType
      case java.sql.Types.FLOAT         => FloatType
      case java.sql.Types.INTEGER       => if (signed) { IntegerType } else { LongType }
      case java.sql.Types.JAVA_OBJECT   => null
      case java.sql.Types.LONGNVARCHAR  => StringType
      case java.sql.Types.LONGVARBINARY => BinaryType
      case java.sql.Types.LONGVARCHAR   => StringType
      case java.sql.Types.NCHAR         => StringType
      case java.sql.Types.NCLOB         => StringType
      case java.sql.Types.NULL          => null
      case java.sql.Types.NUMERIC
        if precision != 0 || scale != 0 => DecimalType.bounded(precision, scale)
      case java.sql.Types.NUMERIC       => DecimalType.SYSTEM_DEFAULT
      case java.sql.Types.NVARCHAR      => StringType
      case java.sql.Types.OTHER         => null
      case java.sql.Types.REAL          => DoubleType
      case java.sql.Types.REF           => StringType
      case java.sql.Types.REF_CURSOR    => null
      case java.sql.Types.ROWID         => LongType
      case java.sql.Types.SMALLINT      => IntegerType
      case java.sql.Types.SQLXML        => StringType
      case java.sql.Types.STRUCT        => StringType
      case java.sql.Types.TIME          => TimestampType
      case java.sql.Types.TIME_WITH_TIMEZONE
                                        => null
      case java.sql.Types.TIMESTAMP     => TimestampType
      case java.sql.Types.TIMESTAMP_WITH_TIMEZONE
                                        => null
      case java.sql.Types.TINYINT       => IntegerType
      case java.sql.Types.VARBINARY     => BinaryType
      case java.sql.Types.VARCHAR       => StringType
      case _                            =>
        throw new SQLException("Unrecognized SQL type " + sqlType)
      // scalastyle:on
    }

    if (answer == null) {
      throw new SQLException("Unsupported type " + JDBCType.valueOf(sqlType).getName)
    }
    answer
  }
```
在拿到 schema 信息之后, 就会生成 分区的信息(根据传入的分字段和上下界等):
```scala
override def createRelation(
      sqlContext: SQLContext,
      parameters: Map[String, String]): BaseRelation = {
    val jdbcOptions = new JDBCOptions(parameters)
    val resolver = sqlContext.conf.resolver
    val timeZoneId = sqlContext.conf.sessionLocalTimeZone
    val schema = JDBCRelation.getSchema(resolver, jdbcOptions)
    val parts = JDBCRelation.columnPartition(schema, resolver, timeZoneId, jdbcOptions)
    JDBCRelation(schema, parts, jdbcOptions)(sqlContext.sparkSession)
  }
```
进入: `JDBCRelation.columnPartition`
```scala
def columnPartition(
      schema: StructType,
      resolver: Resolver,
      timeZoneId: String,
      jdbcOptions: JDBCOptions): Array[Partition] = {
    val partitioning = {
      import JDBCOptions._

      val partitionColumn = jdbcOptions.partitionColumn
      val lowerBound = jdbcOptions.lowerBound
      val upperBound = jdbcOptions.upperBound
      val numPartitions = jdbcOptions.numPartitions

      if (partitionColumn.isEmpty) {
        assert(lowerBound.isEmpty && upperBound.isEmpty, "When 'partitionColumn' is not " +
          s"specified, '$JDBC_LOWER_BOUND' and '$JDBC_UPPER_BOUND' are expected to be empty")
        null
      } else {
        assert(lowerBound.nonEmpty && upperBound.nonEmpty && numPartitions.nonEmpty,
          s"When 'partitionColumn' is specified, '$JDBC_LOWER_BOUND', '$JDBC_UPPER_BOUND', and " +
            s"'$JDBC_NUM_PARTITIONS' are also required")

        val (column, columnType) = verifyAndGetNormalizedPartitionColumn(
          schema, partitionColumn.get, resolver, jdbcOptions)

        // 转换类型
        val lowerBoundValue = toInternalBoundValue(lowerBound.get, columnType)
        val upperBoundValue = toInternalBoundValue(upperBound.get, columnType)
        JDBCPartitioningInfo(
          column, columnType, lowerBoundValue, upperBoundValue, numPartitions.get)
      }
    }

    if (partitioning == null || partitioning.numPartitions <= 1 ||
      partitioning.lowerBound == partitioning.upperBound) {
      return Array[Partition](JDBCPartition(null, 0))
    }

    // 校验合法性
    val lowerBound = partitioning.lowerBound
    val upperBound = partitioning.upperBound
    require (lowerBound <= upperBound,
      "Operation not allowed: the lower bound of partitioning column is larger than the upper " +
      s"bound. Lower bound: $lowerBound; Upper bound: $upperBound")

    val boundValueToString: Long => String =
      toBoundValueInWhereClause(_, partitioning.columnType, timeZoneId)
      
    // 调整 分区数
    val numPartitions =
      if ((upperBound - lowerBound) >= partitioning.numPartitions || /* check for overflow */
          (upperBound - lowerBound) < 0) {
        partitioning.numPartitions
      } else {
        logWarning("The number of partitions is reduced because the specified number of " +
          "partitions is less than the difference between upper bound and lower bound. " +
          s"Updated number of partitions: ${upperBound - lowerBound}; Input number of " +
          s"partitions: ${partitioning.numPartitions}; " +
          s"Lower bound: ${boundValueToString(lowerBound)}; " +
          s"Upper bound: ${boundValueToString(upperBound)}.")
        upperBound - lowerBound
      }
    // Overflow and silliness can happen if you subtract then divide.
    // Here we get a little roundoff, but that's (hopefully) OK.
    // 计算步长
    val stride: Long = upperBound / numPartitions - lowerBound / numPartitions

    var i: Int = 0
    val column = partitioning.column
    var currentValue = lowerBound
    val ans = new ArrayBuffer[Partition]()
    // 根据步长、提供的最大、最小值做步长累计，确定边界后组装 where 查询条件
    while (i < numPartitions) {
      val lBoundValue = boundValueToString(currentValue)
      val lBound = if (i != 0) s"$column >= $lBoundValue" else null
      currentValue += stride
      val uBoundValue = boundValueToString(currentValue)
      val uBound = if (i != numPartitions - 1) s"$column < $uBoundValue" else null
      val whereClause =
        if (uBound == null) {
          lBound
        } else if (lBound == null) {
          s"$uBound or $column is null"
        } else {
          s"$lBound AND $uBound"
        }
      ans += JDBCPartition(whereClause, i)
      i = i + 1
    }
    val partitions = ans.toArray
    logInfo(s"Number of partitions: $numPartitions, WHERE clauses of these partitions: " +
      partitions.map(_.asInstanceOf[JDBCPartition].whereClause).mkString(", "))
    partitions
  }
```
这里得到的结果如下:
```scala
JDBCPartition(`seqid` < 6 or `seqid` is null,0)
JDBCPartition(`seqid` >= 6 AND `seqid` < 12,1)
JDBCPartition(`seqid` >= 12 AND `seqid` < 18,2)
JDBCPartition(`seqid` >= 18 AND `seqid` < 24,3)
JDBCPartition(`seqid` >= 24 AND `seqid` < 30,4)
JDBCPartition(`seqid` >= 30 AND `seqid` < 36,5)
JDBCPartition(`seqid` >= 36 AND `seqid` < 42,6)
JDBCPartition(`seqid` >= 42 AND `seqid` < 48,7)
...
JDBCPartition(`seqid` >= 1794,299)
```
注意: 此方式会产生数据倾斜的可能, 因为 第一个分区 和 最后一个分区 可能会查到更多的数据, 所以这里不建议由Spark动态生成, 而是采用自定义分区条件的方式(也就是 2.2 的方式)

当这一切都产生完成之后 也就代表着 JDBCRelation 生成完成, SparkSession 会把任务加入逻辑行计划中, 当遇到 action 操作的时候, 会转化为 物理执行计划.

```scala
override def buildScan(requiredColumns: Array[String], filters: Array[Filter]): RDD[Row] = {
  // Rely on a type erasure hack to pass RDD[InternalRow] back as RDD[Row]
  JDBCRDD.scanTable(
    sparkSession.sparkContext,
    schema,
    requiredColumns,
    filters,
    parts,
    jdbcOptions).asInstanceOf[RDD[Row]]
}
```
当 JDBCRelation 的 buildScan 执行, 会调用 `JDBCRDD.scanTable` 方法 创建新的 RDD, 在这中间会自动关联上之前的where 合并到 Spark 组装的 sql 语句中:
```scala
def scanTable(
    sc: SparkContext,
    schema: StructType,
    requiredColumns: Array[String],
    filters: Array[Filter],
    parts: Array[Partition],
    options: JDBCOptions): RDD[InternalRow] = {
  val url = options.url
  val dialect = JdbcDialects.get(url)
  val quotedColumns = requiredColumns.map(colName => dialect.quoteIdentifier(colName))
  new JDBCRDD(
    sc,
    JdbcUtils.createConnectionFactory(options),
    pruneSchema(schema, requiredColumns),
    quotedColumns,
    filters,
    parts,
    url,
    options)
}
```
在这里创建 JDBCRDD, 执行到 compute 方法:
```scala
private[jdbc] class JDBCRDD(
    sc: SparkContext,
    getConnection: () => Connection,
    schema: StructType,
    columns: Array[String],
    filters: Array[Filter],
    partitions: Array[Partition],
    url: String,
    options: JDBCOptions)
  extends RDD[InternalRow](sc, Nil) {

  /**
   * Retrieve the list of partitions corresponding to this RDD.
   */
  override def getPartitions: Array[Partition] = partitions

  /**
   * `columns`, but as a String suitable for injection into a SQL query.
   */
  private val columnList: String = {
    val sb = new StringBuilder()
    columns.foreach(x => sb.append(",").append(x))
    if (sb.isEmpty) "1" else sb.substring(1)
  }

  /**
   * `filters`, but as a WHERE clause suitable for injection into a SQL query.
   */
  private val filterWhereClause: String =
    filters
      .flatMap(JDBCRDD.compileFilter(_, JdbcDialects.get(url)))
      .map(p => s"($p)").mkString(" AND ")

  /**
   * A WHERE clause representing both `filters`, if any, and the current partition.
   */
  private def getWhereClause(part: JDBCPartition): String = {
    if (part.whereClause != null && filterWhereClause.length > 0) {
      "WHERE " + s"($filterWhereClause)" + " AND " + s"(${part.whereClause})"
    } else if (part.whereClause != null) {
      "WHERE " + part.whereClause
    } else if (filterWhereClause.length > 0) {
      "WHERE " + filterWhereClause
    } else {
      ""
    }
  }

  /**
   * Runs the SQL query against the JDBC driver.
   *
   */
    // 执行到这来, 该方法 获取 where 条件, 组装 sql, 通过jdbc 查询数据库
  override def compute(thePart: Partition, context: TaskContext): Iterator[InternalRow] = {
    var closed = false
    var rs: ResultSet = null
    var stmt: PreparedStatement = null
    var conn: Connection = null

    def close() {
      if (closed) return
      try {
        if (null != rs) {
          rs.close()
        }
      } catch {
        case e: Exception => logWarning("Exception closing resultset", e)
      }
      try {
        if (null != stmt) {
          stmt.close()
        }
      } catch {
        case e: Exception => logWarning("Exception closing statement", e)
      }
      try {
        if (null != conn) {
          if (!conn.isClosed && !conn.getAutoCommit) {
            try {
              conn.commit()
            } catch {
              case NonFatal(e) => logWarning("Exception committing transaction", e)
            }
          }
          conn.close()
        }
        logInfo("closed connection")
      } catch {
        case e: Exception => logWarning("Exception closing connection", e)
      }
      closed = true
    }

    context.addTaskCompletionListener[Unit]{ context => close() }

    val inputMetrics = context.taskMetrics().inputMetrics
    val part = thePart.asInstanceOf[JDBCPartition]
    conn = getConnection()
    val dialect = JdbcDialects.get(url)
    import scala.collection.JavaConverters._
    dialect.beforeFetch(conn, options.asProperties.asScala.toMap)

    // This executes a generic SQL statement (or PL/SQL block) before reading
    // the table/query via JDBC. Use this feature to initialize the database
    // session environment, e.g. for optimizations and/or troubleshooting.
    options.sessionInitStatement match {
      case Some(sql) =>
        val statement = conn.prepareStatement(sql)
        logInfo(s"Executing sessionInitStatement: $sql")
        try {
          statement.setQueryTimeout(options.queryTimeout)
          statement.execute()
        } finally {
          statement.close()
        }
      case None =>
    }

    // H2's JDBC driver does not support the setSchema() method.  We pass a
    // fully-qualified table name in the SELECT statement.  I don't know how to
    // talk about a table in a completely portable way.
    // 这里 获取 Where 条件(前面 Spark 根据 partitionColumn 等自动生成的 JDBCPartition )
    val myWhereClause = getWhereClause(part)
      
    // 在这里会生成最终被执行的 sql 语句, 例如我这里生成的是:
    // SELECT `sheetid`,`seqid`,`shopid`,`saledate`,`shiftid`,`time`,`reqtime`,`listno`,`sublistno`,`pos_id`,`cashier_id`,`waiter_id`,`vgno`,`goodsid`,`goodsno`,`placeno`,`groupno`,`deptno`,`amount`,`bu_code`,`dt` 
    // FROM (select * from salej where seqid > 1) as T 
    // WHERE `seqid` < 6 or `seqid` is null
    val sqlText = s"SELECT $columnList FROM ${options.tableOrQuery} $myWhereClause"
    stmt = conn.prepareStatement(sqlText,
        ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)
    stmt.setFetchSize(options.fetchSize)
    stmt.setQueryTimeout(options.queryTimeout)
    rs = stmt.executeQuery()
    val rowsIterator = JdbcUtils.resultSetToSparkInternalRows(rs, schema, inputMetrics)
    
    // 将最终的结果返回
    CompletionIterator[InternalRow, Iterator[InternalRow]](
      new InterruptibleIterator(context, rowsIterator), close())
  }
}
```

# 五、总结
1. 为了避免数据倾斜的可能, 尽可能的像 步骤2.2 中那样自定义分区;
2. 如果 dbtable 有加过滤条件, 会有子查询产生, 如果100亿条数据分成 10000个分区查询, 子查询会被重复执行 10000 次, 可能会有性能影响;
3. 因为 Spark 最终是启用多线程通过JDBC的方式查询数据库的, 所以 Spark 读取数据库 是可以走目标数据库创建的索引的, 如果可以, 就创建索引加快速度;