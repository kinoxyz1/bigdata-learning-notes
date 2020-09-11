
* [一、基础数据类型](#%E4%B8%80%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
* [二、 Java 和 Scala 元组](#%E4%BA%8C-java-%E5%92%8C-scala-%E5%85%83%E7%BB%84)
* [三、Scala 样例类](#%E4%B8%89scala-%E6%A0%B7%E4%BE%8B%E7%B1%BB)
* [四、Java简单对象](#%E5%9B%9Bjava%E7%AE%80%E5%8D%95%E5%AF%B9%E8%B1%A1)
* [五、其他](#%E4%BA%94%E5%85%B6%E4%BB%96)

---

Flink 流应用程序处理的是 以数据对象表示的事件流. 所以在 Flink 内部, 我们能够处理这些对象. 它们需要被序列化和反序列化, 以便通过网络传送它们; 或者从状态后端、检查点 和 保存点读取它们. 为了有效的做到这一点, Flink 需要明确的知道应用程序所处理的数据类型. Flink 使用类型信息的概念来表示数据类型, 并为每个数据类型生成特定的序列化器、反序列化器和比较器.

Flink 还具有一个类型提取器, 该系统分析函数的输入和返回类, 以自动获取类型信息, 从而获得序列化器和反序列化器. 但是, 在某些情况下, 例如 lambda 函数或泛型类型, 需要显示地提供类型信息, 才能使应用程序正常的工作或提高其性能.

Flink 支持 Java 和 Scala 中所有常见的数据类型. 使用最广泛的类型有以下几种.

# 一、基础数据类型

Flink 支持所有的 Java 和 Scala 基础数据类型, 包括: Int、Double、Long、String....
```scala 3
val numbers: DataStream[Long] = env.fromElements(1L, 2L, 3L, 4L)
numbers.map( x => x + 1)
```

# 二、 Java 和 Scala 元组
```scala 3
val persons: DataStream[(String, Integer)] = env.fromElements(("kino", 17), ("kino1", 20))
persons.filter(x => x._2 > 18)
```

# 三、Scala 样例类
```scala 3
case class Person(name: String, age: Int)
val persons: DataStream[(String, Integer)] = env.fromElements(
  Persion("Kino1", 17), Persion("Kino2", 20))
persons.filter(x => x.age > 18)
```

# 四、Java简单对象
```java
public class Person {
  public String name;
  public int age;
  public Person() {}
  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }
}
DataStream<Person> persons = env.fromElements(
    new Person("Alex", 42),
    new Person("Wendy", 23));
```

# 五、其他
包括: Arrays, Lists, Maps, Enums.....

Flink 对 Java 和 Scala 中的一些特殊目的的类型也都是支持的, 比如 Java 的 ArrayList、HashMap、Enum等等.