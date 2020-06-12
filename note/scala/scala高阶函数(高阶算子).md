

**定义：参数为<font color='red'>函数</font>的<font color='red'>函数</font>称为高阶函数**

① 在 scala 中， 函数是一等公民，函数可以 `像对象一样进行传递`、`函数可以赋值给变量`

函数赋值给变量例子：
```scala
def main(args: Array[String]): Unit = {
    // 调用 foo 函数, 将 foo 的返回值给 f, foo 的返回值为 Unit, 所以 f 也是 Unit
    //val f = foo

    // foo _ 表示 将 foo 函数整体 传递给 f
    // 此时 f 就是一个函数了
    // 这个写法表示： 函数名叫f, 形参为空(), 期望得到的返回值为 Unit
    // 此处的 () 不能省略, 否则将变成 名传递
    val f: () => Unit = foo _

    foo() // 等价于 foo 
    f()   // 等价于 f
    
}

def foo() = {
    println("foo....")
}
```

上面的例子中，`foo()` 函数并没有参数， 我们写一个可以传入参数的函数传递给变量
```scala
def main(args: Array[String]): Unit = {
    val f: Int => Unit = foo1 _
    f(1)
}

def foo1(n: Int) = {
    println("形参为: "+n)
}

def foo() = {
    println("foo....")
}
```
高阶函数表明：参数为<font color='red'>函数</font>的<font color='red'>函数</font>称为高阶函数**， 也就是说，我们的函数需要接受一个函数才是一个高阶函数
```scala
def main(args: Array[String]): Unit = {
	// 调用 函数: f1并且传入函数 add
    println(f1(add))
}

// 定义一个函数
def add(a: Int, b: Int): Int = a + b

// 函数 f1 需要传入一个函数, 该函数需要两个 Int 类型参数, 并且返回值是 Int 类型
def f1(n: (Int, Int) => Int): Int = {
	// 调用传进来的函数，实际上传进来的是 add 函数，此处声明为 n
    n(2, 4)
}
```
上面例子就是一个 高阶函数 的声明、调用，

可能有朋友就要说了， 这个高阶函数跟我直接调用 add 函数, 有什么区别？

这种写法会简化我们的代码，举个例子： 我想传递一个数组，但是我不确定对这个数组做什么操作(加减乘除都有可能)
```scala
def main(args: Array[String]): Unit = {
	//调用高阶函数, 指定形参列表中的函数(这个函数即要做的动作)
    operation(Array(1,2,3,4), add)
    println()
    operation(Array(1,2,3,4), ride)
}

def add(a: Int): Int = a + 2

def ride(a: Int): Int = a * 2

// arr: 一个数组, 
// op: 函数, 具体这个函数做什么我们不知道, 只对该函数的形参及返回值做要求
//     可以理解成对传入的函数指定了规范: 只允许传入Int 参数, 返回 Int参数的函数
def operation(arr: Array[Int], op: Int => Int): Unit = {
    for (i <- arr) println(op(i))
}
```
上面的例子中，声明了一个高阶函数：`operation(arr: Array[Int], op: Int => Int)`，当调用 `operation` 时，传入一个数组, 一个动作，该动作及表示要做的操作

上面的例子调用高阶函数的时候， 需要声明一个 函数，可以将此过程省略成匿名函数
```scala
def main(args: Array[String]): Unit = {
    operation(Array(1,2,3,4), (a: Int) => a + 2)
    println()
    operation(Array(1,2,3,4), (a: Int) => a * 2)
}

//    def add(a: Int): Int = a + 2
//
//    def ride(a: Int): Int = a * 2

def operation(arr: Array[Int], op: Int => Int): Unit = {
    for (i <- arr) println(op(i))
}
```

因为该匿名函数 `(a: Int) => a + 2` 需要传递给高阶函数，而高阶函数已经声明了，入参必须为 Int，所以匿名函数的 `:Int` 也可以省略， 即：

```scala
def main(args: Array[String]): Unit = {
    operation(Array(1,2,3,4), (a) => a + 2)
    println()
    operation(Array(1,2,3,4), (a) => a * 2)
}
```
在该匿名函数中， 仅有一个形参a，并且该形参a只被引用了一次，所以还可以简写
```scala
def main(args: Array[String]): Unit = {
    operation(Array(1,2,3,4), _ + 2)
    println()
    operation(Array(1,2,3,4), _ * 2)
}
```
`_ + 2` 中的 `_` 表示参数占位符，`_ + 2` 表示第一个参数 `自身+2`，如果有两个参数，可以写成： `_ + _` 表示 第一个参数 + 第二个参数

---

高阶函数的例题

1. 计算 1-100 的 平方，并将该`结果集`打印在控制台
2. 过滤 1-100 的 奇数
3. 计算 1-100 的总和，并将该`结果` 打印在控制台