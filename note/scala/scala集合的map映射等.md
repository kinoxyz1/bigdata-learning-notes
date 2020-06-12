* [一、map 映射： list\.map(fun)](#%E4%B8%80map-%E6%98%A0%E5%B0%84-listmapfun)
* [二、高阶函数的使用](#%E4%BA%8C%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0%E7%9A%84%E4%BD%BF%E7%94%A8)
* [三、集合的压平：list\.flatMap(fun)](#%E4%B8%89%E9%9B%86%E5%90%88%E7%9A%84%E5%8E%8B%E5%B9%B3listflatmapfun)
* [四、集合的过滤: list\.filter(fun)](#%E5%9B%9B%E9%9B%86%E5%90%88%E7%9A%84%E8%BF%87%E6%BB%A4-listfilterfun)
* [五、简化:list\.reduceLeft(fun) list\.reduceRight(fun) list\.reduce(fun)](#%E4%BA%94%E7%AE%80%E5%8C%96listreduceleftfun-listreducerightfun-listreducefun)
* [六、折叠: list\.foldLeft(fun) 等价 /: list\.LeftRight(fun) 等价 :\\](#%E5%85%AD%E6%8A%98%E5%8F%A0-listfoldleftfun-%E7%AD%89%E4%BB%B7--listleftrightfun-%E7%AD%89%E4%BB%B7-)
* [七、 扫描: list\.scanLeft(fun) scanRight(fun)](#%E4%B8%83-%E6%89%AB%E6%8F%8F-listscanleftfun-scanrightfun)
* [八、拉链(合并): list1\.zip(list2) list1\.zipAll(list2, n1, n2) list\.zipWithIndex list\.unzip](#%E5%85%AB%E6%8B%89%E9%93%BE%E5%90%88%E5%B9%B6-list1ziplist2-list1zipalllist2-n1-n2-listzipwithindex-listunzip)
* [九、迭代器](#%E4%B9%9D%E8%BF%AD%E4%BB%A3%E5%99%A8)
* [十、分组：list\.group(fun)](#%E5%8D%81%E5%88%86%E7%BB%84listgroupfun)

---

# 一、map 映射： list.map(fun)
```java
/**
 * foreach 的作用: 遍历数组、集合
 * map 的作用: 调整数据类型, 它返回的集合长度不会增加也不会减少
 */
def main(args: Array[String]): Unit = {
   val list1 = List(1,2,3,4,5,6,7)
   val list2: List[Int] = list1.map(x =>  (x * 2))
   println(s"list2: ${list2.mkString(", ")}")
}
```
先看一个实际需求：将 List(3,5,7,9) 中所有的元素都 * 2，将结果放到一个新的集合中并返回
```java
/**
 * 将 List(3,5,7) 中所有的元素都 * 2, 将其结果放到一个新的集合中返回
 */
def main(args: Array[String]): Unit = {
    def multiple(n: Int): Int = {
        println("multiple 被调用~")
        2 * n
    }

    val list1 = List(3,5,7,9)
    /**
      * 说明 list.map(multiple) 做了什么
      *
      * 1. 将 list 这个集合的元素 依次遍历
      * 2. 将各个元素传递给 multiple 函数 => 新 Int
      * 3. 将得到的新 Int, 放入到一个新的集合并返回
      * 4. 因此 multiple 函数调用了 3次
      */
    val list2: List[Int] = list1.map(multiple)
    println(list2) // 9, 25, 49
}
```

map映射深刻理解-模拟实现
```java
package com.kino.scala.day04.work.high

/**
  * @author kino
  * @date 2019/9/8 20:58
  * @version 1.0.0
  */
object MapOperateDemo2 {
    /**
      * 将 List(3,5,7,9)中的所有元素都 * 2, 并将结果放到一个新的集合中返回
      */
    def main(args: Array[String]): Unit = {
        def multiple(n: Int): Int = {
            println("multiple 被调用~")
            2 * n
        }

        val list1 = List(3,5,7,9)
        /**
          * 说明 list.map(multiple) 做了什么
          *
          * 1. 将 list 这个集合的元素 依次遍历
          * 2. 将各个元素传递给 multiple 函数 => 新 Int
          * 3. 将得到的新 Int, 放入到一个新的集合并返回
          * 4. 因此 multiple 函数调用了 3次
          */
        val list2: List[Int] = list1.map(multiple)
        println(list2) // 9, 25, 49

        // 深刻理解 map 映射函数的机制-模拟实现
        val myList = MyList()
        val myList2: List[Int] = myList.map(multiple)
        println(s"myList2: ${myList2}")

    }
}

class MyList {
    val list1 = List(3, 5, 7, 9)
    //新的集合
    val list2: List[Int] = List[Int]()

    def map(f: Int => Int): List[Int] ={
        for (elem <- this.list1) {
            //过滤, 扁平化
            list2 :+ f(elem)
        }
        list2
    }
}

object MyList {
    def apply(): MyList = new MyList()
}
```



---
# 二、高阶函数的使用
对于一个函数来说，能**接受函数**或者**返回函数**，就称之为**高阶函数**
两个实例：
```java
def main(args: Array[String]): Unit = {

   def myPrint(): Unit = {
        println("hello, world")
    }

    //在 scala中, 可以把一个函数直接赋给一个变量, 但是不执行函数
    val f1 = myPrint _
    f1() //执行

    //1. test 就是一个高阶函数
    //2. f: Double => Double 表示一个函数, 该函数可以接受一个 Double, 返回一个 Double
    //3. n1: Double 普通参数
    //4. f(n1) 在 test 函数中, 指定 你传入进来的函数
    def test(f: Double => Double, n1: Double) = {
        f(n1)
    }

    //普通的函数, 可以接受一个 Double, 返回 Double
    def sum2(d: Double): Double = {
        println("sum2 被调用")
        d + d
    }

    //使用高阶函数
    val res = test(sum2 _, 3.5)
    println(s"res ${res}")
}
```

```java
def main(args: Array[String]): Unit = {
    test2(sayOK)
}

//test2 是一个 高阶函数, 可以接受一个 没有输入, 返回为 Unit 的函数
def test2(f:() => Unit) ={
    f()
}

def sayOK() = {
    println("sayOKKK...")
}

def sub(n1: Int): Unit = {

}
```
---
# 三、集合的压平：list.flatMap(fun)
说明：效果就是将集合中的每个元素的子元素映射到某个函数并返回新的集合
```java
/**
  * 扁平化说明:
  * flatMap: flat 即压扁, 压平, 扁平化, 效果就是将集合中的每个元素的子元素映射到某个函数并返回新的集合
  */
def main(args: Array[String]): Unit = {
    val names = List("Alice", "Bon", "Nick")

    //需求是将 List 集合中的所有元素, 进行扁平化的操作, 即把所有的元素打散
    val names2: Any = names.flatMap(upper)
    println(s"names2: ${names2}")

    val list1 = List("hello world", "kino hello", "hello hello hello")
    //val list2: List[List[String]] = list1.map(x => x.split(" ").toList)
    val list2: List[String] = list1.flatMap(x => x.split(" "))
    println(s"list2: ${list2}")
}

def upper(s: String): String = {
    s.toUpperCase()
}
```
运行结果：
```java
names2: List(A, L, I, C, E, B, O, N, N, I, C, K)
list2: List(hello, world, kino, hello, hello, hello, hello)
```
示例二：
```java
/**
  * 将 val names = List("Alice", "Bon", "Nick") 中所有的单词, 全部转为大写字母, 并返回新的 List
  */
def main(args: Array[String]): Unit = {
    val names = List("Alice", "Bon", "Nick")
    val list1: List[String] = names.map(x => x.toUpperCase)
    println(s"list1: ${list1}")

    val list2: List[String] = names.map(upperCase)
    println(s"list2: ${list2}")
}

def upperCase(s: String): String = {
    s.toUpperCase
}
```
运行结果：
```java
list1: List(ALICE, BON, NICK)
list2: List(ALICE, BON, NICK)
```

---

# 四、集合的过滤: list.filter(fun)
说明： 将符合要求的数据(筛选)放置到新的集合中
```java
/**
 * filter -> 元素的过滤: 将符合要求的数据(筛选)放置到新的集合中
 */
def main(args: Array[String]): Unit = {
    //将 List("Alice", "Bob", "Nick") 集合中的首字母为 'A' 的筛选到新的集合
    val names = List("Alice", "Bob", "Nick")
    val names2: Any = names.filter(x => x.startsWith("A"))
    println(s"names2: ${names2}")

    /**
      * 将 List() 中, Int 类型的字符过滤出来并且 + 1
      */
    val list1 = List(1, 2, 3, "2l1l", 102.1, 'A', 40)
    val list2: List[Any] = list1
                            .filter(_.isInstanceOf[Int])
                            .map(_.asInstanceOf[Int])
                            .map(_ + 1)
    println(s"list2: ${list2}")
}
```
运行结果：
```java
names2: List(Alice)
list2: List(2, 3, 4, 41)
```

---
# 五、简化:`list.reduceLeft(fun)` `list.reduceRight(fun)` `list.reduce(fun)`
两片代码段示例：
```java
/**
  * 简化:
  * List(1,20,30,4,5) 求 list 的和
  */
def main(args: Array[String]): Unit = {
    //使用简化的方式来计算 list 集合的和
    val list1 = List(1,20,30,4,5)
    val res = list1.reduceLeft(sum) //reduce/reduceLeft/reduceRight

    /*
    ① (1 + 20)
    ② (1 + 20) + 30
    ③ ((1 + 20) + 30) + 4
    ④ (((1 + 20) + 30) + 4) + 5 = 60
     */

    println(s"res: ${res}")
}

def sum(n1: Int, n2: Int): Int = {
    println("sum 被调用~~~")
    n1 + n2
}
```
运行结果：
```java
sum 被调用~~~
sum 被调用~~~
sum 被调用~~~
sum 被调用~~~
res: 60
```

```java
def main(args: Array[String]): Unit = {
   val list = List(1,2,3,4,5)

    def minus(n1: Int, n2: Int): Int = {
        n1 - n2
    }

    // ((((1-2)-3)-4)-5
    println(s"reduceLeft: ${list.reduceLeft(minus)}") // -13
    // 1-(2-(3-((4-5))))
    println(s"reduceRight: ${list.reduceRight(minus)}") // 3
    //reduce 等价于 reduceLeft
    println(s"reduce: ${list.reduce(minus)}") // -13

}

def min(n1: Int, n2: Int): Int = {
    if(n1 > n2) n2 else n1
}
```
运行结果：
```java
reduceLeft: -13
reduceRight: 3
reduce: -13
```

---
# 六、折叠: `list.foldLeft(fun) 等价 /:` `list.LeftRight(fun) 等价 :\`
```java
/**
 * 折叠
 */
def main(args: Array[String]): Unit = {
    val list = List(1,2,3,4)

    def minus(n1: Int, n2: Int): Int = {
        n1 -n2
    }

    //说明
    //1. 折叠的理解和简化的运行机制几乎一样
    //理解 list.foldLeft(5)(minus) 理解成 list(5,1,2,3,4) list.reduceLeft(minus)

    //步骤 (5-1)
    //步骤 ((5-1) - 2
    //步骤 (((5-1) - 2) - 3
    //步骤 ((((5-1) - 2) - 3) -4
    println(list.foldLeft(5)(minus)) // 函数的柯里化 -> -5

    //和Left 对比, 就是反着来
    //步骤 (4 - 5)
    //步骤 (3 - (4 -5))
    //步骤 (2 - (3 - (4 - 5)))
    //步骤 1 - (2 - (3 - (4 - 5)))) = 3
    println(list.foldRight(5)(minus)) // 函数的柯里化 -> 3

    //foldLeft 和 LeftRight 缩写方法分别是: /: 和 :\
    val list2 = List(1,9)

    println((1 /: list2) (minus)) //=> list2.foldLeft(1)(minus) -> -9
    println((100 /: list2) (minus)) //=> list2.foldLeft(100)(minus) -> 90
    println((list2 :\ 10) (minus)) //=> list2.foldLeft(1)(minus) -> 2

}
```
运行结果：
```java
-5
3
-9
90
2
```

---
# 七、 扫描: `list.scanLeft(fun)` `scanRight(fun)`
说明：对某个集合的所有元素做 fold 操作, 但是会把产生的所有终检结果放置于一个集合中保存
```java
/**
  * 扫描:
  *     对某个集合的所有元素做 fold 操作, 但是会把产生的所有终检结果放置于一个集合中保存
  */
def main(args: Array[String]): Unit = {
    //普通函数
    def minus(n1: Int, n2: Int): Int = {
        n1 - n2
    }

    // 5 (1,2,3,4,5) => (5,4,2,-1,-5,-10)
    //步骤 -> 5 = 5
    //步骤 -> 5 - 1 = 4
    //步骤 -> 4 - 2 = 2
    //步骤 -> 2 - 3 = -1
    //步骤 -> ....
    println((1 to 5).scanLeft(5)(minus)) // -> Vector(5, 4, 2, -1, -5, -10)

    //普通函数
    def add(n1: Int, n2: Int): Int = {
        n1 + n2
    }

    //(1,2,3,4,5) 5 => (20, 19, 17, 14, 10, 5)
    println((1 to 5).scanRight(5)(add)) // -> Vector(20, 19, 17, 14, 10, 5)
}
```
运行结果：
```java
Vector(5, 4, 2, -1, -5, -10)
Vector(20, 19, 17, 14, 10, 5)
```

---
# 八、拉链(合并): `list1.zip(list2)` `list1.zipAll(list2, n1, n2)` `list.zipWithIndex` `list.unzip`
说明：拉链的本质就是两个集合的合并操作, 合并后, 每个元素是一个 对偶元组(二维元组)
```java
/**
  * 拉链: 拉链的本质就是两个集合的合并操作, 合并后, 每个元素是一个 对偶元组(二维元组)
  *
  * 1. 如果两个集合的个数不对应, 会造成数据丢失
  * 2. 集合不限于 List, 也可以是其他的集合, 例如: Array
  * 3. 如果要取出合并后的各个对偶元组的数据, 可以遍历
  */
def main(args: Array[String]): Unit = {
    val list1 = List(30, 50, 70, 60, 10, 20, 100, 200)
    val list2 = List(3, 5, 7, 6, 1, 2)

    //Zip 将来得到的集合长度, 以少的为主
    val list3: List[(Int, Int)] = list1.zip(list2)
    println(s"list3: ${list3}") // List((30,3), (50,5), (70,7), (60,6), (10,1), (20,2))

    //以多的为主
    val list4: List[(Int, Int)] = list1.zipAll(list2, -1, -2)
    println(s"list4: ${list4}") // List((30,3), (50,5), (70,7), (60,6), (10,1), (20,2), (100,-2), (200,-2))

    //元素和下标进行拉链
    val list5: List[(Int, Int)] = list1.zipWithIndex
    println(s"list5: ${list5}") // List((30,0), (50,1), (70,2), (60,3), (10,4), (20,5), (100,6), (200,7))

    //取下标为整数的集合数
    val list6: List[Int] = list5.filter(_._2 % 2 == 1).map(_._1)
    println(list6)

    //把拉链拉开 -> 分解为 元组, 包含两个集合
    val unzip: (List[Int], List[Int]) = list5.unzip
    println(s"unzip: ${unzip}")
}
```
运行结果：
```java
list3: List((30,3), (50,5), (70,7), (60,6), (10,1), (20,2))
list4: List((30,3), (50,5), (70,7), (60,6), (10,1), (20,2), (100,-2), (200,-2))
list5: List((30,0), (50,1), (70,2), (60,3), (10,4), (20,5), (100,6), (200,7))
List(50, 60, 20, 200)
unzip: (List(30, 50, 70, 60, 10, 20, 100, 200),List(0, 1, 2, 3, 4, 5, 6, 7))
```


---
# 九、迭代器
 通过 iterate 方法从集合获得一个迭代器，该迭代器仅能进行一次遍历
```java
/**
  * 迭代器:
  *     通过 iterate 方法从集合获得一个迭代器,
  *     通过 while 循环或 for 表达式对集合进行遍历
  */
def main(args: Array[String]): Unit = {
    val iterator: Iterator[Int] = List(1,2,3,4,5,6).iterator
    println("--------------遍历方式一: while---------------")
    while (iterator.hasNext) {
        println(s"while遍历: ${iterator.next()}")
    }
    val iterator1: Iterator[Int] = List(1,2,3,4,5,6).iterator
    println("--------------遍历方式二: for---------------")
    for (elem <- iterator1) {
        println(s"for遍历: ${elem}")
    }
    val iterator2: Iterator[Int] = List(1,2,3,4,5,6).iterator
    println("--------------遍历方式三: foreach---------------")
    iterator2.foreach(println(_))
}
```
运行结果
```java
--------------遍历方式一: while---------------
while遍历: 1
while遍历: 2
while遍历: 3
while遍历: 4
while遍历: 5
while遍历: 6
--------------遍历方式二: for---------------
for遍历: 1
for遍历: 2
for遍历: 3
for遍历: 4
for遍历: 5
for遍历: 6
--------------遍历方式三: foreach---------------
1
2
3
4
5
6
```

---
# 十、分组：`list.group(fun)`
说明：根据条件, 对 集合、数组 的数据进行分组(按条件归纳成多个Map)
```java
/**
 * group: 根据条件, 对 集合、数组 的数据进行分组(按条件归纳成多个Map)
 */
def main(args: Array[String]): Unit = {
    val list1: ListBuffer[Int] = ListBuffer(30, 50, 7, 6, 1, 20)
    val map: Map[Boolean, ListBuffer[Int]] = list1.groupBy(x => x % 2 == 1)
    println(s"map: ${map}")

    //worldCount 案例
    val list2: List[String] = List("hello world", "hello hello", "kino kino hello")
    val map2: Map[String, Int] = list2.flatMap(_.split(" ")).groupBy(x => x).map(kv => (kv._1, kv._2.length))
    println(s"map2: ${map2}")

    val list3: List[(String, Int)] = List("hello" -> 2, "hello" -> 3, "kino" -> 4)
    val groupMap: Map[String, List[(String, Int)]] = list3.groupBy(_._1)
    val temp1: Map[String, Int] = groupMap.map(kv => {
        val word = kv._1
        word -> kv._2.map(_._2).sum
    })
    println(temp1)
}
```
运行结果：
```java
map: Map(false -> ListBuffer(30, 50, 6, 20), true -> ListBuffer(7, 1))
map2: Map(world -> 1, kino -> 2, hello -> 4)
Map(kino -> 4, hello -> 5)
```