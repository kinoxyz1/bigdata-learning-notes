scala 拥有两种参数传递的方式：`按值传递` 与 `按名传递`。`按值传递` 避免了参数的重复求值，效率相对较高；而  `按名传递`  避免了在函数调用时刻的参数求值，而将求值推延至实际调用点，但有可能造成重复的表达式求值。

---

**值传递：先计算参数表达式的值，再应用到函数内部；**
```scala
def main(args: Array[String]): Unit = {
    def f: () => Int = () => {
        println("f...")
        10
    }

	foo(f())
}

def foo(a: Int) = {
    println(a)
    println(a)
    println(a)
}
```

运行结果：
```scala
f...
10
10
10
```
**说明：** 调用 `foo(f())` 方法时，会首先调用并执行 函数 `f` 中的代码，并将最后的返回值: `10` 传递给 `foo(a: Int)` 方法 ，所以 `f...` 执行一次， `10` 执行 三次

---

**名传递：将未计算的参数表达式直接应用到函数内部**
```scala
def main(args: Array[String]): Unit = {
    def f: () => Int = () => {
        println("f...")
        10
    }

	foo(f())
}

def foo(a: => Int) = {
    println(a)
    println(a)
    println(a)
}
```
运行结果：
```scala
f...
10
f...
10
f...
10
```
说明： `foo(a: => Int)` 方法中增加了 `=>` 表示 `名传递`，调用 `foo(a: => Int)` 方法时， 会将 函数 `f` 整个传递给 `foo(a: => Int)` 方法，在 该方法中，每次 `println(a)` 都会执行被完整传递过来的 `函数` ，所以 `println 一下 打印一次 f... 和 10`


在上述名传递例子中， 调用 `foo` 方法时需要传入另一个 `函数`，所以我们需要额外定义一个具名函数，而名传递的本质是：**传递函数本身**，所以这个步骤下面精简一下
```scala
def main(args: Array[String]): Unit = {
//    def f: () => Int = () => {
//        println("f...")
//        10
//    }
//    foo(f2())
	
	// 第一次精简
	// 这里直接传递一个匿名函数
//    foo(() => {
//        println("匿名函数...")
//        10
//    })

	// 第二次精简
	//foo的调用还可以将匿名函数的部分省略成
//	foo({
//		println("匿名函数...")
//       10
//	})

	//第三次精简
	// 还可以省略
	foo{
		println("匿名函数...")
        10
	}
	

}

def foo(a: => Int) = {
    println(a)
    println(a)
    println(a)
}
```
运行结果：
```scala
匿名函数...
10
匿名函数...
10
匿名函数...
10
```

---
**抽象控制**
```scala
def main(args: Array[String]): Unit = {
    var i = 1;
    myWhile(i <= 100){
        println(i)
        i += 1
    }
}

def myWhile(flag: => Boolean)(exp: => Unit): Unit = {
    if(flag) {
        exp
        myWhile(flag)(exp)
    } else
        println("循环结束....")
}
```