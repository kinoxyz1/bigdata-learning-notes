顺嘴提一句


scala 在导入包的时候，也可以给指定的类取别名，例如
```scala
import java.util.{ArrayList => JAL}
```
使用的时候，用 定义好的名字 `JAL` 即可，例如
```scala
new JAL[String]()
```

以上为导包时， 给指定类取别名，下面回归正题

---


在工作的时候取名字总是一件让人头大的事情，有时候取的类名名字实在是过长，使用的时候很麻烦，这个时候就用到了  `给类取别名`，例如
```scala
def main(args: Array[String]): Unit = {
    // 给类起别名
    type P = Personhenhenchangdeyigelei
    
    // 使用这个别名创建一个对象
    val p = new P()

	// 使用对象
    println(p.isInstanceOf[P])
    println(p.isInstanceOf[Personhenhenchangdeyigelei])
    println(p.getClass.getSimpleName)
    
}

//定义了一个超长名字的类
class Personhenhenchangdeyigelei
```
