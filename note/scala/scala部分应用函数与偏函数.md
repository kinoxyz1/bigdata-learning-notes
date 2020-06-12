* [一、部分应用函数](#%E4%B8%80%E9%83%A8%E5%88%86%E5%BA%94%E7%94%A8%E5%87%BD%E6%95%B0)
* [二、偏函数](#%E4%BA%8C%E5%81%8F%E5%87%BD%E6%95%B0)


---
# 一、部分应用函数
**部分应用函数：** 是指一个函数有N个参数, 而我们为其提供少于N个参数, 那就得到了一个部分应用函数. 

```scala
def sum(a:Int,b:Int,c:Int) = a + b + c; 
```
那么就可以从这个函数衍生出一个偏函数是这样的:
```scala
def p_sum = sum(1, _:Int, _:Int)
```
于是就可以这样调用p_sum(2,3), 相当于调用sum(1,2,3) 得到的结果是6. 这里的两个_分别对应函数sum对应位置的参数. 所以你也可以定义成
```scala
def p_sum = sum (_:Int, 1, _:Int) 
```

----
# 二、偏函数
**偏函数定义：** 用一对大括号括起来的一系列的 case 语句

偏函数的引出，先看一段需求案例： **`将 List(1, 2, 3, "abc", false) 的非数字 忽略, 将数字 + 1 并返回`**

```python
def main(args: Array[String]): Unit = {
	// 解决方式一: filter + map, 可以解决问题, 但是麻烦
    val list = List(1, 2, 3, "abc", false)

    //选过滤, 在 map
    val list1 = list.filter(x => x.isInstanceOf[Int]) //先过滤 Int 类型的参数
        .map(x => x.asInstanceOf[Int]) // 将过滤过来的值, 转成 Int 类型
        .map(x => x + 1) // 将 转换好的值 + 1
    println(s"filter + map 的方式: ${list1.mkString(", ")}")

    //解决方式二: 模式匹配, 比上面的方法简单, 但是不够完美
    val list2: List[Int] = list.filter(x => x.isInstanceOf[Int]).map(x => {
        x match {
            case x: Int => x + 1
        }
    })
    println(s"模式匹配的方式: ${list2.mkString(", ")}")
}
```
运行结果：
```python
filter + map 的方式: 2, 3, 4
模式匹配的方式: 2, 3, 4
```

上面的两种解决方式比较麻烦，我们**使用偏函数**解决上述需求：
```python
//1. 定义一个偏函数
//2. PartialFunction[Any, Int]: 表示偏函数接受的参数类型是 Any, 返回类型 Int
//3. isDefinedAt(x: Any): 如果返回 true, 就会调用 apply 返回结果, 如果返回 false, 过滤
//4. apply 构造器, 对传入的值 + 1, 并返回(新的集合)
val partialFun: PartialFunction[Any, Int] = new PartialFunction[Any, Int] {
    override def isDefinedAt(x: Any): Boolean = {
        println(s"x=$x")
        x.isInstanceOf[Int]
    }

    override def apply(v1: Any): Int = {
        println(s"v=$v1")
        v1.asInstanceOf[Int] + 1
    }
}

//使用偏函数
//如果使用偏函数, 则不能使用 map, 应该使用 collect
val list3: List[Int] = list.collect(partialFun)
println(s"list3: ${list3.mkString(", ")}")
```

运行结果：
```scala
x=1
v=1
x=2
v=2
x=3
v=3
x=abc
x=false
list3: 2, 3, 4
```

**偏函数的简写**
```scala
def partialFun2: PartialFunction[Any, Int] = {
    case i: Int => i + 1
    case j: Double => (j * 2).toInt
}
val list4: List[Int] = list.collect(partialFun2)
println(s"list4: ${list4.mkString(", ")}")
```
运行结果：
```scala
list4: 2, 3, 4
```

**偏函数的简写再简写**
```python
//再简写
val list5= list.collect({
    case i: Int => i + 1
    case j: Double => (j * 2).toInt
    case k: Float => (k * 3).toInt
})
println(s"list5: ${list5.mkString(", ")}")
```
运行结果：
```scala
list5: 2, 3, 4
```


下面看两个偏函数的实例：

偏函数的应用: `将 集合中的数字 * 2`
```scala
//因为 map 只能调整数据类型, 无法改变 集合 的长度, 所以结果会有空值
val list1 = List(1,2,4,"abc", false)
val list2: List[AnyVal] = list1.map({
    case a: Int => a * a
    case _ =>
})
println(s"list2: ${list2.mkString(",")}") // list2: 1,4,16,(),()

//collect => filter + map 能做到 过滤 和 map 的组合操作
val list3: List[Int] = list1.collect({
    case a: Int => a * a
})
println(s"list3: ${list3.mkString(", ")}") // list3: 1, 4, 16
```

偏函数的应用: `将 Map 的数据重组, 并将新的 Map 返回出来`
```scala
val map: Map[Int, (Int, Int)] = Map(1 -> (2, 20), 10 -> (20, 30), 20 -> (30, 40))
val map1 = map.map({
	//该 _ 是部分应用函数的表现
    case (k, (_, v)) => (k, v)
})
println(s"map1: (${map1.mkString("), (")})") // map1: (1 -> 20), (10 -> 30), (20 -> 40)
```

**偏函数**零碎代码：
```python
/**
 * 如果你想定义一个函数, 而让它只接受和处理其参数定义域范围内的子集,
 * 对于这个参数范围外的参数则抛出异常, 这样的函数就是偏函数(这个函数只处理传入来的部分参数)
 */
def main(args: Array[String]): Unit = {
   //定义一个普通的除法函数
   //val divide = (x: Int) => 100 / x
   /**
     * 当将 0 作为参数传入时会报错,
     * Java 处理该异常的办法:
     *     1. 使用 try/catch 来捕捉异常
     *     2. 对参数进行判断, 看是否等于 0
     *
     * Java 中的处理办法, 在 Scala 的 偏函数中已经封装好了, 看后面的偏函数
     */
   //divide(0) // java.lang.ArithmeticException: / by zero

   //定义一个部分应用函数
   val divide1 = new PartialFunction[Int, Int] {
       //false: 抛出异常, true: 调用 apply 得到结果
       override def isDefinedAt(x: Int): Boolean = x != 0

       override def apply(x: Int): Int = 100 / x
   }

   //上面的偏函数定义麻烦, 偏函数与 Scala 语句结合起来能使代码更简洁, 如下
   val divide2: PartialFunction[Int, Int] = {
       //功能和上面的代码一样, 但是比上面的更加简洁、方便
       case d: Int if d != 0 => 100 /d
   }

   //println("没有用偏函数的部分应用函数: "+divide1(10))
   //println("用了偏函数的部分应用函数: "+divide2(10))

   val rs: PartialFunction[Int, String] = {
       case 1 => "One"
       case 2 => "Two"
       case _ => "Other"
   }
   //println(rs(1))

   //OrElse: 将多个偏函数组合起来使用, 结合起来的效果类似 case 语句
   //          但是每个偏函数里又可以再使用case
   val or1 : PartialFunction[Int, String] = {case 1 => "One"}
   val or2 : PartialFunction[Int, String] = {case 2 => "Two"}
   val or_ : PartialFunction[Int, String] = {case _ => "Other"}

   val or = or1 orElse or2 orElse or_
   println(or(2))
}
```