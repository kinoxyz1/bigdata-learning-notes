


----

Redis是开源的高性能内存Key-Value(键值对)数据库, 可以提供事务和持久化支持

key是String类型

value有五种数据类型, 分别是: String、Hash、Set、Zset 以及 list

# 一、String(字符串)
string是redis最基本的类型, 可以理解成与 Memcached 一模一样的类型，一个 key 对应一个 value, 例如: `hello: world`。
 
string类型是**二进制安全的**, 意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
 
string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M

# 二、Hash(哈希)
Redis hash 是一个键值对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

类似Java里面的Map<String,Object>

# 三、List(列表)
Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。

它的底层实际是个链表



# 四、Set(集合)
Redis的Set是string类型的无序集合。它是通过HashTable实现实现的，


# 五、Zset(有序集合)
Redis zset 和 set 一样也是string类型元素的集合, 且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。

redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。


