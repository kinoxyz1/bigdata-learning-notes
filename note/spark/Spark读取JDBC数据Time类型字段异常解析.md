






---
# 一、问题来源
使用 Spark 读取 MySQL 数据 存入 Hive 时, 突然发现有一个字段值被加上了: `1970-01-01`, 可是自己根本没有做过这种操作, 检查发现这个字段是 Time 类型(只有时分秒)的, 于是直接在读取出 MySQL 数据后打印输出, 发现问题就已经产生了:
```scala
+---+-----+----------+-------------------+---------------------+---------------------+
|id1|name1|t1        |t2                 |t3                   |t4                   |
+---+-----+----------+-------------------+---------------------+---------------------+
|1  |test |2021-03-25|1970-01-01 17:16:10|2021-03-31 17:16:26.0|2021-03-31 17:16:33.0|
|2  |test2|2021-03-03|1970-01-01 18:16:12|2021-03-31 17:16:27.0|2021-03-31 17:16:35.0|
|3  |111  |2021-03-18|1970-01-01 19:16:15|2021-03-31 17:16:29.0|2021-03-31 17:16:36.0|
|5  |4    |2021-03-21|1970-01-01 20:16:18|2021-03-31 17:16:30.0|2021-03-31 17:16:38.0|
|6  |6    |2021-03-12|1970-01-01 21:16:21|2021-03-31 17:16:32.0|2021-03-31 17:16:39.0|
+---+-----+----------+--------+---------------------+--------------------------------+
```

正好在之前优化 Spark 读取 JDBC 的数据源看过 Spark 的源码, 于是再去看看源码到底是哪里出了问题

# 二、源码分析
在 上一篇 * [Spark 读取JDBC数据源优化和源码解析](note/spark/Spark读取JDBC数据源优化.md) 文章中, 已经分析得到 Spark 是如何获取 schema 信息的, 即: 
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
在 `JDBCRDD.resolveTable` 方法返回的 schema 信息, 也就是在这里, 决定了 Spark DataType 和 JDBCType 的映射关系

进入 `JDBCRDD.resolveTable`  方法
```scala
def resolveTable(options: JDBCOptions): StructType = {
    val url = options.url
    val table = options.tableOrQuery
    // 这里产生的 dialect 就是 MySQLDialect
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
在拿到返回的结果集(rs) 后, 执行 JdbcUtils.getSchema 获取源表的 schema 信息

```scala
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
      // 这里就开始得到 源表每个字段的类型了
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
在 `dialect.getCatalystType` 方法中, 就开始得到 每个字段的类型, 如果不存在, 则根据 Spark 提供的 默认规则(即: getCatalystType()) 产生 源和目标 的字段类型映射

Spark 默认的数据类型映射如下:
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
在这里可以看到, 当源表的字段类型是: `java.sql.Types.TIME` 类型时, 被转换成了 Spark 的 `TimestampType` 类型, Time 类型是没有年月日的, 而 `TimestampType` 是 `年月日时分秒` 的, 所以结论就是 Spark 运行在这里的时候, 问题就产生了, 类型一确定, 加载数据就产生了 `1970-01-01`


# 三、解决问题
Spark 不仅定义了一组 通用的 `JDBCType` 转 `Spark DataType` 的规则, 而且还分别对每个不同数据库的特殊类型字段做出了处理, 例如 MySQL 的特殊转换方法就是实现了 `JdbcDialect` 抽象类的 `MySQLDialect`
```scala
private case object MySQLDialect extends JdbcDialect {

  override def canHandle(url : String): Boolean = url.startsWith("jdbc:mysql")

  override def getCatalystType(
      sqlType: Int, typeName: String, size: Int, md: MetadataBuilder): Option[DataType] = {
    if (sqlType == Types.VARBINARY && typeName.equals("BIT") && size != 1) {
      // This could instead be a BinaryType if we'd rather return bit-vectors of up to 64 bits as
      // byte arrays instead of longs.
      md.putLong("binarylong", 1)
      Option(LongType)
    } else if (sqlType == Types.BIT && typeName.equals("TINYINT")) {
      Option(BooleanType)
    } else None
  }

  override def quoteIdentifier(colName: String): String = {
    s"`$colName`"
  }

  override def getTableExistsQuery(table: String): String = {
    s"SELECT 1 FROM $table LIMIT 1"
  }

  override def isCascadingTruncateTable(): Option[Boolean] = Some(false)
}
```
但是在这个方法中, 仅仅对 `BIT` 和 `VARBINARY` 两个类型的字段做出了特殊处理, 想要解决 `Time` 被转成 `TIMESTAMP` 的问题, 可以实现 自定义的 `JdbcDialect` 方法

在 官方的 `JdbcDialect` 方法中, 仅仅实现了几个方法:
```scala
// 判断该JdbcDialect 实例是否能够处理该jdbc url；
def canHandle(url : String)

// 输入数据库中的SQLType，得到对应的Spark DataType的mapping关系；
def getCatalystType

// 输入Spark 的DataType，得到对应的数据库的SQLType；
def getJDBCType

// 引用标识符，用来放置某些字段名用了数据库的保留字（有些用户会使用数据库的保留字作为列名）；
def quoteIdentifier


// 剩下的都是一些其他方法
....
```
该方法还有一个 伴生对象, 也有几个方法
```scala
// 根据database的url获取JdbcDialect 对象；
def get(url: String)

// 将已注册的 JdbcDialect 注销；
def unregisterDialect

// 注册一个JdbcDialect。
def registerDialect
```

所以我们处理的大致步骤是:
1. 实现 `JdbcDialect` 方法
2. 注销当前的 `JdbcDialect`
3. 创建自己的 `JdbcDialect` 方法
4. 重写方法
5. 注册自己的 `JdbcDialect`

```scala
package io.github.interestinglab.waterdrop.spark.dialect

import org.apache.spark.sql.jdbc.{JdbcDialect, JdbcDialects, JdbcType}
import org.apache.spark.sql.types._

import java.sql.Types

/**
 * @description: 自定义 MySQL Dialect 类, 扩展 Spark 对 MySQL 不支持的数据类型
 * @author: Kino
 * @create: 2021-03-31 19:55:33
 */
object MyDialect {
  def useMyJdbcDialect(jdbcUrl: String): Unit = {
    // 将当前的 JdbcDialect 对象unregistered掉
    JdbcDialects.unregisterDialect(JdbcDialects.get(jdbcUrl))

    if (jdbcUrl.contains("jdbc:mysql")) {
      JdbcDialects.registerDialect(new customMySqlJdbcDialect)
    } else if (jdbcUrl.contains("jdbc:postgresql")) {

    } else if (jdbcUrl.contains("jdbc:sqlserver")) {

    } else if (jdbcUrl.contains("jdbc:oracle")) {

    } else if (jdbcUrl.contains("jdbc:informix")) {

    }
  }
}

class customMySqlJdbcDialect extends JdbcDialect {
  override def canHandle(url: String): Boolean = url.startsWith("jdbc:mysql")

  /**
   * 从 MySQL 读取数据到 Spark 的数据类型转换映射.
   *
   * @param sqlType
   * @param typeName
   * @param size
   * @param md
   * @return
   */
  override def getCatalystType(sqlType: Int, typeName: String, size: Int, md: MetadataBuilder): Option[DataType] = {
    if (sqlType == Types.VARBINARY && typeName.equals("BIT") && size != 1) {
      md.putLong("binarylong", 1)
      Option(LongType)
    } else if (sqlType == Types.BIT && typeName.equals("TINYINT")) {
      Option(BooleanType)
    } else if (sqlType == Types.TIMESTAMP || sqlType == Types.DATE || sqlType == -101 || sqlType == -102) {
      // 将不支持的 Timestamp with local Timezone 以TimestampType形式返回
      // Some(TimestampType)
      Some(StringType)
    } else if (sqlType == Types.BLOB) {
      Some(BinaryType)
    } else {
      Some(StringType)
    }
  }

  /**
   * 从 Spark(DataType) 到 MySQL(SQLType) 的数据类型映射
   *
   * @param dt
   * @return
   */
  override def getJDBCType(dt: DataType): Option[JdbcType] = {
    dt match {
      case IntegerType => Option(JdbcType("INTEGER", java.sql.Types.INTEGER))
      case LongType => Option(JdbcType("BIGINT", java.sql.Types.BIGINT))
      case DoubleType => Option(JdbcType("DOUBLE PRECISION", java.sql.Types.DOUBLE))
      case FloatType => Option(JdbcType("REAL", java.sql.Types.FLOAT))
      case ShortType => Option(JdbcType("INTEGER", java.sql.Types.SMALLINT))
      case ByteType => Option(JdbcType("BYTE", java.sql.Types.TINYINT))
      case BooleanType => Option(JdbcType("BIT(1)", java.sql.Types.BIT))
      case StringType => Option(JdbcType("TEXT", java.sql.Types.CLOB))
      case BinaryType => Option(JdbcType("BLOB", java.sql.Types.BLOB))
      case TimestampType => Option(JdbcType("TIMESTAMP", java.sql.Types.TIMESTAMP))
      case DateType => Option(JdbcType("DATE", java.sql.Types.DATE))
      case t: DecimalType => Option(JdbcType(s"DECIMAL(${t.precision},${t.scale})", java.sql.Types.DECIMAL))
      case _ => None
    }
  }
}
```

在 Spark 读取数据之前, 将上面的代码引用进去, 即可解决问题:
```scala
MyDialect.useMyJdbcDialect(conf.getString("url"))
val reader = sparkSession.read
  .format("jdbc")
  .option("url", config.getString("url"))
  .option("dbtable", config.getString("table"))
  .option("user", config.getString("user"))
  .option("password", config.getString("password"))
  .option("driver", driver)
  .load()
```
再次打印输出
```scala
+---+-----+----------+--------+---------------------+---------------------+
|id1|name1|t1        |t2      |t3                   |t4                   |
+---+-----+----------+--------+---------------------+---------------------+
|1  |test |2021-03-25|17:16:10|2021-03-31 17:16:26.0|2021-03-31 17:16:33.0|
|2  |test2|2021-03-03|18:16:12|2021-03-31 17:16:27.0|2021-03-31 17:16:35.0|
|3  |111  |2021-03-18|19:16:15|2021-03-31 17:16:29.0|2021-03-31 17:16:36.0|
|5  |4    |2021-03-21|20:16:18|2021-03-31 17:16:30.0|2021-03-31 17:16:38.0|
|6  |6    |2021-03-12|21:16:21|2021-03-31 17:16:32.0|2021-03-31 17:16:39.0|
+---+-----+----------+--------+---------------------+---------------------+
```
问题被成功解决