

# 常用的数据类型
- 整型
- 浮点型
- 布尔型
- 字符串
- 枚举类型
- 数组
- 元祖
- Date

# 官方其他数据类型
https://clickhouse.tech/docs/en/sql-reference/data-types/uuid/

# 一、整型
https://clickhouse.tech/docs/en/sql-reference/data-types/int-uint/

固定长度的整型, 包括 `有符号整型` 或 `无符号整型` 。

`整型范围`(`-2n-1~2n-1-1`):

Int8 - [-128 : 127]

Int16 - [-32768 : 32767]

Int32 - [-2147483648 : 2147483647]

Int64 - [-9223372036854775808 : 9223372036854775807]

`无符号整型范围`(`0~2n-1`):

UInt8 - [0 : 255]

UInt16 - [0 : 65535]

UInt32 - [0 : 4294967295]

UInt64 - [0 : 18446744073709551615]

# 二、浮点型
`Float32` - `float`

`Float64` – `double`

建议尽可能以整数形式存储数据。例如: 将固定精度的数字转换为整数值, 如时间用毫秒为单位表示, 因为浮点型进行计算时可能引起四舍五入的误差。
```sql
:) select 1-0.9
┌───────minus(1, 0.9)─┐
│ 0.09999999999999998 │
└─────────────────────┘

1 rows in set. Elapsed: 0.011 sec. 
```
与标准SQL相比，ClickHouse 支持以下类别的浮点数:

Inf: 正无穷
```sql
:) select 1/0
┌─divide(1, 0)─┐
│          inf │
└──────────────┘
```
-Inf: 负无穷
```sql
:) select -1/0
┌─divide(1, 0)─┐
│          -inf │
└──────────────┘
```
NaN: 非数字
```mysql
:) select 0/0
┌─divide(0, 0)─┐
│          nan │
└──────────────┘
```

# 三、布尔型
没有单独的类型来存储布尔值。可以使用 UInt8 类型，取值限制为 0 或 1。

# 四、字符串
### ① String
字符串可以任意长度的。它可以包含任意的字节集，包含空字节。

String类型替换了其他DBMS中的VARCHAR，BLOB，CLOB和其他类型。

### ②FixedString(N)
固定长度 `N` 的字符串，`N` 必须是严格的 `正自然数`。当服务端读取长度`小于 N` 的字符串时候，通过在字符串末尾`添加空字节` 来达到 `N` 字节长度。 当服务端读取长度`大于 N` 的字符串时候，将返回错误消息。

与 `String` 相比，极少会使用 `FixedString`, 因为使用起来不是很方便。

## 五、枚举类型
包括 Enum8 和 Enum16 类型。Enum 保存 `'string'= integer` 的对应关系。

Enum8 用 'String'= Int8`[-128, 127]` 对描述。

Enum16 用 'String'= Int16`[-32768, 32767]` 对描述。

使用范例:

创建一个带有一个枚举 Enum8('hello' = 1, 'world' = 2) 类型的列：
```sql
CREATE TABLE t_enum
(
    x Enum8('hello' = 1, 'world' = 2)
)
ENGINE = TinyLog
```
这个 x 列只能存储类型定义中列出的值：'hello'或'world'。如果尝试保存任何其他值，ClickHouse 抛出异常。
```sql
:) INSERT INTO t_enum VALUES ('hello'), ('world'), ('hello')

INSERT INTO t_enum VALUES

Ok.

3 rows in set. Elapsed: 0.002 sec.

:) insert into t_enum values('a')

INSERT INTO t_enum VALUES


Exception on client:
Code: 49. DB::Exception: Unknown element 'a' for type Enum8('hello' = 1, 'world' = 2)
```
从表中查询数据时，ClickHouse 从 Enum 中输出字符串值。
```sql
SELECT * FROM t_enum

┌─x─────┐
│ hello │
│ world │
│ hello │
└───────┘
```
如果需要看到对应行的数值，则必须将 Enum 值转换为整数类型。
```sql
SELECT CAST(x, 'Int8') FROM t_enum

┌─CAST(x, 'Int8')─┐
│               1 │
│               2 │
│               1 │
```
└─────────────────┘

# 六、数组
Array(T): 由 T 类型元素组成的数组。

T 可以是任意类型，包含数组类型。 但不推荐使用多维数组，ClickHouse 对多维数组的支持有限。例如，不能在 MergeTree 表中存储多维数组。

可以使用array函数来创建数组：
```java
array(T)
```
也可以使用方括号：
```java
[]
```
创建数组案例：
```sql
:) SELECT array(1, 2) AS x, toTypeName(x)

SELECT
    [1, 2] AS x,
    toTypeName(x)

┌─x─────┬─toTypeName(array(1, 2))─┐
│ [1,2] │ Array(UInt8)            │
└───────┴─────────────────────────┘

1 rows in set. Elapsed: 0.002 sec.

:) SELECT [1, 2] AS x, toTypeName(x)

SELECT
    [1, 2] AS x,
    toTypeName(x)

┌─x─────┬─toTypeName([1, 2])─┐
│ [1,2] │ Array(UInt8)       │
└───────┴────────────────────┘

1 rows in set. Elapsed: 0.002 sec.
```
# 七、元组
Tuple(T1, T2, ...)：元组，其中每个元素都有单独的类型。

创建元组的示例：
```sql
:) SELECT tuple(1,'a') AS x, toTypeName(x)

SELECT
    (1, 'a') AS x,
    toTypeName(x)

┌─x───────┬─toTypeName(tuple(1, 'a'))─┐
│ (1,'a') │ Tuple(UInt8, String)      │
└─────────┴───────────────────────────┘

1 rows in set. Elapsed: 0.021 sec.
```
# 八、Date
日期类型，用两个字节存储，表示从 1970-01-01 (无符号) 到当前的日期值。

还有很多数据结构，可以参考官方文档：https://clickhouse.yandex/docs/zh/data_types/

