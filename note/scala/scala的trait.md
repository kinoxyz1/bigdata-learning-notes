Scala Trait(特征) 相当于 Java 的接口，实际上它比接口还功能强大。

与接口不同的是，它还可以定义属性和方法的实现。

一般情况下Scala的类只能够继承单一父类，但是如果是 Trait(特征) 的话就可以继承多个，从结果来看就是实现了多重继承。

Trait(特征) 定义的方式与类类似，但它使用的关键字是 trait，如下所示：
```scala
object TraitDemo {
    def main(args: Array[String]): Unit = {
        val usb:Console = new HaweiUsb
        usb.print()
    }
}

//定义 特征 Usb
trait Usb{
	// 定义好三个没有实现的方法
    def insert(): Unit
    def work(): Unit
    def unInsert(): Unit
    
}

//定义 特征 Console
trait Console{
    def print(): Unit
}

// 特征 HaweiUsb 继承于 Usb 和 Console, 并实现每个特征中的方法
class HaweiUsb extends Usb with Console{
    override def insert(): Unit = {
        println("华为的usb插入")
    }
    
    override def work(): Unit = {
        println("华为 usb work")
    }
    
    override def unInsert(): Unit = {
        println("drow huaw usb")
    }
    
    override def print(): Unit = {
        insert()
        work()
        unInsert()
    }
}
```
运行结果：
```scala
华为的usb插入
华为 usb work
drow huaw usb
```

特征构造顺序，由代码演示顺序：
```scala
object TraitDemo2 {
    def main(args: Array[String]): Unit = {
        val abc = new ABC
        abc.foo
    }
}

// 定义 特征 F
trait F {
    println("f")

    def foo = {
        println("f foo")
    }
}

// 特征 A 继承 F, 并实现 F 中的 foo 方法
trait A extends F {
    println("a")

    override def foo = {
        println("a foo")
    }
}

//特征 B 继承 F, 并实现 F 中的 foo 方法
trait B extends F {
    println("b")

    override def foo = {
        println("b foo")
    }
}

// 特征 C 继承 F, 并实现 F 中的 foo 方法
trait C extends F {
    println("c")

    override def foo = {
    	// 调用 父类 中的 foo 方法
    	// super.foo   输出结果实际上调用的是 ABC 类继承顺序中 -> C 的上一层特征的 foo方法 
		
		//super[F].foo: 表示指定调用 父类 -> F 的 foo 方法
        super[F].foo
        println("c foo")
    }
}

// ABC 多继承于 A、B、C(注意继承关系)
class ABC extends A with B with C {
    println("abc")

    /*override def foo= {
        println("abc foo")
    }*/
}
```
运行结果：
```scala
f
a
b
c
abc
f foo
c foo
```
---


**总结：**

特征也可以有构造器，由字段的初始化和其他特征体中的语句构成。这些语句在任何混入该特征的对象在构造时都会被执行。

**构造器的执行顺序：**

- 调用超类的构造器；
- 特征构造器在超类构造器之后、类构造器之前执行；
- 特征由左到右被构造；
- 每个特征当中，父特征先被构造；
- 如果多个特征共有一个父特征，父特征不会被重复构造
- 所有特征被构造完毕，子类被构造。
- 构造器的顺序是类的线性化的反向。线性化是描述某个类型的所有超类型的一种技术规格。