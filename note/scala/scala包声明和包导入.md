```scala
package com.kino.scala.day03.work.pack

// 和 Java 一样, 可以在顶部使用 import 导入, 在这个文件中的所有类都可以使用
//import java.util.ArrayList
//import java.util.List
//import java.util._

//import java.util.{ArrayList => JAL}

import java.util.{ArrayList => _, _}


/**
  * @author kino
  * @date 2019/9/6 19:27
  * @version 1.0.0
  */
object PackDemo1 {

    /**
      * scala
      *     1. 声明包
      *         ① 和 Java 一样
      *         ② 包语句
      *             package a{}
      *
      *     2. 引入包
      *         ① 和 Java 一样, 可以在顶部使用 import 导入
      *             在这个文件中的所有类都可以使用
      *             import java.util.ArrayList
      *             import java.util.List
      *
      *         ② 局部导入: 在一个类中导入两个相同名字的类时使用
      *             val lists: java.util.List[String] = new java.util.ArrayList[String]();
      *
      *         ③ 通配符导入
      *             import java.util._
      *
      *         ④ 给类起别名
      *             import java.util.{ArrayList => JAL}
      *             val lists: JAL[String] = new JAL[String]()
      *
      *         ⑤ 屏蔽类: ArrayList
      *             说明: 给 ArrayList 取别名为 _, 第二个 _ 表示通配符导入util下剩余的类
      *             import java.util.{ArrayList => _, _}
      *
      *         ⑥ 导入多个类:
      *             只想要 util下的部分类
      *             import java.util.{HashSet, ArrayList}
      */
    def main(args: Array[String]): Unit = {
        val lists: List[String] = new util.ArrayList[String]()
        println(lists.size)
    }

}

class packA{

}

package packSub{
    class packB {

    }

    package packSub1{
        class packC{
            def foo = {
                val b: packB = null
            }
        }
    }
}
```