* [一、定长数组](#%E4%B8%80%E5%AE%9A%E9%95%BF%E6%95%B0%E7%BB%84)
* [二、二维定长数组](#%E4%BA%8C%E4%BA%8C%E7%BB%B4%E5%AE%9A%E9%95%BF%E6%95%B0%E7%BB%84)
* [三、可变数组](#%E4%B8%89%E5%8F%AF%E5%8F%98%E6%95%B0%E7%BB%84)
* [总结](#%E6%80%BB%E7%BB%93)
  * [定长数组](#%E5%AE%9A%E9%95%BF%E6%95%B0%E7%BB%84)
  * [可变数组](#%E5%8F%AF%E5%8F%98%E6%95%B0%E7%BB%84)


---


在 scala 中， 数组分为`定长数组`和`可变数组`

---

# 一、定长数组

**声明方式：**
`① new Array[Type](length)：声明一个指定类型，指定长度的空数组`
`② val array: Array[Int] = Array(1,2,3)：通过初始化元素的个数来直接确定数组的长度`

**常用操作：**
- `:+`：在数组的最右边增加元素
- `+:`：在数组的最左边增加元素
- `++`： 拼接两个数组，并返回一个新的数组
- `++=`：拼接两个数组
- `mkString(str)`：将数组转成字符，按照 指定的 "str" 进行拼接 
- `length/size`：获取数组的大小(长度)
- `.sum`：求数组元素的总和(仅对数值型数组有效)
- `.head`： 获取第一个元素
- `.last`： 获取最后一个元素
- `tail`：去除第一个元素, 剩下的元素组成的集合
- `take(2)`：去前 n个元素, 返回时一个新的集合

**实例**
```scala
/**
  * 数组:
  *     定长数组: 一旦声明长度, 不可更改
  *
  *         底层就是 Java 的数组
  *
  *         数组的声明:
  *             1. new Array[Type](length)
  *             2. 通过初始化元素的个数来直接确定数组的长度
  *                 val array: Array[Int] = Array(1,2,3)
  *
  *
  *     可变数组:
  *
  *         数组的声明:
  *             val buffer: ArrayBuffer = new ArrayBuffer[Int]
  */
def main(args: Array[String]): Unit = {
    // 声明定长数组的第一种方式: 初始化并指定数组大小
    val array1 = new Array[Int](10)
    // 声明定长数组的第二种方式: 初始化并赋值, 数组大小由()中参数个数决定
    var array2 = Array(1, 2, 3, 4, 5)

    array2(0) = 10
    // array.size 等同于 array.length
    println(s"array1.size: ${array1.size}")
    println(s"array2.length: ${array2.length}")
    println(s"array2: ${array2.mkString(", ")}")
    println()

    // array2 :+= -1 在数组的最左边插入 -1
    array2 :+= -1
    println(s"array2: ${array2.mkString(", ")}")

    // array2 +:= -100 在数组的最右边插入 -100
    array2 +:= -100
    println(s"array2: ${array2.mkString(", ")}")
	
	val array3 = Array(10,20,30)
	// 两个集合拼接
	val array: Array[Int] = array2 ++ array3
    println(s"array: ${array.mkString("-> ")}")
	
	//两个集合拼接
    array1 ++= array3
    println(s"array2: ${array1.mkString(" -->")}")
}
```

---
# 二、二维定长数组

**声明方式：**
```scala
Array.ofDim[类型](一维数组长度, 二维数组长度)
```

**案例：**
```scala
def main(args: Array[String]): Unit = {
	// 声明二维数组
    val array: Array[Array[Int]] = Array.ofDim[Int](2,3)
    for (arr <- array) {
        for (elem <- arr) {
            println(elem)
        }
    }
}
```


---

# 三、可变数组

**声明方式：**
`① new ArrayBuffer[Int]：声明一个 Int 类型的 可变数组`
`② ArrayBuffer[Int](1,2)：声明一个 Int 类型的 可变数组，并指定初始值`

**常用操作：**
- `:+`：在数组的最右边增加元素
- `+:` 在数组的最左边增加元素
- `+=`：在数组最右边增加元素
- `-=`：删除数组元素，仅删除第一次匹配的元素
- `++`： 拼接两个数组，并返回一个新的数组
- `++=`：拼接两个数组
- `.insert(i, n)`：在指定下标插入元素
- `.remove(n)`：删除指定下标的元素
- `.remove(n, c)`：从指定下标开始删除元素，连续删除 c 个
- `.sum`：求数组元素的总和(仅对数值型数组有效)
- `.head`： 获取第一个元素
- `.last`： 获取最后一个元素
- `tail`：去除第一个元素, 剩下的元素组成的集合
- `take(2)`：去前 n个元素, 返回时一个新的集合

特殊说明：
① 增加数组元素 也可以通过 `+=` `:+= ` `+:=` 来增加元素， 但是没有 `:-=` `-:=` 来删除元素
② `-=`：只删除第一个碰到的元素

**实例：**
```scala
def main(args: Array[String]): Unit = {
    //声明可变数组, 声明时指定需要指定类型
    var buffer: ArrayBuffer[Int] = ArrayBuffer[Int](1,2)
    println(s"buffer.size: ${buffer.size}")
    println(s"buffer.size: ${buffer.mkString(", ")}")

    //增加数组元素
    buffer :+= 100
    println(s":+ 后buffer.size: ${buffer.mkString(", ")}")
    buffer :+= 100
    println(s":+ 后buffer.size: ${buffer.mkString(", ")}")

    buffer +:= -100
    println(s"+: 后buffer.size: ${buffer.mkString(", ")}")

    buffer += 101
    println(s"+= 后buffer.size: ${buffer.mkString(", ")}")

    //insert: 第一个参数 -> 插入元素到指定下标, 插入的元素
    buffer.insert(buffer.size, 102)
    println(s"insert 后buffer.size: ${buffer.mkString(", ")}")

    //删除数组元素(如果该元素在数组中存在多个, 一次只删除一个)
    buffer -= 100
    println(s"-= 后buffer.size: ${buffer.mkString(", ")}")
    
    //根据下标删除第 n 个元素
    buffer.remove(0)
    println(s"remove(0) 后buffer.size: ${buffer.mkString(", ")}")
    //根据下标删除 -> 从下标为 n 的开始删, 删 c个
    buffer.remove(0, 2)
    println(s"remove(0, 2) 后buffer.size: ${buffer.mkString(", ")}")
}
```
**运行结果：**
```scala
buffer.size: 2
buffer.size: 1, 2
:+ 后buffer.size: 1, 2, 100
:+ 后buffer.size: 1, 2, 100, 100
+: 后buffer.size: -100, 1, 2, 100, 100
+= 后buffer.size: -100, 1, 2, 100, 100, 101
insert 后buffer.size: -100, 1, 2, 100, 100, 101, 102
-= 后buffer.size: -100, 1, 2, 100, 101, 102
remove(0) 后buffer.size: 1, 2, 100, 101, 102
remove(0, 2) 后buffer.size: 100, 101, 102
```

# 总结

## 定长数组
1. 声明方式
	1. 初始化并指定数组大小
	2. 初始化并赋值, 数组大小由()中参数个数决定
2. 获取数组长度：`arr.length`,`arr.size`
3. `:+` 和 `=:` 的区别： 
	1.  `:+` 表示在数组的最右边插入指定元素
	2. `=:` 表示在数组的最左边插入指定元素
4. 当去 `修改` 定长数组时，会生成一个新的数组，并自动的返回（和 Java 中 String 类似）

## 可变数组
1. 增加数组元素 也可以通过 `+=` `:+= ` `+:=` 来增加元素， 但是没有 `:-=` `-:=` 来删除元素
2. `-=`：只删除第一个碰到的元素