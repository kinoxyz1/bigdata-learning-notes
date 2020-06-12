* [sorted](#sorted)
* [sortBy](#sortby)
* [sortWith](#sortwith)

---

排序的本质, 就是比较元素的大小

数字类型的可以直接进行比较, 对象类型的, 需要自定义比较规则

Java 中对对象进行排序, 需要继承/实现 Comparable、comparator, 重写对应方法

scala 的排序支持 3 中算子:
- sorted
- sortBy
- sortWith
---
# sorted
**sorted: 对 List<对象> 进行 sorted 的第一种写法**

**实现过程**：对象 继承 Ordered, 并实现 compare 方法, list.sorted 即可实现排序
```scala
object SortDemo1 {
    /**
      * 排序的本质, 就是比较元素的大小
      *     数字类型的可以直接进行比较, 对象类型的, 需要自定义比较规则
      *     Java 中对对象进行排序, 需要继承/实现 Comparable、comparator, 重写对应方法
      *
      * scala 的排序支持 3 中算子:
      *     1. sorted
      *         和 Java 中的 Comparable、comparator 一样,
      *         需要重写 compareTo(other)、compare(o1,o2)
      *
      *     2. sortBy
      *         开发中最常用的
      *
      *     3. sortWith
      *
      * sorted: 对 List<对象> 进行 sorted 的第一种写法:
      *     对象 继承 Ordered, 并实现 compare 方法, list.sorted 即可实现排序
      */
    def main(args: Array[String]): Unit = {
        val list1: ArrayBuffer[Int] = ArrayBuffer(30, 50, 70, 60, 10, 20)
        println(s"list1.sorted: ${list1.sorted}")

        println(new User("kino", 18) > new User("jerry", 10))

        val users = List(new User("kino", 18), new User("jerry", 10),
            new User("tom", 12), new User("sam", 38))
        println(s"users: {${users.sorted.mkString("},{ ")}}")
    }
}

class User(val name: String, val age: Int) extends Ordered[User]{
    override def toString: String = s"name: ${name}, age: ${age}"

    override def compare(o: User): Int = {
        var r = this.age - o.age
        if(r == 0) r = this.name.compareTo(o.name)
        r
    }
}
```


**sorted: 对 List<对象> 进行 sorted 的第二种写法**

**实现过程**：list.sorted(实现 Ordering[T]{重写 compare}) 即可实现排序
```scala
object SortDemo2 {
    /**
      * 排序的本质, 就是比较元素的大小
      *     数字类型的可以直接进行比较, 对象类型的, 需要自定义比较规则
      *     Java 中对对象进行排序, 需要继承/实现 Comparable、comparator, 重写对应方法
      *
      * scala 的排序支持 3 中算子:
      *     1. sorted
      *         和 Java 中的 Comparable、comparator 一样,
      *         需要重写 compareTo(other)、compare(o1,o2)
      *
      *     2. sortBy
      *         开发中最常用的
      *
      *     3. sortWith
      *
      * sorted: 对 List<对象> 进行 sorted 的第二种写法:
      *     list.sorted(实现 Ordering[T]{重写 compare}) 即可实现排序
      */
    def main(args: Array[String]): Unit = {
        val users = List(new User1(20, "lisi"),
                        new User1(10, "zs"),
                        new User1(15, "wangwu"),
                        new User1(15, "abc"))
        val users1 = users.sorted(new Ordering[User1]{
            override def compare(x: User1, y: User1): Int = {
                x.age - y.age
            }
        })
        println(s"users1: ${users1}")
    }
}
class User1(val age: Int, val name: String) {
    override def toString: String = s"name: $name, age: $age"
}
```

---
# sortBy
`list.sortBy[B](f: A => B)(implicit ord: Ordering[B]): Repr = sorted(ord on f)`

```scala
object SortDemo3 {
    /**
      * 排序的本质, 就是比较元素的大小
      *     数字类型的可以直接进行比较, 对象类型的, 需要自定义比较规则
      *     Java 中对对象进行排序, 需要继承/实现 Comparable、comparator, 重写对应方法
      *
      * scala 的排序支持 3 中算子:
      *     1. sorted
      *         和 Java 中的 Comparable、comparator 一样,
      *         需要重写 compareTo(other)、compare(o1,o2)
      *
      *     2. sortBy
      *         开发中最常用的
      *
      *     3. sortWith
      *
      * sortBy: 对 List<对象> 进行 sortBy 的写法:
      *     list.sortBy[B](f: A => B)(implicit ord: Ordering[B]): Repr = sorted(ord on f)
      */
    def main(args: Array[String]): Unit = {
        val users = List(new User1(20, "lisi"),
                new User1(10, "zs"),
                new User1(15, "wangwu"),
                new User1(15, "abc"))
        //按照年龄排序
        println(s"按照年龄排序: {${users.sortBy(user => user.age).mkString("}, {")}}")
        //按照年龄降序
        println(s"按照年龄排序: {${users.sortBy(user => user.age)(Ordering.Int.reverse).mkString("}, {")}}")
        //按照年龄升序, 名字升序 排序
        val userList1: List[User1] = users.sortBy(user => (user.age, user.name))(Ordering.Tuple2(Ordering.Int, Ordering.String))
        println(s"按照年龄升序, 名字升序 排序: {${userList1.mkString("}{")}}")
        //按照年龄升序, 名字降序 排序
        val userList2: List[User1] = users.sortBy(user => (user.age, user.name))(Ordering.Tuple2(Ordering.Int, Ordering.String.reverse))
        println(s"按照年龄升序, 名字降序 排序: {${userList2.mkString("}{")}}")

    }
}
```

--- 
# sortWith
`sortWith(lt: (A, A) => Boolean): Repr = sorted(Ordering fromLessThan lt)`

```scala
def main(args: Array[String]): Unit = {
    val list1 = List(30, 50, 70, 60, 10, 20)
    val list2: List[Int] = list1.sortWith((x, y) => x > y)
    println(s"list2: ${list2}")
}
```