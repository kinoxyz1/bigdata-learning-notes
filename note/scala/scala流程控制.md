* [一、 分支控制 if\-else](#%E4%B8%80-%E5%88%86%E6%94%AF%E6%8E%A7%E5%88%B6-if-else)
* [二、嵌套分支](#%E4%BA%8C%E5%B5%8C%E5%A5%97%E5%88%86%E6%94%AF)
* [三、Switch 分支结构](#%E4%B8%89switch-%E5%88%86%E6%94%AF%E7%BB%93%E6%9E%84)
* [四、For循环控制](#%E5%9B%9Bfor%E5%BE%AA%E7%8E%AF%E6%8E%A7%E5%88%B6)
  * [4\.1 范围数据循环方式 n to m](#41-%E8%8C%83%E5%9B%B4%E6%95%B0%E6%8D%AE%E5%BE%AA%E7%8E%AF%E6%96%B9%E5%BC%8F-n-to-m)
  * [4\.2 范围数据循环方式 n until  m](#42-%E8%8C%83%E5%9B%B4%E6%95%B0%E6%8D%AE%E5%BE%AA%E7%8E%AF%E6%96%B9%E5%BC%8F-n-until--m)
  * [4\.3 循环守卫](#43-%E5%BE%AA%E7%8E%AF%E5%AE%88%E5%8D%AB)
  * [4\.4 循环步长](#44-%E5%BE%AA%E7%8E%AF%E6%AD%A5%E9%95%BF)
  * [4\.5 嵌套循环](#45-%E5%B5%8C%E5%A5%97%E5%BE%AA%E7%8E%AF)
  * [4\.6 引入变量](#46-%E5%BC%95%E5%85%A5%E5%8F%98%E9%87%8F)
  * [4\.7 循环返回值](#47-%E5%BE%AA%E7%8E%AF%E8%BF%94%E5%9B%9E%E5%80%BC)
* [五、While循环控制](#%E4%BA%94while%E5%BE%AA%E7%8E%AF%E6%8E%A7%E5%88%B6)
* [六、do\.\.while循环控制](#%E5%85%ADdowhile%E5%BE%AA%E7%8E%AF%E6%8E%A7%E5%88%B6)
* [七、多重循环控制](#%E4%B8%83%E5%A4%9A%E9%87%8D%E5%BE%AA%E7%8E%AF%E6%8E%A7%E5%88%B6)
* [八、While循环中断](#%E5%85%ABwhile%E5%BE%AA%E7%8E%AF%E4%B8%AD%E6%96%AD)

---


# 一、 分支控制 if-else
和 Java 中的 if-else 一致
1. 单分支, **语法**： 
	```scala
	if  (条件表达式)  {
		执行代码块
	}
	```
	说明：当条件表达式为ture时，就会执行{ }的代码。
	
	**需求**：输入人的年龄，如果该同志的年龄大于18岁，则输出“age > 18”
	```scala
	object TestIfElse {
	    def main(args: Array[String]): Unit = {
	        
	        println("input age:")
	        var age = StdIn.readShort()
	
	        if (age > 18){
	            println("age>18")
	        }
	    }
	}

	```
2. 双分支
	**语法**：
	```scala
	if (条件表达式) {
		执行代码块1
	} else {
		执行代码块2
	}
	```
	**需求**：输入年龄，如果年龄大于18岁，则输出“age >18”。否则，输出“age <= 18”。
	```scala
	object TestIfElse {
	    def main(args: Array[String]): Unit = {
	
	        println("input age:")
	        var age = StdIn.readShort()
	
	        if (age > 18){
	            println("age>18")
	        }else{
	            println("age<=18")
	        }
	    }
	}
	```
3. 多分支
	**语法**： 
	```scala
	if (条件表达式1) {
		执行代码块1
	}
	else if (条件表达式2) {
		执行代码块2
	}
	   ……
	else {
		执行代码块n
	}
	```
	**需求**：岳小鹏参加Scala考试，他和父亲岳不群达成承诺：如果，成绩为100分时，奖励一辆BM；成绩为(80，99]时，奖励一台iphone；其它时，什么奖励也没有。
	```scala
	object TestIfElse {
	    def main(args: Array[String]): Unit = {
	
	        println("请输入成绩")
	        val grade = StdIn.readInt()
	
	        if (grade == 100){
	            println("成绩为100分，奖励一辆BM")
	        }else if (grade > 80 && grade <= 90){
	            println("奖励一台iphone")
	        }else{
	            println("什么奖励也没有")
	        }
	    }
	}
	```
	**需求**：Scala中<font color='red'>**if else表达式其实是有返回值的**</font>，具体返回值取决于满足条件的<font color='red'>代码体的最后一行内容</font>。
	```scala
	object TestIfElse {
	    def main(args: Array[String]): Unit = {
	
	        println("input your age")
	        var age = StdIn.readInt()
	
	        var res = if(age > 18){
	            "您以成人"
	        }else{
	            "小孩子一个"
	        }
	
	        println(res)
	    }
	}
	```

**和 Java 一样，如果大括号{}内的逻辑代码只有一行，大括号可以省略。**

---
# 二、嵌套分支
**语法**
```scala
if(){
	if(){
	
	}else{
	
	}	
}
```
**需求**：参加百米运动会，根据性别提示进入男子组或女子组。如果是女子组，用时8秒以内进入决赛，否则提示淘汰。
```scala
object TestIfElse {

    def main(args: Array[String]): Unit = {

        println("输入性别：")
        var gender = StdIn.readChar()

        if (gender == '男'){
            println("男子组")
        }else{
            println("女子组")
            
            println("输入成绩：")
            var grade = StdIn.readDouble()

            if (grade > 8.0){
                println("你被淘汰了")
            }else{
                println("成功晋级")
            }
        }
    }
}
```
---
# 三、Switch 分支结构
在Scala中没有Switch，而是使用模式匹配来处理。


---

# 四、For循环控制

Scala也为for循环这一常见的控制结构提供了非常多的特性，这些for循环的特性被称为**for推导式或for表达式。**

## 4.1 范围数据循环方式 n `to` m 
**语法：**
```scala
for(i <- 1 to 3){
  print(i + " ")
}
println()
```
- i 表示循环的变量，<- 规定to 
- i 将会从 1-3 循环，<font color='red'>**前后闭合**</font>

**需求**：输出10句 "hi, kino"
```scala
object TestFor {
    def main(args: Array[String]): Unit = {
        for(i <- 1 to 10){
            println("hi, kino"+i)
        }
    }
}
```
---
## 4.2 范围数据循环方式 n `until ` m
**语法：**
```scala
for(i <- 1 until 3) {
  print(i + " ")
}
println()
```
- 这种方式和前面的区别在于i是从1到3-1
- 即使<font color='red'>**前闭合后开**</font>的范围
---
## 4.3 循环守卫
**语法：**
```scala
for(i <- 1 to 3 if i != 2) {
  print(i + " ")
}
println()
```

- 循环守卫，即循环保护式（也称条件判断式，守卫）。保护式为true则进入循环体内部，为false则跳过，类似于continue。

- 上面的代码等价于
```scala
for (i <- 1 to 3){
	if (i != 2) {
		print(i + "")
	}
}
```

**需求**：输出1到10中，不等于5的值
```scala
object TestFor {
    def main(args: Array[String]): Unit = {
        for (i <- 1 to 10 if i != 5) {
            println(i + "")
        }
    }
}
```
---
## 4.4 循环步长
**语法：**
```scala
for (i <- 1 to 10 by 2) {
      println("i=" + i)
}
```
- by表示步长

**需求**：输出1到10以内的所有奇数
```scala
for (i <- 1 to 10 by 2) {
	println("i=" + i)
}
```
**结果输出:**
```scala
i=1
i=3
i=5
i=7
i=9
```
---
## 4.5 嵌套循环
**语法：**
```scala
for(i <- 1 to 3; j <- 1 to 3) {
    println(" i =" + i + " j = " + j)
}
```
- 没有关键字，所以范围后一定要加；来隔断逻辑

上面的代码等价(**当 第一层循环和 第二层循环  中间没有任何操作时才可以使用上面的代码**)
```scala
for (i <- 1 to 3) {
    for (j <- 1 to 3) {
        println("i =" + i + " j=" + j)
    }
}
```
---
## 4.6 引入变量
**语法：**
```scala
for(i <- 1 to 3; j = 4 - i) {
    println("i=" + i + " j=" + j)
}
```
- for推导式一行中有多个表达式时，所以要加；来隔断逻辑
- for推导式有一个不成文的约定：当for推导式仅包含单一表达式时使用圆括号，
- 当包含多个表达式时，一般每行一个表达式，并用花括号代替圆括号，如下
	```scala
	for {
	    i <- 1 to 3
	j = 4 - i
	} {
	    println("i=" + i + " j=" + j)
	}
	```
上面的代码等价于(指的是 `语法` 展示的代码)
```scala
for (i <- 1 to 3) {
    var j = 4 - i
    println("i=" + i + " j=" + j)
}
```

---
## 4.7 循环返回值
**语法：**
```scala
val res = for(i <- 1 to 10) yield i
println(res)
```
- 将遍历过程中处理的结果返回到一个新Vector集合中，使用yield关键字

**需求**：将原数据中所有值乘以2，并把数据返回到一个新的集合中。
```scala
object TestFor {

    def main(args: Array[String]): Unit = {

        var res = for( i <-1 to 10 ) yield {
            i * 2
        }

        println(res)
    }
}

输出结果：
Vector(2, 4, 6, 8, 10, 12, 14, 16, 18, 20)
```
---
# 五、While循环控制
**语法：**
```scala
while (循环条件) {
      循环体(语句)
      循环变量迭代
}
```
- 循环条件是返回一个布尔值的表达式
- while循环是先判断再执行语句
- 与if语句不同，<font color='red'>**while语句没有返回值**</font>，即整个while语句的<font color='red'>**结果是Unit类型()**</font>
- 因为while中没有返回值，所以当要用该语句来计算并返回结果时，就不可避免的使用变量，而变量需要声明在while循环的外部，那么就等同于循环的内部对外部的变量造成了影响，也就违背了函数式编程的重要思想（<font color='red'>**输入=>函数=>输出，不对外界造成影响**</font>），所以不推荐使用，而是<font color='red'>**推荐使用for循环**</font>。

**需求**：输出10句 "hi，kino"
```scala
object TestWhile {
    def main(args: Array[String]): Unit = {
        var i = 0

        while (i < 10) {
            println("hi，kino" + i)
            i += 1
        }
    }
}
```

---
# 六、do..while循环控制
**语法：**
```scala
do{
       循环体(语句)
       循环变量迭代
} while(循环条件)
```
- 循环条件是返回一个布尔值的表达式
- do..while循环是先执行，再判断

**需求**：输出10句 "hi，kino"
```scala
object TestWhile {
    def main(args: Array[String]): Unit = {
        var i = 0
        do {
            println("hi，kino" + i)
            i += 1
        } while (i < 10)
    }
}
```
---

# 七、多重循环控制
**需求**：打印出九九乘法表 
```scala
object TestWhile {
    def main(args: Array[String]): Unit = {
        var max = 9
        for (i <- 1 to max) {
            for (j <- 1 to i) {
                print(j + "*" + i + "=" + (i * j) + "\t")
            }
            println()
        }
    }
}
```
可以简写成
```scala
object TestWhile {
    def main(args: Array[String]): Unit = {
        var max = 9
        for (i <- 1 to max; j <- 1 to i) {
                print(j + "*" + i + "=" + (i * j) + "\t")
                if(j == i) println()
        }
    }
}
```

---
# 八、While循环中断
Scala内置控制结构特地**去掉了break和continue**，是为了更好的适应**函数式编程**，推荐使用函数式的风格解决break和continue的功能，而不是一个关键字。**scala中使用breakable控制结构来实现break和continue功能。**

**需求**：循环遍历10以内的所有数据，数值为5，结束循环（break）

我们一步步推导出 scala 中使用的 breakable
① 
```scala
def main(args: Array[String]): Unit = {
    try{
        for (i <- 1 to 10) {
            println(i)
            if(i == 5) throw new IllegalArgumentException
        }
    } catch {
        case e =>
    }
    println("正常输出....")
}
```
输出结果
```scala
1
2
3
4
5
正常输出....
```
②
此时已经实现了我们的需求， 但是这样写 手动抛出异常  写 try catch 和麻烦
scala 中 体重了 `Breaks` 包 来抛出异常以及捕获异常
```scala
import scala.util.control.Breaks

def main(args: Array[String]): Unit = {
   Breaks.breakable(
        for (i <- 1 to 10) {
            println(i)
            if (i == 5) Breaks.break()
        }
    )
    println("正常输出....")
}
```
③ 
我们在 Java 中， 导入某个包下所有类，可以使用  import scala.util.* 指定导入 util 下所有的类，在这里我们也可以批量导入
```scala
// scala 中
import scala.util.control.Breaks._

object HelloWorld {
    def main(args: Array[String]): Unit = {
        breakable(
            for (i <- 1 to 10) {
                println(i)
                // 这里 break() 的括号中无参数， 也可以省略
                // if (i == 5) break()
                if (i == 5) break
            }
        )
        println("正常输出....")
    }
}
```